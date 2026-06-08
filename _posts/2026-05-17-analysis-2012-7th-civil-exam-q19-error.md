---
title: 2012 국가직 7급 데이터베이스론 19번 문제 출제 오류 (REVOKE GRANT OPTION)
categories: [DB, 7th-civil-exam]
tags: [analysis, error]
---

# 2012 국가직 7급 데이터베이스론 19번 문제 출제 오류 (REVOKE GRANT OPTION)
  
다음과 같은 일련의 권한 부여 SQL 명령어를 실행하였을 때, 이에 대한 설명으로 옳지 않은 것은?  
  
U1 : GRANT SELECT, INSERT ON STUDENT TO U2 WITH GRANT OPTION  
U2 : GRANT SELECT, INSERT ON STUDENT TO U3  
U1 : REVOKE GRANT OPTION FOR SELECT ON STUDENT FROM U2  
U1 : REVOKE INSERT ON STUDENT FROM U2 CASCADE  
  
① U2는 STUDENT에 대한 삽입 권한을 다른 사용자에게 부여할 수 없다.  
② U2는 STUDENT에 대한 검색 권한을 다른 사용자에게 부여할 수 있다.  
③ U2, U3은 STUDENT에 대한 삽입 권한이 없다.  
④ U3은 STUDENT에 대한 검색 권한이 있다.   

  ---
여기서 1~3번 해설은 생략한다.  
4번을 보면 U3이 검색(SELECT) 권한이 있다고 하였고, 저 선지가 답은 아니므로(답은 2번) 기출 답안 기준으로 **옳은** 선지이다.  
하지만 **4번 선지는 옳지 않은 선지**이다.  
  
전세계에서 사용하는 DB 교과서인 Database System (Silverschatz 저)(이하 원서)에 따라 이 문제의 오류를 짚고자 한다.  

<img width="737" height="72" alt="db20120719 (1)" src="https://github.com/user-attachments/assets/17920dfd-01b0-4ed1-9de3-d6fe2a8adc7a" />
  <br>
── *Database System (Silverschatz 저) 4.7.6절*  

원서에 따르면 **연쇄 취소(CASCADE)는 생략할 수 있다**고 한다. (기본값)  
따라서 문제의 "REVOKE GRANT OPTION FOR SELECT ON STUDENT FROM U2" 문장에서 CASCADE나 RESTRICT가 없으므로 (생략되었으므로) **CASCADE라고 생각**을 하고 푸는 것이 맞다.  
  
이때, REVOKE GRANT OPTION 이 무엇을 의미하는지에 대해서 논할 필요가 있다.  
<img width="737" height="105" alt="db20120719 (2)" src="https://github.com/user-attachments/assets/e753a506-d0d7-44cc-a4dc-6d9e6ac48f64" />
  <br>
 ── *Database System (Silverschatz 저) 4.7.6절*  
원서에 따르면, REVOKE GRANT OPTION은 only GRANT OPTION만 회수하는 것이다. (실제 SELECT 권한이 아니라)  
즉, U2의 SELECT 권한이 아니라 GRANT OPTION 권한만 회수하는 것이다.  
이때 U3의 SELECT 권한은 어떻게 되는지가 핵심 쟁점이다.  
  
<img width="750" height="362" alt="db20120719 (3)" src="https://github.com/user-attachments/assets/834cd949-27c7-49cd-9933-3e133682bb62" />  <br>
 ── *Database System (Silverschatz 저) 4.7.5절*  
원서에는 사용자는 권한 그래프의 최상위 노드(즉 데이터베이스 관리자)에서 해당 사용자를 나타내는 노드까지 내려오는 경로가 존재하는 경우에만 권한을 갖는다고 적혀 있다.  
문제에서 U3가 받은 SELECT 권한은 U2가 당시 보유했던 WITH GRANT OPTION이라는 '특권(Privilege)'에 전적으로 의존하여 생성된 것이다.  
그 후 CASCADE로 권한이 회수되었다.  
즉, U2는 현재 GRANT OPTION이 없다. 따라서 U2에서 U3으로 연결되는 화살표 또한 회수된 것이다.  
U2의 SELECT가 존재한다고 해서, 그것이 U3에 대한 SELECT 권한의 근거가 되지 않는다.  
U3의 SELECT 권한의 근거는 U2의 **GRANT OPTION**이었다.  
따라서 **U3**의 **SELECT 권한은 없다.**   
  
<img width="892" height="669" alt="db20120719 (4)" src="https://github.com/user-attachments/assets/2ec0018b-47c7-4423-af9a-e966419f8624" />
  <br>
  
 ── *Database System (Silverschatz 저) PPT*  
저자가 제공하는 PPT에도 명시되어 있다.  
> **취소되는 권한에 의존하는 모든 권한도 함께 취소된다.**  
> **(All privileges that depend on the privilege being revoked are also revoked.)**  

이를 증명하기 위해, PostgreSQL을 깔고 실행해보았다.  
<img width="852" height="697" alt="img" src="https://github.com/user-attachments/assets/89e5f7b9-b921-4a84-9b9d-4dea7b7b4638" />
  <br>
U3의 역할로 SET 한 후, SELECT 문을 실행했을 때, **접근 권한이 없다**고 나온다. (INSERT 또한 당연히 없다.)  
CASCADE를 명시한 이유는 PostgreSQL에서 기본값이 RESTRICT이기 때문이다.  
  
만약 출제위원이 기본값이 RESTRICT라고 가정하고 문제를 냈다고 한다면 그것 또한 출제 오류이다.  
왜냐하면, RESTRICT가 기본값이었다면 REVOKE는 수행되지 않았을 것이고, 그렇다면 2번 선지는 **옳은 선지**가 된다.  
  
따라서 이 문제는 원서에 따라 2번과 4번이 모두 옳지 않으며,  
만약 PostgreSQL의 기준에 따라 기본값을 RESTRICT이라고 가정하고 문제를 냈다고 하더라도 2번과 4번이 모두 옳은 선지가 되어 (정답이 없으므로) 출제 오류이다.  

