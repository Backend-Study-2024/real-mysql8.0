🔗 [블로그 주소_1](https://velog.io/@99mon/Real-MySQL-CH-04.-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98) 
, [블로그 주소_2](https://velog.io/@99mon/Real-MySQL-CH-04.-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%982) 
, [블로그 주소_3](https://velog.io/@99mon/Real-MySQL-CH-04.-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%983) 

MySQL 서버는 **사람의 머리 역할을 담당하는 MySQL 엔진**과 **손과 발의 역할을 담당하는 스토리지 엔진**으로 구분할 수 있다. 

앞으로 MySQL 엔진과 MySQL 서버에서 기본으로 제공되는 InnoDB 스토리지 엔진, 그리고 MyISAM 스토리지 엔진에 대해 살펴보자.

# MySQL 엔진 아키텍쳐
## MySQL의 전체 구조
![](https://velog.velcdn.com/images/99mon/post/df01e208-e1fd-48e8-b1f3-0d3c0ebaaf6a/image.png)
👉 [사진 출처 - Real MySQL 4장](https://wikibook.co.kr/realmysql801/)

- MySQL은 대부분의 프로그래밍 언어로부터 접근 방법을 모두 지원함
  - MySQL 서버에서 다양한 프로그래밍 언어로 서버에서 쿼리 사용 가능
  - C/C++, PHP, 자바, 펄, ...
- MySQL은 표준 SQL(ANSI SQL) 문법 지원함
  - 표준 문법에 따라 작성된 쿼리는 타 DBMS와 호환되어 실행 가능
 
### MySQL 엔진
- **요청된 SQL 문장을 분석하거나 최적화하는 등 DBMS의 두뇌에 해당하는 처리를 수행**
- MySQL 엔진의 중심
  - 클라이언트로부터의 접속 및 쿼리 요청을 처리하는 **커넥션 핸들러**
  - **SQL 파서 및 전처리기**
  - 쿼리의 최적화된 실행을 위한 **옵티마이저**

### 스토리지 엔진
- **실제 데이터를 디스크 스토리지에 저장하거나 디스크 스토리지로부터 데이터를 읽어오는 부분을 담당**
  - 테이블의 모든 읽기 작업이나 변경 작업은 스토리지 엔진이 처리함
- MySQL 서버에서 MySQL 엔진은 하나지만, 스토리지 엔진은 여러 개를 동시에 사용할 수 있음
- 각 스토리지 엔진은 성능 향상을 위해 키 캐시(MyISAM 스토리지 엔진)나 InnoDB 버퍼 풀(InnoDB 스토리지 엔진)과 같은 기능을 내장하고 있음

### 핸들러 API
- 핸들러 요청
  - MySQL 엔진의 쿼리 실행기에서 데이터를 쓰거나 읽어야 할 때 **각 스토리지 엔진에 쓰기 또는 읽기를 요청하는 것**
- 핸들러 API
  - 핸들러 요청에서 사용되는 API
- 즉, **핸들러 API를 이용해 MySQL 엔진과 데이터를 주고 받음**
  - Ex) InnoDB 스토리지 엔진


> **핸들러 API를 통해 얼마나 많은 데이터(레코드) 작업이 있었는지 확인하는 방법**
: `SHOW GLOBAL STATUS LIKE 'Handler%';` 명령
```sql
mysql> SHOW GLOBAL STATUS LIKE 'Handler%';
+----------------------------+-------+
| Variable_name              | Value |
+----------------------------+-------+
| Handler_commit             | 674   |
| Handler_delete             | 8     |
| Handler_discover           | 0     |
| Handler_external_lock      | 6597  |
| Handler_mrr_init           | 0     |
| Handler_prepare            | 0     |
| Handler_read_first         | 41    |
| Handler_read_key           | 1797  |
| Handler_read_last          | 0     |
| Handler_read_next          | 4185  |
| Handler_read_prev          | 0     |
| Handler_read_rnd           | 0     |
| Handler_read_rnd_next      | 166   |
| Handler_rollback           | 0     |
| Handler_savepoint          | 0     |
| Handler_savepoint_rollback | 0     |
| Handler_update             | 332   |
| Handler_write              | 8     |
+----------------------------+-------+
```

## MySQL 스레딩 구조
![](https://velog.velcdn.com/images/99mon/post/24740726-fc8a-4185-aa94-8fa48e2287e3/image.png)
👉 [사진 출처 - Real MySQL 4장](https://wikibook.co.kr/realmysql801/)

- MySQL 서버는 스레드 기반으로 동작함
  - 포그라운드 스레드
  - 백그라운드 스레드
- `performance_schema` 데이터베이스의 `threads` 테이블을 통해 실행 중인 스레드 목록 확인 가능
  ```sql
  mysql> SELECT thread_id, name, type, processlist_user,     processlist_host FROM performance_schema.threads ORDER BY type, thread_id;
  +-----------+---------------------------------------------+------------+------------------+------------------+
  | thread_id | name                                        | type       | processlist_user | processlist_host |
  +-----------+---------------------------------------------+------------+------------------+------------------+
  |         1 | thread/sql/main                             | BACKGROUND | NULL             | NULL             |
  |         2 | thread/mysys/thread_timer_notifier          | BACKGROUND | NULL             | NULL             |
  |         4 | thread/innodb/io_ibuf_thread                | BACKGROUND | NULL             | NULL             |
  |         5 | thread/innodb/io_log_thread                 | BACKGROUND | NULL             | NULL             |
  |         6 | thread/innodb/io_read_thread                | BACKGROUND | NULL             | NULL             |
  |         7 | thread/innodb/io_read_thread                | BACKGROUND | NULL             | NULL             |
  |         8 | thread/innodb/io_read_thread                | BACKGROUND | NULL             | NULL             |
  |         9 | thread/innodb/io_read_thread                | BACKGROUND | NULL             | NULL             |
  |        10 | thread/innodb/io_write_thread               | BACKGROUND | NULL             | NULL             |
  |        11 | thread/innodb/io_write_thread               | BACKGROUND | NULL             | NULL             |
  |        12 | thread/innodb/io_write_thread               | BACKGROUND | NULL             | NULL             |
  |        13 | thread/innodb/io_write_thread               | BACKGROUND | NULL             | NULL             |
  |        14 | thread/innodb/page_flush_coordinator_thread | BACKGROUND | NULL             | NULL             |
  |        15 | thread/innodb/log_checkpointer_thread       | BACKGROUND | NULL             | NULL             |
  |        16 | thread/innodb/log_flush_notifier_thread     | BACKGROUND | NULL             | NULL             |
  |        17 | thread/innodb/log_flusher_thread            | BACKGROUND | NULL             | NULL             |
  |        18 | thread/innodb/log_write_notifier_thread     | BACKGROUND | NULL             | NULL             |
  |        19 | thread/innodb/log_writer_thread             | BACKGROUND | NULL             | NULL             |
  |        20 | thread/innodb/log_files_governor_thread     | BACKGROUND | NULL             | NULL             |
  |        25 | thread/innodb/srv_lock_timeout_thread       | BACKGROUND | NULL             | NULL             |
  |        26 | thread/innodb/srv_error_monitor_thread      | BACKGROUND | NULL             | NULL             |
  |        27 | thread/innodb/srv_monitor_thread            | BACKGROUND | NULL             | NULL             |
  |        28 | thread/innodb/buf_resize_thread             | BACKGROUND | NULL             | NULL             |
  |        29 | thread/innodb/srv_master_thread             | BACKGROUND | NULL             | NULL             |
  |        30 | thread/innodb/dict_stats_thread             | BACKGROUND | NULL             | NULL             |
  |        31 | thread/innodb/fts_optimize_thread           | BACKGROUND | NULL             | NULL             |
  |        32 | thread/mysqlx/worker                        | BACKGROUND | NULL             | NULL             |
  |        33 | thread/mysqlx/acceptor_network              | BACKGROUND | NULL             | NULL             |
  |        34 | thread/mysqlx/worker                        | BACKGROUND | NULL             | NULL             |
  |        38 | thread/innodb/buf_dump_thread               | BACKGROUND | NULL             | NULL             |
  |        39 | thread/innodb/clone_gtid_thread             | BACKGROUND | NULL             | NULL             |
  |        40 | thread/innodb/srv_purge_thread              | BACKGROUND | NULL             | NULL             |
  |        41 | thread/innodb/srv_worker_thread             | BACKGROUND | NULL             | NULL             |
  |        42 | thread/innodb/srv_worker_thread             | BACKGROUND | NULL             | NULL             |
  |        43 | thread/innodb/srv_worker_thread             | BACKGROUND | NULL             | NULL             |
  |        46 | thread/mysqlx/acceptor_network              | BACKGROUND | NULL             | NULL             |
  |        48 | thread/sql/con_sockets                      | BACKGROUND | NULL             | NULL             |
  |        44 | thread/sql/event_scheduler                  | FOREGROUND | event_scheduler  | localhost        |
  |        45 | thread/sql/compress_gtid_table              | FOREGROUND | NULL             | NULL             |
  |        49 | thread/sql/one_connection                   | FOREGROUND | root             | localhost        |
  +-----------+---------------------------------------------+------------+------------------+------------------+
  ```
  - 마지막 `thread/sql/one_connection` 스레드만 실제 사용자의 요청을 처리하는 포그라운드 스레드

### 포그라운드 스레드(클라이언트 스레드)
- 주로 **각 클라이언트 사용자가 요청하는 쿼리 문장을 처리**
- MySQL 서버에 접속된 클라이언트의 수만큼 존재
- 커넥션이 종료되면 해당 커넥션을 담당하던 스레드는 스레드 캐시로 돌아감
  - 스레드 캐시가 꽉 찼다면 스레드를 종료 시킴
  - 즉, 일정 개수의 스레드만 스레드 캐시에 존재하도록 함
    - `thread_cache_size` 시스템 변수로 스레드 캐시에 유지할 수 있는 최대 스레드 개수 설정
- **MySQL의 데이터 버퍼나 캐시로부터 데이터를 가져와서 작업을 처리함**
  - 데이터가 버퍼나 캐시에 없다면, 직접 디스크의 데이터나 인덱스 파일로부터 읽어옴
- MyISAM 테이블
  - 디스크 쓰기 작업까지 포그라운드 스레드가 처리
- InnoDB 테이블
  - 데이터 버퍼나 캐시까지만 포그라운드 스레드가 처리
  - 버퍼로부터 디스크까지 기록하는 작업은 백그라운드 스레드가 처리

#### 백그라운드 스레드
- InnoDB는 여러 가지 작업이 백그라운드로 처리됨 
  - 인서트 버퍼를 병합하는 스레드
  - **로그를 디스크로 기록하는 스레드**
  - **InnoDB 버퍼 풀의 데이터를 디스크에 기록하는 스레드**
  - 데이터를 버퍼로 읽어오는 스레드
  - 잠금이나 데드락을 모니터링하는 스레드
- 가장 중요한 것
  - 로그 스레드(Log thread)
  - 쓰기 스레드(Write thread)
- `innodb_write_io_threads`, `innodb_read_io_threads` 시스템 변수로 스레드의 개수를 설정함  
  - 읽는 작업은 주로 클라이언트 스레드에서 처리됨
    - 읽기 스레드의 개수를 많이 설정할 필요가 없음
  - **쓰기 쓰레드는 많은 작업을 백그라운드로 처리함**
    - 내장 디스크 사용 시에는 2~4개
    - DAS나 SAN과 같은 스토리지 사용 시에는 디스크를 최적으로 사용할 수 있을 만큼 충분히 설정
- 사용자의 요청을 처리할 때, 데이터 쓰기 작업은 지연(버퍼링)되어 처리될 수 있음
  - 그러나 데이터 읽기 작업은 절대 지연될 수 없음
  - 일반적인 상용 DBMS에는 대부분 쓰기 작업을 버퍼링해서 일괄처리하는 기능이 탑재되어 있음(Ex. InnoDB)
  
## 메모리 할당 및 사용 구조
메모리 공간은 **글로벌 메모리 영역과 로컬 메모리 영역**으로 구분된다. 글로벌 메모리 영역과 로컬 메모리 영역은 MySQL 서버 내에 존재하는 **많은 스레드가 공유해서 사용하는 공간인지에 따라 구분된다**. 각 영역에 대해 살펴보자.

### 글로벌 메모리 영역
- 모든 메모리 공간은 MySQL 서버가 시작되면서 운영체제로부터 할당됨
- 클라이언트 스레드 수와 무관하게 **하나의 메모리 공간만 할당**
  - 필요에 따라 2개 이상 할당받을 수 있지만 클라이언트 스레드 수와 무관함
  - 글로벌 영역이 N개라 하더라도 **모든 스레드에 의해 공유됨**
- 대표적인 글로벌 메모리 영역
  - 테이블 캐시
  - InnoDB 버퍼 풀
  - InnoDB 어댑티프 해시 인덱스
  - InnoDB 리두 로그 버퍼

### 로컬 메모리 영역
- **클라이언트 스레드가 쿼리를 처리하는 데 사용하는 메모리 영역**
  - 클라이언트와 스레드가 사용하는 공간이라고 **클라이언트 메모리 영역**이라고도 표현함
  - 클라이언트와 MySQL 서버와의 커넥션을 세션이라고 해서 **세션 메모리 영역**이라고도 표현함
- **각 클라이언트별로 독립적으로 할당**되며, **절대 공유되어 사용되지 않음**
  - 각 쿼리의 용도별로 필요할 때만 공간이 할당되고, 필요하지 않은 경우 메모리 공간 할당조차 하지 않을 수 있음
    - Ex) 정렬 버퍼, 조인 버퍼
  - 커넥션이 열려있는 동안 계속 할당된 상태로 남아있는 공간도 있고, 쿼리를 실행하는 순간에만 할당했다가 다시 해제하는 공간도 있음
- 대표적인 로컬 메모리 영역
  - 정렬 버퍼(Sort buffer)
  - 조인 버퍼
  - 바이너리 로그 캐시
  - 네트워크 버퍼
  
## 플러그인 스토리지 엔진 모델
MySQL의 독특한 구조 중 대표적인 것이 플러그인 모델이다. 만약 부가적인 기능을 제공하는 스토리지 엔진이 필요하면, 직접 스토리지 엔진을 개발하는 것도 가능하다.

또한, MySQL 서버에서는 스토리지 엔진뿐만 아니라 다양한 기능을 플러그인 형태로 지원한다. (Ex. 인증, 전문 검색 파서, 쿼리 재작성 등과 같은 플러그인, 비밀번호 검증과 커넥션 제어 등에 관련된 다양한 플러그인)

### MySQL에서 쿼리가 실행되는 과정
![](https://velog.velcdn.com/images/99mon/post/f7b94929-ddbd-411e-b2be-018719272ac4/image.png)
👉 [사진 출처 - Real MySQL 4장](https://wikibook.co.kr/realmysql801/)

- **거의 대부분의 작업이 MySQL 엔진에서 처리**되고, 마지막 **"데이터 읽기/쓰기" 작업만 스토리지 엔진에 의해 처리됨**
  - 즉, 사용자가 새로운 용도의 스토리지 엔진을 만들더라도 DBMS의 일부분의 기능만 수행하는 엔진을 작성하는 것
- "데이터 읽기/쓰기" 작업은 대부분 1건의 레코드 단위로 처리됨
  - Ex) 특정 인덱스의 레코드 1건 읽기 or 마지막으로 읽은 레코드의 다음 또는 이전 레코드 읽기

### MySQL 사용 시 자주 접하는 단어, 핸들러(Handler)란?
- 프로그래밍 언어에서 어떤 기능을 호출하기 위해 사용하는 운전대와 같은 역할을 하는 객체를 핸들러라고 표현함
- MySQL 엔진은 사람 역할, 스토리지 엔진은 자동차 역할
  - **MySQL 엔진이 스토리지 엔진을 조정하기 위해 핸들러라는 것을 사용함**
- 즉, **MySQL 엔진이 각 스토리지 엔진에게 데이터를 읽어오거나 저장하도록 명령하려면 반드시 핸들러를 통해야 함!**
- `Handler_`로 시작하는 상태 변수는 MySQL 엔진이 각 스토리지 엔진에게 보낸 명령의 횟수를 의미하는 변수

### 다른 스토리지 엔진을 사용한다면?
- 다른 스토리지 엔진을 사용하는 테이블에 쿼리를 실행하는 경우, MySQL 처리 내용은 대부분 동일하고 "데이터 읽기/쓰기" 영역 처리의 차이만 있음
  - 복잡한 처리(`GROUP BY`, `ORDER BY`)는 MySQL의 처리 영역인 "쿼리 실행기"에서 처리함

### 여기서 가장 중요한 내용은?
하나의 쿼리 작업은 여러 하위 작업으로 나뉘는데, **각 하위 작업이 MySQL 엔진 영역에서 처리되는지 아니면 스토리지 엔진 영역에서 처리되는지 구분**할 줄 알아야 한다!

> **MySQL 서버에서 지원되는 스토리지 엔진 확인하기**
```sql
mysql> SHOW ENGINES;
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| Engine             | Support | Comment                                                        | Transactions | XA   | Savepoints |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| MEMORY             | YES     | Hash based, stored in memory, useful for temporary tables      | NO           | NO   | NO         |
| MRG_MYISAM         | YES     | Collection of identical MyISAM tables                          | NO           | NO   | NO         |
| CSV                | YES     | CSV storage engine                                             | NO           | NO   | NO         |
| FEDERATED          | NO      | Federated MySQL storage engine                                 | NULL         | NULL | NULL       |
| PERFORMANCE_SCHEMA | YES     | Performance Schema                                             | NO           | NO   | NO         |
| MyISAM             | YES     | MyISAM storage engine                                          | NO           | NO   | NO         |
| InnoDB             | DEFAULT | Supports transactions, row-level locking, and foreign keys     | YES          | YES  | YES        |
| ndbinfo            | NO      | MySQL Cluster system information storage engine                | NULL         | NULL | NULL       |
| BLACKHOLE          | YES     | /dev/null storage engine (anything you write to it disappears) | NO           | NO   | NO         |
| ARCHIVE            | YES     | Archive storage engine                                         | NO           | NO   | NO         |
| ndbcluster         | NO      | Clustered, fault-tolerant tables                               | NULL         | NULL | NULL       |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
```
- Support 칼럼
  - YES
    - MySQL 서버에 해당 스토리지 엔진이 포함되어 있고, 사용 가능으로 활성화된 상태
  - DEFAULT
    - "YES"와 동일한 상태이지만 필수 스토리지 엔진임을 의미함
  - NO
    - 현재 MySQL 서버(mysqld)에 포함되지 않음
  - DISABLED
    - 현재 MySQL 서버에는 포함되어 있지만, 파라미터에 의해 비활성화된 상태
- Support 칼럼이 `No`인 것을 사용하려면 플러그인 형태로 빌드된 스토리지 엔진 라이브러리를 다운로드해서 끼워넣기만 하면 쉽게 사용할 수 있음

## 컴포넌트
MySQL 8.0부터 기존의 플러그인 아키텍처를 대처하기 위해 컴포넌트 아키텍처가 지원된다. **컴포넌트는 플러그인이 가진 단점들을 보완해서 구현**됐다. 플러그인의 단점은 다음과 같다.
- 플러그인은 오직 MySQL 서버와 인터페이스할 수 있고, 플러그인끼리는 통신 할 수 없음
- 플러그인은 MySQL 서버의 변수나 함수를 직접 호출해서 안전하지 않음(캡슐화 안 됨)
- 플러그인은 상호 의존 관계를 설정할 수 없어서 초기화가 어려움

## 쿼리 실행 구조
![](https://velog.velcdn.com/images/99mon/post/ee58593b-63ca-4247-bcb2-f0401d496d2d/image.png)
👉 [사진 출처 - Real MySQL 4장](https://wikibook.co.kr/realmysql801/)

MySQL의 구조를 기능별로 나눠 살펴보자.

### 쿼리 파서
- **사용자 요청으로 들어온 쿼리 문장을 토큰으로 분리해 트리 형태의 구조로 만들어내는 작업**
  - 토큰은 MySQL이 인식할 수 있는 최소 단위의 어휘나 기호를 뜻함
- **기본 문법 오류를 발견**하고 사용자에게 오류 메시지를 전달

### 전처리기
- 파서 과정에서 만들어진 **파서 트리를 기반으로 쿼리 문장에 구조적인 문제점이 있는지 확인**
- 해당 객체의 존재 여부와 객체의 접근 권한 등을 확인
  - 실제 존재하지 않거나 권한상 사용할 수 없는 개체의 토큰이 걸러짐

### 옵티마이저
- 사용자의 요청으로 들어온 쿼리 문장을 저렴한 비용으로 가장 빠르게 처리하는 방법을 결정하는 역할을 담당
- DBMS의 두뇌에 해당

### 실행 엔진
- 만들어진 계획대로 각 핸들러에게 요청해서 결과를 받고, 그 결과를 또 다른 핸들러 요청의 입력으로 연결하는 역할을 수행
- Ex) 옵티마이저가 `GROUP BY`를 처리하기 위해 임시 테이블을 사용하기로 결정함
  1. 실행 엔진이 핸들러에게 임시 테이블을 만들라고 요청
  2. 다시 실행 엔진은 `WHERE` 절에 일치하는 레코드를 읽어오라고 핸들러에게 요청
  3. 읽어온 레코드들을 1번에서 준비한 임시 테이블로 저장하라고 다시 핸들러에게 요청
  4. 데이터가 준비된 임시 테이블에서 필요한 방식으로 데이터를 읽어 오라고 핸들러에게 다시 요청
  5. 실행 엔진은 최종 결과를 사용자나 다른 모듈로 넘김

### 핸들러(스토리지 엔진)
- MySQL 서버의 가장 밑단에서 **MySQL 실행 엔진의 요청에 따라 데이터를 디스크로 저장하고 읽어 오는 역할**을 담당함
- 결국 스토리지 엔진을 의미함
  - MyISAM 테이블 조작 → 핸들러는 MyISAM 스토리지 엔진
  - InnoDB 테이블 조작 → 핸들러는 InnoDB 스토리지 엔진

옵티마이저는 회사의 경영진, 실행 엔진은 중간 관리자, 핸들러는 각 업무의 실무자로 비유 가능하다!

## 복제
MySQL 서버에서 복제(Replication)은 매우 중요한 역할을 담당한다. 해당 내용은 뒷 장에서 살펴보자.

## 쿼리 캐시
- SQL의 실행 결과를 메모리에 캐시하고, 동일 SQL 쿼리가 실행되면 테이블을 읽지 않고 즉시 결과를 반환함
  - 매우 빠른 성능을 보임
- 그러나 테이블의 데이터가 변경되면 캐시에 저장된 결과 중에서 변경된 테이블과 관련된 것들을 모두 삭제해야 함
  - 심각한 동시 처리 성능 저하를 유발함

그래서 MySQL 8.0에서는 쿼리 캐시가 MySQL 서버의 기능에서 완전히 제거되었다!

## 스레드 풀
MySQL 서버 엔터프라이즈 에디션은 스레드 풀 기능을 제공한다. 그러나 커뮤니티 에디션은 지원하지 않는다. 따라서 Percona Server에서 제공하는 스레드 풀 기능을 살펴보도록 하자.

### MySQL 엔터프라이즈 vs Percona Server
- MySQL 엔터프라이즈 스레드 풀은 MySQL 서버 프로그램에 내장됨
- Percona Server의 스레드 풀은 플러그인 형태로 작동하게 구현됨

### 스레드 풀의 사용 목적 
- MySQL 서버의 **CPU가 제한된 개수의 스레드 처리에만 집중하도록 하여 서버의 자원 소모를 줄이는 것을 목적**으로 함

> 👉 감당할 만큼만 받아서 최적의 서비스를 지원해주겠다! 라는 의미 같다. 유명 식당의 음식 수를 제한하고 제한된 양에 최선을 다해서 제공하는 느낌?

### 스레드 풀 기능
- CPU 프로세서 친화도를 높이려면 CPU 코어의 개수와 스레드 그룹의 수를 맞추는 것이 좋음
- 모든 스레드 그룹의 스레드가 각자 작업을 처리하고 있는 상태에서 새로운 쿼리 요청이 들어온다면, `thread_poll_stall_limit` 시간 동안 기다려야 스레드 풀이 요청을 처리할 수 있음
  - 응답 시간에 민감한 서비스라면 `thread_poll_stall_limit` 시스템 변수를 적절히 낮춰서 설정해야 함
- 선순위 큐와 후순위 큐를 이용해서 특정 트랜잭션이나 쿼리를 우선적으로 처리할 수 있는 기능 제공

## 트랜잭션 지원 메타데이터
### 메타데이터란?
- **테이블의 구조 정보와 스토어드 프로그램 등의 정보**를 뜻하며, 데이터 딕셔너리라고도 부름
- MySQL 5.7까지는 파일 기반으로 관리
  - 파일 기반의 메타 데이터는 생성 및 변경 작업에 대한 트랜잭션 지원 X
  - 데이터베이스 또는 테이블이 깨지는 경우가 발생함
- MySQL 8.0부터는 **메타데이터를 모두 InnoDB 테이블에 저장하도록 개선함**
  - 스키마 변경 작업 중간에 MySQL 서버가 비정상적으로 종료되더라도 완전한 성공 또는 완전한 실패로 정리됨
- MySQL 서버가 작동하는데 기본적으로 필요한 테이블을 묶어서 시스템 테이블이라고 부름
  - 사용자의 인증과 권한에 관련된 테이블이 있음
- 시스템 테이블과 메타데이터를 모두 모아서 mysql DB에 저장함
  - mysql DB는 `mysql.ibd`라는 이름의 테이블 스페이스에 저장됨

### InnoDB 스토리지 엔진 이외의 스토리지 엔진
- InnoDB 스토리지 엔진의 메타데이터는 InnoDB 테이블 기반의 딕셔너리에 저장함
- MyISAM이나 CSV 등과 같은 스토리지 엔진의 경우도 메타 정보를 저장할 공간이 필요함
  - 이를 위해 SDI(Serialized Dictionary Information) 파일을 사용
  - SDI는 직렬화를 위한 포맷 → InnoDB 테이블의 구조도 SDI 파일로 변환 가능

# InnoDB 스토리지 엔진 아키텍처
InnoDB 스토리지 엔진은 레코드 기반의 잠금을 제공한다. 따라서 높은 동시성 처리가 가능하고, 안정적이며 성능이 뛰어나다. InnoDB의 구조는 다음과 같다.
![](https://velog.velcdn.com/images/99mon/post/1f062ac3-ce87-4922-8df7-b2aa3de8aab3/image.png)
👉 [사진 출처 - Real MySQL 4장](https://wikibook.co.kr/realmysql801/)

InnoDB의 각 부분에 대해 살펴보자. 

## 프라이머리 키에 의한 클러스터링
- InnoDB의 모든 테이블은 **프라이머리 키를 기준으로 클러스터링되어 저장됨**
  - 즉, **프라이머리 키 값의 순서대로 디스크에 저장됨**
  - 세컨더리 인덱스는 프라이머리 키의 값을 논리적인 주소로 사용함
- 쿼리의 실행 계획에서 프라이머리 키는 다른 보조 인덱스에 비해 비중이 높게 설정됨(= 선택될 확률이 높음)
- MyISAM 스토리지 엔진에서는 클러스터링 키를 지원하지 않음
  - 프라이머리 키와 세컨더리 인덱스는 구조적으로 아무 차이가 없음
  - 모든 인덱스는 물리적인 레코드의 주소 값(ROWID)을 가짐

## 외래 키 지원
- InnoDB 스토리지 엔진 레벨에서 지원하는 기능
  - MyISAM이나 MEMORY 테이블에서는 사용 불가
- **부모 테이블과 자식 테이블 모두 해당 칼럼에 인덱스 생성이 필요함**
- 변경 시에는 반드시 부모 테이블이나 자식 테이블에 데이터가 있는지 체크하는 작업이 필요함
  - **잠금이 여러 테이블로 전파됨** → 데드락이 발생할 때가 많으므로, 개발 시 외래 키의 존재에 대해 주의해야 함
  
### 외래 키 관계에 대한 체크 작업 멈추기
- `foreign_key_checks` 시스템 변수를 OFF로 설정하면 일시적으로 멈출 수 있음
  - `SET foreign_key_checks=OFF;`
  - `ON DELETE CASCADE`와 `ON UPDATE CASCADE` 옵션이 무시됨
- 그렇다고 해서 부모와 자식 테이블 간의 관계가 깨진 상태로 유지해도 된다는 것은 아님
  - 반드시 일관성을 맞춰준 후 다시 외래 키 체크 기능을 활성화해야 함
    - `SET foreign_key_checks=ON;`

## MVCC(Multi Version Concurrency Control)
- 레코드 레벨의 트랜잭션을 지원하는 DBMS가 제공하는 기능
- 가장 큰 목적은 **잠금을 사용하지 않는 일관된 읽기를 제공하는 것**
  - **언두 로그**를 이용해 이 기능을 구현함
  - 멀티 버전이라는 것은 하나의 레코드에 대해 여러 개의 버전이 동시에 관리된다는 의미
- 아직 COMMIT이나 ROLLBACK이 되지 않은 상태에서 작업 중인 레코드를 조회하면 어디에 있는 데이터를 조회할까?
  - MySQL 서버의 시스템 변수에 설정된 격리 수준에 따라 다름
  - `READ_UNCOMMITTED`
    - InnoDB 버퍼 풀이 가지고 있는 데이터를 읽어서 반환
  - `READ_COMMITED`, `REPEATABLE_READ`, `SERIALIZABLE`
    - 언두 영역의 데이터를 반환
- 이러한 과정을 MVCC라고 표현함
  - 즉, **하나의 레코드에 대해 2개의 버전이 유지되고, 여러 상황에 따라 보여지는 데이터가 달라지는 구조**
  
## 잠금 없는 일관된 읽기(Non-Locking Consistent Read)
- InnoDB 스토리지 엔진은 **MVCC 기술을 이용해 잠금을 걸지 않고 읽기 작업을 수행함**
  - 읽기 작업 시 다른 트랜잭션이 가지고 있는 잠금을 기다리지 않아도 됨
- 이를 "잠금 없는 일관된 읽기"라고 표현함

## 자동 데드락 감지
- 잠금이 교착 상태에 빠지지 않았는지 체크하기 위해 **잠금 대기 목록을 그래프(Wait-for List) 형태로 관리함**
- **데드락 감지 스레드**가 주기적으로 잠금 대기 그래프를 검사 → 교착 상태에 빠진 트랜잭션 중 하나를 강제 종료함
  - 언두 로그 레코드를 더 적게 가진 트랜잭션이 종료 대상이 됨
    - 롤백 시 언두 처리를 할 내용이 적기 때문
- 동시 처리 스레드가 매우 많은 경우, 데드락 감지 스레드가 더 많은 CPU 자원을 소모할 수 있음
  - 잠금 목록이 저장된 리스트에 새로운 잠금을 걸고 데드락 스레드를 찾기 때문
  - `innodb_deadlock_detect`를 OFF로 설정하면 작동하지 않음

## 자동화된 장애 복구
- InnoDB 데이터 파일은 **MySQL 서버가 시작될 때 항상 자동 복구를 수행함**
  - Ex) 완료되지 못한 트랜잭션, 디스크에 일부만 기록된(Partial write) 데이터 페이지 등
- 자동으로 복구될 수 없는 손상이 있다면 자동 복구를 멈추고 MySQL 서버가 종료됨
- MySQL 서버가 종료된 경우, `innodb_force_recovery` 시스템 변수를 설정해서 MySQL 서버를 시작해야 함
  - 1부터 6까지 설정할 수 있음
  - 값이 커질수록 심각한 상황 → 데이터 손실 가능성 증가, 복구 가능성 감소
  - 그래도 시작되지 않으면 다시 구축해야 함

## InnoDB 버퍼 풀
- InnoDB 스토리지 엔진에서 가장 핵심적인 부분
- **디스크의 데이터 파일이나 인덱스 정보를 메모리에 캐시해 두는 공간**
  - 쓰기 작업을 지연시켜 일괄 작업으로 처리할 수 있게 해주는 버퍼 역할도 같이 함
- 버퍼 풀이 변경된 데이터를 모아서 처리하면 **랜덤한 디스크 작업의 횟수를 줄일 수 있음**

### 버퍼 풀의 크기 설정
- MySQL 5.7 버전부터 **InnoDB 버퍼 풀의 크기를 동적으로 조절**할 수 있음
  - `innodb_buffer_poll` 시스템 변수로 크기 설정
  - 가능하면 버퍼 풀의 크기를 줄이는 작업은 하지 않도록 주의
- InnoDB 버퍼 풀은 **128MB 청크 단위로 쪼개어 관리됨** → 버퍼 풀의 크기를 조정할 때 128MB 단위로 처리됨
  - 각 버퍼 풀을 버퍼 풀 인스턴스라고 표현함
  - 버퍼 풀을 여러 개로 쪼개어 관리함으로서 **내부 잠금 경합을 줄일 수 있음**
- 운영체제의 전체 메모리 공간이 8GB 미만인 경우
  - 50% 정도만 InnoDB 버퍼 풀로 설정
- 그 이상인 경우
  - 전체 메모리의 50%에서 시작해서 조금씩 올려가며 최적점 찾기

### 버퍼 풀의 구조
- 버퍼 풀이라는 거대한 메모리 공간을 **페이지 크기의 조각으로 쪼갬** → InnoDB 스토리지 엔진이 데이터를 필요로 할 때, **해당 데이터 페이지를 읽어서 각 조각에 저장함**
- 3개의 자료구조를 사용해서 페이지 조각을 관리함
  - LRU(Least Recently Used) 리스트
    - 디스크로부터 한 번 읽어온 페이지를 최대한 오랫동안 InnoDB 버퍼 풀의 메모리에 유지해서 디스크 읽기를 최소화하는 것이 목적
    - 한 번 읽힌 데이터 페이지가 자주 사용된다면 MRU 영역에서 살음
    - 거의 사용되지 않으면 LRU의 끝으로 밀려나 결국 제거됨
  - 플러시 리스트
    - 디스크로 동기화되지 않은 데이터를 가진 데이터 페이지(= 더티 페이지)의 변경 시점 기준의 페이지 목록을 관리함
    - 데이터가 변경되면 변경 내용을 리두 로그에 기록하고, 버퍼 풀의 데이터 페이지에도 변경 내용을 반영함
    - 내용이 변경된 데이터 페이지는 플러시 리스트에 관리되고, 특정 시점이 되면 디스크로 기록됨
  - 프리 리스트
    - 실제 사용자 데이터로 채워지지 않은 비어있는 페이지들의 목록
    - 사용자의 쿼리로 새롭게 디스크의 데이터 페이지를 읽어와야 하는 경우에 사용됨

### 버퍼 풀과 리두 로그
- 서버 성능 향상을 위해 데이터 캐시와 쓰기 버퍼링 기능이 존재함
  - 메모리 공간을 늘리면 데이터 캐시 기능을 향상시킬 수 있음
  - 쓰기 버퍼링 기능까지 향상시키려면 버퍼 풀과 리두 로그의 관계를 이해해야 함
- InnoDB 버퍼 풀은 디스크를 읽은 상태에서 전혀 변경되지 않은 **클린 페이지**와 변경된 데이터를 가진 **더티 페이지**를 가지고 있음
- 데이터 변경이 발생하면 리두 로그 파일에 기록됐던 로그 엔트리는 새로운 로그 엔트리로 덮어 쓰임
  - 재사용 가능한 공간과 재사용 불가능한 공간을 구분해서 관리해야 함
    - 재사용 불가능 공간을 활성 리두 로그라고 함
- **주기적으로 체크포인트 이벤트를 발생시켜 리두 로그와 더티 페이지를 디스크로 동기화** 시켜야 함
  - InnoDB 버퍼 풀의 **더티 페이지는 특정 리두 로그 엔트리와 관계**를 가짐

### 버퍼 풀 플러시(Buffer Poll Flush)
- 버퍼 풀에서 아직 디스크로 기록되지 않은 **더티 페이지들을 성능상의 악영향 없이 동기화하기 위해 2개의 플러시 기능을 백그라운드로 실행**함
  - 플러시 리스트 플러시
  - LRU 리스트 플러시

#### 플러시 리스트 플러시
- 리두 로그 공간의 재활용을 위해 주기적으로 오래된 리두 로그 엔트리가 사용하는 공간을 비워야 함
- 주기적으로 **플러시 리스트 플러시 함수를 호출해서 오래전에 변경된 데이터 페이지 순서대로 디스크에 동기화하는 작업을 수행함**
- 더티 페이지가 많을수록 디스크 쓰기 폭발 현상이 발생할 가능성이 높아짐
  - 일정 수준 이상의 더티 페이지가 발생하면 조금씩 디스크에 기록하여 문제 완화 가능

#### LRU 리스트 플러시
- LRU 리스트에서 사용 빈도가 낮은 데이터 페이지들을 제거해서 새로운 페이지들을 읽어올 공간을 만들어야 함
- 이를 위해 LRU 리스트 플러시 함수가 사용됨

### 버퍼 풀 상태 백업 및 복구
- 디스크의 데이터가 풀에 적재되어 있는 상태를 워밍업이라고 함
- **버퍼 풀이 잘 워밍업된 상태에서는 그렇지 않은 경우보다 몇십 배의 쿼리 처리 속도를 보임**
- MySQL 서버를 셧다운하기 전에 **현재 InnoDB 버퍼 풀의 상태를 백업**할 수 있음
  - 서버를 다시 시작하면 **백업된 버퍼 풀의 상태 다시 복구**할 수 있음
  - 버퍼 풀을 자동으로 백업하고 복구할 수 있는 기능도 제공함
```sql
# MySQL 서버 셧다운 전에 버퍼 풀의 상태 백업
mysql> SET GLOBAL innodb_buffer_poll_dump_now=ON;
  
# MySQL 서버 재시작 후, 백업된 버퍼 풀의 상태 복구
mysql> SET GLOBAL innodb_buffer_load_dump_now=ON;
```

### 버퍼 풀의 적재 내용 확인
- MySQL 8.0부터 버퍼 풀의 상태를 확인하기 위해 `information_schema` 데이터베이스의 `innodb_cached_indexes` 테이블이 추가됨
- 해당 테이블을 이용하면 테이블의 인덱스별로 데이터 페이지가 얼마나 InnoDB 버퍼 풀에 적재되어 있는지 확인할 수 있음

## Double Writer Buffer
![](https://velog.velcdn.com/images/99mon/post/8a02cc80-cb0b-40be-bd1f-337a73cb6e0d/image.png)
👉 [사진 출처 - Real MySQL 4장](https://wikibook.co.kr/realmysql801/)

- 리두 로그는 공간의 낭비를 막기 위해 페이지의 변경된 내용만 기록함 → 더티 페이지를 디스크 파일로 플러시할 때, **일부만 기록되면 그 페이지 내용을 복구할 수 없는 문제가 발생할 수 있음**
   - 파셜 페이지 또는 톤 페이지라고 함
- 이러한 문제를 막기 위해 **Double-write 기법**을 이용함
  - 실제 데이터 파일에 변경 내용을 기록하기 전에, **더티 페이지들을 묶어서 DoubleWrite 버퍼에 기록함**
  - 스토리지 엔진이 재시작될 때, 항상 DoubleWrite 버퍼의 내용과 데이터 파일의 페이지를 모두 비교함
    - 다른 내용을 담고 있는 페이지가 있다면 DoubleWrite 버퍼의 내용을 데이터 파일의 페이지로 복사함
- 데이터 안정성을 위해 자주 사용됨
- 데이터 무결성이 중요한 서비스에서는 DoubleWrite의 활성화를 고려하는게 좋음

## 언두 로그
트랜잭션과 격리 수준을 보장하기 위해 변경 전 데이터를 별도로 백업한다. 이렇게 **백업된 데이터를 언두 로그**라고 한다.

InnoDB 스토리지 엔진에서 중요한 역할을 담당하는 언두 로그에 대해 살펴보자. 

### 언두 로그 모니터링
- 언두 로그의 데이터는 크게 두 가지 용도로 사용됨
  - **트랜잭션의 롤백 대비용**
    - 롤백 시, 언두 로그에 백업해둔 이전 버전의 데이터를 이용해 복구함
  - **트랜잭션의 격리 수준을 유지하면서 높은 동시성 제공**
- 트랜잭션이 완료되었다고 해서 해당 트랜잭션이 생성한 언두 로그를 즉시 삭제할 수 있는 것은 아님
  - 활성 상태의 트랜잭션이 없어야 함
- 활성 상태의 트랜잭션이 장시간 유지되는 것은 좋지 않음
  - 언두 로그가 얼마나 증가했는지 항상 모니터링하는 것이 좋음
```sql
# MySQL 서버의 모든 버전에서 사용 가능한 명령
mysql> SHOW ENGINE INNODB STATUS \G;

## MySQL 8.0 버전에서 사용 가능한 명령
mysql> SELECT count
    -> FROM information_schema.innodb_metrics
    -> WHERE SUBSYSTEM='transaction' AND NAME='trx_rseg_history_len';
```

### 언두 테이블스페이스 관리
- **언두 로그가 저장되는 공간을 언두 테이블스페이스**라고 함
- 하나의 언두 테이블스페이스는 1개 이상 128개 이하의 롤백 세그먼트를 가지고, 롤백 세그먼트는 1개 이상의 언두 슬롯을 가짐
  - 언두 슬롯이 부족하다면 트랜잭션을 시작할 수 없는 심각한 문제가 발생함
- MySQL 8.0부터
  - 언두 로그는 **항상 시스템 테이블스페이스 외부의 별도 로그 파일에 기록됨**
  - `CREATE UNTO TABLESPACE`나 `DROP TABLESPACE` 명령으로 새로운 언두 테이블스페이스를 동적으로 추가하고 삭제할 수 있음

## 체인지 버퍼
- RDBMS에서 레코드가 INSERT되거나 UPDATE 될 때, 데이터 파일뿐 아니라 테이블에 포함된 인덱스를 업데이트하는 작업도 필요함
  - 인덱스는 랜덤하게 디스크를 읽음 → 인덱스가 많다면 상당히 많은 자원을 소모함
- 변경해야 할 인덱스 페이지를 디스크로부터 읽어와서 업데이트해야 한다면, 즉시 실행하지 않고 **임시 공간에 저장해 두고 사용자에게 결과를 반환하는 형태로 성능을 향상**시킴
  - 이때 사용하는 **임시 메모리 공간을 체인지 버퍼**라고 함
- 기본적으로 InnoDB 버퍼 풀로 설정된 메모리 공간의 25%까지 사용할 수 있음

## 리두 로그 및 로그 버퍼
- 리두 로그는 여러 가지 문제점으로 서버가 비정상적으로 종료됐을 때 **데이터 파일에 기록되지 못한 데이터를 잃지 않게 해주는 안전장치**
  - **영속성과 가장 밀접하게 연관**되어 있음
- 데이터베이스 서버는 비정상 종료가 발생하면 **리두 로그의 내용을 이용해 데이터 파일을 서버가 종료되기 직전의 상태로 복구함**
  - 커밋됐지만 데이터 파일에 기록되지 않은 데이터 → 리두 로그에 저장된 데이터로 복구 가능
- 롤백됐지만 데이터 파일에 이미 기록된 데이터 → 리두 로그로 복구 X, 언두 로그에 저장된 데이터로 복구 가능
### 리두 로그 아카이빙
- 데이터 변경이 많으면 아직 복사하지 못한 리두 로그가 덮어쓰일 수 있음
  - 백업 툴이 리두 로그 엔트리를 복사할 수 없어서 백업에 실패함
- **리두 로그 아카이빙 기능은 데이터 변경이 많아서 리두 로그가 덮어쓰인다 하더라도 백업이 실패하지 않게 함**

### 리두 로그 활성화 및 비활성화
- MySQL 서버에서 트랜잭션이 커밋돼도 데이터 파일은 즉시 디스크로 동기화되지 않지만, **리두 로그(트랜잭션 로그)는 항상 디스크로 기록됨**
- MySQL 8.0 부터 수동으로 리두 로그를 활성화하거나 비활성화할 수 있음
  - 서버가 비정상적으로 종료되어도 특정 시점의 일관된 데이터를 가질 수 있도록 **되도록이면 리두 로그를 활성화하기 **

## 어댑티브 해시 인덱스
- InnoDB 스토리지 엔진에서 **사용자가 자주 요청하는 데이터에 대해 자동으로 생성하는 인덱스**
  - 어댑티브 해시 인덱스 기능을 활성화하거나 비활성화할 수 있음
- B-Tree 검색 시간을 줄여주기 위해 도입된 기능
- **자주 읽히는 데이터 페이지의 키 값을 이용해 해시 인덱스를 만듦**
  - 필요할 때마다 검색해서 레코드가 저장된 데이터 페이지를 즉시 찾아갈 수 있음
  - 인덱스 키 값은 'B-Tree 인덱스의 고유 번호'와 'B-Tree 인덱스의 실제 키 값'의 조합
- 어댑티브 해시 인덱스는 하나만 존재
  - 내부 잠금(세마포어) 경합을 줄이기 위해 **파티션 기능**을 제공함
- **버퍼 풀에 올려진 데이터 페이지에 대해서만** 어댑티브 해시 인덱스로 관리됨
  - 버퍼풀에서 없어지면 어댑티브 해시 인덱스에서도 없어짐
- 어댑티브 해시 인덱스의 도움을 많이 받을수록 **삭제 또는 변경 작업은 더욱 치명적인 작업**이 됨
  
## InnoDB와 MyISAM, MEMORY 스토리지 엔진 비교
- MySQL 8.0으로 업그레이드 되면서 MySQL 서버의 **모든 시스템 테이블이 InnoDB 스토리지 엔진으로 교체됨**
- InnoDB 스토리지 엔진에 대한 기능이 개선됨
  - MyISAM 스토리지 엔진과 MEMORY 스토리지 엔진은 향후 없어질 것으로 예상함

# MyISAM 스토리지 엔진 아키텍처
## 키 캐시
- InnoDB의 버퍼 풀과 비슷한 역할
- 인덱스만을 대상으로 작동
- 인덱스의 디스크 쓰기 작업에 대해서만 부분적으로 버퍼링 역할을 함

## 운영체제의 캐시 및 버퍼
- MyISAM 테이블의 데이터 읽기나 쓰기 작업은 **항상 운영체제의 디스크 읽기 또는 쓰기 작업으로 요청됨**
  - 디스크의 I/O를 해결할 만한 캐시나 버퍼링 기능을 갖고 있지 않기 때문
- InnoDB처럼 전문적으로 캐시나 버퍼링을 하지는 못하지만, 없는 것보다는 나음
- 운영체제의 캐시 공간은 남는 메모리를 사용함
  - 쿼리 처리가 느려지지 않도록 캐시 공간을 위한 메모리를 비워둬야 함

## 데이터 파일과 프라이머리 키(인덱스) 구조
- 데이터 파일이 **힙(Heap) 공간처럼 활용됨** 
  - 프라이머리 키 값과 무관하게 **INSERT 되는 순서대로 데이터 파일에 저장됨** (↔ InnoDB는 클러스터링 되어 저장됨)
- 저장되는 레코드는 모두 ROWID라는 물리적인 주솟값을 가짐

# MySQL 로그 파일
로그 파일을 이용하면 MySQL 서버의 깊은 내부 지식이 없어도 MySQL의 상태나 부하를 일으키는 원인을 쉽게 찾아서 해결할 수 있다.

## 에러 로그 파일
- MySQL이 **실행되는 도중에 발생하는 에러나 경고 메시지가 출력되는 로그 파일**
- 에러 로그 파일의 위치
  - MySQL 설정 파일(`my.cnf`)에서 `log_error` 라는 이름의 파라미터로 정의된 경로 
  - 데이터 디렉터리에 `.err`라는 확장자가 붙은 파일로 생성됨
- 자주 보게되는 메시지
  - MySQL이 시작하는 과정과 관련된 정보성 및 에러 메시지
  - 마지막으로 종료할 때 비정상적으로 종료된 경우 나타나는 InnoDB의 트랜잭션 복구 메시지
  - 쿼리 처리 도중에 발생하는 문제에 대한 에러 메시지
  - 비정상적으로 종료된 커넥션 메시지(Aborted connection)
  - InnoDB의 모니터링 또는 상태 조회 명령(`SHOW ENGINE INNODB STATUS` 같은)의 결과 메시지
  - MySQL의 종료 메시지

## 제너럴 쿼리 로그 파일(제너럴 로그 파일, General log)
- MySQL 서버에서 **실행되는 쿼리의 전체 목록을 뽑아서 검토해 볼 때**, 쿼리 로그를 활성화해서 쿼리를 쿼리 로그 파일로 기록 → 기록한 파일을 검토하면 됨

## 슬로우 쿼리 로그
- **서비스에서 사용되는 쿼리 중에서 어떤 쿼리가 문제인지 판단하는 데 많은 도움**이 됨
- 쿼리는 정상적으로 완료됐지만, 실행하는데 걸린 시간이 `long_query_time`에 정의된 시간보다 많이 걸린 쿼리가 기록됨

제너럴 로그 파일이나 슬로우 쿼리 로그 파일의 내용이 상당히 많아서 직접 쿼리를 검토하기에 시간이 많이 걸리거나 어느 쿼리를 집중적으로 튜닝해야 할지 식별하기가 어려울 수 있다. 
⇒ Percona에서 개발한 Percona Toolkit의 pt-query-digest 스크립트를 이용하면 빈도나 성능별로 쿼리를 정렬해서 쉽게 살펴볼 수 있다.
