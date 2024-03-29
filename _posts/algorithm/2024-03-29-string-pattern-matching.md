---
title: "[Algorithm] 문자열 패턴 매칭"
excerpt: "라빈 카프, KMP 알고리즘을 이용하여 부분 문자열의 포함 여부를 찾아봅시다."
categories:
    - Algorithm
tags:
    - [Algorithm]
use_math: true

toc: true
toc_sticky: true
toc_label: "목차"
toc_icon: "sticky-note"

date: 2024-03-29
last_modified_at: 2024-03-29
---

# 문자열 패턴 매칭

다음과 같이 200개 문자로 이루어진 문자열 T 안에 10개의 문자로 이루어진 P가 존재하는지 찾아보자.

```java
T = "
TGTTАACCAAGGAATGGATCTGTGTCGTTCCACGTTCGAAGGCCTTTTCTGATGАААТGААGАТАGGТТТСААСТССАСАGЄ
ТТАТТСТСGТАТGАТСТТААССАААААТGАТGААGТТТТСТССААGАТТАСТGАААААССТGААТТGАТТААСGАТАТСТТАТТGG
ААТGТGGТТТСССАААСАСТТСТGGТСАААААСС
"

P = "СТТАТТGGАA"
```

## Brute-Force 방법

### 아이디어

본문 문자열을 처음부터 끝까지 차례대로 순회하면서 패턴 내의 문자들을 하나씩 비교하는 방식이다.

<div style="text-align: center;">
    <img src = "/image/posts/algorithm/string-pattern-matching/pattern.png">
</div>


### 시간 복잡도

패턴 매칭이 가장 많은 연산의 경우는, 항상 P의 마지막 글자에서 틀리는 경우이다. 

문자열 T의 길이 N이 문자열 P의 길이 M보다 길다고 가정했을때,  연산 횟수는 약 (N-M + 1) * M 이다. 따라서 O(N * M)이 된다.

## Rabin-Karp 알고리즘

### 아이디어

패턴 내 문자들을 일일이 비교하는 대신에 패턴의 해시값과 본문의 부분 문자열의 해시값을 비교한다.

#### 해싱

본문의 부분 문자열을 해싱 할 때는, 슬라이딩 윈도우 기법으로 맨 앞의 문자의 해시 값을 빼고, 슬라이딩 윈도우를 한 칸 밀어서 다음 문자의 해시 값을 추가 한 후, 부분 문자열에 해시 값을 한번 더 곱한다.

#### 시뮬레이션

타겟 T = “315265”에서 패턴 P = “26” 을 찾는 과정을 시뮬레이션하였다.

해시 키로 33을 사용하였고, 26을 해싱하면 1704가 나온다.

이후 슬라이딩 윈도우를 통해서 타켓 문자열을 해싱하며 비교하였다.

<div style="text-align: center;">
    <img src = "/image/posts/algorithm/string-pattern-matching/rabin-karp.jpeg">
</div>

### 시간 복잡도

문자열 T의 길이가 N이고, 문자열 P의 길이가 M일때, 문자열 P를 해싱하는데 필요한 연산 횟수는 M이다. 문자열 T를 슬라이딩 윈도우 방식으로 문자열 P의 해시 값과 일치하는지 확인한다. 총 N번의 문자를 슬라이딩 윈도우에 추가, 삭제, 비교가 발생하기 때문에 연산 횟수는 N번이다.

따라서 시간 복잡도는 O(M + N)이다.

### Code

```java
long mod = (int)(1e9) + 7;
String T = "315265";
String P = "26";

long key = 33;

long t_hash = 0;
long p_hash = 0;
long before = 1;

for (int i = 0; i < P.length(); i++) {
    t_hash = (t_hash * key) % mod;
    t_hash = (t_hash + T.charAt(i)) % mod;

    p_hash = (p_hash * key) % mod;
    p_hash = (p_hash + P.charAt(i)) % mod;

    before *= key;
    before %= mod;
}

for (int i = P.length(); i < T.length(); i++) {
    System.out.println(t_hash + (t_hash == p_hash ? " == " : " != ") + p_hash);

    char c = T.charAt(i - P.length());
    t_hash = t_hash * key % mod;
    t_hash = t_hash  - (before * c) % mod + mod;
    t_hash = (t_hash + T.charAt(i)) % mod;
}

System.out.println(t_hash + (t_hash == p_hash ? " == " : " != ") + p_hash);
```

## KMP 알고리즘

kmp 알고리즘은 문자열 매칭 문제를 failure 함수를 활용하여 전처리한 패턴 정보를 활용하여 해결하는 알고리즘이다.

문자열 T에서 패턴 P를 찾는 문제를 O(N+M)시간안에 해결한다.

### 문제 변환

KMP는 문자열 매칭 알고리즘을 패턴 P에 존재하는 모든 문자의 가장 긴 접두사를 찾는 문제로 변환하여 최적화 한다.

### Failure Function

KMP 알고리즘은 실패 함수를 사용한다.

실패 함수가 존재하는 이유는 패턴 P와 T의 부분 문자열이 불일치하면 P의 접두사와 T의 부분 문자열의 접미사가 일치하지 않는 부분은 건너 뛰어도 된다.

따라서 불일치가 발생했을 때 ,탐색을 건너뛰기 위한 값을 미리 구해야 하여 패턴 매칭이 실패했을때 어떻게 해야 하는지 알려주어 실패 함수라고 부른다. 

#### 정의

실패 함수 F(k)의 의미는 k번째 인덱스까지 문자열이 갖는 접두사와  k번째 인덱스까지 문자열이 갖는 접미사가 일치하는 최대 길이이다.

최대 길이를 기록하는 이유는 최대 길이보다 작은 길이를 이동하면 비교 횟수가 늘어나기 때문에 그리디한 선택을 한다.

#### 아이디어

실패 함수를 계산하는 방법은 간단하다. 실패 함수는 패턴 P를 자기 자신과 비교하여 계산한다.

따라서 실패 함수를 계산하는데는 P와 P’의 문자열 매칭 알고리즘 즉, KMP를 사용하여 계산할 수 있다.

초기에 검사시 suffix[1] ≠ prefix[0]이기 때문에 ff[1]은 0으로 초기화 된다.

| suffix | a | b | a | b | a | b |  |
| --- | --- | --- | --- | --- | --- | --- | --- |
| prefix |  | a | b | a | b | a | b |
| ff | 0 | 0 |  |  |  |  |  |

다음 검사시 suffix[2] == prefix[0]이기 때문에 ff[2]는 1로 초기화 된다.

| suffix | a | b | a | b | a | b |  |  |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| prefix |  |  | a | b | a | b | a | b |
| ff | 0 | 0 | 1 |  |  |  |  |  |

패턴이 일치했으므로 2번째 prefix를 검사한다. 2번째 prefix도 일치하기 때문에 ff[3]은 2로 초기화 된다.

| suffix | a | b | a | b | a | b |  |  |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| prefix |  |  | a | b | a | b | a | b |
| ff | 0 | 0 | 1 | 2 |  |  |  |  |

ff를 모두 채운 테이블은 다음과 같다.

| suffix | a | b | a | b | a | b |  |  |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| prefix |  |  | a | b | a | b | a | b |
| ff | 0 | 0 | 1 | 2 | 3 | 4 |  |  |

### Code

```java
static void foo(String s) {
    fail = new int[s.length()]; // 실패 함수를 초기화한다.
    int patternIdx = 0; // 검사할 패턴의 인덱스를 가리킴
		
    //문자열 s에 대한 실패 함수를 계산하기 위해서 1번부터 반복
    for (int i = 1; i < s.length(); i++) {

        // 중간에 패턴이 불일치하면 패턴 위치를 이전 패턴의 실패 함수 값으로 이동 
        while (patternIdx > 0 && s.charAt(i) != s.charAt(patternIdx)) {
            patternIdx = fail[patternIdx-1];
        }

        // 패턴이 일치했다면 다음 패턴으로 넘어가고 실패 함수 값을 현재 패턴의 길이로 초기화
        if(s.charAt(patternIdx) == s.charAt(i)) {
            patternIdx++;
            fail[i] = patternIdx;
        } 
        // 첫번째 패턴이 i번째 문자와 다르면 실패 함수 값은 0으로 초기화
        // 즉, i번째 패턴 매칭이 실패하면 처음부터 계산해야 함
    }
}
```

### 문자열 매칭

실패 함수를 사용하여 문자열 매칭을 진행하면 된다.

#### 원리

kmp 알고리즘의 원리는 다음과 같다.

- T[i]와 P[j]가 일치할 경우
    - 완전히 일치한다면 j를 f[j]로 초기화한다.
    - 그렇지 않다면 j를 1 증가시킨다.
- 불일치 할 경우
    - j를 f[j-1]로 초기화 하고 다시 비교한다.
    - j == 0이라면 T[i+1]로 넘어간다.

완전히 일치하는 경우에는 현재값의 실패 함수를 사용하여 다시 검사할 위치로 넘어간다.

중간에 불일치 하면 실패 함수의 값을 이용하여 다시 검사해야 할 패턴의 위치로 초기화한다.

### Code

다음은 문자열 T에 패턴 P가 몇개 존재하는지 찾아내는 코드이다.

```java
int patternIdx = 0;
int cnt = 0;
for (int i = 0; i < T.length(); i++) {
    // 중간에 패턴이 틀렸다면 실패 함수의 값을 이용하여 이전 패턴으로 돌아간다.
    while (patternIdx > 0 && P.charAt(patternIdx) != T.charAt(i)) {
        patternIdx = fail[patternIdx-1];
    }
    // 만약 패턴이 일치했다면
    if(T.charAt(i) == P.charAt(patternIdx)) {
        // 패턴이 모두 일치했다면
        if(patternIdx == P.length() -1) {
            cnt++;  // 갯수를 센다.
            
            //마지막 패턴의 실패 함수 값을 이용하여 이전 패턴으로 돌아간다
            patternIdx = fail[patternIdx]; 
        } else { // 패턴 중간이라면  
            patternIdx++;  // 다음 패턴으로 넘어간다.
        }
    }
}
```