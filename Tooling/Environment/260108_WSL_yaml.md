# TIL: WSL에서 conda 환경 구성 시 윈도우 yaml 파일 사용하는 법 

## 한 줄 요약
WSL 환경에서 grep으로 Windows용 conda YAML을 필터링해 Linux용 환경을 구성한다
--- 

## 배경
- 수업에서 mini-forge를 사용하여 환경을 구성하라고 하였는데 나는 WSL으로 전체적인 환경 관리를 하는 것이 목표였기 때문에 윈도우 용 yaml 파일을 wsl용으로 변경할 필요성이 있었다. 
- 보통 windows에서 만들어진 conda 환경을 `conda env export`로 뽑는다고 한다. 근데 이걸 wsl에서 그대로 `conda env create -f`로 만드려고 하면 다음과 같은 에러로 막힌다. 
- `LibMambaUnsatisfiableError`
- `does not exist (perhaps a typo or a missing channel)`
- `opencv ... requires hdf5 >=... <... but none of the providers can be installed`
- `qt6-main, pyside6, libopencv, py-opencv, libpng, pcre2, libclang13 등 의존성 충돌`

## 원인 분석 
1. 플랫폼 차이: 같은 패키지라도 바이너리가 다르다. 
- conda 패키지는 소스 코드가 아니라 플랫폼 별로 빌드된 바이너리로 배포되는 경우가 많다. 따라서 윈도우용 런타임/툴체인 패키지가 포함될 수 있는데 이건 리눅스 플랫폼에서는 애초에 제공되지 않거나 의미가 없다. 그래서 solver는 없는 패키지를 발견하고 즉시 실패한다. 

2. ABI(바이너리 호환성)와 python_abi 제약 
- 에러에 자주 등장하는 `python_abi`는 파이썬 런타임 호환성 조건이다. C/C++ 확장 모듈(opencv, numpy)은 파이썬 버전과 강하게 엮인다. YAML에서 강하게 고정되어 있는데 다른 패키지가 cp311/cp312로 맞춰진 조합을 요구하면 solver가 출구를 잃는다. 즉, python 버전 고정은 재현성에는 좋지만, 플랫폼이 바뀌거나 패키지 생태계가 달라지면 충돌 확률을 폭증시킨다. 

3. Conda solver는 결국 버전 범위/채널/플랫폼 제약을 모두 만족하는 패키지 조합을 찾는 문제를 푼다. 이는 제약 만족 문제(CSP, Constraint Satisfaction Problem) 혹은 SAT 문제와 유사한 구조이며,
하나의 패키지를 고정할수록 전체 해 공간이 급격히 줄어든다. 따라서 충돌의 중심 패키지를 제거해 제약을 줄여야 solver가 해를 찾는다. 

4. opencv, pyside6/qt6은 내부적으로 이미지 코덱, GUI, 글꼴, 컴파일러 런타임, 시스템 라이브러리 등 많은 하위 패키지를 끌어온다. 

## 해결 전략 요약 
1. WSL에서 conda-forge + strict로 채널 일관성 확보
2. Windows YAML에서 Windows 전용 패키지 / 충돌 중심 패키지(opencv/qt) 제거
3. python 고정을 완화(삭제하거나 3.11로 이동)
4. conda로 환경 생성 성공시키기
5. 필요한 대형 패키지는 환경 생성 후 pip로 분리 설치
- pip wheel은 이미 플랫폼별로 빌드된 바이너리를 직접 내려받기 때문에
conda solver처럼 전체 의존성을 다시 풀 필요가 없다.
6. 설치가 오래 걸릴 때는 “멈춤 vs 진행 중”을 프로세스/소켓으로 판별

## 3. 실제 작업 기록 

### 3.1 채널 설정: conda-forge 우선 + strict

```bash
conda config --add channels conda-forge
conda config --set channel_priority strict

conda config --show channels
conda config --show channel_priority
```
- conda의 채널은 패키지 저장소 목록이다. `pip`가 `PyPI`에서 받듯이, conda는 채널에서 패키지를 받는다. 대표채널에는 defaults(anaconda 기본), `conda-forge`(커뮤니티 중심, 패키지 많음) 등이 있다. 
- conda는 설치할 때 이 패키지를 어느 채널에서 받을까? 그 채널의 빌드가 다른 패키지들과 잘 맞을까?를 결정한다. 
- strict는 채널 우선순위로, 상위 우선 채널에서 가능한 전부 해결하도록 유도할 수 있다. 즉 한 채널에서 해결 가능한 경우 다른 채널 패키지를 섞지 않겠다는 정책이다.

---

### 3.2 Windows 다운로드 폴더로 이동 + YAML 확인

WSL에서 Windows 파일 시스템은 `/mnt/c`로 접근한다.

```bash
cd "/mnt/c/Users/SSAFY/Downloads"
ls | grep -E '\.ya?ml$'
```
- grep은 텍스트에서 특정 패턴을 찾아서 출력하는 도구로 
- `grep "python"`: python이 들어간 줄만 보여줘 
- `grep -v "python"`: python이 안 들어간 줄만 보여줘 
---

### 3.3 1차 필터링: Windows 전용/명백히 충돌 유발 패키지 제거

원본 파일이 `prediction_wsl.yml`로 존재했음.

```bash
WIN_YML="prediction_wsl.yml"
OUT_YML="prediction_wsl_clean.yml"

grep -v -E '^[[:space:]]*-[[:space:]]*(_libavif_api|intel-openmp|khronos-opencl-icd-loader|libintl|libwinpthread|py-opencv|libopencv|qt6-main|ucrt|vc$|vc14_runtime|vs2015_runtime|pywin32|pywinpty)[=[:space:]]' \
  "$WIN_YML" > "$OUT_YML"
```
- `-E`는 확장 정규식
- YAML 파일에서 특정 패키지 줄을 빼고 새 YAML 파일을 만들라는 명령어 

**포인트**

- `_libavif_api`, `intel-openmp` 등은 “현재 플랫폼/채널에서 존재하지 않음” 형태로 로그에 등장했음
- `py-opencv` / `libopencv` / `qt6-main`는 충돌 중심축이었음
- Windows 전용 런타임(`ucrt`/`vc`/...) 및 `pywin32`/`pywinpty`는 WSL에서 의미가 없음

---

### 3.4 2차 필터링: OpenCV/Qt 체인을 강하게 제거 (solver가 계속 막힐 때)

실제로 solver는 `opencv`가 `hdf5` 버전 제약 때문에 계속 실패했다.  
그래서 환경 생성 우선을 위해 OpenCV/Qt 계열을 더 과감히 제거했다.

```bash
grep -v -E '^[[:space:]]*-[[:space:]]*(opencv|py-opencv|libopencv|pyside6|qt6-main|pcre2|libpng)[=[:space:]]' \
  "$OUT_YML" > prediction_wsl_min.yml
```

---

### 3.5 python 고정 완화: python 라인 제거 (solver 자유도 확보)

로그에서 `python=3.10.18` 고정이 `python_abi` 충돌을 유발했다.  
그래서 python 라인을 제거해 solver가 가능한 조합을 찾도록 했다.

```bash
grep -v -E '^[[:space:]]*-[[:space:]]*python=.*' prediction_wsl_min.yml > prediction_wsl_min2.yml
```

---

### 3.6 환경 생성

```bash
conda env create -f prediction_wsl_min2.yml
conda env list
conda activate <환경이름>
```

---

## 4. 환경 생성 후: 필요한 패키지 분리 설치 (pip)

### 4.1 OpenCV (WSL에서 안정적 선택: headless)

Conda `opencv`가 깨지면, pip wheel 기반 `opencv-python-headless`가 훨씬 간단하게 끝나는 경우가 많다.

```bash
pip install opencv-python-headless
python -c "import cv2; print('cv2 ok', cv2.__version__)"
```

---

### 4.2 PySide6(Qt GUI)

GUI가 꼭 필요하면 pip로 분리 설치를 우선 고려한다.

```bash
pip install pyside6
python -c "import PySide6; print('pyside ok', PySide6.__version__)"
```

---

## 5. “Installing pip dependencies…”가 오래 걸릴 때 판단법

pip 단계가 10분 이상 걸릴 수 있다.  
여기서 중요한 건 진짜 멈춘 건지, 그냥 대용량 휠 다운로드/설치 중인지 구분하고 싶었다. 

### 5.1 pip/conda 프로세스가 살아있는지

```bash
ps aux | egrep "pip|python|conda" | head
```

예: 실제로 아래처럼 pip install 프로세스가 떠 있으면 진행 중일 가능성이 높다.

- `python -m pip install -U -r ...requirements.txt`

### 5.2 네트워크 연결이 살아있는지(HTTPS 443)

```bash
ss -tpn | head
```

`ESTAB ... :443`가 보이면 다운로드/통신 중으로 판단 가능.

### 5.3 진행 중이면 기다리는 게 맞는 케이스

- `%CPU`가 0이 아니고
- `etime`이 증가하고
- `ESTAB` 연결이 존재하면 정상적으로 진행 중일 확률이 높다고 한다. 

```bash
ps -p <pip_pid> -o pid,etime,%cpu,%mem,cmd
```

---

## 6. 가상환경 종료

```bash
conda deactivate
```

여러 번 실행하면 `(prediction-wsl) → (base) → (표시 없음)` 순으로 내려간다.

---

## 7. 정리 / 배운 점

- Windows에서 export된 conda YAML은 다른 플랫폼(WSL Linux)에서 그대로 재현하기 어렵다.
- conda solver는 제약 만족 문제이고, 충돌 중심 패키지(OpenCV/Qt 같은 바이너리 큰 녀석)가 있으면 해가 없어지기 쉽다.
- 해결은 “제약을 줄이는 방향”이 효과적:  
  필터링(플랫폼 종속 제거) → 최소 환경 생성 → 대형 패키지는 pip로 분리 설치
- 설치가 오래 걸릴 때는 감으로 끊지 말고,  
  프로세스/네트워크(ESTAB 443)/CPU 사용률로 진행 중 여부를 판단하면 불필요한 재시도를 줄일 수 있다.
