---
title: "이진탐색 알고리즘 O(logN)"

categories:
- Algorithm

tags:
- Algorithm
---

# Binary Search 사용 조건

1. 정렬된 배열
2. 배열의 중간값에 접근이 가능해야함(Linked List에서는 사용 불가)
3. 찾으려는 정확한 값이 내부에 존재해야함   

   
```java
    // arr = [1, 2, 3, 4, 5, 6, 7, 8, 9]
    // target = 6
    public int solution(int target, int[] arr) {
        int min = 0;
        int max = arr.length();

        while(min <= max) {
            int mid = (min + max) / 2;
            if(arr[mid] == target)
                return mid;
            else if(arr[mid] < target)
                min = mid + 1;
            else
                max = mid - 1;
        }

        return -1;
    }
```
이러한 경우에 mid 값은 내가 찾으려는 임의의 중간 값이다.   
target일 경우에 그대로 반환 target이 아닐때는 조건을 수행하는데   
1. mid가 target보다 작으면 **min = mid + 1**
2. mid가 target보다 크면 **max = mid - 1**   

이때 mid를 재탐색할때 mid + 1, mid - 1등 값을 꼭 변경시켜줘야 무한루프에 빠지지 않음

# 정확히 찾으려는 값이 없을 경우

정확히 어떤 값을 찾는게 아니고 주어진 조건을 만족하는 값들 중 하나를 찾으려면
Binary Search 알고리즘을 이용한 [Lower Bound, Upper Bound]가 있다
이 알고리즘들 역시 배열이 정렬된 경우에만 사용 가능
<br>
<br>
<br>

### 1. Lower Bound
**찾고자 하는 값이 처음으로 나타나는 위치를 찾음**   
[1, 3, 5, 7, 9] 에서    
- 6이상의 값? 7이 나옴   
- 7이상의 값? 7이 나옴

   
```java
  // arr = [1, 3, 5, 7, 9]
  // target = 6 (정확한 값이 아닌 6 이상의 값)
  public int lowerBound(int[] arr, int target) {
      int left = 0;
      int right = arr.length;
    
      while (left < right) {
          int mid = left + (right - left) / 2;
        
          if (arr[mid] < target)
                left = mid + 1;
          else 
                right = mid;
      }
    
      return left;
  } 
```

#### 이진탐색과의 차이
이진탐색은 값을 못찾으면 -1을 반환했지만 여기선 이상인 값만 찾으면 되기에 left 값을 그대로 반환함

```java
    // 1번 (left < right)의 경우
    public int lowerBound(int[] arr, int target) {
        int left = 0;
        int right = arr.length;
      
        while (left < right) {
            int mid = left + (right - left) / 2;
      
            if (arr[mid] < target)
                left = mid + 1;
            else
                right = mid;
        }
      
        return left;
    }
  
    // 2번 (left <= right)의 경우
    public int lowerBound(int[] arr, int target) {
        int left = 0;
        int right = arr.length;

        while (left <= right) {
            int mid = left + (right - left) / 2;

            if (arr[mid] < target)
                left = mid + 1;
            else
                right = mid - 1;
        }

        return left;
  }
```  
사실 둘 다 별 차이 없는 코드인데   
**left <= right** 는 [left, right]로 탐색 범위를 잡고 left와 right가 같을 때 까지 탐색함      
**left < right** 는 [left, right)로 탐색 범위를 잡고 탐색을 함 **right가 포함되지 않음**   
   
많이 헷갈리는 코드였는데 딱히 별 차이는 없고 조건만 약간 다름   
그냥 편한대로 내가 편한대로 코드 짜면 될 것 같음


### 2. Upper Bound
**찾으려고 하는 값을 초과하는 값의 처음 위치를 찾음**
```java
    // arr = [1, 3, 5, 7, 9]
    // target = 6
    public int upperBound(int[] arr, int target) {
        int left = 0;
        int right = arr.length;
    
        while (left < right) {
            int mid = left + (right - left) / 2;

            if (arr[mid] <= target)
                left = mid + 1;
            else
                right = mid;
        }

        return left;
    }
```

얘도 lower bound와 특별히 차이나는건 없고   
그냥 조건문에서 arr[mid] <= target 여기서 <= 를 신경 써주면 됨

lower bound에선 arr[mid] < target으로 비교해서 현재 중간 값이 타겟 보다 작으면    
left를 증가시키고 다음 범위를 탐색함

upper bound에선 arr[mid] <= target으로 비교해서 현재 중간 값이 타겟 값보다 작거나 같으면 left를 증가시킴
그렇게 해서 left 값이 right를 넘을 때까지 탐색해서 초과 값을 찾음

# 결론
binary Search는 '일치'를 찾음   
lower bound는 '이상'을 찾음   
upper bound는 '초과'를 찾음

차근차근 생각하면 이해가 되는데 빨리빨리 써먹을려면 문제를 많이 풀어봐야 할것 같음 + 부등호에 특별히 신경을 써야함