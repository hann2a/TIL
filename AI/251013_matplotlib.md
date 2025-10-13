# 01_AI_Math_matplotlib

## 1. drive.mount()
```python 
from google.colab import drive
drive.mount('/content/drive')
```
> Google Colab에서 내 구글 드라이브를 세션에 연결(마운트)해서, 드라이브의 파일을 로컬 폴더처럼 읽고/쓸 수 있게 해주는 코드

- `from google.colab import drive`
  Colab 전용 drive 유틸을 불러옴 (로컬 Jupyter나 Kaggle에선 동작하지 않음)
- `drive.mount('/content/drive')`
  Colab 런타임에 마운트 지점(폴더) /content/drive를 만들고, 여기에 내 Google Drive를 연결
  - 실행하면 인증 링크가 뜨고, 내 구글 계정으로 로그인 → 인증 코드를 붙여넣으면 연결 완료.
  - 성공하면 내 드라이브의 기본 경로는:
    내 드라이브: /content/drive/MyDrive
    공유 드라이브: /content/drive/Shareddrives/<드라이브명>

## 2. 데이터 불러오기 및 구조 확인 
pandas를 이용해 CSV 파일 세 개를 불러오고,
각 데이터셋의 기본 정보(컬럼명, 결측치, 데이터 타입 등)를 확인하는 과정

```python 
import pandas as pd

# 분포 비교를 위한 별도의 점수 데이터셋들 로드
score_normal = pd.read_csv(base_path + 'final_score.csv')
score_left_skew = pd.read_csv(base_path + 'final_score_left.csv')
score_right_skew = pd.read_csv(base_path + 'final_score_right.csv')

# 각 데이터 셋 확인
print(score_normal.info())
print(score_left_skew.info())
print(score_right_skew.info())
```

`import pandas as pd`
  데이터 분석에 널리 쓰이는 라이브러리 pandas를 불러옴. 
  pd라는 약칭(alias)을 사용해 코드에서 간결하게 호출할 수 있게 함.

`pd.read_csv()`
  CSV(Comma-Separated Values) 파일을 읽어 데이터프레임(DataFrame) 형태로 불러옴.
  base_path는 파일들이 저장된 경로를 담고 있는 변수이며,
  파일 이름을 이어 붙여 완전한 파일 경로를 만듦.

`print(df.info())`
  각 데이터셋의 기본 구조를 확인합니다.
  출력 정보에는 다음이 포함됩니다:
  - 컬럼명과 데이터 타입 (int64, float64, object 등)
  - 결측치(Non-Null Count)
  - 전체 행(row) 수

## 3. 기본 선 그래프 그리기 
`matplotlib.pyplot` 모듈을 사용해 x, y 데이터의 관계를 시각적으로 표현하는 기본 선 그래프(line plot)를 그리기. 

```python 
from matplotlib import pyplot as plt
    
# x와 y데이터
x = [1, 2, 3, 4, 5]
y = [2, 4, 6, 8, 10]

# 선 그래프 그리기
plt.plot(x, y)
plt.title('Example plot')
plt.xlabel('X-axis')
plt.ylabel('Y-axis')
plt.show()
```

`from matplotlib import pyplot as plt`
  matplotlib은 파이썬의 대표적인 데이터 시각화 라이브러리. 
  그중 pyplot은 그래프를 단계별로 쉽게 그릴 수 있는 인터페이스를 제공.
  plt라는 약칭으로 불러오는 것이 관례.

`데이터 정의 (x, y)`
  x: 그래프의 가로축(X축) 값
  y: 그래프의 세로축(Y축) 값
  두 리스트의 길이가 같아야 함.(1대1 대응 관계)

`plt.plot(x, y)`
  x와 y 데이터를 선(line)으로 연결해 그림.
  기본적으로 파란색 실선(line)으로 표시됨.
  점 스타일, 색상, 선 모양도 지정할 수 있음.
  - 예: plt.plot(x, y, 'r--') → 빨간색 점선

### 그래프 꾸미기
  plt.title('Example plot'): 그래프 제목 추가
  plt.xlabel('X-axis'): X축 이름 설정
  plt.ylabel('Y-axis'): Y축 이름 설정
  plt.show()
  - 그래프를 실제로 화면에 표시.
  - (Jupyter/Colab 환경에서는 마지막 줄에 있어야 그래프가 정상 출력됨.)

## 4. 히스토그램 그리기 
```python 
# figure(): 그래프 크기 설정 (가로 8인치, 세로 6인치)
plt.figure(figsize=(8, 6))

# hist(): 히스토그램 그리기
#  - bins=20: 구간을 20개로 나눔 (막대 개수)
#  - edgecolor='black': 막대 테두리 색
#  - alpha=0.7: 막대 투명도(0~1)
plt.hist(score_normal, bins=20, edgecolor='black', alpha=0.7)

plt.title('Normal Distribution of Scores', fontsize=15)
plt.xlabel('Score', fontsize=12)
plt.ylabel('Frequency', fontsize=12)

# grid: 격자 표시
plt.grid(True)
plt.show()
```

`plt.figure(figsize=(8, 6))`
  출력 캔버스 크기를 인치 단위로 지정합니다. (DPI에 따라 실제 픽셀 크기는 달라짐)

`plt.hist(score_normal, ...)`
  score_normal의 값을 구간(bins)별로 세어 막대 그래프로 시각화합니다.
  - bins=20은 구간 개수를 20개로 설정해 분포의 굴곡을 좀 더 자세히 봅니다.
  - edgecolor, alpha로 가시성/미관을 조절합니다. (선 색, 투명도)

`title/xlabel/ylabel`
  그래프 제목과 축 라벨로 의미를 명확히 합니다.

`plt.grid(True)`
  읽기 편하도록 격자를 표시합니다.

`plt.show()`
  렌더링을 완료하고 화면에 표시합니다.

### 여러 개 그리기 
1) `label`
  어디에 쓰나? → plot(), hist(), scatter() 같은 **아티스트(artist)**에 붙여 범례에서 보일 이름.

  예:
  ```python
    plt.plot(x, y1, label='Train')
    plt.plot(x, y2, label='Validation')
  ```
   - 붙여도 legend()를 호출해야 화면에 보임.
   - 판다스 df.plot(...)도 내부적으로 같은 방식으로 label을 씀.

2) `legend()`
  `plt.legend()`  # label이 설정된 아티스트를 자동 수집해 범례 표시