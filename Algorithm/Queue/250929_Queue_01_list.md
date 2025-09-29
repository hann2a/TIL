"""
큐를 구현하여 다음 동작을 확인해봅시다. 
1. 세 개의 데이터 1, 2, 3을 차례로 큐에 삽입하고 
2. 큐에서 세 개의 데이터를 차례로 꺼내서 출력한다. 
"""

"""
1. 리스트로 풀기 
append로 추가하면 순서대로 들어간다. 
.pop(0)를 하면 맨 왼쪽 꺼를 pop할 수 있다. 
0.000256 sec
"""
```python
queue = []
queue.append(1)
queue.append(2)
queue.append(3)

print(queue.pop(0))
print(queue.pop(0))
print(queue.pop(0))
```