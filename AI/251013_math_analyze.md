# 산점도 그리기 
`plt.scatter()`
```python 
# 1. 분석할 두 변수(특성) 선택
assignment_rate = df_main['assignment_rate']
final_score = df_main['final_score']

# 2. 산점도 그리기
plt.figure(figsize=(10, 6))
plt.scatter(assignment_rate, final_score, alpha=0.7)  # alpha=점 투명도

# 3. 그래프에 제목과 라벨
plt.title('Assignment Rate vs. Final Score')
plt.xlabel('Assignment Submission Rate (%)')
plt.ylabel('Final Exam Score')
plt.grid(True)
plt.show()
```

## 상관 계수를 이용한 정량적 관계 분석
`.corr()`메서드를 통해 산점도에서 확인한 시각적 패턴을 정량적인 수치로 확인

- pandas의 corr() 함수는 기본적으로 피어슨 상관계수를 계산
- cov(X, Y): X와 Y의 공분산(covariance)
  - (공분산: 두 변수 간의 관계를 나타내는 지표)
- E: 기대값 (평균)
- $cov(X, Y) = E[(X - mean(X)) * (Y - mean(Y))]$
- $r = cov(X, Y) / (std(X) * std(Y))$

단, student_id와 같은 식별자 열은 상관계수 계산에서 제외할 것.
원본 df_main은 그대로 두고, 상관 계수 계산용 복사본만 별도로 담아서 사용

### 데이터프레임 복사 및 열 제거 
```python 
# 복사본 생성 후 student_id 열 제거
df_main_copy = df_main.copy().drop(columns=['student_id'])
```
`df_main.copy()`
- 원본 df_main의 깊은 복사(deep copy) 를 만듦.
- 즉, 새 데이터프레임 df_main_copy는 원본과 메모리를 공유하지 않기 때문에, 이후 수정해도 원본이 영향을 받지 않음.
- (얕은 복사인 df_main 그대로 수정과 달리 안전.)

`.drop(columns=['student_id'])`
- 지정한 열(student_id)을 제거한 새 데이터프레임을 반환.
- columns 인자는 여러 열을 리스트로 받을 수 있음.
- `df.drop(columns=['col1', 'col2'])`

결과
- df_main_copy는 df_main의 모든 데이터 중 student_id 열만 빠진 완전한 사본.
- 원본 df_main은 그대로 유지됨.

### 상관계수 계산 
```python
# final_score와 assignment_rate의 상관관계 추출
# corr(): 상관계수 계산 함수
# 상관계수는 -1 ~ 1 사이의 값으로 표현됨
#  1에 가까울수록 → 강한 양의 상관관계 (한 변수가 커질수록 다른 변수도 커짐)
# -1에 가까울수록 → 강한 음의 상관관계 (한 변수가 커질수록 다른 변수는 작아짐)
correlation_matrix = df_main_copy.corr()
print("\n상관계수 행렬:\n", correlation_matrix)
```

`df_main_copy.corr()`
- pandas의 내장 함수로, 수치형 열들 간의 상관계수(correlation coefficient) 를 계산.
- 결과는 상관계수 행렬(correlation matrix) 형태의 DataFrame으로 반환.
- 기본 계산 방식은 피어슨 상관계수(Pearson’s correlation coefficient).

### .loc[]로 특정 상관계수 추출하기
```python 
# loc: 특정 행과 열을 선택하는 함수
# 'final_score' 행과 'assignment_rate' 열의 상관계수 추출
final_score_assignment_rate_corr = correlation_matrix.loc['final_score', 'assignment_rate']
print("final_score와 assignment_rate의 상관관계: ", final_score_assignment_rate_corr)
```

`.loc[행, 열]`
- pandas의 라벨 기반 인덱싱(label-based indexing) 기능.
- 이름(라벨)을 직접 지정해서 특정 행과 열의 교차점 값을 선택.
```python
df.loc['행이름', '열이름']
```