# Chapter 5

### 😶 트랜잭션과 잠금

- `트랜잭션`은 **작업의 완전성을 보장**해 주는 것이다.
- `트랜잭션`은 `작업`의 `일부`만 **적용되는 현상이 발생하지 않도록 만들어준다.**
- `잠금`은 `동시성`을 제어하기 위한 기능이고, `트랜잭션`은 **데이터의 정합성 보장을 위해 필요한 기능**이다.
- 격리 수준은 하나의 `트랜잭션`이나 여러 `트랜잭션` 간의 작업 내용을 **어떻게 공유하고 차단할 것인지 결정**하는 수준을 의미한다.

### 🤓 트랜잭션

- `트랜잭션`을 지원하지 않는 `스토리지 엔진`의 테이블은 더 많은 고민거리를 만들어 낸다.
- `MyISAM`이나 `MEMORY` 엔진은 **트랜잭션을 지원하지 않는다.**

### 🤔 MySQL에서의 트랜잭션

- `트랜잭션`은 꼭 여러 개의 `변경 작업`을 수행하는 `쿼리`가 `조합`됐을 때만 **의미 있는 개념**은 아니다.
- 하나의 `논리적인 작업 셋`에 대하여 100% 적용 되거나, **아무것도 적용되지 않아야 함을 의미**한다.

그렇다면 `트랜잭션` 관점에서 `InnoDB` 테이블과 `MyISAM` 테이블의 차이를 비교해보자

먼저 하나는 `InnoDB` 엔진을 사용하는 테이블, 다른 하나는 `MyISAM` 엔진을 사용하는 테이블로 생성한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/26844e67-bb8f-4d50-99c2-bf9cd5c4314a)

테스트용 테이블에 `레코드`를 한 건씩 저장한 후 `AUTO-COMMIT` 모드에서 아래의 `쿼리문`을 입력한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/5d4094f2-c9e5-47a3-b568-42ef61cf7edf)

두 `쿼리문`은  `프라이머리 키`가 `중복`되어 모두 `에러`가 나지만, `조회`해보면 **결과가 달리 나오는 것**을 볼 수 있다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/76f49293-dacc-4e3a-9b8d-593384cc1336)

`MyISAM` 테이블에는 오류가 발생했음에도 `1`과 `2`는 **INSERT 된 상태가 유지**된 것을 볼 수 있다.

`MyISAM` 테이블에서 실행되는 쿼리는 이미 `INSERT`된 데이터를 두고 `쿼리 실행`을 `종료`한 것이다.

`MyISAM`은 `트랜잭션`을 지원하지 않고, `InnoDB`는 `트랜잭션`을 지원하기 때문에 같은 쿼리라도 

동작은 다르다.

`InnoDB`는 쿼리 중 일부라도 오류가 발생하면 전체를 원 상태로 복구한다.

`MyISAM` 테이블에서 발생하는 `부분 업데이트 문제`는 **테이블 데이터 정합성을 맞추는 데 문제**가 된다.

`트랜잭션`은 애플리케이션 개발에서 고민해야 할 문제를 줄여주는 `필수적인 DBMS`의 기능이다.

`트랜잭션`이 지원되는 `InnoDB` 테이블을 사용하면 `애플리케이션 코드` 구현이 훨씬 쉬워진다.

```sql
try {
  START TRANSACTION
  INSERT INTO tab_a ...;
  INSERT INTO tab_b ...;
  COMMIT;
} catch (exception) {
  ROLLBACK;
}
```

### 🫠 주의사항

- `트랜잭션` 또한 **필요한 최소의 코드에만 적용**하는 것이 좋다.

사용자가 `게시판`에 `게시물`을 쓴다고 했을때 `트랜잭션`이 처리되는 과정을 순서대로 써본다면 아래와 같은데, 

왜 `최소한`의 `코드`에만 `적용`해야 하는지 누구나 알 수 있다.

```sql
1) 처리 시작
   -> 데이터베이스 커넥션 생성
   -> 트랜잭션 시작
2) 사용자의 로그인 여부 확인
3) 사용자의 글쓰기 내용의 오류 여부 확인
4) 첨부로 업로드된 파일 확인 및 저장
5) 사용자의 입력 내용을 DBMS에 저장
6) 첨부 파일 정보를 DBMS에 저장
7) 저장된 내용 또는 기타 정보를 DBMS에서 조회
8) 게시물 등록에 대한 알림 메일 발송
9) 알림 메일 발송 이력을 DBMS에 저장
   <- 트랜잭션 종료(COMMIT)
   <- 데이터베이스 커넥션 반납
10) 처리 완료
```

위 처리 절차 중에서 `트랜잭션 처리`에 `악영향`을 끼칠 만한 포인트는 아래와 같다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/4b478d0f-5671-4bbf-abd4-3a940488cafa)

`핵심 포인트`는 1~10번까지 한 `트랜잭션`에 묶일 필요가 없다는 것이다.

그렇다면 `개선한 절차` 는 어떻게 변경될까?

```sql
1) 처리 시작
2) 사용자의 로그인 여부 확인
3) 사용자의 글쓰기 내용의 오류 발생 여부 확인
4) 첨부로 업로드된 파일 확인 및 저장
   -> 데이터베이스 커넥션 생성(또는 커넥션 풀에서 가져오기)
   -> 트랜잭션 시작
5) 사용자의 입력 내용을 DBMS에 저장
6) 첨부 파일 정보를 DBMS에 저장
   <- 트랜잭션 종료(COMMIT)
7) 저장된 내용 또는 기타 정보를 DBMS에서 조회
8) 게시물 등록에 대한 알림 메일 발송
   -> 트랜잭션 시작
9) 알림 메일 발송 이력을 DBMS에 저장
   <- 트랜잭션 종료(COMMIT)
   <- 데이터베이스 커넥션 종료(또는 커넥션 풀에 반납)
10) 처리 완료
```

`프로그램`의 코드가 **데이터베이스 커넥션**을 가지고 있는 범위와 `트랜잭션`이 

활성화돼 있는 `프로그램`의 범위를 **최소화해야 한다는 것**이다.

`트랜잭션`에 네트워크 작업이 있는 경우에는 그 `코드` 한두 줄이라고 해도,

`DBMS 서버`가 높은 부하 상태로 빠지거나 **위험한 상태에 빠질 수 있기 때문에 배제**해야 한다.

### 😚 MySQL 엔진의 잠금

- `MySQL`에서 사용하는 잠금은 크게 **스토리지 엔진 레벨**과 **MySQL 엔진 레벨**로 나눌 수 있다.
- `MySQL` **엔진 레벨의 잠금**은 모든 **스토리지 엔진**에 `영향`을 미친다.
- `스토리지` **엔진 레벨의 잠금**은 **스토리지 엔진** 간 `상호 영향`을 미치지 않는다.

`MySQL` 엔진에서는 **테이블 데이터 동기화**를 위한 `테이블 락` 이외에도,

`테이블`의 구조를 잠그는 **메타데이터 락**이나 사용자의 필요에 맞게 사용하는 `네임드 락`도 지원한다.

### 🥲 글로벌 락(GLOBAL LOCK)

- `글로벌 락`은 **FLUSH TABLES WITH READ LOCK** 명령으로 `획득`할 수 있다.
- `MySQL`에서 제공하는 `잠금` 가운데 가장 범위가 크다.

한 `세션`에서 `글로벌 락`을 획득하면 다른 세션에서 `SELECT`를 제외한 대부분의 DDL, DML

문장은 `글로벌 락`이 해제될 때까지 `대기 상태`로 남는다.

`글로벌 락`이 영향을 미치는 범위는 `MySQL` 서버 전체이다.

**작업 대상 테이블**이나 `데이터베이스`가 다르더라도 동일하게 영향을 미친다.

여러 데이터베이스에 존재하는 `MyISAM`이나 `MEMORY` 테이블에 대해

`mysqldump`로 일관된 백업을 받아야 할 때는 **글로벌 락을 사용**해야 한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/7e6c0479-4ed5-450c-81a2-36975d4b2d40)

`InnoDB` 스토리지 엔진은 `트랜잭션`을 지원하므로, **일관된 데이터 상태를 위해 모든 데이터 변경 작업**을

멈출 필요가 없어졌다.

따라서, 조금 더 가벼운 `글로벌 락`의 필요성이 생겼다.

`MySQL 8.0` 버전부터는 **Xtrabackup**이나 **Enterprise Backup**과 같은 백업 툴들의 `안정적인 실행`을 위해

`백업 락`이 `도입`되었다.

```sql
mysql> LOCK INSTANCE FOR BACKUP;

-- // 백업 실행
mysql> UNLOCK INSTANCE;
```

`특정 세션`에서 백업 락을 획득하면 모든 `세션`에서 다음과 같이 **테이블의 스키마**나

`인증 관련 정보`를 변경할 수 없게 된다.

- `데이터베이스` 및 `테이블` 등 모든 객체 **생성 및 변경, 삭제**
- `REPAIR TABLE` 과 `OPTIMIZE TABLE` 명령
- 사용자 관리 및 비밀번호 변경

`백업 락`은 일반적인 `테이블`의 데이터 변경은 허용된다.

`MySQL` 서버의 구성은 `소스 서버`와 `레플리카 서버`로 구성된다.

주로 백업은 `레플리카 서버`에서 실행되며, 하지만 백업이 **FLUSH TABLES WITH READ LOCK** 명령으로 

`글로벌 락`을 획득하면 복제는 `백업 시간`만큼 `지연`될 수 밖에 없다.

`레플리카 서버`에서 `백업`을 실행하는 도중에 `소스 서버`에 문제가 생기면,

`레플리카 서버`의 데이터가 `최신 상태`가 될 때까지 `서비스`를 멈춰야 할 수도 있다.

`백업 툴`이 실행되는 도중에 `스키마 변경`이 실행되면 백업은 실패한다.

따라서 `MySQL 서버`의 백업 락은 **정상적으로 복제는 실행하고** `백업`의 `실패`를 막는다.

예를 들어, `DDL 명령`이 실행된다면 **복제를 일시 중지**해서 `백업`의 실패를 막는다.

### 🥶 테이블 락

- `테이블 락`은 **개별 테이블 단위로 설정되는 잠금**이다.
- `명시적` 또는 `묵시적`으로 특정 테이블의 락을 획득할 수 있다.

명시적으로 `락`을 획득하려면 아래의 `명령어`로 락을 획득할 수 있다.

```sql
-- 잠금 획득
LOCK TABLES table_name [ READ | WRITE ]

-- 잠금 해제
UNLOCK TABLES
```

명시적인 `테이블 락`은 `글로벌 락`과 동일하게 온라인 작업에 상당한 영향을 끼친다.

따라서 `애플리케이션 레벨`에서 사용할 필요가 거의 없다.

묵시적인 테이블 락은 `MyISAM`이나 `MEMORY` 테이블에 데이터를 변경하는 쿼리를 실행하면 발생한다.

`MySQL` 서버가 테이블에 잠금을 설정 후, 데이터를 변경한 후 잠금을 해제하는 방식으로 동작한다.

`InnoDB` 테이블의 경우 스토리지 엔진 차원에서 **레코드 기반의 잠금을 제공**한다.

따라서 단순 `데이터 변경 쿼리`로 인해 **묵시적인 테이블 락**이 설정되지는 않는다.

### 🤑 네임드 락

- `네임드 락`은 `GET_LOCK()` 함수를 이용해 **임의의 문자열에 대해 잠금**을 설정할 수 있다.
- `네임드 락`은 테이블이나 레코드와 같은 데이터베이스 객체를 대상으로 하지 않는다.

`네임드 락`은 단순히 사용자가 **지정한 문자열에 대해 획득하고 반납하는 잠금**이다.

`네임드 락`을 사용하는 예제를 간단히 살펴보면 아래와 같다.

```sql
-- 'mylock' 이라는 문자열에 대해 잠금을 획득
-- 이미 잠금을 사용중이면 2초 동안만 대기 -> 2초 이후 자동 잠금 해제
mysql> SELECT GET_LOCK('mylock', 2);

-- 'mylock' 이라는 문자열에 대해 잠금이 설정되어 있는지 확인한다.
mysql> SELECT IS_FREE_LOCK('mylock');

-- 'mylock' 이라는 문자열에 대해 획득했던 잠금을 반납한다.
mysql> SELECT RELEASE_LOCK('mylock');

-- 3개 함수 모두 정상적으로 락을 획득하거나 해제한 경우 -> 1을 반환
-- 아닐 경우 NULL 이나 0을 반환한다.
```

`네임드 락`은 많은 레코드에 대해 복잡한 요건으로 `레코드`를 변경하는 `트랜잭션`에 유용하다.

한꺼번에 `많은 레코드`를 변경하는 쿼리는 `데드락`의 원인이 되곤 한다.

**동일 데이터를 변경하거나 참조하는 프로그램**끼리 분류해서 `네임드 락`을 걸고 쿼리를 실행하면 

이 `문제`를 간단히 해결할 수 있다.

`MySQL 8.0` 부터는 `네임드 락`을 **중첩해서 사용**할 수 있게 되었다.

현재 `세션`에서 획득한 `네임드 락`을 **한 번에 모두 해제하는 기능도 추가**되어 편의성도 증가했다.

```sql
mysql> SELECT GET_LOCK('mylock_1', 10);

-- mylock_1에 대한 작업 실행
mysql> SELECT GET_LOCK('mylock_2', 10);
-- mylock_1과 mylock_2에 대한 작업 실행

mysql> SELECT RELEASE_LOCK('mylock_2');
mysql> SELECT RELEASE_LOCK('mylock_1');

-- mylock_1과 mylock_2를 동시에 모두 해제하려면 RELEASE_ALL_LOCKS() 함수를 사용해야 한다.
mysql> SELECT RELEASE_ALL_LOCKS();
```

### 🙉 메타데이터 락

- `메타데이터 락`은 데이터베이스 객체의 `이름`이나 `구조`를 변경하는 경우 **획득하는 잠금**이다.
- `메타데이터 락`은 명시적으로 `획득`하거나 `해제`할 수 없다.
- `RENAME TABLE` 명령은 **원본 이름**과 **변경될 이름** 두 개 모두 한꺼번에 `잠금`을 `설정`한다.

두 번의 `RENAME TABLE` 명령이 수행되는 하나의 예시를 살펴보자.

- rank → rank_backup
- rank_new → rank

`RENAME` 작업을 한꺼번에 실행하면 실제 `애플리케이션`에서 문제가 생기지 않는다.

만약 이 문장을 2개로 나눠서 실행하면 **아주 짧은 시간**이지만 `rank 테이블`이 존재하지 않기 때문에

**Table not found 오류**가 발생한다.

`메타데이터 잠금`과 `InnoDB 트랜잭션`을 동시에 사용해야 하는 경우도 있다.

예를 들어 아래와 같은 **INSERT 작업**만 하는 `로그 테이블`이 있다고 해보자.

```sql
mysql> CREATE TABLE access_log (
        id BIGINT NOT NULL AUTO_INCREMENT,
        client_ip INT UNSIGNED, 
        access_dttm TIMESTAMP,
        PRIMARY KEY(id) 
        );

```

만약 이 테이블의 구조를 변경해야 한다면 어떻게 할 수 있을까?

물론 `MySQL` 서버의 `Online DDL`을 사용해서 변경할 수 있다.

하지만 시간이 너무 오래 걸린다면, `언두 로그`의 증가와 `Online DDL`이 실행되는 동안 누적되는

**Online DDL 버퍼의 크기** 등 고민해야 할 것이 많을 것이다.

`MySQL`의 `DDL`은 **단일 스레드로 작동**하기 때문에 `상당히 많은 시간`이 `소모`된다는 것도 문제다.

이럴때는 **새로운 구조의 테이블을 생성**하고 먼저 `최근의 데이터`까지는 `프라이머리 키`인 id 값을

범위별로 나눠서 **여러 개의 스레드로 빠르게 복사**하는 것이 좋다.

이렇게 처리하는 과정을 한 번 살펴보면 아래와 같다.

```sql
-- 테이블 압축을 적용하기 위해 KEY_BLOCK_SIZE=4 옵션을 추가
mysql> CREATE access_log_new (
        id BIGINT NOT NULL AUTO_INCREMENT,
        client_ip INT UNSIGNED,
        access_dtm TIMESTAMP,
        PRIMARY KEY(id)
       ) KEY_BLOCK_SIZE=4;
       
       
- // 4개의 스레드를 이용해 id 범위별로 레코드를 신규 테이블로 복사
mysql_thread1> INSERT INTO access_log_new SELECT * FROM access_log WHERE id>=0 AND id<10000; 
mysql_thread2> INSERT INTO access_log_new SELECT * FROM access_log WHERE id>=10000 AND id<20000;
mysql_thread3> INSERT INTO access_log_new SELECT * FROM access_log WHERE id>=20000 AND id<30000;
mysql_thread4> INSERT INTO access_log_new SELECT * FROM access_log WHERE id>=30000 AND id<40000;
```

이제 나머지 데이터는 `트랜잭션`과 `테이블 잠금`, `RENAME TABLE` 명령으로 애플리케이션 중단 없이 실행된다.

**남은 데이터를 복사하는 시간** 동안은 테이블 잠금으로 인해 `INSERT`를 할 수 없다.

따라서 가능하면 아주 `최근 데이터`까지 복사해 둬야 `잠금 시간`을 `최소화`할 수 있다.

```sql
--  트랜잭션을 autocommit으로 실행(BEGIN이나 START TRANSACTION으로 실행하면 안 됨) 
mysql> SET autocommit=0;

--  작업 대상 테이블 2개에 대해 테이블 쓰기 락을 획득 
mysql> LOCK TABLES access_log WRITE, access_log_new WRITE;

--  남은 데이터를 복사
mysql> SELECT MAX(id) as @MAX_ID FROM access_log_new;
mysql> INSERT INTO access_log_new SELECT * FROM access_log WHERE pk>@MAX_ID; 
mysql> COMMIT;

-- 새로운 테이블로 데이터 복사가 완료되면 RENAME 명령으로 새로운 테이블을 서비스로 투입 
mysql> RENAME TABLE access_log TO access_log_old, access_log_new TO access_log;
mysql> UNLOCK TABLES;

-- 불필요한 테이블 삭제
mysql> DROP TALBE access_log_old;
```

### 😚 InnoDB 스토리지 엔진 잠금

- `InnoDB 스토리지 엔진`은 엔진 내부에서 **레코드 기반 잠금 방식을 탑재**하고 있다.
- **레코드 기반 잠금 방식**은 `MyISAM`보다 훨씬 뛰어난 `동시성 처리`를 제공한다.
- **InnoDB 스토리지 엔진**에서 사용되는 `잠금`에 대한 `정보`는 `MySQL` 명령을 이용해 접근하기 까다롭다.

서버에서 `InnoDB`의 **잠금 정보를 진단**할 수 있는 방법은 `InnoDB`의 잠금 정보를 **테이블을 생성해 덤프**하거나,

**SHOW ENGINE INNODB STATUS** 명령이 전부였고, 이조차도 이해하기 어려웠다.

최근 버전에서는 `InnoDB`의 `트랜잭션`과 `잠금`, 잠금 대기 중인 `트랜잭션`의 목록을 조회할 수 있게 되었다.

`information_schema` 데이터베이스에 존재하는 아래의 `테이블`을 `조인`해서 조회하면 된다.

- INNODB_TRX
- INNODB_LOCKS
- INNODB_LOCK_WAITS

이 `테이블`들을 조인해서 조회하면 어떤 `트랜잭션`이 잠금을 대기하고 있고 해당 `잠금`은 누가 가지고 있는지

확인할 수 있다. 또한 `장시간 잠금`을 가지고 있는 **클라이언트를 강제로 종료**시킬 수도 있다.

`InnoDB`의 잠금에 대한 모니터링도 더욱 강화되어 **Performance Schema를 이용**하여

`InnoDB` 스토리지 엔진의 `내부 잠금(세마포어)`에 대한 `모니터링`도 가능해졌다.

### 🫠 InnoDB 스토리지 엔진의 잠금

- `InnoDB` 스토리지 엔진은 잠금 정보가 `상당히 작은 공간`으로 관리된다.
- `레코드 락`이 **페이지 락**으로, `페이지 락`이 **테이블 락**으로 레벨업되는 경우는 없다.
- 상용 `DBMS`와는 다르게 `InnoDB 스토리지 엔진`은 레코드 락뿐 아니라 레코드와 `레코드 사이`의 간격을 잠그는 `GAP 락`이 `존재`한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/ef9b9d11-1ec4-4aca-a9c1-ea7852e223a3)

지금부터 다양한 락의 종류에 대해서 살펴보자

### 😚 레코드 락

- `레코드` 자체만을 잠그는 것을 `레코드 락`이라고 한다.
- `InnoDB 스토리지 엔진`은 레코드 자체가 아니라 **인덱스의 레코드를 잠근다.**
- `인덱스`가 하나도 없는 테이블도 내부적으로 **자동 생성된 클러스터 인덱스**를 이용해 `잠금`을 `설정`한다.

`InnoDB`에서는 대부분 `보조 인덱스`를 이용한 변경 작업은 `넥스트 키 락`이나 `갭 락`을 사용하지만,

`프라이머리 키` 또는 `유니크 인덱스`에 의한 변경 작업에서는 **레코드 자체에서만 락**을 건다.

### 🥱 갭 락

- `갭 락`은 레코드 자체가 아니라 **레코드와 바로 인접한 레코드 사이의 간격**만을 잠그는 것을 의미한다.
- `갭 락`의 역할은 레코드와 레코드 사이의 간격에 **새로운 레코드가 생성되는 것을 제어**하는 것이다.
- `갭 락`은 **넥스트 키 락의 일부로 자주 사용**된다.

### 🤥 넥스트 키 락

- `레코드 락`과 `갭 락`을 합쳐 높은 형태의 잠금을 **넥스트 키 락** 이라고 한다.

`STATEMENT` 포맷의 바이너리 로그를 사용하는 MySQL 서버에서는

`REPEATABLE READ` 격리 수준을 사용해야 한다.

`innodb_locks_unsafe_for_binlog` 시스템 변수가 비활성화되면 변경을 위해 검색하는

레코드에는 **넥스트 키 방식으로 잠금**이 걸린다.

`InnoDB`의 `갭 락`이나 `넥스트 키 락`은 바이너리 로그에 기록되는 쿼리가 레플리카 서버에서 

실행될 때 **소스 서버에서 만들어 낸 결과와 동일한 결과를 만들어내도록 보장**하는 것이 `목적`이다.

`넥스트 키락`과 `갭 락`으로 인해 **데드락이 발생하거나 다른 트랜잭션을 기다리게 만드는 일이 발생**한다.

바이너리 로그 포맷을 ROW 형태로 바꿔서 넥스트 키락이나 갭 락을 줄이는 것이 좋다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/e485c946-1f3d-4e44-8075-1b8f89745cde)

### 🙂 자동 증가 락

- `자동 증가`하는 숫자 값을 추출(채번)하기 위해 `AUTO_INCREMENT`라는 `컬럼 속성`을 제공한다.
- `InnoDB` 스토리지 엔진에서는 `AUTO_INCREMENT` 락을 이용해 테이블 수준의 잠금으로 **각 레코드마다 중복되지 않는 일련번호 값을 보장**한다.

`AUTO_INCREMENT 락`은 `INSERT`와 `REPLACE` 명령과 같이 새로운 레코드를 저장할때만 필요하다.

따라서 `UPDATE`나 `DELETE`에는 걸리지 않는다.

InnoDB의 다른 잠금과는 달리 `AUTO_INCREMENT` 락은 트랜잭션과 관계없이 `INSERT`나 `REPLACE` 

문장에서 **AUTO_INCREMENT 값을 가져오는 순간만 락이 걸렸다가 즉시 해제**된다.

`AUTO_INCREMENT 락`은 **테이블에 단 하나만 존재**한다.

따라서 `두 개`의 **INSERT 쿼리가 동시에 실행**되면 하나의 쿼리만 락을 획득하고 나머지 쿼리는 기다려야 한다.

`AUTO_INCREMENT 컬럼`에 명시적으로 값을 설정해도 **자동 증가 락은 걸린다.**

`AUTO_INCREMENT 락`을 명시적으로 `획득`하고 `해제`하는 방법은 없다.

`AUTO_INCREMENT 락`은 **아주 짧은 시간 동안 걸리기 때문에 대부분 문제가 되지 않는다.**

`MySQL 5.1` 버전 이상부터는 `innodb_autoinc_lock_mode`라는 

`시스템 변수`를 이용해 **자동 증가 락의 작동 방식을 변경**할 수 있다.

- `innodb_autoinc_lock_mode=0`

MySQL 5.0과 동일한 잠금 방식으로 **모든 INSERT 문장은 자동 증가 락을 사용**한다.

- `innodb_autoinc_lock_mode=1`

단순히 한 건 또는 여러 건의 레코드를 INSERT 하는 SQL 중에서 MySQL 서버가 INSERT 되는 레코드의 건수를

정확히 예측할 수 있을 때는 `자동 증가 락`을 사용하지 않고 훨씬 가볍고 빠른 `래치(뮤텍스)`를 이용해 처리한다.

`개선된 래치`는 자동 증가 락과 달리 **아주 짧은 시간 동안만 잠금을 걸고** 필요한 자동 증가 값을 가져오면 

잠금이 즉시 해제된다.

`INSERT .. SELECT`와 같이 MySQL 서버가 쿼리를 실행하기 전에 예측할 수 없는 경우에는

예외적으로 **자동 증가 락을 사용**한다.

대량 INSERT가 수행될 때는 `여러 개`의 자동 증가 값을 한 번에 할당받아서 `INSERT` 되는 레코드에 사용한다.

이 때 사용되지 못한 값은 `폐기`되므로 대량 `INSERT` 실행 이후의 **자동 증가 값은 연속되지 않을 수 있다.**

- `innodb_autoinc_lock_mode=2`

`innodb_autoinc_lock_mode`가 2로 설정되면 `InnoDB 스토리지 엔진`은 절대 자동 증가 락을 걸지 않고

경량화된 `래치(뮤텍스)`를 사용한다.

하나의 `INSERT` 문장으로 `INSERT`되는 레코드라고 하더라도 **연속된 자동 증가 값을 보장**하지 않는다.

이 설정 모드에서는 `INSERT … SELECT`와 같은 대량 `INSERT` 문장 실행 중에도 다른 `커넥션`에서 

`INSERT … SELECT`를 수행할 수 있기 때문에 **동시 처리 성능이 높아진다는 장점**이 있다.

이 설정에서 작동하는 **자동 증가 기능**은 `유니크한 값`이 생성된다는 것만 `보장`한다.

`STATEMENT` 포맷의 바이너리 로그를 사용하는 `복제`에서는 **소스 서버와 레플리카 서버의 자동 증가 값**이

달라질 수 있다는 점에 `주의`해야 한다.

자동 증가 값이 한 번 증가하면 절대 줄어들지 않는 이유는 `AUTO_INCREMENT` 잠금을 최소화하기 위해서다.

`INSERT 쿼리`가 실패해도 **AUTO_INCREMENT 값은 다시 줄어들지 않고 그대로 남는다.**

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/f260e102-8d82-48c4-aa1b-24b9e1934c58)

### 😊 인덱스와 잠금

- `InnoDB`의 잠금은 레코드를 잠그는 것이 아니라 **인덱스를 잠그는 방식으로 처리**한다.
- 변경해야 할 `레코드`를 찾기 위해 검색한 **인덱스의 레코드를 모두 락을 걸어줘야 한다.**

정확한 이해를 위해서 `UPDATE` 문장을 하나 살펴보자

```sql
--  예제 데이터베이스의 employees 테이블에는 아래와 같이 first_name 칼럼만
--  멤버로 담긴 ix_firstname이라는 인덱스가 준비돼 있다.
--  KEY ix_firstname (first_name)
--  employees 테이블에서 first_name=‘Georgi'인 사원은 전체 253명이 있으며,
--  first_name='Georgi'이고 last_name二'Klassen'인 사원은 딱 1 명만 있는 것을 아래 쿼리로 
--  확인할 수 있다.
mysql> SELECT COUNT(*) FROM e叩loyees WHERE first_name='Georgi'; 
+------------+
|        253 |
+------------+

mysql> SELECT COUNT(*) FROM employees WHERE first_name='Georgi1' AND last_name='Klassen';
+------------+
|          1 |
+------------+

--  employees 테이블에서 first_name='Georgi'이고 last_name='Klassen'인 사원의
--  입사 일자를 오늘로 변경하는 쿼리를 실행해보자.
mysql> UPDATE employees SET hire_date=NOW() WHERE first_name='Georgi' AND last_name='Klassen';
```

1건의 레코드 업데이트를 위해서 몇 개의 레코드에 락을 걸어야 할까?

이 UPDATE 문장의 조건에서 인덱스를 사용할 수 있는 조건은 `first_name=’Georgi’` 이며,

last_name 컬럼은 인덱스에 없기 때문에 `first_name=’Georgi’` 인 레코드 `253건`이 잠긴다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/7b688392-19c5-4f41-943c-0ba7fcd464b8)

이 예제는 몇 건의 `레코드`만 잠그지만, `UPDATE` 문장을 위해 `적절한 인덱스`를 `준비`하지 않았다면

각 `클라이언트` 간의 `동시성`이 상당히 떨어져서 한 `세션`에서 `UPDATE` 작업을 하는 중에는 다른 `클라이언트`는

그 `테이블`을 `업데이트` 하지 못하고 기다려야 한다.

`테이블`에 `인덱스`가 하나도 없다면 정말 끔찍해진다.

이런 경우에는 `테이블`을 풀 스캔하면서 `UPDATE 작업`을 수행하기 때문에 예를 들어 `30만건의 데이터`가 있다면

`30만건의 데이터`에 모두 레코드를 잠근다.

이를 통해 `MySQL`의 **InnoDB의 인덱스 설계**가 `중요한 이유`를 알 수 있다.

### 🥶 레코드 수준의 잠금 확인 및 해제

- `레코드` 수준의 잠금은 테이블의 `레코드` 각각에 잠금이 걸리므로 그 `레코드`가 자주 사용되지 않는다면 오랜 시간 동안 **잠겨진 상태로 남아 있을 수도 있다.**
- MySQL 5.1 버전부터는 `레코드 잠금`과 `잠금 대기`에 대한 `조회`가 가능하도록 `개선`되었다.
- 강제로 `잠금`을 `해제`하려면 `KILL` 명령으로 프로세스를 `강제 종료`하면 된다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/7967f08b-48ed-48c1-a567-21d1ae69fa2c)

예를 들어, 위와 같은 `잠금 시나리오`가 있다고 해보자.

각 `트랜잭션`이 어떤 잠금을 기다리고 있고, 기다리고 있는 잠금은 어떤 `트랜잭션`이 가지고 있는지

쉽게 **메타 정보를 통해 조회**할 수 있다.

버전 별로 조회할 수 있는 스키마와 테이블이 상이하다.

- `MySQL 5.1`
    - information_schema
        - INNODB_TRX, INNODB_LOCKS, INNODB_LOCKS_WAITS

- `MySQL 8.0`
    - performance_schema
        - data_locks, data_lock_waits

`MySQL 8.0` 버전부터 information_schema의 정보는 조금씩 `Deprecated` 되고 있다.

따라서, `8.0` 버전 이상을 사용한다면 `performance_schema`의 테이블을 이용해 잠금을 조회해야 한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/eecfdeb6-be6d-43ff-b3f4-550a6f908fc3)

위의 내용은, `MySQL` 서버에서 앞의 3개의 `UPDATE` 명령이 실행된 상태의 **프로세스 목록을 조회**한 것이다.

어떤 내용이 들어있는지 살펴보면 다음과 같다.

`17번 스레드`는 지금 아무것도 하지 않고 있지만 `트랜잭션`을 시작하고 `UPDATE`가 `완료` 되었다.

`17번 스레드`가 `COMMIT`을 실행하지 않았으므로, **업데이트한 레코드의 잠금을 그대로 가지고 있는 상태**다.

`18번 스레드`는 그 다음으로 `UPDATE 명령`을 실행했고, **19번 스레드가 이어서 UPDATE 명령을 실행**했다.

따라서, `18번`과 `19번` 스레드는 `잠금 대기`로 인하여 아직 `UPDATE 명령`을 실행중인 것으로 표시된다.

`performance_schema`의 **data_locks, data_lock_waits 테이블을 조인**해서 `잠금 대기 순서`를 조회하는

쿼리를 작성하면 아래와 같다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/c184024f-4123-4019-9b82-7f970660bf93)

`쿼리`의 실행 결과를 보면, 현재 대기 중인 스레드는 `18번`과 `19번`인 것을 확인할 수 있다.

`18번` 스레드는 `17번` 스레드를, `19번` 스레드는 `17`번과 `18`번 스레드를 기다리고 있다.

`잠금 대기 큐`의 내용을 그대로 보여주기 때문에 이런 내용을 표시하게 되는 것이다.

따라서 `17번` 스레드가 잠금을 해제해야 `18번` 스레드가 작업을 완료하고 `19번`스레드가 작업을 수행할 수 있다.

`17번` 스레드가 어떤 잠금을 가지고 있는지 더 상세하게 확인하고 싶다면 

`data_locks` 테이블이 가진 `컬럼`을 모두 살펴보면 된다.

```sql
mysql> SELECT * FROM performance_schema.data_locks\G
```

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/3a25d389-11be-4157-bf35-15c3e0acd8d4)

위의 결과로 비추어 보면, `employees 테이블`에 대해 `IX 잠금(Intentional Exclusive)`을 가지고 있고,

`employees` 테이블의 특정 레코드에 대해서 `쓰기 잠금`을 가지고 있다는 것을 알 수 있다.

`REC_NOT_GAP` 표시가 있으므로 `레코드 잠금`은 **갭이 포함되지 않은 순수 레코드에 대한 잠금**만 

가지고 있는 것을 알 수 있다.

만약, `17번 스레드`가 잠금을 가진 상태에서 상당히 `오랜 시간` 멈춰 있다면 **아래의 명령으로 강제 종료**할 수 있다.

```sql
mysql> KILL 17;
```

### 👿 MySQL의 격리 수준

- `트랜잭션`의 격리 수준이란 여러 `트랜잭션`이 동시에 처리될 때 **특정 트랜잭션이 다른 트랜잭션에서 변경하거나 조회하는 데이터를 볼 수 있게 허용할지 말지를 결정**하는 것이다.
- `격리 수준`은 4가지 단계가 존재한다.
- `READ UNCOMMITED`와 `SERIALIZABLE`은 거의 사용되지 않는다.

`격리 수준`이 높아질 수록 **동시 처리 성능이 떨어지는 것**이 일반적이다.

하지만 `SERIALIZABLE` 격리 수준이 아니면 **크게 성능의 개선이나 저하는 발생하지 않는다.**

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/47d36a27-ecdd-403b-8b97-bf1e958c0b8c)

`SQL-92` 또는 `SQL-99` 표준에 따르면 **REPEATABLE_READ** 격리 수준에서는 

`PHANTOM_READ`가 발생할 수 있지만, `InnoDB 엔진`을 사용하면 `독특한 특성`으로 인해 발생하지 않는다.

일반적인 온라인 서비스 용도로 `데이터베이스`를 사용한다면 

`READ_COMMITED` 또는 `REPEATABLE_READ`를 사용하는 것이 일반적이다.

### 🙃 READ_UNCOMMITED

- 각 `트랜잭션`의 변경 내용이 `COMMIT`이나 `ROLLBACK` 여부에 상관없이 **다른 트랜잭션에서 보인다.**

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/5a44533e-b668-4a85-a6d5-26b7b8e82edf)

위의 그림을 보면 **사용자 B**는 **사용자 A**가 `INSERT`한 사원의 정보를 `커밋`되지 않았는데 `조회`할 수 있다.

`A`가 어떤 문제로 **이 데이터를 롤백**하면 사용자 B는 `Lara`가 정상적인 사원이라고 생각해서 `처리`할 것이다.

이런 현상을 `더티 리드(Dirty Read)`라고 부른다.

`더티 리드`는 RDBMS 표준에서 **트랜잭션의 격리 수준으로 인정하지 않을 정도로 정합성에 문제**가 많다.

### 😲 READ_COMMITED

- `오라클 DBMS`에서 기본으로 사용되는 `격리 수준`이다.
- `온라인 서비스`에서 **가장 많이 선택되는 격리수준** 이기도 하다.
- `Dirty Read`가 발생하지 않고 `COMMIT`이 완료된 데이터에 대해서만 **다른 트랜잭션이 조회**할 수 있다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/581f0d46-a117-4bce-aba5-d755dac51b9c)

`A 사용자`가 데이터를 변경해서 커밋하는 도중에, 다른 `트랜잭션`이 조회를 시도하면

어떤 `데이터`를 반환하는지 시나리오를 한 번 살펴보자.

1. `A`는 **emp_no=500000**인 사원의 `first_name`을 Lara → Toto로 `변경`
2. 새로운 값인 `Toto`는 **테이블에 즉시 기록**되고, Lara는 언두 영역으로 백업
3. `B`가 **emp_no=500000**인 사원을 조회하면 `언두 영역`에 있는 `데이터`를 반환

즉, `A`가 변경된 내용을 `커밋`하기 전에는 다른 `트랜잭션`에서는 **변경된 데이터를 참조할 수 없다.**

`READ_COMMITED` 격리 수준에서도 `NON_REPEATABLE_READ`라는 **부정합 문제는 존재**한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/f0d7613e-c58c-426c-b753-ca86c912cd6a)

위의 그림에서 `NON_REPEATABLE_READ`가 왜 문제가 되는지 살펴보자.

1. 사용자 `B`가 BEGIN 명령으로 `트랜잭션`을 `시작`
2. 사용자 `B`는 `first_name`이 **Toto인 사용자를 검색**했고, 일치하는 결과가 없음
3. 사용자 `A`가 사원번호가 500000인 사원의 이름을 `Toto`로 변경하고 `커밋`
4. 사용자 `B`가 SELECT 쿼리로 `first_name`이 `Toto`인 사용자를 검색해서 **1건의 결과를 얻음**

간단하게 생각해보면 `커밋` 된 데이터가 조회되는 현상인데 이게 왜 문제가 될까 싶다.

하지만 `사용자 B`가 실행한 **두 번의 조회 명령은 한 트랜잭션에서 실행**되었다.

즉, `사용자 B`가 하나의 트랜잭션 내에서 **똑같은 SELECT 쿼리를 실행**했을 때는 `항상 같은 결과`를

가져와야 한다는 **REPEATABLE_READ 정합성에 어긋나는 것이다.**

이런 `부정합 현상`은 일반적인 웹 프로그램에서는 큰 문제가 되지 않을 수 있다.

하지만 `하나의 트랜잭션`에서 동일 데이터를 **여러 번 읽고 변경하는 작업이 있다면 문제**가 될 수도 있다.

`트랜잭션` 내에서 실행되는 `SELECT` 문장과 트랜잭션 없이 실행되는 `SELECT` 문장의 차이를

혼동하는 경우가 생각보다 많다.

`READ_COMMITED` 격리 수준에서는 **트랜잭션의 유무가 크게 중요하지 않다.**

`REPEATABLE_READ` 격리 수준에서는 **SELECT 쿼리 문장도 트랜잭션 범위 내에서만 작동**한다.

`START TRANSACTION(BEGIN)` 명령으로 트랜잭션을 시작하고,

다른 `트랜잭션`이 데이터를 수정해도 같은 `조회 쿼리`에 대해서는 실행해도 `같은 결과`만 받아보게 된다.

### 😊 REPEATABLE READ

- MySQL `InnoDB` 스토리지 엔진에서 **기본으로 사용되는 격리 수준**이다.
- 바이너리 로그를 가진 `MySQL` 서버에서는 `REPEATABLE_READ` **격리 수준 이상을 사용**해야 한다.

이 격리 수준에서는 `NON_REPEATABLE_READ` 부정합이 발생하지 않는다.

`InnoDB` 스토리지 엔진은 트랜잭션 `ROLLBACK`에 대비하기 위해 **변경 전 레코드를 언두 공간에 백업**해두고 실제 레코드 값을 변경한다.

이런 방식을 `MVCC`라고 부르며, `REPEATABLE_READ`는 이 `MVCC`를 위해 **언두 영역에 백업된 이전 데이터**를

이용해 `동일 트랜잭션` 내에서는 `동일한 결과`를 보여줄 수 있게 `보장`한다.

`REPEATABLE_READ`와 `READ_COMMITED`의 차이는 언두 영역에 백업된 레코드의

`여러 버전` 가운데 **몇 번째 이전 버전까지 찾아 들어가야 하느냐**에 있다.

모든 `InnoDB`의 트랜잭션은 **고유한 트랜잭션 번호**를 가지며,

`언두 영역`에 백업된 `모든 레코드`에는 **변경을 발생시킨 트랜잭션의 번호가 포함**되어 있다.

`언두 영역`의 백업된 데이터는 `InnoDB` 스토리지 엔진의 판단에 의해 **주기적으로 삭제**된다.

`REPEATABLE_READ` 격리 수준에서는 `MVCC`를 보장하기 위해 실행 중인 `트랜잭션` 가운데 가장

오래된 `트랜잭션 번호`보다 **트랜잭션 번호가 앞선 언두 영역의 데이터는 삭제**할 수 없다.

물론 그렇다고 해서, `가장 오래된 트랜잭션 번호` 이전의 트랜잭션에 의해 변경된 모든 언두 데이터가

필요하다는 이야기는 아니다.

정확하게 표현하면 **특정 트랜잭션 번호의 구간 내에서 백업된 언두 데이터가 보존**되어야 한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/9923c5b8-fcf4-405d-8be2-299459f6b012)

`REAPEATABLE_READ` 격리 수준이 어떻게 `작동`하는지 `시나리오`를 살펴보자.

1. `employees` 테이블의 데이터는 **6번 트랜잭션에 의해 INSERT** 
2. `A`가 트랜잭션 `12번`, `B`는 트랜잭션 `10번`이다.
3. 사용자 `A`가 사원의 이름을 `Toto`로 **변경하고 커밋**을 수행
4. `B`가 `emp_no=500000`인 사원을 A **트랜잭션 변경 전후 한 번씩 SELECT** 했으나 항상 `Lara` 라는 값이 반환된다.
5. `B`가 트랜잭션 `10번`이므로, 트랜잭션 번호가 `10`보다 작은 **트랜잭션 번호에서 변경한 것만 보이기 때문이다.**

`언두 영역`에 백업된 데이터가 하나만 있는 그림이지만, 실제로는 더 많을 수 있다.

`트랜잭션`을 시작하고 장시간 종료하지 않으면 **언두 영역이 백업된 데이터로 무한정 커질 수 있기 때문에**

`트랜잭션`을 **작업이 완료 되었다면 빨리 종료**해야 한다.

물론 `REPEATABLE_READ` 격리 수준에서도 다음과 같은 `부정합`이 발생할 수 있다.

만약 `INSERT`를 실행하는 도중에 **SELECT … FOR UPDATE 쿼리로 조회를 시도**하면 어떻게 될까?

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/438c699e-5ffc-4d06-9c36-d50fa3fe35d7)

위의 그림의 시나리오로 `PHANTOM_READ`에 대해 알아보자.

1. 사용자 `B`가 트랜잭션을 시작한다.
2. 사용자 B는 `SELECT … FOR UPDATE` 쿼리로 `Lara`를 반환 받았다.
3. `A`가 `Georgi` 라는 이름을 가진 사원을 `INSERT` 하고 데이터를 COMMIT 한다.
4. 다시 사용자 `B`가 `SELECT … FOR UPDATE` 쿼리를 실행하면 **Lara, Georgi를 반환**받는다.

같은 `트랜잭션`에서 `B`가 두 번의 `SELECT` 쿼리를 실행했는데 다른 결과가 반환된 것이다.

이렇게 다른 `트랜잭션`에서 수행한 변경 작업에 의해 `레코드`가 보였다 안 보였다 하는 현상을 

`PHANTOM_READ`라고 부른다.

`SELECT … FOR UPDATE` 쿼리는 `SELECT` 하는 레코드에 쓰기 잠금을 걸어야 하는데,

`언두 레코드`에는 잠금을 걸 수 없다.

따라서, `SELECT … FOR UPDATE`나 `SELECT … LOCK IN SHARE MODE`로 조회되는

`레코드`는 **언두 영역의 변경 전 데이터를 가져오지 않고 현재 레코드의 값을 가져온다.**

### 😚 SERIALIZABLE

- 가장 `단순한 격리 수준`이면서 동시에 가장 `엄격한 격리 수준`이다.
- `동시 처리 성능`도 **다른 트랜잭션 격리 수준보다 떨어진다.**

`InnoDB` 테이블에서 기본으로 순수한 `SELECT` 작업은 아무런 `레코드 잠금`도 설정하지 않는다.

하지만 `트랜잭션`의 격리 수준이 `SERIALIZABLE` 이라면 읽기 작업도 `공유 잠금`을 획득해야 한다.

따라서 동시에 다른 `트랜잭션`은 레코드를 변경하지 못하게 된다.

한 `트랜잭션`에서 읽고 쓰는 레코드를 다른 `트랜잭션`에서는 절대 접근할 수 없다.

`SERIALIZABLE` 격리 수준에서는 `PHANTOM_READ`가 발생하지 않는다.

`InnoDB` 스토리지 엔진에서는 `갭 락`과 `넥스트 키 락`이 **PHANTOM_READ를 막아준다.**