---
title: "Windows 인증 구성요소: LSA / SAM / SRM"
categories: [IS]
tags: [is, summary]
---


## 1. 전체 인증 처리 흐름

```
① 사용자 로그인 정보 입력 (Winlogon)
→ ② LSA: 인증 패키지(MSV1_0 등)를 통해 SAM 데이터베이스를 조회·비교하여 인증 여부를 검증
→ ③ SRM: 인증 성공 시 SID 부여, SID 기반으로 리소스 접근 허용 여부 결정, 감사 메시지 생성
→ ④ LSA: 계정명-SID 매칭(조회), SRM이 생성한 감사 로그 기록
```

<br>
<br>

## 2. LSA (Local Security Authority)

- 시스템의 로컬 보안 정책을 관리하는 보호된 시스템 프로세스(LSASS)
- **사용자의 로그인 입력 정보를 SAM 데이터베이스와 비교하여 인증 여부를 검증하는 주체**
  - 정확히는 LSA(LSASS) 내부에 로드된 인증 패키지(MSV1_0 등)가 SAM DB를 조회·비교하고, 그 결과를 LSA에 반환하는 구조
- 시스템 자원(파일 등)에 대한 접근 권한 검사
- 계정명과 SID 간 매칭(변환/조회) 서비스 제공
- SRM이 생성한 감사 로그를 기록
- NT 보안의 중심 서브시스템(Security Subsystem)으로 불림

<br>
<br>


## 3. SAM (Security Account Manager)

- 사용자/그룹 계정 정보(계정명, 패스워드 해시, SID 등)를 저장하는 **데이터베이스**
- 파일 경로: `C:\Windows\System32\config\SAM`
- LSA가 인증 판단 시 조회하는 **수동적 저장소** — SAM 자체가 능동적으로 비교·판단을 수행하지는 않음

<br>
<br>


## 4. SRM (Security Reference Monitor)

- LSA로부터 인증 성공 결과를 받으면 사용자에게 **SID(Security Identifier) 부여**
- 부여된 SID를 기반으로 파일·디렉터리에 대한 **접근 허용 여부 결정**
- 접근 처리 결과에 대한 **감사 메시지 생성**

<br>
<br>


## 5. 요약 비교

| 구성요소 | 핵심 역할 |
|---|---|
| **LSA** | 로그인 정보-SAM DB 비교·인증 검증, 계정명-SID 매칭, 감사 로그 기록 |
| **SAM** | 계정 정보 데이터베이스 저장·관리 (조회 대상) |
| **SRM** | SID 부여, 접근 허용 여부 결정, 감사 메시지 생성 |

<br>
<br>


## 6. 근거

- Microsoft Learn, *Credentials Processes in Windows Authentication*: "the LSA can validate user information by checking the Security Accounts Manager (SAM) database"
- Microsoft Learn, *MSV1_0 Authentication Package*: "The MSV1_0 package checks the local security accounts manager (SAM) database to determine whether the logon data belongs to a valid security principal and then returns the result of the logon attempt to the LSA."
