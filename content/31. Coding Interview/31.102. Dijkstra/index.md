---
title: 디익스트라 알고리즘
---
## 디익스트라 알고리즘이란?
- BFS는 그래프 중에서 weight가 없는 상황에서만 사용할 수 있다는 단점이 있다. 모든 weight가 같을 때는 BFS를 사용해도 무방하다
- 디익스트라 알고리즘은 BFS와 비슷하지만 weight가 있는 그래프에 쓰인다는 점이 다르다
- 가장 "가벼운" / "짧은" 길을 찾는 알고리즘이다
- weight가 음수가 아닌 양수라는 조건이 필요하다
## 알고리즘 설명
`그림에서 오른쪽 아래는 E이어야 함`
`edges = [[A,B,10], [A,C,3], [B,D,2], [C,B,4], [C,D,8], [C,E,2], [D,E,5]]`

- BFS와 거의 동일하다. 다만 모든 가까운 루트들을 탐색하는 것이 아니다. 지금까지 가장 짧게 도달한 루트를 기준으로 다음 루트를 탐색하는 것이라 생각하면 된다.
- 예를 들어 A에서 도달 가능한 노드는 B와 C이다, B와 C를 우선 다음 루트로서 탐색 경로에 넣는다
- 이 중에서 가장 짧게 도달 가능한 것은 3인 C이다. C부터 주변 루트를 또 다시 탐색한다. C에서는 B, D, E에 도달할 수 있다. 여기서 C에 도달하는 가장 짧은 루트는 3이라고 우리는 보장할 수 있다. A에서 시작하여 B를 거치면 어떤 방법을 사용해도 C까지는 3보다 클 수 밖에 없기 때문이다
(B, 10)
(B, 7)
(D, 11)
(E, 5)
- 이어서 E가 가장 짧게 도착할 수 있고 E까지는 5가 최단거리로 기록을 한다. E에서는 더 이상 갈 곳이 없다. 다음으로 작은 것은 7으로 B이다. B에서는 D로 갈 수 있다. 9만큼 걸린다.
(B, 10)
(D, 11)
(D, 9)
- D까지를 9가 최단거리인 것으로 기록한다. D에서는 E로 14만큼 이어져 있지만 이미 모든 알파벳은 최단거리가 기록되어 있는 상태이다.
## 코드
두 부분으로 나누어져 있다
input은 edges, n, src

- 이웃들을 표시하는 부분
```python
adj = {}
for i in range(1, n + 1):
	adj[i] = []

# s = src, d = dst, w = weight
for s, d, w in edges:
	adj[s].append([d, w])
```
- minHeap을 사용하여 최단거리를 뽑고 저장하는 부분
```python
shortest = {}
minHeap = [[0, src]]
while minHeap:
	w1, n1 = heapq.heappop(minHeap)
	if n1 in shortest:
		continue
	shortest[n1] = w1

	for n2, w2 in adj[n1]:
		if n2 not in shortest:
			heapq.heappush(minHeap, [w1 + w2, n2])
```