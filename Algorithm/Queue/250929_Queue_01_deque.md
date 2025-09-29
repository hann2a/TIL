```python 
"""
2. deque로 풀기 
실행시간: 0.003001초
Deque(데크)는 double-ended-queue의 줄임말로, 양방향에서 데이터를 처리할 수 있는 queue형 자료구조이다.

deque.append(x) : x를 데크의 오른쪽 끝에 삽입한다
deque.appendleft(x) : x를 데크의 왼쪽 끝에 삽입한다
deque.pop() : 데크의 오른쪽 끝 엘리먼트를 가져오는 동시에 데크에서는 삭제한다
deque.popleft() : 데크의 왼쪽 끝 엘리먼트를 가져오는 동시에 데크에서는 삭제한다

deque 객체 사용하는 법 
queue = deque()
"""

from collections import deque 
queue = deque()

queue.append(1)
queue.append(2)
queue.append(3)

print(queue.popleft())
print(queue.popleft())
print(queue.popleft())
```