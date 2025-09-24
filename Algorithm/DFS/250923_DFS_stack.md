```python 
"""
250923
스택: 후입선출 
무조건 input부터 쓰는 습관 오늘부터 들이기 
"""

import sys

sys.stdin = open('input.txt', 'r')

# ---------------------------------------------
"""
pop 시점에 방문처리를 한다 
첫 시작은 직접 해준다 (직접 스택에 넣어주어야 하므로)
0번을 안쓸 것이므로 visited 배열은 V+1개를 만든다 
path는 탐색 경로를 저장한다. 
stack이 다 빌 때까지 해당 함수는 계속된다. 
if, while 등은 바로 변수: 를 쓰면 '비지 않았으면'의 뜻이다. 
시작할 때 현재 노드를 정한다. 
같은 변수가 들어갈 수도 있으므로 if not visited가 필요하다 
- pop 시점에 방문처리하므로, 스택에 들어갔으나 아직 빠져나오지 않은 노드는 중복으로 스택에 들어갈 수 있다. 
다루는 숫자는 모두 노드의 번호 
visited = [False] * (V+1) 이렇게 적어야 한 겹 안에 들어간다 
"""
def dfs(start):
    visited = [False] * (V+1)
    stack = [start]
    path = [] 

    while stack: 
        now_node = stack.pop()

        if not visited[now_node]:
            visited[now_node] = True 
            path.append(now_node)

            """
            현재 노드와 연결된 인접 노드들을 추가한다 
            """
            for next_node in adj_list[now_node]:
                if not visited[next_node]:
                    stack.append(next_node)

    return path 


# --------------------------------------------------------------------------------------------

V, E = map(int, input().split())
numbers = list(map(int, input().split()))

# 인접 리스트 생성 
adj_list = [[] for _ in range(V+1)]

"""
스택으로 할 때는 인접리스트에 append(없으면 없음)
간선 정보 입력 - 양방향 
행은 확인할 정점, 
열은 그 정점과 연결된 정점들을 의미함 
range(E): 간선의 양쪽 끝에 정점들이 있으니까 인접 정보를 모두 추가하려면 간선만큼 돌아야 한다
first, second로 홀수번째, 짝수번째를 명확하게 한다 
"""
for i in range(E):
    first, second = numbers[2 * i], numbers[2 * i + 1]
    adj_list[first].append(second)
    adj_list[second].append(first)

"""
각 인접리스트를 sort할 때 reverse=True인 이유: 그래야 마지막에 들어가는 게 젤 작은 거(스택의 원리)
정점
"""
for i in range(1, V+1):
    adj_list[i].sort(reverse=True)

"""
DFS 실행 
시작 노드의 번호를 넣는다 
"""

path = dfs(1)
print(''.join(map(str, path)))
```