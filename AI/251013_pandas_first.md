# 주피터 노트북 첫 공부 

## 1. 주피터 노트북 팁
**목적:** 노트북 환경에서 패키지를 정확히 현재 커널에 설치/관리하기.

- 설치(현재 커널에 적용):  
  ```python
  %pip install --upgrade "pandas>=2.0" "seaborn==0.13.2"
  ```
  - `%pip`(매직) 권장: 노트북 커널과 연동되어 의도한 환경에 깔립니다.
  - `!pip`도 가능하지만 커널과 분리된 환경에 설치되는 문제가 발생할 수 있습니다.
- 버전 확인:
  ```python
  import pandas as pd, seaborn as sns
  print(pd.__version__, sns.__version__)
  ```
- 설치/업그레이드 후에는 **커널 재시작**이 필요할 수 있습니다.

---

## 2. 판다스: 컬럼 생성 & 조건 필터링
**목적:** 계산된 비율 컬럼을 만들고, 조건에 맞는 행만 선택하기.

- 새 컬럼(벡터연산: 빠르고 간결)
  ```python
  df["tip_percent"] = df["tip"] / df["total_bill"]
  # 보기 좋게 반올림
  df["tip_percent"] = df["tip_percent"].round(3)
  ```
  - 벡터연산은 `apply`보다 일반적으로 빠르고 메모리 효율적입니다.
- 조건 필터링
  ```python
  # 20% 이상만
  high_tippers = df[df["tip_percent"] >= 0.2].copy()
  # NaN 방어(필요 시)
  high_tippers = df[df["tip_percent"].ge(0.2).fillna(False)].copy()
  ```
- 실수 방지 포인트
  - 슬라이싱 후 수정할 계획이면 `.copy()`로 **사본**을 만들어 SettingWithCopyWarning을 피하세요.
  - 0으로 나누기 가능성이 있으면 `total_bill.replace(0, pd.NA)`로 방지.

---

## 3. 그룹 집계 & 피벗
**목적:** 성별×요일 별 평균 팁을 표 형태(행: 성별, 열: 요일)로 만들기.

- 절차(핵심 두 단계: 집계 → 피벗)
  ```python
  grouped = df.groupby(["sex", "day"], observed=True)["tip"].mean().reset_index()
  pivot_table = grouped.pivot(index="sex", columns="day", values="tip")
  ```
- 왜 `reset_index()`가 필요한가?
  - `groupby` 결과의 그룹키가 인덱스에 들어가므로, 열로 다시 쓰려면 먼저 일반 열로 복원.
- `observed`란?
  - 카테고리형 그룹키에서 **실제로 관측된 조합만 포함(True)** 할지, **모든 가능한 조합(False)** 을 포함할지.
  - 판다스의 기본값 변경 예정 경고가 있어 **명시**하는 습관을 추천합니다.
- 자주 생기는 문제
  - `pivot`의 `index`/`columns`/`values` 지정 누락 → 원하는 형태가 안 생김.
  - 피벗 후 열/행 정렬이 어색하면 `pivot_table = pivot_table.sort_index().sort_index(axis=1)`.

---

## 4. 형태 변환: melt (Wide → Long)
**목적:** 여러 측정값 열을 행으로 녹여서 분석/시각화/그룹화에 유리한 **Long-format** 으로 변환.

- 사용 상황
  - 여러 수치 열(`total_bill`, `tip`)을 하나의 항목 열(`item`)과 값 열(`amount`)로 모으고 싶을 때.
- 예시
  ```python
  melted = (
      df.loc[:, ["size", "total_bill", "tip"]]
        .melt(
            id_vars="size",
            value_vars=["total_bill", "tip"],
            var_name="item",
            value_name="amount"
        )
  )
  # 결과 컬럼: ['size', 'item', 'amount']
  ```
- 장점
  - `groupby`, `pivot_table`, `seaborn`의 시각화 등에서 **하나의 값 열**을 쓰기 쉬워짐.
- 흔한 실수
  - `id_vars`(유지 열)와 `value_vars`(녹일 열)를 뒤섞어서 지정.

---

## 5. 통계값 기반 행 추출
**목적:** 어떤 지표의 최댓값/최솟값을 가진 관측치를 **동점 포함**하여 모두 찾기.

- 최댓값 가진 행 모두 선택
  ```python
  max_val = df["tip_percent"].max()
  top_tipper = df.loc[df["tip_percent"].eq(max_val)].copy()
  ```
- 대안(상위 N개 뽑기)
  ```python
  top3 = df.nlargest(3, "tip_percent")
  ```
- 주의
  - 동점 처리를 원하면 `eq(max_val)` 방식이 더 명확합니다.

---

## 6. 자주 쓰는 판다스 패턴 (요약)
- 인덱스/정렬
  ```python
  df = df.reset_index(drop=True)
  df = df.sort_index()                  # 인덱스 기준
  df = df.sort_values(["col1", "col2"]) # 열 값 기준
  ```
- 라벨/정수 위치 선택
  ```python
  df.loc[row_sel, col_sel]   # 라벨 기반
  df.iloc[i, j]              # 정수 위치 기반
  ```
- 범위 필터
  ```python
  df[df["value"].between(0.1, 0.5)]
  ```
- 슬라이스 사본
  ```python
  subset = df.loc[mask, cols].copy()
  ```

---

### Micro-Checklist
- 벡터연산 우선, 불필요한 `apply` 지양.
- `groupby → reset_index → pivot` 순서 기억.
- 카테고리 그룹키는 `observed` 명시로 경고/의도 차이를 제거.
- 슬라이스 수정 전 `.copy()`로 안전하게 작업.
