---
title: "[Algorithm] 순열, 조합, 부분집합"
excerpt: "집합 문제에서 순열, 조합, 부분 집합을 구해보자."
categories:
    - Algorithm
tags:
    - [Algorithm]

toc: true
toc_sticky: true
toc_label: "목차"
toc_icon: "sticky-note"

date: 2024-01-23
last_modified_at: 2024-01-24
---

## 순열(Permutation)

### 정의

순열이란 크기가 N인 ~~순서가 정해지지 않은~~ 집합 A에서 각 원소들을 순서대로 나열한 모든 경우의 수이다.

<div style="text-align: center;">
    <img width="300" height="300" src="/image/posts/algorithm/permutation/img1.png">
</div>

크기가 N인 순열의 모든 경우의 수는 N!개이다.

크기가 3인 집합에서 순열은 다음과 같이 구현할 수 있다.

<script src="https://gist.github.com/ma-an-jong/6de3309e96c9bd790768f3d0a150155e.js"></script>

크기가 3인 순열에서 3개의 수를 선택하기 위한 모든 경우의 수를 탐색하기 위해서 반복문을 사용하였다. 즉, 크기가 N인 집합에서 순열을 만들려면 for문을 N번 중첩해야한다. 

### 문제점

하지만 이 방법은 N이 크거나, 동적으로 크기가 달라지는 집합에서는 사용할 수 없다. N이 15가 된다면 for문을 15번 중첩 시키는 코드를 짜야 할 뿐만 아니라, 각 N이 올 수 있는 모든 경우를 직접 코딩해야한다. 

### 재귀

이런 불편함에서 벗어나기 위해서  for문을 N번 중첩시키기 위해서 재귀함수를 이용한다. 각 함수는 반복문을 1개 가지고 있으며, k번째에 어떤 원소가 올지를 선택한다. 이를 k가 N일때까지 반복하면 총 N개의 원소가 선택된것으로 이를 통해 순열을 찾을 수 있다.

### 구현

각 재귀함수에서는 k번째 원소를 선택한다. 이미 선택된 원소는 visited 배열로 체크하여 중복을 피할 수 있다. 또한 선택한 수는 select 배열에 저장한다.

<script src="https://gist.github.com/ma-an-jong/b97f18faab8d9223baee1250b6594dd6.js"></script>

## 조합(Combination)

### 정의

조합은 크기가 N인 집합에서 K개의 원소를 선택한 결과이며, 순서가 중요하지 않다.

### 순열과 유사성

순열도 조합과 마찬가지로 반복문을 K번 중첩하여 이를 구할 수 있다, 그러나 순열과 마찬가지로 동적으로 변하는 K와 여러 반복문을 중첩하여 구현하는것은 매우 번거롭다.

### 재귀

조합도 마찬가지로 재귀로 구할 수 있다. 그러나 반복문을 K번 중첩한다는 개념보다는, 현재 index의 원소를 선택할것인가? 선택하지 않을것인가? 개념으로 봐서 선택한다면 cnt의 개수를 1증가시키고, 선택하지 않는다면 cnt를 증가시키지 않고 다음 index+1로 넘어간다. 이렇게 index가  n이 되거나, cnt가 K가 될때까지 이를 수행하면 모든 조합을 찾을 수 있다.

### 구현

방법은 순열과 비슷하게 선택된 원소를 저장하는 select를 이용하고 cnt를 k까지 증가시키면서 모든 조합을 탐색할 수 있다.

<script src="https://gist.github.com/ma-an-jong/029eda096448fd7f0258604098abe2f5.js"></script>

## 부분집합(Subset)

### 정의

부분 집합은 집합 A의 일부 원소를 갖는 집합이다. 조합에서는 N개 원소 중 K개를 선택한 모든 경우의 수라면 부분 집합은 K가 0~N일때, 모든 조합이 부분 집합이며 경우의 수는 총 2^N이다.


<div style="text-align: center;">
    <img src="/image/posts/algorithm/permutation/img2.png">
</div>

### 부분 집합의 특징

부분 집합은 선택하지 않는 경우의 수도 있는것이 특징이다. 보통 N이 63보다 작거나 같다면 비트 마스킹을 통해서 부분 집합을 표현할 수 있다. 여기서는 재귀를 이용한 방법을 이야기한다.

### 재귀

부분 집합은 선택하지 않는 경우의 수도 있기 대문에 반복문을 N번 중첩하는것으로 구현하는것은 일반적이지 않다. 따라서 재귀를 이용하여 현재 집합의 원소를 선택할것인가? 선택하지 않을것인가로 구분하여 index가 N이 될 때 ,선택된 원소들이 부분 집합이 된다.

### 구현

<script src="https://gist.github.com/ma-an-jong/5b72133d5bdfef24bf197b5db7e67429.js"></script>