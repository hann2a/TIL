```python 
"""
3. 원형 큐로 풀기 
실행시간 0.000360초 
__init__(생성자): 인스턴스를 만들 때 한 번 실행되는 초기화 함수 
                - 여기서는 용량계산, 배열 준비, 포인터 초기화 
인스턴스 속성: 각 객체가 들고 있는 상태값
메서드: 객체가 할 수 있는 동작
<front, rear 포인터>
- front: 비어 있는 칸을 가리킵니다(첫 원소 “바로 앞” 위치).
- rear: 마지막 원소가 들어간 칸을 가리킵니다.    
enqueue 한 다음에는 rear를 한 칸 이동 
dequeue 한 다음에는 front를 한 칸 이동             
"""


class CircularQ:
    """
    self.이름 = 값
    """
    def __init__(self, n):
        self.capacity = n + 1 # 포화 상태 구분을 위해 한 칸 비워둠
        self.items = [None] * self.capacity
        self.front = 0 # 큐의 시작 위치 (비어 있음을 가리킴)
        self.rear = 0 # 큐의 끝 위치 (마지막 원소를 가리킴)
    
    def is_empty(self):
        # 큐가 비어있는지 확인하는 메서드 
        # front와 rear 포인터가 같은 위치에 있으면 비어있는 상태 
        return self.front == self.rear
    
    def is_full(self):
        # 큐가 가득찼는지 확인하는 메서드 
        # rear 포인트의 다음 위치가 front와 같다면 가득 찬 상태 
        return (self.rear + 1) % self.capacity == self.front 
    
    def enqueue(self, item):
        if self.is_full():
            # 큐가 가득 찼다면 더 이상 추가할 수 없음 
            print('Queue is full')
            return 

        # rear 포인터를 시계방향으로 한 칸 이동 
        self.rear = (self.rear +1) % self.capacity
        # 새로운 위치에 데이터 삽입
        self.items[self.rear] = item 
    
    def dequeue(self):
        if self.is_empty():
            print("Queue is empty")
            return None 
        
        self.front = (self.front +1) % self.capacity
        return self.items[self.front]

q = CircularQ(3)

q.enqueue(1)
q.enqueue(2)
q.enqueue(3)

print(q.dequeue())
print(q.dequeue())
print(q.dequeue())
```