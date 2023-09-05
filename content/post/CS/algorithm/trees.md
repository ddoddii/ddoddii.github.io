+++
author = "Soeun"
title = "Trees"
date = "2023-05-02"
description = "Trees & Binary Trees"
categories = [
    "CS"
]
tags = [
    "자료구조"
]
image = ""
+++
## Trees

### General Trees
- Tree의 구조는 계층적(hierachical)이다. 즉 데이터를 계층적 구조로 저장한다.
- Top element를 제외한 다른 원소들은 모두 부모 원소와 0개 이상의 자식 원소들이 있다. 
- Top element = Root of Tree
- edge of tree : u가 v의 부모이거나 그 반대일때 노드 (u,v) 의 집합이다.
- path of tree : edge 들의 연속적인 집합이다. 
- leaf of tree : 자식이 없는 노드
- depth of p in Tree : p가 가진 조상들의 수 (p는 제외)
    ```python
    def depth(self,p):
        if self.is_root(p):
            return 0
        else:
            return 1 + self.depth(self.parent(p))
    ```
- height of p in Tree : p가 leaf 면 height = 0, 아니면 1 + max(p 자식의 height)
    1. 모든 leaf node 의 depth 를 다 비교 -> leaf 의 가장 큰 depth = height
    2. 1 + max(자식 height) -> recursion
    ```python
    def height1(self,p):
        return max(self.depth(p)) for p in self.positions() if self.is_leaf(p)

    def height2(self,p):
        if self.is_leaf(p):
            return 0 
        else:
            return 1 + max(self.height2(c) for c in self.children(p)) 
    ```
### Binary Trees
- Binary Tree 의 조건 3가지
  - 자식의 개수는 0,1,2 중 1개이다.
  - 각 자식은 left 또는 right child 로 라벨되어 있다.
  - left child 의 순서가 먼저이다.
- Proper Binary Tree (=Full Tree) : 각 노드는 0 또는 2 개의 자식 노드가 있다.
- Perfect Tree : 모든 노드의 자식이 2개이다.
- Complete Tree : 왼쪽에서 오른쪽으로 자식 노드들을 채운다. 

### Implementing Trees
- DFS(Depth First Search)
  - Preorder traversal
    - root 먼저 방문
    - children 에 대해 recursion
    - 우선순위 : 부모 -> 왼 -> 오
  - Postorder traversal
    - root 를 가장 마지막에 방문
    - 우선순위 : 왼 -> 오 -> 부모
  - Inorder traversal
    - root가 중간에 있음
    - 우선순위 : 왼 -> 부모 -> 오
- BFS(Breadth First traversal)
  - 같은 depth 에 있는 모든 node 방문
  - Queue 로 구현 (FIFO)

### Binary Search Tree(BST)
- 새로 노드가 들어오면 root 노드와 비교해서, 작으면 왼쪽에 크면 오른쪽에 붙인다.
- 최종적으로 root 의 왼쪽에 있는 모든 노드들은 root 보다 작고, 오른쪽에 있는 모든 노드들은 root 보다 크다.

### BigO
- Perfect Binary Tree 라고 가정하자. 이때 노드를 찾는 것(lookup), 새로운 노드 삽입(insert), 노드 제거(remove) 모두 O(logn)이다. 왜냐하면 Perfect Binary Tree 에서 계속 Divide & Conquer 를 사용할 수 있기 때문이다. 
    ![img](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/dea2c9a6-5e7a-403b-b055-35eef083808c)
- Worst case 는 tree 가 갈라지지 않고 계속 이어질 때이다. 이때 연산의 시간복잡도는 O(n)이다. 하지만 BST 에서는 worst case 시나리오를 생각하기 보다는, perfect binary tree 일 경우를 생각해서 시간복잡도는 O(logn)이라 한다. 
    ![img](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/99e1f2e5-874c-447f-a3a4-aa56647ed298)
- Linked List 와 BST 의 비교 
    |연산|Linked List|BST|
    |:---:|:---:|:---:|
    |lookup()|O(n)|O(logn)|
    |remove()|O(n)|O(logn)|
    |insert()|<span style="background-color: #FFF59D">O(1)</span>|O(logn)|
    - 여기서 중요한 점이 Linked List 에서 뒤에 insert 하는 것은 언제나 O(1)이라는 것이다. 그래서 remove, lookup 을 많이 안하고 추가하는 경우가 많은 데이터는 linked list 방식으로 저장하는 것이 더 낫다. 
## Reference
- 2023-1 데이터 사이언스를 위한 자료구조 및 알고리즘, 연세대학교 송경우 교수님
- Python Data Structures & Algorithms , Udemy