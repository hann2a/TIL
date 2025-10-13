# 특정 패턴의 컬럼 선택하기 (리스트 컴프리헨션 + f-string)
```python 
# 1. 8개 난이도별 실습 제출률 컬럼만 선택
submission_cols = [f'sub_rate_d{i+1}' for i in range(8)]
# print(submission_cols)
difficulty_df = df_main[submission_cols]
print(difficulty_df.head())
```
- range(8) → i = 0 ~ 7
- `f'sub_rate_d{i+1}'` → sub_rate_d1, sub_rate_d2, ..., sub_rate_d8 문자열 생성

결과: ['sub_rate_d1', ..., 'sub_rate_d8']
- 원본에서 원하는 열만 슬라이싱

`df_main[submission_cols]` → 위 리스트에 있는 열만 추출해 새 DataFrame difficulty_df 생성

미리보기
- `difficulty_df.head()` → 상위 5행만 출력 (형태/결측치 대략 확인)

# 난이도별 평균 제출률 라인 플롯 
```python 
# 2. 각 난이도별 평균 제출률 계산
mean_submission_rates = difficulty_df.mean()

# 3. 라인 플롯 그리기
plt.figure(figsize=(10, 6))
plt.plot(range(1, 9), mean_submission_rates, marker='o', linestyle='--')

# 4. 그래프 요소
plt.title('Average Submission Rate by Difficulty Level')
plt.xlabel('Difficulty Level')
plt.ylabel('Average Submission Rate (%)')
plt.grid(True)
plt.show()
```
`difficulty_df.mean()`
- 기본값 axis=0 → 각 열의 평균을 계산하여 Series 반환 (인덱스=컬럼명).
- 결측치는 기본적으로 무시(skipna=True) 하고 계산.

`plt.plot(range(1, 9), mean_submission_rates)`
- X축: 난이도 1~8, Y축: 각 난이도 평균 제출률.`

# 세 가지 설명변수와 결과 변수의 관계를 3개의 산점도 서브플롯으로 비교
```python 
# 1. 비교할 세 특성과 결과 변수 선택
assign_ment_rate = df_main['assignment_rate']
sub_rate_d1 = df_main['sub_rate_d1']
sub_rate_d8 = df_main['sub_rate_d8']
final_score = df_main['final_score']

# 2. 두 개의 산점도를 나란히 그리기
    # subplots 함수로 1행 2열의 서브플롯 생성
    # output: 1행 2열의 서브플롯이 있는 Figure 객체와 각 서브플롯에 대한 Axes 객체 반환
        # Figure: 전체 그래프 영역
        # Axes: 실제 데이터가 그려지는 영역
fig, (ax1, ax2, ax3) = plt.subplots(1, 3, figsize=(16, 6)) # 1행 3열의 서브플롯 생성

# 과제 제출률 산점도
ax1.scatter(assign_ment_rate, final_score, alpha=0.5, color='red')
ax1.set_title('Assignment Rate vs. Final Score')
ax1.set_xlabel('Assignment Rate (%)')
ax1.set_ylabel('Final Exam Score')
ax1.grid(True)

# 가장 쉬운 난이도(d1) 산점도
ax2.scatter(sub_rate_d1, final_score, alpha=0.5)
ax2.set_title('Difficulty 1 Rate vs. Final Score')
ax2.set_xlabel('Submission Rate for Difficulty 1 (%)')
ax2.set_ylabel('Final Exam Score')
ax2.grid(True)

# 가장 어려운 난이도(d8) 산점도
ax3.scatter(sub_rate_d8, final_score, alpha=0.5, color='green')
ax3.set_title('Difficulty 8 Rate vs. Final Score')
ax3.set_xlabel('Submission Rate for Difficulty 8 (%)')
ax3.set_ylabel('Final Exam Score')
ax3.grid(True)

plt.tight_layout() # 그래프 간격 자동 조절
plt.show()
```
- `plt.subplots(1, 3, ...)` → Figure(전체) + 세 개의 Axes(각 그래프 영역) 반환
- 각 ax.scatter(x, y, ...)에서 점 투명도(alpha)로 겹침을 완화
- `tight_layout()`이 서브플롯 간격을 보기 좋게 자동 조정

# 제출률과 final_score의 상관계수를 한 번에 게산하고 내림차순 정렬 후 막대그래프로 시각화 
```python 
# 1. 분석할 모든 제출률 관련 컬럼 리스트 생성
submission_cols = ['assignment_rate'] + [f'sub_rate_d{i+1}' for i in range(8)]

# 2. 각 특성과 final_score 간의 상관 계수만 계산하여 Series로 저장
correlations = df_main[submission_cols + ['final_score']].corr()['final_score']

# final_score 자기 자신과의 상관관계(1.0)는 제외
correlations = correlations.drop('final_score')

# 3. 상관 계수를 내림차순으로 정렬
correlations_sorted = correlations.sort_values(ascending=False)

print("Correlation with Final Score (Sorted):")
print(correlations_sorted)

# 4. 막대그래프로 시각화
plt.figure(figsize=(12, 7))
correlations_sorted.plot(kind='bar', color='skyblue', edgecolor='black')

# 5. 그래프에 제목과 라벨 추가
plt.title('Correlation of Various Submission Rates with Final Score', fontsize=15)
plt.xlabel('Submission Rate Feature', fontsize=12)
plt.ylabel('Correlation Coefficient', fontsize=12)
plt.xticks(rotation=45, ha='right') # x축 라벨이 겹치지 않도록 회전
plt.grid(axis='y', linestyle='--')
plt.tight_layout()
plt.show()
```