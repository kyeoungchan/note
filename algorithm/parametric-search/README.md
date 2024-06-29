# 파라메트릭서치
## 파라메트릭 서치 정의
Parametric Search란, 이진탐색을 응용한 알고리즘으로, 한 번의 접근으로 남은 배열 크기의 1/2 부분을 제외시킬 수 있는 효율적인 방법을 말한다.
어떤 것들을 만족하는 범위 내에서 최댓값, 혹은 최솟값을 구하는 문제에서 사용할 수 있다.  
<br>
문제 링크: https://www.acmicpc.net/problem/2805
백준에 "나무 자르기" 문제를 보면, 적어도 M미터의 나무를 집에 가져가기 위해 절단기에 설정할 수 있는 높이의 최댓값을 구하는 프로그램을 구하라고 되어있다.
- 높이를 H만큼 지정하여 절단했을 때, 얻은 나무의 길이의 합이 M미터보다 크거나 같은지를 확인해야한다.
- 다르게 말하자면, 최소 M미터의 나무를 확보할 수 있도록 설정한 절단기의 높이 중에서 최댓값을 구해야한다.

## 구현
```java
public int parametricSearch(int[] arr, int find) {
    int left = 0, right = arr.length - 1;
    
    while (left <= right) {
        mid = (left + right) / 2;
        if (arr[mid] > find) { // 부등호 포함 여부에 따라 만족하는 범위 내에서 왼쪽으로 갈지 오른쪽으로 갈지 결정된다.
            right = mid - 1;
        } else {
            left = mid + 1;
        }
    }
    return right;
}
```
네 가지의 경우로 나눌 수 있다.
1. 만족하는 범위 내에서 가장 오른쪽이면서, 만약 값이 없다면 작은 값 
2. 만족하는 범위 내에서 가장 오른쪽이면서, 만약 값이 없다면 큰 값
3. 만족하는 범위 내에서 가장 왼쪽이면서, 만약 값이 없다면 큰 값
4. 만족하는 범위 내에서 가장 왼쪽이면서, 만약 값이 없다면 작은 값
> 위 코드는 1번의 경우에 해당한다.

### 1번의 구현 코드
```java
// 만족하는 범위 내에서 가장 오른쪽이면서, 만약 값이 없다면 작은 값
public int parametricSearch(int[] arr, int find) {
    int left = 0, right = arr.length - 1;
    
    while (left <= right) {
        int mid = (left + right) / 2;
        if (arr[mid] > find) { // 부등호 포함 여부에 따라 만족하는 범위 내에서 왼쪽으로 갈지 오른쪽으로 갈지 결정된다.
            right = mid - 1;
        } else {
            left = mid + 1;
        }
    }
    return right;
}
```
- 만족하는 범위는 else로 가게 되므로 계속 범위가 좁혀짐에 따라 오른쪽으로 가게 된다.
- 최종적으로 left = right가 되었을 때 left = right + 1로 가게 되면서 반복문이 끝이난다.
- 따라서 범위 내에서 가장 오른쪽은 right가 되고, 혹은 값을 못 찾았다면 right의 값이 작은 값, left가 큰 값이므로 right가 반환값이 된다.

### 2번의 구현 코드
```java
// 만족하는 범위 내에서 가장 오른쪽이면서, 만약 값이 없다면 큰 값
public int parametricSearch(int[] arr, int find) {
    int left = 0, right = arr.length - 1;
    int answer = -1;
    
    while (left <= right) {
        int mid = (left + right) / 2;
        if (arr[mid] > find) { // 부등호 포함 여부에 따라 만족하는 범위 내에서 왼쪽으로 갈지 오른쪽으로 갈지 결정된다.
            right = mid - 1;
        } else {
            left = mid + 1;
            if (arr[mid] == find) answer = mid;
        }
    }
    if (answer != -1) return answer;
    return left;
}
```
- 만족하는 범위는 else로 가게 되므로 계속 범위가 좁혀짐에 따라 오른쪽으로 가게 된다.
- else문에서 만족하는 값이라면 answer에 계속 넣어준다.
- 최종적으로 left = right가 되었을 때 left = right + 1로 가게 되면서 반복문이 끝이난다.
- 만약 정답을 찾았다면 left가 오른쪽, right는 answer와 같은 상태로 반복문이 끝이 나게 된다.
- 하지만 정답을 못 찾았다면 큰 값을 찾아야하는 문제조건이기 때문에 left를 반환해야한다.

### 3번의 구현 코드
```java
// 만족하는 범위 내에서 가장 왼쪽이면서, 만약 값이 없다면 큰 값
public int parametricSearch(int[] arr, int find) {
    int left = 0, right = arr.length - 1;
    
    while (left <= right) {
        int mid = (left + right) / 2;
        if (arr[mid] >= find) { // 부등호 포함 여부에 따라 만족하는 범위 내에서 왼쪽으로 갈지 오른쪽으로 갈지 결정된다.
            right = mid - 1;
        } else {
            left = mid + 1;
        }
    }
    return left;
}
```
- 만족하는 범위는 if문을 통해서 계속 범위가 좁혀짐에 따라 왼쪽으로 가게 된다.
- 최종적으로 left = right일 때 right가 왼쪽으로 가면서 반복문이 끝난다.
- 따라서 left가 정답인 범위 내에서 제일 왼쪽이 된다.
- 그리고 정답이 없더라면 right가 작은 값, left가 큰 값이므로 left가 반환값이다.

### 4번의 구현 코드
```java
// 만족하는 범위 내에서 가장 왼쪽이면서, 만약 값이 없다면 작은 값
public int parametricSearch(int[] arr, int find) {
    int left = 0, right = arr.length - 1;
    int answer = -1;
    
    while (left <= right) {
        int mid = (left + right) / 2;
        if (arr[mid] >= find) { // 부등호 포함 여부에 따라 만족하는 범위 내에서 왼쪽으로 갈지 오른쪽으로 갈지 결정된다.
            right = mid - 1;
            if (arr[mid] == find) answer = mid;
        } else {
            left = mid + 1;
        }
    }
    if (answer != -1) return answer;
    return right;
}
```
- 만족하는 범위는 if문으로 가게 되므로 계속 범위가 좁혀짐에 따라 왼쪽으로 가게 된다.
- if문에서 만족하는 값이라면 answer에 계속 넣어준다.
- 최종적으로 left = right가 되었을 때 right = left - 1로 가게 되면서 반복문이 끝이난다.
- 만약 정답을 찾았다면 left는 answer와 같은 상태, right는 왼쪽에 위치한 상태로 반복문이 끝이 나게 된다.
- 하지만 정답을 못 찾았다면 작은 값을 찾아야하는 문제조건이기 때문에 right를 반환해야한다.
