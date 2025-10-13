# 가중치와 편향 
## 가중치(Weight, W)
각 입력 변수가 결과 변수에 미치는 영향력 또는 중요도를 나타내는 게수
가중치의 절대값이 클수록 해당 특성이 예측에 더 큰 영향을 미침
## 편향(Bias, b)
모든 입력 변수가 0일 때의 기본 예측값
모델이 데이터 전체의 기본적인 편향을 학습할 수 있도록 돕는 절편(intercept) 역할
---
# 산점도 위에 가설 모델을 그려 최적의 직선을 찾아보자 
```python 
import numpy as np
# 1. 가장 상관관계가 높았던 특성과 결과 변수 선택
sub_rate_d1 = df_main['sub_rate_d1']
final_score = df_main['final_score']

plt.figure(figsize=(10, 6))
plt.scatter(sub_rate_d1, final_score, alpha=0.5, label='Actual Data')
plt.title('Finding the Best-Fit Line', fontsize=15)
plt.xlabel('Submission Rate for Difficulty 1 (%)', fontsize=12)
plt.ylabel('Final Exam Score', fontsize=12)
plt.grid(True)

# 3. 임의의 직선(모델) 그려보기
# 가중치와 편향을 변경해 보며 최적의 직선을 찾아보자.
W_example = 1  # 가상의 가중치 (기울기)
b_example = 25   # 가상의 편향 (y절편)

# 특성
x_line = np.array(sub_rate_d1) 
# 모델에 의한 예측값
y_line = W_example * x_line + b_example
# label은 legend에 표시될 문자열
plt.plot(x_line, y_line, color='red', linewidth=3, label=f'Model Example (y = {W_example}x + {b_example})')
plt.legend()
plt.show()
```

## `x_line = np.array(sub_rate_d1)`

- sub_rate_d1이 pandas.Series 타입이라면, 이걸 NumPy 배열(ndarray) 로 변환하는 코드.

|객체	| 형태	|설명|
|----|------|----|
|pandas.Series	|표 구조 데이터의 1차원 열	| pandas용|
| numpy.ndarray	|순수 수치형 배열	|수학 연산용 |

NumPy 배열로 바꾸면, 벡터 연산이 즉시 가능해진다. 
  예를 들어:
```python 
x_line = np.array([10, 20, 30])
y_line = 2 * x_line + 5
# → 결과: [25, 45, 65]
```
- 여기서 2 * x_line + 5는 for문 없이 배열의 모든 요소에 연산이 동시에 적용됨. 
- 이걸 `브로드캐스팅(broadcasting)` 이라고 부름.