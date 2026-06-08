---
title: 2021 국가직 7급 데이터베이스론 25번 문제 출제 오류 (k-way merge)
categories: [DB, 7th-civil-exam]
tags: [analysis, error]
math: true
---
  
릴레이션 R에 K 단계 합병(k-way merge) 정렬을 수행하고자 한다. 정렬이 완료될 때까지 최소 몇 번의 디스크 접근(읽기＋쓰기 횟수)이 필요한가? (단, 문제에서 언급되지 않은 다른 기법은 고려하지 않는다)

○ 각 디스크 블록에는 레코드 200개씩 저장되어 있음 </br>
○ 주 기억장치의 최대 사용가능한 크기: 레코드 1,000개 크기</br>
○ 전체 정렬해야 할 레코드의 개수: 3,200개</br>

① 32회 ② 64회 ③ 96회 ④ 128회

  ---
이 문제의 최종 정답은 3번이다.

하지만 실제 세계적으로 데이터베이스의 교과서로 불리는 Database System (Silberschatz 저) 및 Fundamentals of Database Systems (Elmasri & Navathe 저)에 따르면 이 문제의 정답은 **2번**이다. 이유는 다음과 같다.

---
먼저, 외부 정렬-합병 알고리즘의 정렬 과정은 "**초기 런 생성 → 합병**"이다.

* $b_r$ : 전체 데이터 크기, 블록 단위
  
디스크 블록 하나에 200개의 레코드가 들어가므로, 전체 레코드를 저장하는 데 필요한 블록 수(b)는 **16 블록**이다. (3200 ÷ 200)

* $M$ : 주 기억장치(메모리)에서 사용 가능한 블록 수
  
레코드 1,000개 분량이므로, 블록당 200개씩 나누면 가용 메모리 블록 수(M)는 **5 블록**이다. (1000 ÷ 200)

* $b_b$ : 버퍼 블록 크기 (별도의 언급이 없으므로 1로 가정)

---
</br>

## 1. 초기 런 (Run) 생성
<img width="1192" height="118" alt="image" src="https://github.com/user-attachments/assets/b3f76c11-a438-4253-b175-07cb68d9be2d" />

── *사진 1 (Database System (Silberschatz 저) 15.4.2절)*

**생성된 초기 런의 개수**는 $\lceil b_r / M \rceil = \lceil 16 / 5 \rceil = \mathbf{4}$**개**이다.

(16 ÷ 5 = 3.2, 각 런은 5, 5, 5, 1 블록 크기이다.)

이때, **비용**은 총 **32회**($2b_r$)이다. (16블록 읽기 + 16블록 쓰기)</br></br></br></br>
## 2. 합병 (Merge) 
### 1) 초기 패스 (initial pass)
<img width="922" height="256" alt="image" src="https://github.com/user-attachments/assets/0634819b-b590-4326-b4ff-ac2af7991a43" />

── *사진 2 (Database System (Silberschatz 저) 15.4.1절)* </br>
> **"It merges the first M - 1 runs to get a single run for the next pass." </br>**
> **(다음 패스에서 하나의 런으로 사용하기 위해 처음에 나타나는 M - 1개의 런을 합병한다.)**


이때 **런의 개수는 $M - 1$의 비율로 줄어들게** 된다.

참고로, 여기서 1을 빼는 이유는 동시에 읽어올 수 있는 런(Run)들 외에, 합친 **결과**를 담아서 내보낼 **출력 버퍼**가 하나 필요하기 때문이다. (사진 3 참조)

<img width="858" height="92" alt="image" src="https://github.com/user-attachments/assets/5b5f4f41-2693-499c-b798-5c9bd3bd5885" />

── *사진 3 (Database System (Silberschatz 저) 15.4.1절)*

문제에서는 $M = 5$이므로, 동시에 합병할 수 있는 차수(k-way)는 $M - 1 = \mathbf{4}$이다.
여기서 4-way merge라는 것은, 위에서 확인한 것과 같이 4개의 런을 합병하며, 4의 비율로 줄어든다는 것을 의미한다. (사진 2 참조)

즉, **4개를 한 번에 합병할 수 있다**는 의미이다.

초기 런이 4개이며, 4개를 한 번에 합병하므로 런의 개수는 **1개**가 된다.

이때, **비용**은 총 **32회**($2b_r$)이다. (16블록 읽기 + 16블록 쓰기)
</br></br></br>

### 2) 추가 패스 확인 (anothor pass)
> **If this reduced number of runs is still greater than or equal to M, another pass is made, with the runs created by the first pass as input.** </br>
> **"만약 줄어든 런의 개수가 M보다 크거나 같은 경우 첫 번째 패스의 결과를 입력으로 하는 또 다른 패스를 생성한다."** (사진 2 참조)


문제에서 정렬이 완료될 때까지 **최소** 몇 번의 디스크 접근이 필요하냐고 물었으므로, 우리는 최솟값을 찾으면 된다.

초기 패스(Initial Pass) 단계에서 4개의 런을 한 번에 다 합병했기 때문에, **남은 런의 개수는 1개**이다.

줄어든 런의 개수가 $M(5)$보다 작으므로, 더 이상 **추가적인 패스 없이 종료**된다.



각 패스마다 $\lfloor M/b_b \rfloor - 1$의 비율로 런의 개수가 줄어들기 때문에 $\lceil \log_{\lfloor M/b_b \rfloor - 1}(b_r/M) \rceil$만큼의 합병 패스가 필요하다. (사진 4 참조)

여기서 $b_b$란, 디스크에서 주 기억장치(RAM)로 데이터를 한 번에 읽어오거나 쓸 때 사용하는 블록의 단위 크기로, 보통 별도의 조건이 없으면 기본 단위인 $b_b = 1$로 간주하여 위와 같이 합병 차수를 $M-1$로 단순화한다.


<img width="916" height="96" alt="image" src="https://github.com/user-attachments/assets/4306537b-215a-422c-bac1-29c9e582e2da" />


── *사진 4 (Database System (Silberschatz 저) 15.4.2절)*

</br></br></br></br>

## 3. 최종 디스크 접근 횟수 계산
각 단계마다 모든 블록(16개)을 한 번씩 읽고(Read) 한 번씩 쓰므로(Write),

* **초기 런 생성 단계**: 16블록 읽기 + 16블록 쓰기 = **32회**
* **합병 단계 (초기 패스)**: 16블록 읽기 + 16블록 쓰기 = **32회**
* **총합**: $32 + 32 = \mathbf{64}$**회**

따라서 **정렬이 완료될 때까지 필요한 최소 디스크 접근 횟수는 64회**이다.
</br></br></br></br>
## 4. 공식 대입
<img width="756" height="752" alt="image" src="https://github.com/user-attachments/assets/57739bd7-e4eb-48e2-9e0d-d0ccfadd9b46" />

── *사진 5 (Database System (Silberschatz 저) 15.4장 그림 15.4)*


외부 합병 정렬의 총 디스크 블록 전송 횟수 공식은 다음과 같다.</br>

$$b_{r}\left(2\left\lceil \log _{\lfloor M/b_{b}\rfloor -1}(b_{r}/M)\right\rceil +1\right)$$

위에서 본 것과 같이, 본 문제의 조건을 대입하면 로그의 밑은 $M(5) - 1 = 4$이다.


전체 블록인 $16 ÷ M(5)$는 $3.2$ 이므로 $\left\lceil \log _{4}(3.2)\right\rceil $에서 $\log_{4}(3.2)$는 $0.x$이다.
이를 올림하면 1이며, 여기에 2를 곱한 후 1을 더하면 3이다.
따라서, $b_r(16) × 3 = 48$이다.
하지만 Silberschatz는 이 공식이 "**최종 결과를 디스크에 쓰는 비용을 제외한 수치**"임을 명시하고 있다. (사진 6 참조)

따라서 최종 결과를 디스크에 쓰는 비용(16회)를 더해 64회가 된다.

<img width="938" height="97" alt="image" src="https://github.com/user-attachments/assets/908278b2-aa2b-4859-ae1f-4ff4b02b3257" />

── *사진 6 (Database System (Silberschatz 저) 15.4.2절)*




</br></br></br></br>

## 5. Elmasri & Navathe의 《Fundamentals of Database Systems》
<img width="937" height="417" alt="image" src="https://github.com/user-attachments/assets/b2bc90fc-5bd9-4665-bbb3-bf8b2c4218d0" />

── *Fundamentals of Database Systems (Elmasri & Navathe 저) 18.2장*


> **Hence, $d_M$ is the smaller of ($n_B - 1$) and $n_R$,** </br>
> **and the number of merge passes is $\lceil \log_{d_M}(n_R) \rceil$**


Silberschatz의 Database System 뿐만 아니라, 또 다른 데이터베이스 표준 교과서인 Elmasri & Navathe의 Fundamentals of Database Systems에서도 동일한 메커니즘을 명시하고 있다.

해당 저서에서는 가용 버퍼 블록이 $n_{B}$개일 때, 합병 차수 $d_{M}$은 출력 버퍼 1개를 제외한 $n_B - 1$로 정의한다. 

또한 예시에서 205개의 초기 런을 4-Way 합병($n_B = 5$)할 때 단수 블록 처리를 모두 올림($\lceil \dots \rceil$)하여 최종적으로 4패스(Four passes)가 소요되었다. 앞서 Silberschatz의 Database System에서 본 가용 메모리 블록($n_{B}$)에서 출력 버퍼 1개를 제외한 값($n_B - 1$)이 실제 합병 차수($d_{M}$)가 된다는 핵심 원칙을 똑같이 설명하고 있다. 이는 데이터베이스 이론에서 외부 합병 정렬의 패스 및 I/O 계산 규칙이 학계 전반에서 통용되는 표준 정의임을 증명한다.

본 문제에 대입할 경우 생성된 초기 런의 개수($n_R$, initial sorted runs)은 4이고, $n_B - 1$은 4이므로 $d_M = \min(n_B - 1, n_R) = \min(5 - 1, 4) = \min(4, 4) = \mathbf{4}$ (4-Way Merge)

이때 $\lceil \log_{d_M}(n_R) \rceil = \lceil \log_{4}(4) \rceil = \lceil 1 \rceil = \mathbf{1}$ 패스가 필요하다.



</br></br></br></br>

## 6. 전체 단계를 3단계(Pass 0, 1, 2)로 계산한 경우의 오류
아마 출제위원은 전체 단계를 3단계(초기 런, 초기 패스, 추가 패스)로 가정했기 때문에 정답을 96이라고 도출한 것으로 보인다.</br>
1) 만약 초기 런의 개수가 합병 차수(4)보다 커서(예: 초기 런이 5개 이상), 합병을 두 번 거쳐야 했다면 96회가 되었을 것이다.

하지만 본 문제에서는 초기 런이 4개이고 합병 차수 또한 $4(M−1)$이므로, Silberschatz의 Database System 및 Elmasri & Navathe의 Fundamentals of Database Systems의 과정 및 공식에 따라 **단 1회의 합병 Pass만 필요**하다.

</br>

2) 만약 단순히 한 번에 2개의 런만 합치는 것(2-Way Merge)으로 가정했다면, 이 경우에도 합병 단계가 2회($log_24=2$) 필요하게 되므로 총 디스크 접근 횟수가 96회(32 × 3)였을 것이다.

하지만 위의 초기 패스 단계에서 언급한 바와 같이 동시에 합병할 수 있는 차수(k-way)는 $M - 1 = \mathbf{4}$이다.

메모리에 4개의 런으로부터 각각 1블록씩 총 4블록을 동시에 올려서 한꺼번에 처리할 수 있음에도 불구하고, 자원을 남겨두고 2개씩만 처리하는 방식을 택한 것은 불필요한 자원 낭비이며, 디스크 접근을 최소로 하라는 문제의 전제에 어긋난다.

</br></br>
본 문제는 **4-Way Merge**이며, 최소 디스크 접근 횟수는 위에서 증명한 바와 같이 **64회**이다.

따라서 본 문제는 출제 오류이다.



</br></br></br></br></br>

