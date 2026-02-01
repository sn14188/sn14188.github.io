---
title: "Bellman-Ford Algorithm"
date: 2025-12-03
---

벨만-포드 알고리즘을 공부하고 정리해본 기록입니다.
<br><br>

## Bellman-Ford Algorithm

벨만-포드 알고리즘은 단일 출발 정점에서 가중치가 있는 방향 그래프의 모든 다른 정점까지의 최단 경로를 계산하는 알고리즘입니다.

- 그래프: 정점과 정점들을 연결하는 간선으로 이루어진 자료구조
- 방향 그래프: (한 정점에서 다른 정점으로) 간선에 방향이 있어 이동 가능한 경로가 정해져 있는 그래프
- 가중치: 각 간선에 부여된 값으로, 해당 간선을 따라 이동할 때의 비용을 뜻합니다

동일한 문제에 대해 다익스트라 알고리즘보다 느리지만, 일부 가중치가 음수인 그래프를 처리할 수 있으므로 더 일반적이라고 할 수 있습니다.<br>
그래프에 도달 가능한 "음의 사이클"이 있으면 최소 비용 경로는 없습니다. 이 사이클을 돌면 계속 비용이 줄어들기 때문입니다. 벨만-포드 알고리즘은 이런 경우를 감지하고 리포트할 수 있습니다.

## Relaxation

relaxation이란 그래프를 순회하며 경로를 찾는 과정에서 더 비용이 작은 경로가 발견되면 그 정보를 갱신하는 과정을 말합니다.<br>
다음과 같은 상황을 가정해보겠습니다.

```py
B ---w---> C
```

- `A`: 시작 정점
- `B`: 중간 경로 후보 정점
- `C`: 거리를 갱신하려는 대상 정점
- `w`: `B -> C`의 가중치

알고 있는 정보:

- `dist[B]`: `A`에서 `B`까지의 최소 비용 거리
- `dist[C]`: `A`에서 `C`까지의 최소 비용 거리

relaxation은 다음을 확인합니다:

```py
if dist[C] > dist[B] + w:
    dist[C] = dist[B] + w
```

이 경우에 `dist[C]`가 10, `dist[B]`가 3, `w`가 5라면 `dist[C]`는 10에서 8로 갱신됩니다. 벨만-포드 알고리즘에서는 이 과정을 모든 간선에 대해 수행합니다.<br>
알고리즘 구현에서는 시작 정점의 비용을 0으로, 다른 정점들까지의 비용을 무한대로 초기화합니다. 그러면 시작 정점의 실제 값을 바탕으로 연결된 정점들의 비용이 유한한 값으로 갱신되기 시작하고, 이후 반복을 통해 더 먼 정점들까지 갱신이 확장됩니다.<br><br>

<img src='/images/bellman-ford/happy.png' width="800">

### 1회 Relaxation으로 충분하지 않은 이유

relaxation 로직에 비용을 비교하고 더 작은 값으로 업데이트하는 과정이 포함돼있어서, 처음에는 위 경우처럼 1회 relaxation으로도 최단 경로를 구할 수 있지 않을까 생각했습니다.<br>
하지만 그래프의 구조나 간선의 순서를 알 수 없으며, 아래 경우처럼 최단 경로를 이루는 간선들이 반드시 순서대로 등장하는 것도 아니기 때문에 여러 차례 반복이 필요합니다.<br><br>

<img src='/images/bellman-ford/sad.png' width="800">

### 그렇다면 얼마나 반복해야 하는가

벨만-포드 알고리즘은 한 번의 반복에서 최소 비용의 정보를 간선 한 단계만큼 전파할 수 있습니다. 시작 정점에서 직접 연결된 정점들은 첫 반복에서 갱신되지만, 두 개 이상의 간선을 거쳐 도달하는 정점들은 여러 번의 반복을 거쳐야 최소 비용이 반영될 수 있습니다.<br>
위 예시에서 `A`에서 `X`까지의 최단 경로는 `A -> C -> D -> X`으로 세 개의 간선으로 구성됩니다. 비효율적인 순서의 간선에서 첫 반복에서는 `C`만 갱신되고, 두 번째 반복에서야 `D`가 갱신되며, 세 번째 반복에서 비로소 `X`가 최단 거리 3으로 갱신됩니다.<br>
이처럼 최단 경로의 앞부분이 늦게 등장하면 한 번의 반복으로는 전체 경로를 반영할 수 없으며, 벨만-포드 알고리즘이 $$|V|−1$$번의 반복을 수행하는 이유가 여기에 있습니다.

## Implementation

[Baekjoon Online Judge](https://www.acmicpc.net/problem/11657)에 있는 문제를 살짝 변형해 단순하게 만들어봤습니다.<br>
변형된 입력 형식은 간선 정보와 시작 정점을 받고, 출력은 시작 정점을 제외한 각 정점까지의 최소 비용을 반환합니다.<br>
테스트 케이스는 음의 사이클이 없는 경우와 있는 경우를 하나씩 작성했고, 음의 사이클이 존재하는 경우에는 `[-1]`를 반환하도록 했습니다.

```py
assert solution(["A B 4", "A C 3", "B C -1", "C A -2"], "A") == [4, 3]
assert solution(["A B 4", "A C 3", "B C -4", "C A -2"], "A") == [-1]
```

먼저 입력과 출력 형식에 맞게 함수 시그니처를 정의했습니다.

```py
def solution(info: list[str], start: str) -> list[int]:
    answer = []
    return answer
```

`info` 문자열 리스트에서 간선 정보를 추출해 `edges`에 저장했고, 정점 수를 파악하기 위해 `vertices` 집합을 사용했습니다.<br>
또 시작 정점을 제외한 모든 정점의 비용을 무한대로, 시작 정점의 비용을 0으로 초기화했습니다.

```py
import math


def solution(info: list[str], start: str) -> list[int]:
    edges = []
    vertices = set()
    for line in info:
        u, v, w_str = line.split()
        edges.append((u, v, int(w_str)))
        vertices.add(u)
        vertices.add(v)

    dist = {vertex: math.inf for vertex in vertices}
    dist[start] = 0

    answer = []
    return answer
```

<br>
모든 간선에 대해 $$|V|-1$$번 relaxation을 수행합니다.

```py
import math


def solution(info: list[str], start: str) -> list[int]:
    edges = []
    vertices = set()
    for line in info:
        u, v, w_str = line.split()
        edges.append((u, v, int(w_str)))
        vertices.add(u)
        vertices.add(v)

    dist = {vertex: math.inf for vertex in vertices}
    dist[start] = 0

    n = len(vertices)
    for _ in range(n - 1):
        for edge in edges:
            u, v, w = edge
            if dist[u] + w < dist[v]:
                dist[v] = dist[u] + w

    answer = []
    return answer
```

음의 사이클이 있다면 최소 비용이 계속 작아지는 부분이 있을 것입니다.<br>
그래서 마지막으로 모든 간선을 확인해 비용이 더 감소하는 경우가 있는지 검사했습니다. 감소가 발생한다면 이는 음의 사이클이 존재한다는 의미이므로 `[-1]`을 반환하고, 그렇지 않다면 계산된 최소 비용을 `answer`에 담아 반환합니다.

```py
import math


def solution(info: list[str], start: str) -> list[int]:
    edges = []
    vertices = set()
    for line in info:
        u, v, w_str = line.split()
        edges.append((u, v, int(w_str)))
        vertices.add(u)
        vertices.add(v)

    dist = {vertex: math.inf for vertex in vertices}
    dist[start] = 0

    n = len(vertices)
    for _ in range(n - 1):
        for edge in edges:
            u, v, w = edge
            if dist[u] + w < dist[v]:
                dist[v] = dist[u] + w

    for u, v, w in edges:
        if dist[u] + w < dist[v]:
            return [-1]

    answer = [dist[vertex] for vertex in sorted(vertices) if vertex != start]

    return answer
```

<br>
이 구현은 모든 간선에 대해 $$|V|−1$$번 relaxation을 수행하므로 시간 복잡도는 $$O(VE)$$입니다.

## Dijkstra Algorithm과 비교

두 알고리즘 모두 가중치 그래프에서 최단 경로를 찾지만, 다익스트라 알고리즘은 모든 간선의 가중치가 0 이상이어야만 올바르게 작동합니다.<br>
왜냐하면 다익스트라는 현재까지 가장 최소 비용인 정점을 확정하며 진행되기 때문입니다. 그런데 음의 가중치 간선이 존재하면 이미 확정된 정점의 비용보다 더 작은 비용의 경로가 이후에 발견될 수 있어 이 전제가 망가지게 됩니다.<br>
다익스트라 알고리즘은 별도 포스트에서 더 깊게 다뤄보겠습니다.

## Takeaways

1. 벨만-포드 알고리즘은 음의 비용인 간선이 포함된 그래프에서도 올바르게 최단 경로를 구할 수 있습니다
2. relaxation은 한 번의 반복에서 경로를 한 단계씩만 확장하기 때문에, 최대 $$\lvert V\rvert -1$$번 반복이 필요합니다
3. 알고리즘의 시간 복잡도는 $$O(VE)$$입니다
   <br><br>

_출처:<br>
[1] Wikipedia (["Bellman–Ford algorithm"](https://en.wikipedia.org/wiki/Bellman%E2%80%93Ford_algorithm))<br>_
