💡 [블로그 주소](https://velog.io/@99mon/Real-MySQL-CH-01-03)

💡 본 내용은 [Real MySQL 8.0 1권](https://wikibook.co.kr/realmysql801/)을 읽고 정리한 내용입니다. 자세한 내용은 책을 참고해 주세요🙂

# CH 01. 소개
## 다른 DBMS와 비교할 때, MySQL의 경쟁력은?
MySQL과 오라클을 비교한다면, MySQL의 경쟁력은 **가격이나 비용**일 것이다.

최근 10여년간, 전자제품의 발전과 서버 컴퓨터 시장의 변화로 새롭고 엄청난 양의 데이터가 만들어지기 시작했다. 데이터는 점점 더 많아지는데 방대한 양의 데이터를 저장하기엔 오라클 RDBMS는 너무 비싸다. 

따라서 비교적으로 저렴한 MySQL 서버를 사용하는 곳이 증가할 가능성이 높다.

## DBMS 선택하는 기준은? 
**자기가 가장 잘 활용할 수 있는 DBMS가 가장 좋은 DBMS**이다. 어떤 DBMS를 사용할지 고민된다면 다음 순서대로 고려하면 된다.

- **안정성**
- 성능과 기능
- 커뮤니티나 인지도

성능이나 기능은 돈이나 노력으로 해결되지만, 안정성은 그렇지 않다. 따라서 가장 먼저 안정성을 고려하고 다음으로 성능과 기능, 그 다음으로 커뮤니티나 인지도를 고려해야 한다. 

커뮤니티나 인지도가 적은 DBMS는 필요한 경험이나 지식을 구하기 어렵고, DBMS를 관리할 전문가도 구하기 어렵기 때문에 DBMS를 선택할 때 함께 고려해야 한다.

<br/>

아래 사진은 DB-Engines.com에서 제공하는 DBMS 서버 랭킹으로, MySQL이 상위권에 위치하는 것을 확인할 수 있다. 뿐만 아니라, MySQL은 오픈소스라는 무기를 가지고 있기 때문에 MySQL 서버는 충분히 좋은 선택지가 될 수 있다.
![](https://velog.velcdn.com/images/99mon/post/a156887d-1424-4736-9e36-8dee2866f887/image.png)

# CH 02. 설치와 설정
## MySQL 서버의 시작과 종료
MySQL 서버에서 실제 트랜잭션이 정상적으로 커밋돼도 데이터 파일에 변경된 내용이 기록되지 않고 로그 파일(Redo 로그)에만 기록될 수 있다. 심지어 MySQL 서버가 종료되고 다시 시작된 이후에도 계속 동일한 상태로 남아있을 수 있다.

사용량이 많은 MySQL 서버에서 일반적인 현상으로, 비정상적인 상황은 아니다. 그러나 **종료될 때 모든 커밋된 내용을 데이터 파일에 기록하고 종료하고 싶다면**, MySQL 서버의 옵션을 변경하고 서버를 종료하면 된다.

```sql
mysql > SET GLOBAL innodb_fast_shutdown=0;

# 원격으로 MySQL 종료 시
mysql > SET GLOBAL innodb_fast_shutdown=0;
mysql > SHUTDOWN;
```

이렇게 커밋된 모든 데이터를 데이터 파일에 적용하고 종료하는 것을 **클린 셧다운**이라고 한다. 클린 셧다운으로 종료되면 다시 MySQL 서버가 기동할 때, **별도의 트랜잭션 복구과정을 실행하지 않기 때문에 빠르게 시작**할 수 있다.

> 👉 클린 셧다운을 사용하면 커밋된 내용이 데이터 파일에 다 적용되었기 때문에 복구과정이 없어 빠르게 시작할 수 있다!

## 서버 연결 테스트
다음과 같이 여러 가지 형태의 명령행 인자를 넣어 MySQL 서버에 접속할 수 있다.
- MySQL 소켓 파일을 이용해서 접속하는 경우
  - `linux> mysql -uroot -p --host=localhost --socket=/tmp/mysql.sock`
- TCP/IP를 통해 127.0.0.1(로컬 호스트)로 접속하는 경우
  - 원격 호스트에 있는 MySQL 서버에 접속할 때 사용
  - `linux> mysql -uroot -p --host=127.0.0.1 --port=3306`
- 호스트 주소와 포트를 명시하지 않는 경우
  - 기본값: 호스트 - `localhost`, 소켓 파일을 사용해서 접속
  - `linux> mysql -uroot -p`

<br/>

MySQL 서버에 접속한 후, `SHOW DATABASES` 명령으로 데이터베이스의 목록을 확인할 수 있다.
![](https://velog.velcdn.com/images/99mon/post/4330ef8a-8511-48ed-8706-86fdc2e4269c/image.png)

# CH 03. 사용자 및 권한
## 사용자 식별
MySQL의 사용자는 사용자의 계정뿐만 아니라 **사용자의 접속 지점(클라이언트가 실행된 호스트명이나 도메인 또는 IP 주소)도 계정의 일부**가 된다. 따라서 계정을 언급할 때는 항상 아이디와 호스트를 포함해야 한다.
Ex) `svc_Id`라는 아이디로 접속하는 경우, `'svc_id' @ '127.0.0.1'`
(단, 외부 컴퓨터에서 `svc_id`라는 이름으로 접속 불가능

모든 외부 컴퓨터에서 접속 가능한 사용자 계정을 생성하려면 호스트 부분을 `%`로 대체하면 된다. (`%`: 모든 IP 또는 모든 호스트)

또한, **동일한 아이디가 있을 때** MySQL은 계정 정보 가운데 **범위가 좁은 것을 먼저 선택**한다. 
- `'svc_id' @ '192.168.0.10'` vs `'svc_id' @ '%'`
  - `%`가 포함되지 않은 `'svc_id' @ '192.168.0.10'` 계정 정보를 이용해서 인증

> 👉 MySQL의 사용자에는 사용자의 계정과 사용자의 접속 지점(IP 주소)이 포함된다! 또한 사용자 아이디가 동일하다면 범위가 좁은 것을 먼저 선택한다!

## 사용자 계정 관리
### 시스템 계정과 일반 계정
`SYSTEM_USER` 권한을 가지고 있느냐에 따라 시스템 계정과 일반 계정으로 구분된다.
- **시스템 계정**
  - `SYSTEM_USER` 권한 O
  - **데이터베이스 관리자를 위한 계정**
  - 일반 계정을 관리할 수 있음(생성/삭제 및 변경 가능)
- **일반 계정**
  - `SYSTEM_USER` 권한 X
  - **응용 프로그램이나 개발자를 위한 계정**
  - 시스템 계정을 관리할 수 없음

DBA 계정과 일반 사용자 계정을 분리하기 위해 시스템 계정과 일반 계정을 도입했다. 따라서 다음과 같이 데이터베이스 서버 관리와 관련된 중요한 작업은 시스템 계정으로만 수행해야 한다. 
- 계정 관리
- 다른 세션 또는 그 세션에서 실행 중인 쿼리 강제 종료
- 스토어드 프로그램 생성 시 DEFINER를 타 사용자로 설정

> **MySQL 서버에는 내장된 계정들이 있다!**
내장된 계정은 각기 다른 목적으로 사용되니 삭제되지 않도록 주의하자!
아래의 계정들은 처음부터 잠겨있는 상태이므로, 보안을 걱정하지 않아도 된다.
![](https://velog.velcdn.com/images/99mon/post/3124b8ff-f149-4958-9f1e-160fc52b5657/image.png)

> 👉 `SYSTEM_USER` 권한이 있느냐 없느냐에 따라 시스템 계정과 일반 계정으로 나뉜다. 시스템 계정은 데이터베이스 관리자가, 일반 계정은 개발자가 사용한다! 

### 계정 생성
**`CREATE USER` 명령으로 계정을 생성**하고, **`GRANT` 명령으로 권한을 부여**할 수 있다. 계정을 생성할 때, 다음과 같이 다양한 옵션 설정이 가능하다.
- 계정의 인증 방식과 비밀번호
- 비밀번호 관련 옵션
  - 비밀번호 유효 기간, 비밀번호 이력 개수, 비밀번호 재사용 불가 기간
- 기본 역할(Role)
- SSL 옵션
- 계정 잠금 여부

일반적으로 많이 사용되는 옵션을 가진 `CREATE USER` 명령은 다음과 같다.
```sql
CREATE USER 'user' @ '%'
	IDENTIFIED WITH 'mysql_native_password' BY 'password'
    REQUIRE NONE
    PASSWORD EXPIRE INTERVAL 30 DAY
    ACCOUNT UNLOCK
    PASSWORD HISTORY DEFAULT
    PASSWORD REUSE INTERVAL DEFAULT
    PASSWORD REQUIRE CURRENT DEFAULT;
```

계정 생성 명령에서 각 옵션을 살펴보자.

#### IDENTIFIED WITH
**사용자의 인증 방식과 비밀번호를 설정**한다. `WITH` 뒤에는 반드시 인증 방식을 명시해야 하며, MySQL 서버의 기본 인증 방식은 `IDENTIFIED BY 'PASSWORD'` 형식이다. 
EX) `IDENTIFIED WITH 'mysql_native_password' BY 'password'`

#### REQUIRE
MySQL 서버에 접속할 때 **암호화된 SSL/TLS 채널을 사용할지 여부**를 설정한다. 별도로 설정하지 않으면 비암호화 채널로 연결하게 된다.
EX) `REQUIRE NONE`

#### PASSWORD EXPIRE
**비밀번호의 유효 기간을 설정**하는 옵션이다. 개발자나 데이터베이스 관리자의 비밀번호에는 유효기간을 설정하는 것이 보안상 안전하지만, 응용 프로그램 접속용 계정에 유효 기간을 설정하는 것은 위험할 수 있으니 주의해야 한다.

설정 가능한 옵션은 다음과 같다.
- `PASSWORD EXPIRE`
  - 계정 생성과 동시에 비밀번호 만료 처리
- `PASSWORD EXPIRE NEVER`
  - 계정 비밀번호의 만료 기간 없음
- `PASSWORD EXPIRE DEFAULT`
  - `default_password_life` 시스템 변수에 저장된 기간으로 유효 기간 설정
    ![](https://velog.velcdn.com/images/99mon/post/67a1d2bc-1050-435a-bfdc-d332fcc61422/image.png)
- `PASSWORD EXPIRE INTERVAL n DAY`
  - 비밀번호의 유효 기간을 오늘부터 n일자로 설정

#### PASSWORD HISTORY
**한 번 사용했던 비밀번호를 재사용하지 못하게 설정**하는 옵션이다.

설정 가능한 옵션은 다음과 같다.
- `PASSWORD HISTORY DEFAULT`
  - `password_history` 시스템 변수에 저장된 개수만큼 비밀번호의 이력을 저장
  - 저장된 이력에 남아있는 비밀번호는 재사용 불가
    ![](https://velog.velcdn.com/images/99mon/post/a9ebdef3-0add-46fb-9f15-9876e513f028/image.png)
- `PASSWORD HISTORY n`
  - 최근 n개의 비밀번호 이력만 저장
  - 저장된 이력에 남아있는 비밀번호 재사용 불가

#### PASSWORD REUSE INTERVAL
한 번 사용했던 비밀번호의 **재사용 금지 기간을 설정**하는 옵션이다.

설정 가능한 옵션은 다음과 같다.
- `PASSWORD REUSE INTERVAL DEFAULT`
  - `password_reuse_interval` 시스템 변수에 저장된 기간으로 설정
    ![](https://velog.velcdn.com/images/99mon/post/fec2d097-8aaa-466e-82ac-f466b770652f/image.png)
- `PASSWORD REUSE INTERVAL n DAY`
  - n일자 이후에 비밀번호를 재사용할 수 있게 설정

#### PASSWORD REQUIRE
비밀번호가 만료되어 새로운 비밀번호로 변경할 때, **현재 비밀번호를 필요로 할지 말지를 결정**하는 옵션이다.

설정 가능한 옵션은 다음과 같다.
- `PASSWORD REQUIRE CURRENT`
  - 비밀번호를 변경할 때 현재 비밀번호를 먼저 입력하도록 설정
- `PASSWORD REQUIRE OPTIONAL`
  - 비밀번호를 변경할 때 현재 비밀번호를 입력하지 않아도 되도록 설정
- `PASSWORD REQUIRE DEFAULT`
  - `password_require_current` 시스템 변수의 값으로 설정
    ![](https://velog.velcdn.com/images/99mon/post/eb078f87-f243-4deb-bbf5-c34735bec6b1/image.png)

#### ACCOUNT LOCK / UNLOCK
계정 생성 시 또는 `ALTER USER` 명령을 사용해 **계정 정보를 변경할 때 계정을 사용하지 못하게 잠글지 여부를 결정**하는 옵션이다.

설정 가능한 옵션은 다음과 같다.
- `ACCOUNT LOCK`
  - 계정을 사용하지 못하게 잠금
- `ACCOUNT UNLOCK`
  - 잠근 계정을 다시 사용 가능한 상태로 잠금 해제

> 👉 계정을 생성할 때 다양한 옵션을 적용할 수 있다!

## 비밀번호 관리
### 고수준 비밀번호
MySQL 서버는 비밀번호를 쉽게 유추할 수 있는 단어들이 사용되지 않게 **글자의 조합을 강제하거나 금칙어를 설정하는 기능**도 있다. 비밀번호의 유효성 체크 규칙을 적용하려면 `validate_password` 컴포넌트를 이용하면 된다.

우선 `validate_password` 컴포넌트를 설치해야 한다.
```sql
## validate_password 컴포넌트 설치
mysql> INSTALL COMPONENT 'file://component_validate_password';

## 설치된 컴포넌트 확인
mysql> SELECT * FROM mysql.component;
```

`validate_password` 컴포넌트를 설치하면, 해당 컴포넌트에서 제공하는 시스템 변수를 확인할 수 있다. 
![](https://velog.velcdn.com/images/99mon/post/0d711162-f23a-431a-97c6-576ee14a3d70/image.png)

**비밀번호 정책**은 크게 3가지가 있으며, 기본값은 MEDIUM으로 설정된다.
- **LOW**: 비밀번호 길이만 검증
- **MEDIUM**: 비밀번호의 길이, 숫자와 대소문자, 그리고 특수문자의 배합을 검증
- **STRONG**: MEDIUM 레벨의 검증을 모두 수행하며, 금칙어 포함 여부까지 검증

참고로 금칙어 파일은 금칙어들을 한 줄에 하나씩 기록해서 텍스트 파일로 저장한 뒤 `validate_password.dictionary_file` 시스템 변수에 등록하면 된다.
```sql
## MySQL 서버에 금칙어 파일 등록
mysql> SET GLOBAL validate_password.dictionary_file='prohibitive_word.data';

## 금칙어는 STRONG일 경우에만 작동하므로 비밀번호 정책 변경
mysql> SET GLOBAL validate_password.policy='STRONG';
```

> 👉 비밀번호 정책은 3가지로 나눌 수 있으며 비밀번호의 길이, 조합, 금칙어까지 설정할 수 있다!

### 이중 비밀번호
데이터베이스 계정의 비밀번호는 보안을 위해 주기적으로 변경해야 된다. 그러나 서비스가 실행 중일 때 비밀번호 변경이 불가능하기 때문에, 처음 설정한 상태로 계속 사용되는 경우가 많다.

이러한 문제를 해결하기 위해 MySQL 8.0부터 **2개의 비밀번호를 사용할 수 있는 기능인 이중 비밀번호**가 추가되었다. 

이중 비밀번호를 사용하는 경우 **2개 중 하나의 비밀번호만 일치하면 로그인이 통과**되며, 프라이머리(Primary)와 세컨더리(Secondary)로 구분된다. 
- 프라이머리 비밀번호: 최근에 설정한 것
- 세컨더리 비밀번호: 이전에 설정한 것

이중 비밀 번호를 사용하려면 비밀번호 변경 구문에 `RETAIN CURRENT PASSWORD` 옵션만 추가하면 된다.
```sql
## 프라이머리 비밀번호: old_password, 세컨더리 비밀번호: 없음
mysql> ALTER USER 'root' @ 'localhost' IDENTIFY BY 'old_password';

## 프라이머리 비밀번호: new_password, 세컨더리 비밀번호: old_password
mysql> ALTER USER 'root' @ 'localhost' IDENTIFY BY 'new_password' RETAIN CURRENT PASSWORD;
```

<br/>

참고로 MySQL 서버에 접속하는 모든 응용 프로그램이 재시작 되면 보안을 위해 세컨더리 비밀번호를 삭제하는 것이 좋다.
`mysql> ALTER USER 'root' @ 'localhost' DISCARD OLD PASSWORD;`

> 👉 이중 비밀번호를 사용하면 서비스가 실행 중일 때도 비밀번호를 변경할 수 있다! 비밀번호 변경이 끝난 후에는 세컨더리 비밀번호를 삭제하는 것이 보안상 좋다!

## 권한(Privilege)
### 권한의 종류
권한은 글로벌 권한, 객체 권한, 동적 권한으로 나눌 수 있다.
- **글로벌 권한**
  - 데이터베이스나 테이블 이외의 객체에 적용되는 권한
  - GRANT 명령에서 특정 객체를 명시하지 말아야 함 
- **객체 권한**
  - 데이터베이스나 테이블을 제어하는데 필요한 권한
  - GRANT 명령에서 특정 객체를 반드시 명시해야 함
- **동적 권한**
  - MySQL 8.0 이후부터 추가
  - MySQL 서버가 시작되면서 동적으로 생성하는 권한
(+ 정적 권한: MySQL 서버의 소스코드에 고정적으로 명시되어 있는 권한)
  
**권한을 부여할 때는 `GRANT` 명령을 사용**한다. 권한의 범위에 따라 `ON` 절에 명시되는 오브젝트(DB나 테이블)의 내용이 바뀌어야 한다. 이때, 사용자를 먼저 생성하고 권한을 부여해야 오류가 발생하지 않는다.
`mysql> GRANT privilege_list ON db.table TO 'user' @ 'host';`

### 글로벌 권한
DB 단위나 오브젝트 단위로 부여할 수 있는 권한이 없으므로 `GRANT` 명령의 `ON`절에 **항상 `*.*`를 사용**해야 한다.
`mysql> GRANT SUPER ON *.* TO 'user' @ 'localhost';`

### DB 권한
특정 DB에서만 권한을 부여하거나 서버에 존재하는 모든 DB에 권한을 부여할 수 있으므로 **`*.*`절과 `employees.*`가 가능**하다. (employees: DB명)
```sql
mysql> GRANT EVENT ON *.* TO 'user' @ 'localhost';
mysql> GRANT EVENT ON employees.* TO 'user' @ 'localhost';
```

### 테이블 권한
서버에 존재하는 모든 DB와 특정 DB, 그리고 특정 DB의 특정 테이블에 대해 권한을 부여할 수 있으므로 **`*.*`절과 `employees.*`, 그리고 `employees.department`가 가능**하다.
```sql
## 서버의 모든 DB에 권한 부여
mysql> GRAND SELECT, INSERT, UPDATE, DELETE ON *.* TO 'user' @ 'localhost';

## 특정 DB의 오브젝트에 대해서만 권한 부여
mysql> GRAND SELECT, INSERT, UPDATE, DELETE ON employees.* TO 'user' @ 'localhost';

## 특정 DB의 특정 테이블에 대해서만 권한 부여
mysql> GRAND SELECT, INSERT, UPDATE, DELETE ON employees.department TO 'user' @ 'localhost';

```

<br/>

특정 칼럼에 대해서만 권한을 부여할 수도 있는데, INSERT, UPDATE, SELECT의 GRANT 명령이 달라진다. 

employees DB의 department 테이블에서 `dept_name` 칼럼만 업데이트 할 수 있도록 권한 뒤에 칼럼을 명시하여 권한을 부여하면 된다.
`mysql> GRANT SELECT, INSERT, UPDATE(dept_name) ON employees.department TO 'user' @ 'localhost';`

하지만 칼럼 단위의 권한이 하나라도 설정되어 있으면 모든 테이블의 모든 칼럼에 대해서도 권한을 체크하므로 전체적인 성능에 영향을 줄 수 있다. 따라서 칼럼 단위 권한은 잘 사용되지 않는다.

> 👉 권한은 글로벌 권한, 객체 권한, 동적 권한으로 나눌 수 있다! 글로벌 권한은 DB나 테이블 이외에, 객체 권한은 DB나 테이블에 적용되는 권한이다! 칼럼 단위의 권한 사용은 성능 저하를 일으킬 수 있으므로 되도록이면 피하자!

## 역할(Role)
MySQL에서 역할을 사용하는 예시를 살펴보자.

### 역할 정의하기
```sql
## role_emp_read, role_emp_write라는 이름의 역할 정의
mysql> CREATE ROLE
		role_emp_read,
        role_emp_write;

## role_emp_read 역할에는 읽기(SELECT) 권한
## role_emp_write 역할에는 데이터 변경(INSERT, UPDATE, DELETE) 권한 부여
mysql> GRANT SELECT ON employees.* TO role_emp_read;
mysql> GRANT INSERT, UPDATE, DELETE ON employees.* TO role_emp_write;
```  
- 실행 결과
  <br/>
  ![](https://velog.velcdn.com/images/99mon/post/4f6bfcc1-cfda-407f-b6bd-89c0879347a3/image.png)
### 계정 생성하기
역할은 그 자체로 사용될 수 없고 계정에 부여해야 하므로 계정을 생성해야 한다.
```sql
## reader와 writer라는 계정 생성하기
mysql> CREATE USER reader@'127.0.0.1' IDENTIFY BY 'PASSWORD';
mysql> CREATE USER writer@'127.0.0.1' IDENTIFY BY 'PASSWORD';
```
- 실행 결과
  <br/>
  ![](https://velog.velcdn.com/images/99mon/post/648ae126-0dc6-4bdb-b75d-1e3942b1cab6/image.png)
### 역할 부여하기
계정들에 아무 권한이 부여되지 않았으므로 역할을 부여해야 한다.
```sql
## reader와 writer에 역할 부여
mysql> GRANT role_emp_read TO reader@'127.0.0.1';
mysql> GRANT role_emp_read, role_emp_write TO writer@'127.0.0.1';
```
- 실행 결과
  <br/>
  ![](https://velog.velcdn.com/images/99mon/post/dfdedc4f-2e84-4fe5-b148-3725aef46ced/image.png)
### 데이터 조회/변경 시 오류 발생
현재 `reader`나 `writer` 계정은 부여된 역할이 없기 때문에, 데이터를 조회/변경하려고 하면 권한이 없다는 오류가 발생한다.
- 부여된 역할 없음
    <br/>
    ![](https://velog.velcdn.com/images/99mon/post/4fde2cfd-988e-45c3-93e9-d89c9c27e726/image.png)
- 권한이 없다는 오류 발생
    <br/>
    ![](https://velog.velcdn.com/images/99mon/post/ce763689-b17e-4e12-ad16-8cdfec99cdaf/image.png)
### 역할 활성화하기
`reader` 계정이 `role_emp_read` 역할을 사용할 수 있게 하려면 `SET ROLE` 명령을 통해 역할을 활성화해야 한다.
- `SET ROLE 'role_emp_read';`
- 실행 결과
    ![](https://velog.velcdn.com/images/99mon/post/0f406d06-644d-4bd0-88b2-8931292db3df/image.png)

그러나 로그아웃하면 다시 초기화되기 때문에, 자동으로 부여된 역할을 활성화시키고 싶다면 `SET GLOBAL activate_all_roles_on_login=ON;`으로 설정하면 된다.

<br/>

MySQL 서버의 역할은 사용자 계정과 거의 같은 모습을 하고 있으며, **MySQL 서버 내부적으로 역할과 계정은 동일한 객체로 취급된다.** 하나의 사용자 계정에 다른 사용자 계정이 가진 **권한을 병합해서 권한 제어가 가능**해진 것이다. 

다음을 통해 실제 권한과 사용자 계정이 구분없이 저장된 것을 확인할 수 있다.
![](https://velog.velcdn.com/images/99mon/post/2d1a48f1-86a3-4b62-b393-4723b4709ea2/image.png)

> 👉 MySQL 서버 내부에서 역할과 계정을 동일한 객체로 취급한다! 하나의 사용자 계정에 다른 사용자 계정이 가진 권한을 병합해서 권한을 제어할 수 있다!

### CREATE ROLE 명령과 CREATE USER 명령을 구분해서 지원하는 이유는?
데이터베이스 관리의 직무를 분리할 수 있게 해서 **보안을 강화하는 용도로 사용**될 수 있다. `CREATE USER` 명령에 대해서는 권한이 없고 `CREATE ROLE` 명령만 실행이 가능한 사용자는 역할을 생성할 수 있다. 생성된 역할은 계정과 동일한 객체를 생성하지만, `account_locked` 값이 Y로 설정되어있어 로그인 용도로 사용 불가능하다.
