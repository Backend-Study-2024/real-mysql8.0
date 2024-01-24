🔗 [블로그 주소](https://velog.io/@99mon/Real-MySQL-8.0-CH-05.-%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98)

> 💡 본 내용은 [Real MySQL 8.0 1권](https://wikibook.co.kr/realmysql801/)을 읽고 정리한 내용입니다. 자세한 내용은 책을 참고해 주세요🙂 <br/>
📷 본문에 포함된 사진의 출처는 [Real MySQL 5장](https://wikibook.co.kr/realmysql801/)입니다!

> **잠금 vs 트랜잭션** <br/>
**잠금은 동시성을 제어하기 위한 기능**으로, 여러 커넥션에서 동시에 동일한 자원을 요청할 경우 **한 시점에는 하나의 커넥션만 변경할 수 있게 해주는 역할**을 한다. 
**트랜잭션은 데이터의 정합성을 보장하기 위한 기능**으로, **작업의 완전성을 보장**하여 일부만 적용되는 현상(Partial update)이 발생하지 않게 만들어주는 기능이다.

# 트랜잭션
트랜잭션을 지원하는 InnoDB와 트랜잭션을 지원하지 않는 MyISAM의 처리 방식 차이에 대해 살펴보자.

## MySQL에서의 트랜잭션
트랜잭션은 **논리적인 작업 셋 자체가 100% 적용되거나 아무것도 적용되지 않아야 함을 보장해 주는 것**이다.

### 트랜잭션 관점에서 InnoDB 테이블과 MyISAM 테이블의 차이
우선 테스트용 테이블을 만들고 각각 레코드를 1건씩 저장한다.
```sql
mysql> CREATE TABLE tab_myisam (fdpk INT NOT NULL, PRIMARY KEY (fdpk) ) ENGINE=MyISAM;
mysql> INSERT INTO tab_myisam (fdpk) VALUES (3);

mysql> CREATE TABLE tab_innodb (fdpk INT NOT NULL, PRIMARY KEY (fdpk) ) ENGINE=INNODB;
mysql> INSERT INTO tab_innodb (fdpk) VALUES (3);
```

그 후, `AUTO-COMMIT` 모드에서 다음 쿼리 문장을 각각 실행한다.
```sql
# AUTO-COMMIT 활성화
mysql> SET autocommit=ON;

mysql> INSERT INTO tab_myisam (fdpk) VALUES (1), (2), (3);
mysql> INSERT INTO tab_innodb (fdpk) VALUES (1), (2), (3);
```

테스트 결과는 다음과 같다.
```sql
mysql> INSERT INTO tab_myisam (fdpk) VALUES (1), (2), (3);
ERROR 1062 (23000): Duplicate entry '3' for key 'tab_myisam.PRIMARY'

mysql> INSERT INTO tab_innodb (fdpk) VALUES (1), (2), (3);
ERROR 1062 (23000): Duplicate entry '3' for key 'tab_innodb.PRIMARY'

mysql> SELECT * FROM tab_myisam;
+------+
| fdpk |
+------+
|    1 |
|    2 |
|    3 |
+------+
3 rows in set (0.00 sec)

mysql> SELECT * FROM tab_innodb;
+------+
| fdpk |
+------+
|    3 |
+------+
1 row in set (0.00 sec)
```

- 두 INSERT 문장 모두 프라이머리 키 중복 오류로 쿼리가 실패함
- MyISAM
  - 1, 2가 저장된 이후 중복 키 오류 발생 → 이미 1, 2가 저장됨 ⇒ **부분 업데이트(Partial Update)**
    - 테이블 데이터의 정합성을 맞추는데 상당히 어려움
- InnoDB
  - 쿼리 중 일부라도 오류가 발생 → **INSERT 문장을 실행하기 전 상태로 복구**

## 주의사항
프로그램 코드에서 트랜잭션의 범위를 최소화하는 것이 좋다!

### 최적의 트랜잭션 설계는?
- **데이터베이스 커넥션을 가지고 있는 범위와 트랜잭션이 활성화되어 있는 범위를 최소화**해야 함
- **네트워크 작업이 있는 경우, 반드시 트랜잭션에서 배제**해야 함
  - DBMS 서버가 높은 부하 상태에 빠지거나 위험한 상태에 빠지는 경우가 빈번히 발생하기 때문
- 예시
  ```
  1. 처리 시작
  2. 사용자의 로그인 여부 확인
  3. 사용자의 글쓰기 내용의 오류 여부 확인
  4. 첨부로 업로드된 파일 확인 및 저장
  => 데이터베이스 커넥션 생성 (또는 커넥션 풀에서 가져오기)
  => 트랜잭션 시작
  5. 사용자의 입력 내용을 DBMS에 저장
  6. 첨부 파일 정보를 DBMS에 저장
  <= 트랜잭션 종료 (COMMIT)
  7. 저장된 내용 또는 기타 정보를 DBMS에서 조회
  8. 게시물 등록에 대한 알림 메일 발송 (= 네트워크 작업)
  => 트랜잭션 시작
  9. 알림 메일 발송 이력을 DBMS에 저장
  <= 트랜잭션 종료 (COMMIT)
  <= 데이터베이스 커넥션 종료 (또는 커넥션 풀에 반납)
  10. 처리 완료
  ```

# MySQL 엔진의 잠금
MySQL에서 사용되는 잠금은 크게 스토리지 엔진 레벨과 MySQL 엔진 레벨로 나눌 수 있다. **MySQL 엔진 레벨의 잠금은 모든 스토리지 엔진에 영향**을 미치지만, **스토리지 엔진 레벨의 잠금은 스토리지 엔진 간에 상호 영향을 미치지 않는다**.

## 글로벌 락
- `FLUSH TABLES WITH READ LOCK` 명령으로 획득 가능
- MySQL에서 제공하는 잠금 중 범위가 가장 큼
  - 한 세션에서 글로벌 락 획득 & 다른 세션에서 DDL 문장이나 DML 문장 실행 (SELECT 제외) → 글로벌 락이 해제될 때까지 대기
- **영향을 미치는 범위는 MySQL 서버 전체**
  - 작업 대상 테이블이나 데이터베이스가 다르더라도 동일하게 영향을 미침

## 테이블 락
- **개별 테이블 단위로 설정되는 잠금**
- `LOCK TABLES table_name [ READ | WRITE ]` 명령으로 특정 테이블의 락 획득 가능
  - `UNLOCK TABLES` 명령으로 반납 가능
- **묵시적 테이블 락**이 발생하는 경우
  - MyISAM, MEMROY 테이블에 데이터 변경 쿼리를 실행할 때
  - **InnoDB 테이블에 스키마를 변경하는 쿼리(DDL)를 실행할 때**
    - 데이터 변경(DML) 쿼리는 무시됨

## 네임드 락
- `GET_LOCK()` 함수를 이용해 **임의의 문자열에 대한 잠금**을 설정
  - 자주 사용되지 않음
- 유용하게 사용 가능한 경우
  - 여러 클라이언트가 상호 동기화를 처리해야 할 때
  - 많은 레코드에 대해 복잡한 요건으로 레코드를 변경할 때
- 중첩 사용 & 한 번에 모두 해제 가능

## 메타데이터 락
- **데이터베이스 객체의 이름이나 구조를 변경하는 경우**에 자동으로 획득하는 잠금
- `RENAME TABLE` 명령의 경우, 원본 이름과 변경될 이름 모두 한꺼번에 잠금을 설정함
  - 2개로 나눠서 실행하면 아주 짧은 시간이지만 테이블이 존재하지 않는 순간이 생겨 오류가 발생함
- 테이블의 구조를 변경해야 하는 경우
  - 새로운 구조의 테이블 생성 → 최근 데이터까지는 id 값을 범위 별로 나눠서 여러 개의 스레드로 빠르게 복사
  - 나머지 데이터를 복사할 때 트랜잭션과 테이블 잠금, 그리고 `RENAME TABLE` 명령으로 응용 프로그램의 중단 없이 실행할 수 있음

# InnoDB 스토리지 엔진 잠금
InnoDB 스토리지 엔진은 MySQL에서 제공하는 잠금과 별개로, **스토리지 엔진 내부에서 레코드 기반의 잠금 방식**을 탑재하고 있다. 때문에 MyISAM보다 **훨씬 뛰어난 동시성 처리**를 제공한다.

## InnoDB 스토리지 엔진의 잠금
- InnoDB 스토리지 엔진은 **레코드 기반의 잠금 기능**을 제공
- 레코드와 레코드 사이의 간격을 잠그는 갭(GAP) 락 존재
![](https://velog.velcdn.com/images/99mon/post/864580ec-c350-4856-8486-fe8a3c72e97f/image.png)

### 레코드 락
- **레코드 자체만을 잠그는 것**
- **InnoDB 스토리지 엔진은** 레코드 자체가 아닌 **인덱스의 레코드를 잠금**
  - 인덱스가 하나도 없는 테이블 → 내부적으로 자동 생성된 클러스터 인덱스를 이용해 잠금을 설정함
- 프라이머리 키 또는 유니크 인덱스에 의한 변경 작업 → 레코드 자체에만 락을 검
  - 보조 인덱스를 이용한 변경 작업 → 넥스트 키 락 또는 갭 락을 사용

### 갭 락
- **레코드 사이의 간격만을 잠그는 것**
- 레코드와 레코드 사이의 간격에 **새로운 레코드가 생성되는 것을 제어**
- 넥스트 키 락의 일부로 자주 사용됨

### 넥스트 키 락
- 레코드 락과 갭 락을 합쳐 놓은 형태의 잠금

### 자동 증가 락
- **AUTO_INCREMENT 칼럼이 사용된 테이블에서 사용하는 테이블 수준의 잠금**
  - 동시에 여러 레코드가 INSERT 되는 경우, **저장되는 레코드가 중복되지 않고 저장된 순서대로 증가하는 일련번호 값을 가지기 위해 사용**함
- 트랜잭션과 관계 없이, INSERT나 REPLACE 문장에서 AUTO_INCREMENT 값을 가져오는 순간에만 락이 걸렸다가 즉시 해제됨
- 테이블에 단 하나만 존재
  - 하나의 쿼리가 AUTO_INCREMENT 락을 걸면 나머지 쿼리는 기다려야 함
- 자등 증가 값이 한 번 증가하면 절대 줄어들지 않음
  - AUTO_INCREMENT 잠금을 최소화하기 위해
  
## 인덱스와 잠금
- InnoDB의 잠금은 인덱스를 잠그는 방식을 처리됨
  - 변경해야 할 레코드를 찾기 위해 검색한 인덱스의 레코드에 모두 락을 걸어야 함
  - Ex) `first_name`이 `Georigi`인 레코드가 253개, 그 중 1개를 변경하려고 함 → 253개 레코드에 모두 락을 걸어야 함
- 테이블에 인덱스가 없다면 테이블을 풀 스캔하면서 UPDATE 작업을 함
  - 이때, 테이블에 있는 모든 레코드를 잠그게 됨

⇒ MySQL의 InnoDB에서 인덱스 설계가 중요한 이유!

## 레코드 수준의 잠금 확인 및 해제
- 테이블 잠금은 잠금의 대상이 테이블 자체 → 문제의 원인이 쉽게 발견되고 해결될 수 있음
- 레코드 수준의 잠금은 테이블의 레코드 각각에 잠금 → 그 레코드가 자주 사용되지 않는다면 오랜 시간 잠겨진 상태로 남아있어도 잘 발견되지 않음
- MySQL 5.1부터 **레코드 잠금과 잠금 대기에 대한 조회 가능**
- 강제로 잠금을 해제하려면 `KILL` 명령을 통해 MySQL 서버의 프로세스를 강제 종료하면 됨

# MySQL의 격리 수준
트랜잭션의 격리 수준은 여러 트랜잭션이 동시에 처리될 때, 특정 트랜잭션이 다른 트랜잭션에서 변경하거나 조회하는 데이터를 볼 수 있게 허용할지 말지를 결정하는 것이다. `READ UNCOMMITED`, `READ COMMITED`, `REPEATABLE READ`, `SERIALIZABLE`의 4가지가 있으며, 뒤로 갈수록 트랜잭션 간의 데이터 격리 정도가 높아지고 동시 처리 성능도 떨어진다.
![](https://velog.velcdn.com/images/99mon/post/796288d2-26ca-4fd4-be86-8c66edba4a99/image.png)

일반적인 온라인 서비스 용도의 데이터베이스는 `READ COMMITED`와 `REPETABLE READ` 중 하나를 사용한다.

## READ UNCOMMITTED
- 각 트랜잭션에서의 변경 내용이 **COMMIT이나 ROLLBACK 여부에 상관없이 다른 트랜잭션에서 보임**
![](https://velog.velcdn.com/images/99mon/post/6b4ca748-2711-48e7-9d2a-8a5bbe8ba792/image.png)

### 부정합의 문제 - Dirty read
- 어떤 트랜잭션에서 처리한 작업이 완료되지 않았는데도 다른 트랜잭션에서 볼 수 있는 현상인 더티 리드가 발생함
- 데이터가 나타났다가 사라졌다하는 현상

## READ COMMITTED
- 오라클 DBMS에서 기본으로 사용되는 격리 수준
  - 온라인 서비스에서 가장 많이 선택되는 격리 수준
- 더티 리드 같은 현상은 발생하지 않음
  - **COMMIT이 완료된 데이터만 다른 트랜잭션에서 조회**할 수 있기 때문
![](https://velog.velcdn.com/images/99mon/post/fa982980-ba6d-4d67-8fd9-9cd6e03dd534/image.png)
- 새로운 값은 테이블에 즉시 기록되고, 이전 값은 언두 영역으로 백업됨
- 커밋 수행 전에 SELECT → 언두 영역에 백업된 레코드에서 값을 가져옴
- 변경한 내용이 커밋되기 전까지는 다른 트랜잭션에서 변경 내역을 조회할 수 없음


### 부정합의 문제 - NON-REPETABLE READ
- 변경된 데이터를 커밋하기 전/후에 조회할 때 결과가 달라짐
- 하나의 트랜잭션 내에서 똑같은 SELECT 쿼리를 실행했을 때 **항상 같은 결과를 가져와야 한다는 `REPETABLE READ` 정합성에 어긋남**
![](https://velog.velcdn.com/images/99mon/post/79d23102-889a-4113-ba14-c1acbb177e91/image.png)

## REPEATABLE READ
- MySQL의 InnoDB 스토리지 엔진에서 기본으로 사용되는 격리 수준
- MVCC를 위해 **언두 영역에 백업된 데이터를 이용해 동일 트랜잭션 내에서는 동일한 결과를 보여줄 수 있게 보장함**
  - InnoDB 스토리지 엔진은 트랜잭션에 ROLLBACK 될 가능성에 대비해, 변경되지 전 레코드를 언두 공간에 백업해두기 때문에 가능
![](https://velog.velcdn.com/images/99mon/post/9ac9bcc4-98bb-470f-941b-229aef04ae7e/image.png)
- 사용자 B의 트랜잭션은 10
  - 그 트랜잭션 안에서 실행되는 모든 SELECT 쿼리는 트랜잭션 번호가 10보다 작은 트랜잭션 번호를 가진 것만 보임

### 부정합의 문제 - PHANTOM READ
- SELECT ... FOR UPDATE 쿼리를 여러 번 수행하면 결과가 다름
  - 다른 트랜잭션에서 수행한 변경 작업에 의해 레코드가 보였다 안 보였다 하는 현상을 PHANTOM READ라고 함
![](https://velog.velcdn.com/images/99mon/post/cb11eeae-0852-4da9-be9f-324076ff193f/image.png)
- SELECT ... FOR UPDATE 쿼리는 SELECT하는 레코드에 쓰기 잠금을 걸어야 함
  - 그러나 **언두 레코드에는 잠금을 걸 수 없음** → 현재 레코드의 값을 가져오게 됨

## SERIALIZABLE
- 가장 단순한 격리 수준이면서 가장 엄격한 격리 수준
- 동시 처리 성능도 다른 트랜잭션 격리 수준보다 떨어짐
- 한 트랜잭션에서 읽고 쓰는 레코드를 다른 트랜잭션에서 절대 접근할 수 없음
  - 읽기 작업도 공유 잠금(읽기 잠금)을 획득해야 함
- InnoDB 스토리지 엔진에서는 갭 락과 넥스트 키 락 덕분에 REPEATBABLE READ 격리 수준에서도 `PHANTOM READ`가 발생하지 않음
  - 굳이 이 격리 수준을 사용할 필요는 없음
