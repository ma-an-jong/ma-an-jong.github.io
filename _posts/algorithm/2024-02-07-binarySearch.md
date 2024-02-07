---
title: "[Algorithm] 이분탐색(Binary Search)"
excerpt: "lower bound, upper bound 그리고 매개변수 탐색에 대하여 알아봅시다."
categories:
    - Algorithm
tags:
    - [Algorithm]
use_math: true

toc: true
toc_sticky: true
toc_label: "목차"
toc_icon: "sticky-note"

date: 2024-02-07
last_modified_at: 2024-02-07
---

## 이분 탐색

### Lower Bound & Upper Bound

이분 탐색은 두 개의 파티션으로 분할된 구간에서 분할 경계의 위치를 O(logN)에 찾아주는 알고리즘이다.

C++에는 std::lower_bound와 std::upper_bound가 있지만 자바에는 없다. ~~(정말 부럽다)~~

정렬된 배열에서 x의 lower bound는 정렬 상태를 유지한 채 x를 삽입할 수 있는 첫번째 위치이며 upper bound는 마지막 위치이다. 따라서 lower bound는 x보다 크거나 같은 원소의 첫번째 위치이며 upper bound는 x보다 큰 원소의 첫번째 위치이다.

<div style="text-align: center;">
    <img src = "/image/posts/algorithm/binarySearch/binarySearch.jpeg">
</div>

### 이분 탐색적인 설명

lower bound는 배열의 값이 x 미만인지, x 이상인지에 따라서 배열을 False/True 두 파티션으로 나눌 수 있다.

upper bound는 배열의 값이 x이하인지 ,x 초과인지에 따라서 False / True 두 파티션으로 나눌 수 있다.

두 함수 모두 첫 번째 True가 가리키는 위치를 찾아준다.

<div style="text-align: center;">
    <img src = "/image/posts/algorithm/binarySearch/bound.jpeg">
</div>


### 첫 번째 True 찾기

MIN이상 MAX이하의 구간에서 f[i] = True인 첫번째 i를 찾는 코드는 다음과 같다.

```java
// f[i] = true인 첫번째 i를 리턴한다.
// 만약 닫힌 구간 [MIN , MAX]가 전부 false라면 MAX+1을 리턴한다.
int first_true(int min , int max) {
	int l = min , r = max + 1; 
	while(l != r) { // [l,r)은 false일 가능성이 있는 미확인 구간이다.
		// (l+r) / 2는 오버플로우 위험이 있고 
		// (l+r)이 음수인 경우 나누기 2가 의도와 다르게 동작할 수 있다.
		int mid = l + (r-l)/2;

		//결과에 따라서 남은 구간을 반으로 줄인다.
		if(f[mid]) { // 중간 값이 true이면 [mid+1,r) 구간은 true
			r = mid; // [l,mid) 미확인 구간 탐색
		} else { // 중간 값이 false이면 [l,mid] 구간은 false
			l = mid +1; // [mid+1,) 미확인 구간 탐색
		}
	}

	//미확인 구간이 전부 없어지면 l은 첫번째 true를 가리킨다.
	return l;
}
```

중간 값이 true가 된다면 [m+1,r) 구간은 항상 true이다.

따라서 왼쪽 구간인 [l,m)을 추가 탐색하면된다.

<div style="text-align: center;">
    <img src = "/image/posts/algorithm/binarySearch/midTrue.jpeg">
</div>

중간값이 false가 된다면 [l,m) 구간은 항상 false이다.

따라서 오른쪽 구간인 [m+1,r)을 추가 탐색하면 된다.

<div style="text-align: center;">
    <img src = "/image/posts/algorithm/binarySearch/midFalse.jpeg">
</div>

#### 첫번째 True 찾기 방식의 lower bound 구현 

```java
int lowerBound(int low, int high, int x) {

	while(low < high) {
		int mid = low + (high - low) / 2;
		if(mid >= x) {
				high = mid;
		} else {
				low = mid+1;
		}
	}

	return low;
}
```

#### 첫번째 True 찾기 방식의 upper bound 구현

```java
int upperBound(int low, int high, int x) {
	while(low < high) {
		int mid = low + (high - low) / 2;
		if(mid <= x) {
			low = mid + 1;
		} else {
			high = mid;
		}
	}

	return low;
}
```

### 마지막 True 찾기

구간이 true/false로 나눠져 있을 때 마지막 true를 찾는 방법으로도 구현할 수 있다.

첫번째 true를 이용한 논리를 뒤집으면 된다.

MIN이상 MAX이하인 구간에서 f[i] = true인 마지막 i를 찾는 코드는 다음과 같다.

```java
//f[i] = true인 마지막 i를 탐색

// [MIN, MAX]가 전부 false라면 MIN - 1을 리턴한다
int last_true(int min, int max) {
	int l = min -1 , int r = max;

	while(l < r) {
		int m = r - (r-l) / 2;
		
		if(f[m]) {
			l = mid;
		} else {
			r = mid-1;
		}
	}

	return l; // 또는 r
}
```

### 주의사항

이분 탐색을 구현하는 방법은 사람마다 다양하다.

STL 구현 방식은 위에 처럼 탐색 구간 끝에 1을 더해서 반열린 구간으로 만드는 방법이다.

종료 조건과 구간을 쪼개는 과정에서 무한 루프를 조심해야 한다. 본문 구현에서 +1, -1을 다른 쪽에 하거나, m값을 구하는 방법을 바꾼다면 무한 루프가 되거나 어느 한 점은 탐색하지 않고 건너 뛸것이다.

올바른 이분 탐색 사고는 항상 구간을 절반으로 나눠서 생각한다. 아래 코드처럼 3가지 이상의 경우를 고려하는 방식은 피하는것이 좋다.

```java
while (l != r) {
	 int m = l + (r - l) / 2;
	 if (array[m] < key) {
	   doSomething();
	 } else if (array[m] == key) {
	   doSomething();
	 } else {
	   retrun m;
	 }
}
```

## Parametric Search

이분 탐색을 단순히 정렬된 배열 안에서 원소를 찾는 알고리즘으로 한정하면 안된다.

이를 응용하여 최적화 문제를 결정 문제로 바꾸어 푸는 알고리즘이 있다.

### 매개변수 탐색

매개 변수 탐색 알고리즘은 최적화 문제를 결정 문제로 바꾸어 해결하는 알고리즘이다.

- 최적화 문제 → f(x) = true가 되는 x의 최댓값은?
- 결정 문제 → 어떤 x에서 f(x) = true인가?

### 결정 문제

가능한 모든 x값마다 f(x)가 true인지 false인지 검사하면 최적화 문제를 결정 문제로 풀 수 있다. 

하지만 이 방법은 탐색할 범위가 증가하거나 쿼리의 양이 증가 할수록 비 효율적이다.

### 결정 문제 최적화

#### 조건

x1 < x2 , f(x1) < f(x2) 을 만족한다면 가능한 모든 x값마다 f(x)가 true인지 false인지 검사하는 결정 문제를 최적화 할 수 있다. 

#### 아이디어

f(x)가 정렬된 형태이기 때문에 이분 탐색 아이디어를 그대로 적용할 수 있다.

임의의 값 x에  대하여 true인지 false인지 검사하면 f(x)가 true일 경우 x 이하도 전부 true이며 f(x)가 false일 경우 x이상도 전부 false이다.

마치 이분 탐색을 하듯 x에 대한 범위를 절반 씩 줄일 수 있고, O(log(x의 범위))의 결정 문제로 최적화 할 수 있다.

#### 예시

> n개의 섬과 m개의 다리가 있다.
> 
> 
> 다리마다 무게 제한(1<= 제한 <= 10^9)이 있어서 제한보다 무거운 차량이 지나가면 다리가 무너진다. 
> 
> 0번 섬에서 n – 1번 섬으로 이동할 수 있는 차량 무게의 최댓값은 얼마인가?
> 

다익스트라, 분리 집합, 매개 변수 탐색 등을 이용하여 풀 수 있다.

매개 변수 탐색을 이용한 해법은 다음과 같다.

- 무게가 x인 차량이 0번 섬에서 n-1번 섬으로 이동할 수 있다면, 그 길을 그대로 따라서 x보다 가벼운 차량도 이동할 수 있다. 만약 무게가 x인 차량이 0번 섬에서 n - 1번 섬으로 이동할 수 없다면 x보다 무거운 차량도 이동할 수 없다.
- 따라서 f(i) = 무게가 i인 차량이 0번 섬에서 n - 1 번 섬으로 이동 여부로 이분 탐색이 가능하다.

<div style="text-align: center;">
    <img src = "/image/posts/algorithm/binarySearch/parametric.jpeg">
</div>

```java
boolean f(int limit) {
	// 제한이 limit 이하인 다리만 사용하여 0 -> n-1 이동이 가능한가?
}

// 0 -> n-1로 이동이 가능한 차량 무게의 최댓값을 리턴
// 만약 0이 리턴되면 0 -> n-1로 다리가 존재하지 않는 경우

int solve(){
	int l = 0, r = 1e9;

	while(l != r) {
		int m = r - (r-l) / 2;
		
		if(f(m)) {
			l = m;
		} else {
			r = m - 1;
		}
	}

	return l;
}
```