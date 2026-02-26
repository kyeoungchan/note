# 🧑🏻‍💻 B-Tree
<hr>

> [!IMPORTANT]
> 시간복잡도: `O(longN)`
- B-tree란 자식 노드가 2개 이상인 트리
- 균형 트리(Balanced Tree)로서, 최상위 루트 노드에서 리프 노드까지의 거리가 모두 동일하다.
    - ❗️ Binary Tree가 아니다.

## ✅ 구조 및 특성
<hr>

> [!NOTE]
> B-Tree는 트리 구조의 최상위에 하나의 Root Node가 존재하고, 그 하위에 자식 노드가 붙어 있는 형태다.  
> 가장 하위에 붙어 있는 노드가 Leaf Node, 중간의 노드를 Branch Node라고 한다.  
> 데이터베이스에서 인덱스와 실제 데이터가 저장된 데이터는 따로 관리되는데, 인덱스의 Leaf Node는 항상 실제 데이터 레코드를 찾아가기 위한 주솟값을 가지고 있다.

![b_tree_index_structure.png](../res/b_tree_index_structure.png)  
위 그림과 같이, 인덱스의 키 값은 모두 정렬돼 있지만, 데이터 파일의 레코드는 정렬돼 있지 않고 임의의 순서로 저장돼 있다.

> [!TIP]
> 대부분의 RDBMS의 데이터 파일에서 레코드는 특정 기준으로 정렬되지 않고 임의의 순서로 저장된다.  
> 하지만 MySQL의 InnoDB 테이블에서 레코드는 클러스터되어 디스크에 저장되므로 기본적으로 프라이머리 키 순서로 정렬되어 저장된다.  
> ➡️ 이는 오라클의 IOT(Index organized table)이나 MS-SQL의 클러스터 테이블과 같은 구조를 말한다.  
> 다른 DBMS에서는 클러스터링 기능이 선택 사항이지만, MySQL의 InnoDB에서는 사용자가 별도의 명령이나 옵션을 선택하지 않아도 디폴트로 클러스터링 테이블이 생성된다.




<br>

출처
[Real MySQL 8.0](https://product.kyobobook.co.kr/detail/S000001766482)
