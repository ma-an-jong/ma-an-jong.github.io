---
title: "[Algorithm] 접미어 배열과 lcp 배열"
excerpt: "접미어 배열과 최장 공통 접두사에 대하여 알아봅시다."
categories:
    - Algorithm
tags:
    - [Algorithm]
use_math: true

toc: true
toc_sticky: true
toc_label: "목차"
toc_icon: "sticky-note"

date: 2024-04-02
last_modified_at: 2024-04-02
---

# 접미어 배열(Suffix Array)

문자열의 접미어들을 사전순으로 나열한 배열이다.

문자열 인덱싱, 데이터 압축, 부분 문자열 탐색에 사용된다.

## 접미어 사전순 정렬

### 정렬 결과

$s = abab$일때 

접미어 사전순 정렬 결과는 $\{ ab, abab, b, bab \}$이다.

접미어 배열은 i번째 위치의 부분 문자열의 시작 위치를 기록한다. 

따라서 접미어 배열은 다음과 같이 생성된다.

|  | ab | abab | b | bab |
| --- | --- | --- | --- | --- |
| suffixArray | 2 | 0 | 3 | 1 |

### 성질

- 접미어 배열은 $0 \sim N-1$이 반드시 한개씩 등장한다.
- 모든 접미사의 길이가 서로 다르기 때문에 두 접미사가 서로 같을 수 없다.

## brute-force 접근법

### 개념

완전 탐색으로 길이가 $N$인 문자열을 순회하며 substring을 한 뒤, 정렬한다.

### 시간복잡도

길이가 $1,2,3…. N$인 문자열을 저장한 접미어 배열의 정렬은 문자열 길이 비교가 필요한 정렬이기 때문에 $O(N^{2} * log{N})$이 소모된다.

```java
List<String> str = new ArrayList<>(s.length());	
for(int i = 1; i < s.length(); i++) {
	String subString = s.substring(i);
	str.add(subString);
}
str.sort(String::compareTo);
```

## 멘버-마이어스 알고리즘

### 개념

완전 탐색 접근법보다 좀 더 효율적인 알고리즘이다.

접미사의 $d$번째 문자열이 같은 접미사끼리는 같은 그룹(연속한 위치)에 존재하도록 정렬한다. 이후 그룹이 총 $N$개로 분리될 때까지 $d$를 2배씩 증가시키며 이를 반복한다.

초기에는 문자열의 제일 앞 1번째 글자만 보고 비교하여 정렬한다.

이후, 2번째 글자까지 보고 정렬한다. 그 다음은 4번째 글자까지, 8번째 글자까지 보며 정렬한다.

이렇게 길이가 $N$인 문자열은 $O(log{N})$번 정렬이 필요하다.

### 시뮬레이션

문자열 “banana”의 결과는 다음과 같다.

<div style="text-align: center;">
    <img src = "/image/posts/algorithm/suffix-array-lcp/banana-suffix-array.png" style = "width:75%;">
</div>

<br>

<div style="text-align: center;">
    <img src = "/image/posts/algorithm/suffix-array-lcp/sa0.jpg" style = "width:75%;">
</div>

$d = 1$인 경우, 문자열의 맨 앞글자들이 같은 그룹에 속한다. 따라서 사전순으로 제일 앞서는 $\{a, ana, anana\}$는 0번 그룹에 속하고 $\{banana\}$는 1번, $\{na, nana\}$는 2번 그룹에 속한다.

<div style="text-align: center;">
    <img src = "/image/posts/algorithm/suffix-array-lcp/sa1.jpg" style = "width:75%;">
</div>

$d = 2$인 경우, 2번째 문자까지 같은 그룹으로 묶는다. 0번 그룹에 존재하던 $a$와 $ana$를 비교할 때, $a$와 $ana$는 서로 다른 그룹으로 묶여야한다. 

따라서 $a$가 0번 그룹에 남고 $\{ana,anana\}$는 이 다음번 그룹으로 이동한다. $banana$도 1번에서 2번으로 이동하고, $na$와$nana$도 다음 그룹으로 이동한다. 

<div style="text-align: center;">
    <img src = "/image/posts/algorithm/suffix-array-lcp/sa2.jpg" style = "width:75%;">
</div>

$d = 4$인 경우, 1번 그룹에 속하던 $ana$와 $anana$, 3번 그룹에 속하면 $na$와 $nana$도 이제 서로 다른 그룹으로 분리된다.

총 $N$개의 서로 다른 그룹이 생성되었으니 반복문을 종료한다.

이후 생성된 $group$의 값은 접미사 배열의 사전 순 정렬 결과를 의미한다.

### Code

#### Comparator

```java
static Comparator<Integer> comparator = new Comparator<>() {

  @Override
  public int compare(Integer o1, Integer o2) {
      // 두 접미사가 서로 다른 그룹에 속했다면
      if(groupNumber[o1] != groupNumber[o2]) {
          return groupNumber[o1] - groupNumber[o2]; // 그룹 번호가 작은것 우선
      }

      // 두 suffix의 위치에서 +d만큼 떨어진 위치로 이동

      // 만약, 두 suffix가 out of index인 경우
      if((o1 + d >= maxLength) && (o2 + d >= maxLength)) {
          return 0; //둘은 서로 같은 그룹임
      }

      // 만약 o1 + d칸 이동 결과가 out of index인 경우
      // 최소한 o1번째 suffix의 길이 < o2번째 suffix의 길이
      if(o1 + d >= maxLength) {
          return -groupNumber[o2+d]; // o1은 o2보다 앞의 그룹임
      }

      // 만약 o2 + d칸 이동 결과가 out of index인 경우
      // 최소한 o1번째 suffix의 길이 > o2번째 suffix의 길이
      if(o2 + d >= maxLength) {
          return groupNumber[o1+d];
      }

      // suffix에서 d칸 떨어진 suffix의 그룹 번호를 통해서
      // 사전 순 정렬
      return groupNumber[o1 + d] - groupNumber[o2 + d];
  }
};
```

#### Suffix Array 생성

```java
static Integer[] getSuffixArray(String s) {
    maxLength = s.length();

    groupNumber = new int[maxLength]; // 문자열 s의 i번째 문자가 몇번째 그룹인지 저장
    Integer[] suffix = new Integer[maxLength]; // 문자열 s의 접미사의 시작 위치를 저장

    for (int i = 0; i < maxLength; i++) {
        // suffix의 시작 위치를 저장
        suffix[i] = i;

        // 문자열 s의 i번째 문자의 그룹을 저장
        // 초기에는 문자의 ascii 코드 값으로 d = 1일때, 같은 문자는 그룹으로 초기화된다
        groupNumber[i] = s.charAt(i);
    }

    d = 1; // d = 1부터 시작

    while(true) {
        // suffix를 d번째 글자까지 고려했을때 사전순으로 정렬
        Arrays.sort(suffix,comparator);

        if(d >= maxLength) { // 만약, d가 문자열의 길이를 넘어갔으면 종료
            break;
        }

        // 같은 그룹에 속한 원소를 새 그룹으로 이동
        int[] moveGroup = new int[maxLength];

        for (int i = 1; i < maxLength; i++) {
            // i번째 suffix와 i+1번째 suffix를 비교
            boolean isFrontThan = comparator.compare(suffix[i-1],suffix[i]) < 0;
            
            // 같은 그룹에 넣기
            moveGroup[suffix[i]] = moveGroup[suffix[i-1]];

            // 만약, i번째 suffix가 i+1번째 suffix보다 사전순으로 앞선다면
            // i+1번째 원소를 다음 그룹을 이동
            if(isFrontThan) {
                moveGroup[suffix[i]]++;
            }
        }
        d *= 2; // 비교할 문자의 길이를 증가시키기
        groupNumber = moveGroup; // 그룹 배열 복사
    }
    return suffix;
}
```

### 시간 복잡도

접미어 배열은 $O(N)$ 공간 복잡도를 가지며 $O(N * (log{N}) ^{2})$ 시간에 생성된다.

길이가 $N$인 텍스트에 길이가 $P$인 패턴의 존재를 $O({P} + log{N})$에 계산 가능하다.

# LCP(Longest Common Prefix)

### 개념

접미어 배열의 보조적인 자료구조로 최장 공통 접두어 배열이다.

정렬된 접미어 배열에서 연속적인 2개의 접미어들 사이의 최장 공통 접두어의 길이를 저장한다.

접미어 배열의 순회나 패턴 매칭을 효율적으로 수행하는데 사용된다.

<div style="text-align: center;">
    <img src = "/image/posts/algorithm/suffix-array-lcp/banana-suffix-array.png" style = "width:75%;">
</div>

LCP[4] = 3인데 A[3] = S[4], A[4] = S[2]가 갖는 공통 접두어는 ana가 되기 때문이다. 

### 원리

어떤 두 인접한 접미사 $X$와 $Y$가 존재할때, $X$와 $Y$의 공통 접두사의 길이가 $L$이면, 앞에서 $X$,$Y$에서 한 글자를 뺀 공통 접두사의 길이는 $L’ ≥ max(L-1, 0)$이 된다.

즉, 사전순으로 $banana$와 인접한 접미사인 $na$와 비교한 공통 접두사의 길이가 $L$이면, $banana$에서 앞 한 글자를 지운 접미사 $anana$와 인접한 접미사 $banana$와 공통 접두사의 길이 $L’$는 최소 $L-1$보다는 크다는 뜻이다.

이후 $anana$에서 한 글자를 지운 접미사 $nana$와 인접한 접미사는 존재하지 않기 때문에 건너뛰고,

$ana$와 인접한 접미사인 $anana$의 공통 접두사 길이는 최소 $L’-1$보다는 크다.

### 시뮬레이션

- 접미사 배열의 역 인덱스 배열을 생성한다.
- 문자 $S$의 접미어를 순차적으로 탐색한다. $i$
- 문자 $S$의 접미어가 사전순으로 몇번째 위치에 있는지 확인한다. $A[i]$
- 사전 순으로 다음 위치에 있는 접미어의 시작 위치($A[i] + 1$)를 얻는다. $j$
- 두 인접한 접미사의 시작 위치($i, j$)에서 이전 LCP 값 $L-1$만큼 떨어진 문자부터, 공통 접두사의 길이를 구한다. $L’$

<div style="text-align: center;">
    <img src = "/image/posts/algorithm/suffix-array-lcp/lcp0.jpg" style = "width:75%;">
</div>

$i = 0$일 때, $banana$와 인접한 접미사 배열은 $na$이다. $L = 0$이기 때문에 두 접미사의 첫부분부터 비교한다. 일치하는 부분이 없기 때문에 $L$은 $0$이다.

<div style="text-align: center;">
    <img src = "/image/posts/algorithm/suffix-array-lcp/lcp2.jpg" style = "width:75%;">
</div>

$i = 2$일 때, $nana$는 사전순으로 마지막 접미사이다. 인접한 접미사 배열은 존재하지 않기 때문에 건너뛴다. 

<div style="text-align: center;">
    <img src = "/image/posts/algorithm/suffix-array-lcp/lcp3.jpg" style = "width:75%;">
</div>

$i = 3$일 때, $ana$의 인접 접미사는 $anana$이다. $L = 0$이였기 때문에 문자의 처음부터 비교한다.

총 3개의 prefix가 일치했기 때문에 $L = 3$으로 갱신된다.

<div style="text-align: center;">
    <img src = "/image/posts/algorithm/suffix-array-lcp/lcp4.jpg" style = "width:75%;">
</div>

이전 $L$값이 3이였기 때문에, 다음 접미사는 최소 길이가 $L-1$ 이상의 공통 접두사를 갖는다. 따라서 $na$와 $nana$를 비교할때, $L-1$만큼은 건너뛰고 비교할 수 있다.

$L-1$ 이후로 일치하는 공통 접두사가 없기 때문에 $L = L-1$로 갱신된다.

### Code

```java
static int[] getLCP(Integer[] suffix,String s) {
    //문자열 s의 i번째 접미사 == s.substring(i);

		// 문자열 s의 i번 접미사의 최장 공통 접두사 길이 저장
    int[] lcp = new int[maxLength - 1];
    // suffix 배열의 역인덱스 배열 -> 문자열 s의 i번째 접미사의 사전순 위치를 역추적
    int[] inversionSuffix = new int[maxLength]; 

    for (int i = 0; i < maxLength; i++) {
        inversionSuffix[suffix[i]] = i;
    }

    int L = 0;
    for (int i = 0; i < maxLength; i++) {
        int suffixIdx = inversionSuffix[i]; //i번째 접미사의 사전 순 번호
        
        if(suffixIdx == maxLength-1) continue; // 만약, 마지막 접미사면 건너뛰기

        L = Math.max(L-1, 0); // L 크기는 최소 이전 LCP 길이 - 1임
        int j = suffix[suffixIdx+1]; // 인접한 접미사의 시작 인덱스
        // i+L 문자와 j+L 문자가 일치하는지 확인하며 길이 증가
        while(i + L < maxLength && j + L < maxLength && s.charAt(i+L) == s.charAt(j+L)) {
            L++;
        }
        
        lcp[suffixIdx] = L;
    }

    return lcp;
}
```

## 부분 문자열 탐색

### 최장 반복 부분 문자열

만들어진 lcp 배열에서 최댓값이 정답이다. 반복 부분 문자열은, 모든 접미사 쌍의 공통 접두사가 반복 부분 문자열이 되는데, 이때 가장 긴 부분 문자열만 구하면 되기 때문에 lcp 배열의 최댓값이 정답이다. 

### 서로 다른 부분 문자열의 개수

길이가 $N$인 문자열에서 부분 문자열의 개수는 길이가 1인 부분 문자열, 길이가 2인 부분 문자열 … 길이가 $N$인 부분 문자열의 수의 합과 같다.

따라서 $ \displaystyle \sum_{k=1}^N k = {(N + 1) * N \over 2}$ 이다.

여기서 부분 문자열의 중복을 고려해야한다.

lcp 배열의 값은 공통 문자열의 길이이고, lcp가 $abc$였다면, $\{a, ab, abc\}$ 가 중복되었다는 의미이다.

따라서 모든 lcp배열의 합을 구한 뒤 부분 문자열의 총 개수의 합과 차이가 서로 다른 부분 문자열의 개수이다.