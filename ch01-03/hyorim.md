
# 1장🎉

---

## 1.1 소개

- MySQL은 오픈소스
- 무료
- 오라클이 인수함

<br>

## 1.2 왜 MySQL인가?

- 다른 DBMS와 비교할 때 MySQL은 <u>**비용과 가격**</u>에서 경쟁력이 있음 
- DBMS를 고르는 기준
  1. 안정성
  2. 성능과 기능
  3. 커뮤니티나 인지도 
- DBMS 활용도가 높다 → 성능, 안정성, 기능이 모두 포함된 게 아니겠나

<br>

# 2장🎉

---

### 2.1 MySQL 서버 설치

- MySQL 서버 설치 및 디렉터리 준비
- MySQL 서버의 시작과 종료 
- 설치 과정 설명 전 MySQL 서버의 버전 및 에디션 (엔터프라이즈 vs 커뮤니티)

<br>

### 2.1.1 버전과 에디션(엔터프라이즈와 커뮤니티) 선택

- 서버 버전 선택 - 가능한 한 최신버전을 설치하는 게 좋음 
- 최소 패치 버전이 15~20번 이상 릴리스된 버전을 선택하는 것이 안정적인 서비스에 도움 됨
  - MySQL 8.0 버전이라면 MySQL 8.0.15 부터 8.0.20 사이의 버전부터 시작 권장 
  - 갓 출시된 메이저버전은 위험
  - - 메이저버전은 많은 변화를 거친 버전, 갓 출시된 상태에서는 치명적이거나 보완하는데 많은 시간 걸리는 버그발생 가능

<br>

- 초기 버전의 MySQL 서버는 엔터프라이즈 에디션과 커뮤니티 에디션이 MySQL 서버의 기능에 차이 ㄴㄴ 기술 지원의 차이만 있었음 
- MySQL 5.5 버전부터는 커뮤니티와 앤터프라이즈 에디션 기능이 달라지면서 소스코드도 달라짐
  - 앤터프라이즈 에디션의 소스코드는 공개되지 x 
- MySQL 서버의 상용화전략: 핵심 내용은 동일, 특정 부가 기능들만 상용버전인 엔터프라이즈 에디션에 포함: <u>**오픈코어 모델!**</u>
  - 핵심 기능은 거의 차이 x
  - 부가적인 기능과 서비스들은 엔터프라이즈 에디션에서만 지원됨
      - thread pool
      - enterprise audit
      - MySQL 기술지원 등등
- 성능 자체는 다르지 않아. 지원하는 거 잘 보고 앤터프라이즈 시스템을 선택하자

<br>

### 2.2 MySQL 설치

1. MySQL 서버 (MySQL 클라이언트 프로그램이 포함돼 있음)
2. MySQL Shell
3. MySQL Router

3개 설치하기

- MySQL 서버의 고가용성 옵션 선택가능

- 복제없이 단일 서버 실행 모드인 Standalone MySQL Server/ Classic MySQL Replication 선택 
- 서버를 어떤 방식으로 접속하게 할 지 설정함


- Config Type: Development Computer
    - MySQL 서버가 허용하는 커넥션의 개수를 적게 설정하게 됨
    - 서비스용 MySQL 서버를 설치한다면 그에 맞게 옵션을 변경 ㄱㄱ
- Connectivity 옵션은 일반적으로 많이 사용되는ㄴ TCP/IP 로 선택
- Port는 서버의 기본 포트인 3306으로, X Protocol Port도 기본 포트인 33060 그대로 유지 ㄱㄱ

사용자 로그인 시점에 사용할 비밀번호 인증 방식 선택 ㄱㄱ

Use Strong Password Encrytion for Atuh 은 Caching SHA-2 Authentication 플러그인을 사용하는 것임

Lecacy Authentication MEthod는 Native Authentication 플러그인을 사용하는 것임

- 테스트용이니까 이거 사용 ㄱㄱ

관리자 계정 비번 설정 ㄱㄱ

MySQL 서버 설정 모두 완료되면 MySQL Router 옵션을 설정하는 화면이 나타남 → 이 화면은 취소하고 다음으로 넘어가자

서버 셸 라우터 깔았다.

ProgramData\MySQL\MySQL Server 8.0\Data\my.ini 는 MySQL 서버의 설정파일!

설정파일의 경로를 반드시 기억하자

MySQL 서버 디렉터리의 구조는 다음과 같다. 이 디렉터리는 삭제되면 MySQL 서버 정상실행이 안될 수 있음

- bin: MySQL 서버와 클라이언트 프로그램, 그리고 유틸리티를 위한 디렉터리
- include: C/C++ 헤더 파일들 저장
- lib: 라이브러리 파일들이 저장
- share: 다양한 지원 파일들 저장, 에러 메시지나 샘플 설정파일(my.ini)이 잇는 디렉터리

<br>

### 2.2 MySQL 서버의 시작과 종료

리눅스 OS에서  MySQL 서버의 설정파일 / MySQL 서버를 시작, 종료하는 방법을 알아봄

<br>

### 2.3.3 MySQL 8.0 업그레이드

<br>

### 2.4 서버 설정

MySQL 서버는 단 하나의 설정파일을 사용함

- 유닉스 계열: my.cnf
- 윈도우 계열: my.ini

- MySQL 서버는 시작될 때만 이 설정 파일을 참조함
- 경로가 고정은 ㄴㄴ 지정된 여러 개의 디렉터리를 순차적으로 탐색하면서 처음 발견된 my.cnf 파일을 사용함
    - 직접 MySQL 을 컴파일해서 설치한 경우에는 디렉터리 다르게 설정될 수도 있음
    - 경로 확인 가능함
- 즉 실제 MySQL 서버는 단 하나의 설장 파일만 사용/ 설정파일이 위치한 디렉터리는 여러 곳일 수 ㅇ
    - MySQL 서버가 어느 디렉터리  my.cnf 파일을 먼저 읽는지 화ㄱ인 가능
- 하나의 서버에 2개 이상의 MySQL 서버 를 실행하는 형태로 서비스하는 곳 잘 없다

<br>

### 2.4.1 설정 파일의 구성

- MySQL 설정 파일은 하나의 my.cnf나 my.ini 파일에 여러 개의 설정 그룹을 담을 수 ㅇ
- 대체로 실행 프로그램 이름을 그룹명으로 사용함
    - mysqldump 프로그램 - [mysqldump]
    - mysqld 프로그램은 [mysqld]
    - mysqld_safe 프로그램은 [m ysqld_dsafe] [mysqld] 섹션 참조함
- 설정파일이 MySQL 서버만을 위한 서정파일이라면 - [ mysqld]  그룹만 명시해도 무방함
- 하지만 MySQL 서버 뿐 아니라 MySQL 클라이언트나 백업을 위한 mysqld 프로그램이 실행될 때도 이 설정 파일을 공용으로 사용하고 싶다면 [mysql] [mysqldump] 등의 그룹을 함께 설정해 둘 수도 있음
- socket이나 port 같은 설정은 모든 프로그램에 공통으로 필요한 설정값 → 각 설정 그룹에 여러 번 설정됨

ex)

MySQL 서버 프로그램은 3306 포트를 사용

MySQL 클라이언트 프로그램은 3304 번 포트를 이용해 MySQL 서버에 접속하려고 함

설정파일의 각 그룹은 같은 파일을 공유하지만 서로 무관하게 적용됨

<br>

### 2.4.6 my.cnf파일

- MySQL 8.0 서버의 시스템 변수는 대략 570개
    - 사용하는 플러그링ㄴ이나 컴포넌트 따라 시스템 변수 개수 더 늘어날 수 ㅇ

<br>

# 3장🎉

---

- 사용자 생성, 각 계정의 권한을 성정하는 방법 다른 DBMS랑 차이있음

1. 단순히 사용자의 아이디뿐 아니라 해당 사용자가 어느 IP에서 접속하고 있는지도 확인함
2. MySQL 8.0 버전부터는 권한을 묶어서 관리하는 역할의 개념이 도입 → 각 사용자의 권한으로 미리 준비된 권한 세트인 Role 부여 가능

데이터베이스 서버의 보안은 정말 중요해! 반드시 계정의 식별 방식과 권한, 역할에 대한 내용 숙지 ㄱㄱ

<br>

## 3.1 사용자 식별

- MySQL의 사용자는 다른 DBMS와는 조금 다르게 사용자의 계정뿐 아니라 사용자의 접속 지점(클라이언트가 실행된 host명 or 도메인 or IP주소)도 계정의 일부가 됨 
- 아이디와 호스트 함께 명시해야함 → 사용자의 계정!!


💡 사용자 계정 생성 시 주의!
- MySQL은 권한이나 계정 정보에 대해 둘 중 범위가 가장 작은 것을 항상 먼저 선택함

```
`svc_id`@`192.168.0.10` -> 비밀번호 123
`svc_id`@`%` -> 비밀번호 345
```

IP가 명시된 계정정보를 이용해 사용자를 인증함

<br>

### 3.2 사용자 계정 관리

<br>

### 3.2.1 시스템 계정과 일반계정

- MySQL 8.0부터 계정은 SYSTEM_USER 권한을 가지고 있느냐에 따라 시스템 계정 & 일반 계정으로 구분됨
    - MySQL서버 내부적으로 실행되는 백그라운드 스레드와 무관
    - 사용자를 위한 계정임 ㅇㅇ 시스테 ㅁ계정도
- 시스템 계정은 일반계정을 생성삭제변경 가능하지만 일반 계정은 시스템계정 관리 불가능
- 디비 서버 관리와 관련된 중요한 작업은 시스템 계정으로만 수행가능
    - 계정관리(생성, 삭제, 권한부여 제거)
    - 다른 세션 or 그 세션에서 실행중인 쿼리 강제종료
    - 스토어드 프로그램 생성 시 DEFINER를 타 사용자로 설정
- 이 시스템계정, 일반계정 도입 이유: DBA계정에는 SYSTEM_USER권한을 할당하고 일반 사용자를 위한 계정에는 해당 권한 부여 ㄴㄴ

MySQL 서버에는 다음 내장된 계정들이 있는데, `root`@`localhost`제외 3개 계정은 내부적으로 각기 다른 목적으로 사용됨 - 삭제되지 않게 주의

- ‘mysql.sys’@’localhost’: MySQL 8.0부터 기본으로 내장된 sys 스키마의 객체(뷰, 함수, 프로시저)들의 DEFINER로 사용되는 계정
- ‘mysql.session’@’localhost’: MySQL 플러그인이 서버로 접근할 때 사용되는 계정
- ‘mysql.infoschema’@’localhost’: information_schema에 정의된 뷰의 DEFINER로 사용되는 계정
- 셋 다 처음부터 잠겨있는 (account_locked) 계정이라 보안걱정 ㄴㄴ
    - SELECT user, host, account_locked FROM mysql.user WHERE user LIKE ‘mysql.%’

<br>

### 3.2.2 계정 생성

- MySQL 8.0 버전부터는 계정의 생성은 CREATE USER 명령으로, 권한부여는 GRANT 명령으로 구분해서 실행하도록 바뀜
- 계정 생성 시 다음과 같은 다양한 옵션 설정가능
    - 계정의 인증 방식과 비밀번호
    - 비밀번호 관련 옵션(비번 유효 기간, 비번 이력 개수, 비밀번호 재사용 불가 기간)
    - 기본 역할(Role)
    - SSL 옵션
    - 계정 잠금 여부

```sql
CREATE USER 'user'@'%'
	IDENTIFIED WITH 'mysql_natve_password' BY 'password'
	REQUIRE NONE
	PASSWORD EXPIRE INTERVAL 30 KAY
  ACCOUNT UNLOCK
  PASSWORD HISTORY DEFAULT
  PASSWORD REUSE INTERVAL DEFAULT
  PASSWIRD REQUIRE CURRENT DEFAULT;
```

<br>

### 3.2.2.1 IDENTIFIED WITH

```sql
CREATE USER 'user'@'%'
	IDENTIFIED WITH 'mysql_natve_password' BY 'password'
```

- 사용자 인증 방식과 비밀번호를 설정한다
- IDENTIFIED WITH 뒤에는 반드시 인증 방식을 명시해야함 //인증 플러그인의 이름
    - MySQL 서버의 기본 인증방식 사용하고자 한다면 IDENTIFIED BY ‘password’ 형식으로 명시
- MySQL 서버에서는 다양한 인증방식을 플러그인 형태로 제공함. 대표적 4가지 방식
    - Native Pluggable Authentication
        - MySQL 5.7 버전까지 기본으로 사용되던 방식
        - 단순히 비밀번호에 대한 해시(SHA-1 알고리즘)값을 저장해두고, 클라이언트가 보낸 값과 해시값일 일치하는지 비교하는 인증 방식
    - PAM Pluggable Authentication
        - 유닉스나 리눅스 패스워드 또는 LDAP 같은 외부 인증을 사용할 수 있게 해주는 인증방식
        - MySQL 엔터프라이즈 에디션에서만 사용가능
    - LDAP Pluggable Authentication
        - LDAP을 이용한 외부인증을 사용할 수 있게 해주는 인증방식
        - MySQL 엔터프라이즈 에디션에서만 사용가능

- MySQL 5.7 버전까지는 Native Authentication이 기본 인증방식으로 사용됐지만, MySQL 8.0 부터는 Cahing SHA-2 Pluggable Authentication 이 기본 인증으로 바뀌었음
- 하지만 Caching 저거는 SSL/TLS 또는 RSA 키페어를 필요로 하기 때문에 기존 MySQL 5.7 까지의 연결 방식과는 다른 방식으로 접속해야함
- 보안 수준은 좀 낮아지겠지만 기존 버전과의 호환성을 고려한다면 Native Authentication 인증방식으로 계정을 생성해야할 수도 있음
    - 이러려면 MySQL 설정을 변경하거나 my.cnf 설정 파일에 추가하면 됨

CREATE USER 또는 ALTER USER 명령을 이용해 MySQL 서버 계정을 생성 or 변경할 때 연결 방식과 비밀번호옵션, 자원 사용과 관련된 여러 옵션 설정가능

<br>

### 3.2.2.2 REQUIRE

- MySQL 서버에 접속할 때 암호화된 SSL/TLS 채널을 사용할지 여부를 설정함
- 별도로 설정 x : 비암호화 채널로 연결하게 됨
- 하지만 REQUIRE 옵션을 SSL로 설정하지 않았다고 하더라도 Caching SHA-2 Authentication 인증 방식을 사용하면 암호화된 채널만으로 MySQL 서버에 접속가능

<br>

### 3.2.2.3 PASSWORD EXPIRE

- 비밀번호의 유효 기간을 설정하는 옵션이며, 별도로 명시하지 않으면 default_password_lifetime 시스템 변수에 저장된 기간으로 유효 기간이 설정됨
- 개발자나 디비 관리자의 비밀번호는 유효기간을 설정하는 것이 보안상 안전하지만 응용 프로그램 접속용 계정에 유효기간을 설정하는 거 위험
- 설정가능 옵션
    - PASSWORD EXPIRE: 계정 생성과 동시에 비밀번호 만료처리
    - PASSWORD EXPIRE  NEVER: 만료기간 x
    - PASSWORD EXPIRE DEFAULT: default_password_lifetime 시스템 변수에 저장된 기간으로 비밀번호의 유효 기간을 설정
    - PASSWORD EXPIRE INTERVAL n DAY: 비밀번호 유효기간을 오늘부터 n일자로 설정

<br>

### 3.2.2.4 PASSWORD HISTORY

- 한 번 사용했던 비밀번호를 재사용하지 못하게 설정하는 옵션
- PASSWORD HISTORY 설정 가능한 옵션
    - PASSWORD HISTORY DEFAULT: password_history **시스템 변수에 저장된 개수만큼** 비밀번호의 이력을 저장하며, 저장된 이력에 남은 비번 재사용 ㄴㄴ
    - PASSWORD HISTORY n: 비밀번호의 이력을 **최근 n개**까지만 저장, 저장된 이력에 남아있는 비번 재사용 불가능
- 이전 사용 비번은 mysql DB의 password_history 테이블을 사용함

<br>

### 3.2.2.5 PASSWORD REUSE INTERVAL

- 한 번 사용했던 비밀번호의 재사용 금지 기간 설정옵션
- 별도로 명시하지 않으면 password_reuse_interval 시스템 변수에 저장된 기간으로 설정됨
- PASSWORD REUSE INTERVAL 절에 설정 가능한 옵션
    - PASSWORD REUSE INTERVAL DEFAULT: 시스템 변수에 저장된 기간
    - PASSWORD REUSE INTERVAL n DAY: n일자 이후에 비밀번호를 재사용가능

<br>

### 3.2.2.7 ACCOUNT LOCK/ UNLOCK

- 계정 생성 시 or ALTER USER 명령을 사용해 계정 정보를 변경할 때 계정을 사용하지 못하게 잠글지 ㅇㅇ 여부
- ACCOUNT LOCK/ ACCOUNT UNLOCK

<br>

### 3.3 비밀번호 관리

<br>

### 3.3.1 고수준 비밀번호

- 비밀번호를 쉽게 유추할 수 있는 단어들이 사용되지 않도록 글자의 조합을 강제하거나 금칙어를 설정하는 기능 있음
- MySQL 서버에서 비밀번호의 유효성 체크 규칙을 적용하려면 validate_password 컴포넌트를 이용하면 됨
    - validate_password 컴포넌트 설치해야한다
    - MySQL서버 프로그램에 내장되어 있어서 파일경로 적을 필요 x
- validate_password 컴포넌트가 설치되면 컴포넌트에서 제공하는 시스템변수 확인가능
- 비밀번호 정책은 선택가능 - 기본값은 MEDIUM으로 자동 설정됨
    - LOW: 비밀번호 길이만 검증
    - MEDIUM: 길이검증 숫자 대소문자 특수문자 배합 검증
    - STRONG: MEDIUM + 금칙어 포함 여부 검증
    - validate_password.policy 시스템 변수를 바꾸면 된다 ㅇㅇ
- 시스템 변수 설정
    - validate_password.length 시스템 변수에 설정된 길이 이상의 비밀번호 길이 검증
    - 숫자 대소문자 특수문자 mixed_case_count, number_count, special_char_count 시스템 변수에 설정된 글자 수 이상을 포함하고 있는지 검증
    - 금칙어는 dictionary_file 시스템변수

높은 수준의 보안 요구하는 서비스: 비밀번호 금칙어 ㅇㅇ 설정

한 줄에 하나씩 기록, 텍스트파일로 작성~ 리스트 깃헙에 있으니 참고하세요

- MySQL 8.0 버전부터는 validate_password가 플러그인 형태가 아닌 컴포넌트 형태로 제공됨

<br>

### 3.3.2 이중 비밀번호

- 많은 응용프로그램 서버들이 공용으로 디비를 많이 사용함 → 디비 서버의 계정정보는 응용 프로그램 서버로부터 공용으로 사용되는 경우가 많음 ㅇㅇ 구현특성
    - 디비 서버의 계정 정보가 쉽게 변경하기 어ㅓ려움
    - 디비 계정의 비밀번호는 서비스가 실행 중인 상태에서 변경이 불가능했음. 서비스를 모두 멈추지 않고서는 비번 변경 불가능했다!
- MySQL 8.0 버전부터는 계정의 비밀번호로 2개의 값을 동시에 사용할 수 있는 기능이 추가됐음
    - Dual password 기능 ㅇㅇ
    - 하나만 일치해도 로그인이 통과되는 것
- Primary 비밀번호, Secondary 비밀번호
    - Primary: 최근에 설정된 비밀번호
    - Secondary: 이전 비밀번호

<br>

### 3.4 권한

- MySQL 5.7 버전까지는 글로벌 권한과 객체 단위의 권한으로 구분됐음
    - 디비나 테이블 이외의 객체에 적용되는 권한 → 글로벌 권한/ GRANT 명령에서 특정 객체 명시 x
    - 디비나 테이블을 제어하는 데 필요한 권한 → 객체 권한/  GRANT 명령으로 권한 부여 시 반드시 특정객체를 명시해야함
- ALL은글로벌과 객체 권한 두 가지 용도로 사용가능
    - 특정 객체에 ALL 권한이 부여되면 해당 객체에 적용될 수 있는 모든 객체 권한을 부여
    - 글로벌로 ALL 이 사용되면 글로벌 수준에서 가능한 모든 권한 부여하게 됨
-
- MySQL 8.0 버전부터는 MySQL 5.7 버전의 권한에 동적 권한이 더 추가됐음
    - 이전에 있던 권한들은 정적권한이라고 불림: MySQL 서버의 소스코드에 고정적으로 명시돼 있는 권한
    - 동적권한은 MySQL 서버가 시작되면서 동적으로 생성하는 권한
    - MySQL 서버의 컴포넌트나 플러그인이 설치되면 그때 등록되는 권한 = 동적권한
- 이전 SUPER 권한이 잘게 쪼개져서 동적 권한으로 분산된 것임
    - MySQL 서버 8.0 버전부터는 백업 관리자와 복제 관리자 개별로 꼭 필요한 권한만 부여할 수 있게 된 것

- 사용자에게 권한 부여: GRANT 명령 사용

```sql
GRANT privilege_list ON db.table TO 'user'@'host';
```

- MySQL 8.0 버전부터는 존재하지 않는 사용자에 대해 GRANT 명령이 실행되면 에러발생함 ㅇㅇ 주의
- 어떤 DB 어떤 오브젝트에 / 권한을 부여할 대상 사용자 명시

<br>

### 글로벌 권한

```sql
GRANT SUPER ON *.* TO 'user'@'localhost';
```

- 글로벌 권한은 특정 DB나 테이블에 부여될 수 없기 때문에 글로벌 권한을 부여할 때 GRANT 명령의 ON 절에는 항상 *.* 를 사용함
    - *.* 는 디비의 모든 오브젝트(테이블과 스토어드 프로시저나 함수 등)를 포함한 MySQL 서버 전체를 의미
    - CREATE USER나 CREATE ROLE 과 같은 글로벌 권한은 DB단위나 오브젝트 단위로 부여할 수 있는 권한이 아니라서 ㅇㅇ

<br>

### DB권한

```sql
GRANT EVENT ON *.* TO 'user'@'localhost';
GRANT EVENT ON employees.* TO 'user'@'localhost';
```

- DB권한은 특정 DB대해서만 && 서버 존재 모든 DB 권한부여 가능함
- *.*   employees.* 다 사용가능
- DB는 디비 내부에 존재하는 테이블 뿐만 아니라 스토어드 프로그램들도 모두 포함함
- 하지만 DB 권한만 부여하는 경우에는 테이블까지 명시 불가능
    - 서버의 모든 DB에 적용할 수 있으므로  대상에 *.*을 사용가능
    - 특정 DB에 대해서만 권한 부여 가능, db.* 가능

<br>

### 테이블 권한

```sql
GRANT SELECT,INSERT,UPDATE,DELETE ON *.* TO 'user'@'localhost';
GRANT SELECT,INSERT,UPDATE,DELETE ON employees.* TO 'user'@'localhost';
GRANT SELECT,INSERT,UPDATE,DELETE ON employees.department TO 'user'@'localhost';
```

- 테이블의 특정 컬럼에 대해서만 권한 부여고 가능함
- 컬럼에 부여가능 권한은 DELETE 제외 SELECT, INSERT, UPDATE, DELETE임
    - 각 권한 뒤에 컬럼 ㅁ영시하는 형태로 부여
- 업뎃 불가능하게 가능함

테이블이나 컬럼단위 권한 잘 사용안함

컬럼 단위 권한이 하나라도 설정되면 나머지 모든 테이블의 ㅗㅁ든 컬럼에 대해서도 권한 체크 → 전체적인 성능에 영향 미칠 수 ㅇ

컬럼 단위 접근권한이 꼭 필요하면 GRANT명령보단 별도의 뷰를 만들어 사용하는 방법 고려 ㄱㄱ

- 뷰도 하나의 테이블로 인식됨
- 뷰의 컬럼에 대해 권한체크 ㄴㄴ 뷰 자체에 대한 권한만 체크한다

<br>

### 3.5 역할

- MySQL 8.0 버전부터 권한을 묶어서 역할을 사용할 수 있음
- 실제 MySQL 서버 내부적으로 역할은 계정과 똑같은 모습을 하고있음

<br>

```sql
CREATE ROLE
	role_emp_read,
	role_emp_write;
```



- 이건 빈 껍데기만 있는 역할을 정의한 것
- 이후 GRANT 명령으로 각 역할에 대해 실질적인 권한을 부여 ㄱㄱ

<br>

```sql
GRANT SELECT ON emplyees.* TO role_emp_read;
GRANT INSERT, UPDATE, DELETE ON employees.* TO role_emp_write;
```

- 역할은 그 자체로 사용될 수 없고 계정에 부여해야함

<br>

```sql
CREATE USER reader@'127.0.0.1' IDENTIFIED BY 'qwerty';
CREATE USER writer@'127.0.0.1' IDENTIFIED BY 'qwerty';
//아무런 권한 부여 x

GRANT role_emp_read TO reader@'127.0.0.1';
GRANT role_emp_read, role_emp_write TO writer@'127.0.0.1';

SHOW GRANTS; //계정이 가진 권한을 확인

SELECT * FROM emplyees.employees LINIT 10;//권한 없다는 에러가 나옴

SELECT current_role();//계정에 활성화된 역할 조회 시 역할 없음

SET ROLE 'role_emp_read'; //해당 역할을 활성화해야함. 그래야 계정이 역할 사용가능
```

- SET ROLE 실행해서 해당 역할을 활성화해야한다
- 역할이 활성화되면 그 역할이 가진 권한은 사용할 수 있는 상태가 된다
- 계정 로그아웃 후 로그인 → 역할이 활성화되지 않은 상태로 초기화돼 버림 
  - MySQL 서버의 역할이 자동으로 활성화되지 않게 설정돼 있기 때문!

<br>

사용자가 MySQL 서버에 로그인할 때 역할이 자동으로 활성화할지 여부를 activate_all_roles_on_login 시스템 변수에 설정가능

- 매번 SET ROLE 명령으로 역할을 활성화하지 않아도 로그인과 동시에 부여된 역할이 자동으로 활성화됨

<br>

- MySQL 서버의 역할을 사용자 계정과 거의 같은 모습을 하고 있음
- MySQL 서버 내부적으로 역할과 계정은 동일한 객체로 취급됨
    - 단지 하나의 사용자 계정에 다른 사용자계정이 가진 권한을 병합해서 권한 제어가 가능해졌을 뿐
- mysql DB의 user 테이블을 살펴보면 실제 권한과 서용자 계정이 구분 없이 저장되어 있음 
  - 역할과 계정의 차이는 account_locked뿐

<br>

계정과 권한 구분 → 하나의 계정에 다른 계정의 권한을 병합하기만 하면 돼서 구분 필요 x

일반적으로 user 생성시 `CREAETE USER reader@’127.0.0.1’` 과 같이 계정 이름과 호스트 부분을 함께 명시한다

역할 생성시에는 호스트 부분을 별도로 명시하지 않았음

- 호스트 부분 명시하지 않으면 자동으로 모든 호스트(%)가 추가됨
- host컬럼값은 %가 된다

<br>

역할과 계정을 구분해서 지원하는 이유

- 데이터베이스 관리 직무를 분리할 수 있게 해서 보안을 가오하하는 용도로 사용할 수 있게 하기 위해서임
- CREATE USER 권한은 없지만 CREATE ROLE 명령만 실행 가능한 사용자는 역할을 생성가능
- 계정과 동일한 객체를 생성하지만 실제 이 역할은 account_locked 컬럼값이 Y라서 로그인 용도로사용 불가능

<br>

- 계정의 기본 역할 또는 역할에 부여된 역할 그래프 관계는 SHOW GRANT를 사용
- 표 형태로 깔끔하게 보고자하면 mysql DB의 권한 관련 테이블 참조하면 됨