```python 
"""
250924
스택 방식과 차이점 
- sort 안해도 됨 
- 돌다가 바로 다음 노드 발견하면 갔다가 옴 
- 한 번만 가면 되기 때문에 백트랙킹을 안하면 됨 (visited가 추적함)
"""

import sys
sys.stdin = open('input.txt', 'r')

def dfs(now_node, adj_list, visited, path):
    # 1. 현재 노드 방문 처리 
    visited[now_node] = True 
    path.append(now_node)

    """
    # 2. 현재 노드와 연결된 노드들을 직접 순회 
    """
    for next_node in adj_list[now_node]:
        if not visited[next_node]:
            dfs(next_node, adj_list, visited, path)


V, E = map(int, input().split())
numbers = list(map(int, input().split()))

adj_list = [[] for _ in range(V+1)]

for i in range(E):
    first, second = numbers[2* i], numbers[2*i + 1]
    adj_list[first].append(second)
    adj_list[second].append(first)

"""
안에서 while 문이 도는 게 아니라면 밖에서 visited나 시작 배열을 만들어야 하고 
안에서 while 문이 돌아서 반복되지 않는 부분이 있다면 안에서 해도 괜찮고, 또 밖에서 할 이유가 없다. 
"""
visited = [False] * (V+1)
path = []

dfs(1, adj_list, visited, path)
print(''.join(map(str, path)))