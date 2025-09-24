```python
"""
250924 복습 
"""

import sys
sys.stdin = open('input.txt', 'r')

V, E = map(int, input().split())
numbers = list(map(int, input().split()))

def dfs(start):
    
    visited = [False] * (V+1)
    path = []

    stack = [start]

    while stack:
        # print(stack)
        now_node = stack.pop()

        if not visited[now_node]:
            visited[now_node] = True
            path.append(now_node)

            for next_node in adj_list[now_node]:
                # print(next_node)
                if not visited[next_node]:
                    stack.append(next_node)
    
    return path 
"""
인접 행렬 만들고 채우기 
"""
adj_list = [[] for _ in range(V+1)]

for i in range(E):
    first, second = numbers[2* i], numbers[2*i + 1]
    adj_list[first].append(second)
    adj_list[second].append(first)

for i in range(V+1):
    adj_list[i].sort(reverse=True)

path = dfs(1)
print(''.join(map(str, path)))

```