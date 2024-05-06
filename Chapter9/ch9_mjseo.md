# Chapter 9

### 🤓 쿼리 실행 절차

`MySQL 서버`에서 쿼리가 실행되는 과정은 크게 `세 단계`로 나눌 수 있다.

1. 사용자로부터 요청된 `SQL 문장`을 잘게 쪼개서 `MySQL 서버`가 이해할 수 있는 수준으로 `분리`한다.
2. `SQL`의 파싱 정보를 확인하면서 어떤 `테이블`로부터 읽고 **어떤 인덱스를 사용할지 선택**한다.
3. 두 번째 단계에서 결정된 `테이블`의 읽기 순서나 선택된 `인덱스`를 이용해서 `스토리지 엔진`으로부터 데이터를 가져온다.

각 단계를 상세히 살펴보면 아래와 같다.

첫 번째 단계를 `SQL 파싱`이라고 부른다. `SQL 파싱`은 `SQL 파서`라는 **모듈로 처리**하게 된다.

`SQL 문장`이 문법적으로 잘못된 경우, 이 단계에서 걸러지게 된다.

첫 번째 단계에서 `SQL 파스 트리`가 만들어지게 되는데, MySQL 서버는 `SQL 파스 트리`를 이용해 **쿼리를 실행**한다.

두 번째 단계에서는 첫 번째 단계에서 만들어진 `SQL 파스 트리`를 `참조`하면서 아래와 같은 작업을 한다.

- `불필요한 조건` 제거 및 `복잡한 연산`의 단순화
- 여러 테이블의 `조인`이 있는 경우 **어떤 순서로 테이블을 읽을지 결정**
- 각 `테이블`에 사용된 조건과 `인덱스 통계 정보`를 이용해 **사용할 인덱스를 결정**
- 가져온 `레코드`들을 임시 테이블에 넣고 다시 한번 `가공`해야 하는지 `결정`

no

위의 `작업` 이외에도 `많은 작업`들을 하게 된다.

두 번째 단계는 `최적화 및 실행 계획 수립`이라고 부른다. 이 작업은 `“옵티마이저”`가 처리한다.

두 번째 단계가 완료되면 쿼리의 `“실행 계획”`이 만들어지고, 세 번째 단계에서 수립된 `실행 계획`대로 스토리지 엔진에 `레코드`를 읽어오도록 요청하고 MySQL 엔진이 받은 `레코드`를 조인하거나 `정렬`하는 작업을 `수행`한다.

정리해보면, 첫 번째 단계와 두 번째 단계는 `MySQL 엔진`이 대부분의 `작업`을 `처리`하며

세 번째 작업은 `MySQL 엔진`과 `스토리지 엔진`이 **동시에 참여해서 처리**한다.

### 😲 옵티마이저의 종류

- `옵티마이저`는 데이터베이스 서버에서 두뇌와 같은 역할을 담당한다.
- `옵티마이저`는 **비용 기반 최적화 방법**과 **규칙 기반 최적화 방법**으로 나눌 수 있다.

`규칙 기반 최적화`와 `비용 기반 최적화 방법`에 대해 알아보면 아래와 같다.

**규칙 기반 최적화**

- 대상 `테이블`의 `레코드 건수`나 `선택도` 등을 고려하지 않고 `옵티마이저`에 내장된 **우선순위에 따라 실행 계획을 수립**
- `통계 정보`를 조사하지 않고 `실행 계획`이 `수립`되기 때문에 같은 `쿼리`에 대해서는 거의 같은 `실행 방법`을 만든다.
- 사용자의 `데이터`는 `분포도`가 매우 다양하므로, 현재는 거의 `사용`되지 않는다.

**비용 기반 최적화**

- `쿼리`를 처리하기 위한 여러 가지 가능한 방법을 만들고, 작업의 `비용 정보`와 대상 테이블의 예측된 `통계 정보`를 이용해 실행 계획별 비용을 산출한다.
- 산출된 `실행 방법`별로 비용이 `최소`로 `소요`되는 처리 방식을 선택해 **최종적으로 쿼리를 실행**한다.

`규칙 기반 최적화`는 각 `테이블`이나 `인덱스`의 통계 정보가 거의 없고 상대적으로 느린 `CPU 연산`을 이유로

`비용 계산 과정`이 부담스럽다는 이유로 사용되던 `최적화 방법`이고 **현재는 대부분 사용하지 않는다.**

### 🙃 풀 테이블 스캔과 풀 인덱스 스캔

- `풀 테이블 스캔`은 인덱스를 사용하지 않고 `테이블`의 데이터를 **처음부터 끝까지 읽어서 요청을 처리**하는 것을 말한다.

`옵티마이저`는 아래와 같은 조건이 일치할 때 주로 `풀 테이블 스캔`을 `선택`한다.

- `테이블`의 `레코드 건수`가 너무 작아서 `인덱스`를 통해 읽는 것보다 `풀 테이블 스캔`을 하는 편이 더 빠른경우
- `WHERE` 절이나 `ON` 절에 인덱스를 이용할 수 있는 적절한 조건이 없을 때
- `인덱스 레인지 스캔`을 사용할 수 있는 `쿼리`라고 하더라도 `옵티마이저`가 판단한 조건 일치 `레코드 건수`가 너무 많은 경우**(B-Tree를 샘플링해서 조사한 통계 정보 기준)**

일반적으로 `테이블`의 전체 크기는 `인덱스`보다 훨씬 크다.

대부분의 DBMS는 `풀 테이블 스캔`을 실행할 때 **한꺼번에 여러 개의 블록이나 페이지를 읽어오는 기능**을 `내장`하고 있다.

`InnoDB 스토리지 엔진`은 특정 테이블의 연속된 데이터 페이지가 읽히면 `백그라운드 스레드`에 의해,

`Read-Ahead` 작업이 자동으로 시작된다.

`풀 테이블 스캔`이 실행되면 **처음 몇 개의 데이터 페이지**는 `Foreground-Thread`가 페이지를 읽어오지만,

특정 시점부터는 `Background-Thread`로 넘긴다.

`백그라운드 스레드`가 읽기를 넘겨받는 시점부터는 한 번에 `4개` 또는 `8개`씩 페이지를 읽으면서 수를 `증가`시킨다.

한 번에 최대 `64개`의 **데이터 페이지를 읽어서 버퍼 풀에 저장**해 둘 수 있다.

`클라이언트 스레드`가 **버퍼 풀에 준비된 데이터를 가져다 사용**하기만 하면 되므로 `쿼리`가 **상당히 빨리 처리**되는 것이다.

`MySQL 서버`에서는 `innodb_read_ahead_threshold` 시스템 변수를 이용해서, `InnoDB 스토리지 엔진`이

언제 `Read-Ahead`를 시작할지 `임계값`을 `설정`할 수 있다.

`포그라운드 스레드`에 의해 `시스템 변수`에 `설정된 개수`만큼의 **연속된 데이터 페이지**가 읽히면,

`InnoDB 스토리지 엔진`은 백그라운드 스레드를 이용해 **대량으로 그 다음 페이지를 읽어서 버퍼 풀에 적재**한다.

일반적으로 `디폴트 설정`으로도 충분하지만, `데이터 웨어하우스`용으로 `MySQL`을 사용한다면,

이 옵션을 `더 낮은 값`으로 `설정`해서 더 빨리 `리드 어헤드`가 시작되게 `유도`하는 것도 좋은 방법일 수 있다.

`Read-Ahead`는 **풀 테이블 스캔**에서만 사용되는 것이 아니라 **풀 인덱스 스캔**에서도 `동일`하게 `처리`된다.

`풀 인덱스 스캔`은 인덱스를 **처음부터 끝까지 스캔**하는 것을 `의미`한다.

```sql
mysql> SELECT COUNT(*) FROM employees;
```

위와 같은 `쿼리`를 `처리`한다고 생각해보자.

아무런 조건 없이 `employees` 테이블의 `레코드 건수`를 `조회`하고 있으므로 **풀 테이블 스캔**을 할 것 같지만

**풀 인덱스 스캔**을 할 가능성이 더 높다.

`MySQL 서버`는 앞의 예제와 같이 **단순히 레코드의 건수만 필요로 하는 쿼리**라면 용량이 작은 `인덱스`를 

선택하는 것이 `디스크 읽기 횟수`를 줄일 수 있기 때문이다.

하지만, 아래의 `쿼리`처럼 레코드에만 있는 `컬럼`이 필요한 경우에는 `풀 테이블 스캔`을 한다.

```sql
mysql> SELECT * FROM employees;
```

### 😚 병럴 처리

- `MySQL 8.0` 버전부터 용도가 한정돼 있긴 하지만 `쿼리의 병렬 처리`가 가능해졌다.
- `병렬 처리`는 하나의 쿼리를 `여러 스레드`가 작업을 나누어 **동시에 처리**한다는 것을 `의미`한다.

`MySQL 8.0`에서는 `innodb_parallel_read_threads` 시스템 변수를 이용해 하나의 쿼리를 최대 몇 개의

**스레드를 이용해서 처리**할지 `변경`이 `가능`하다.

8.0 버전에서는 `WHERE` 조건 없이 **단순히 테이블의 전체 건수를 가져오는 쿼리**만 `병렬`로 `처리`할 수 있다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/ad35b0ff-0041-4d5e-9a6f-f56d4345d7e8)

`병렬 처리용 스레드` 개수가 늘어날수록 `쿼리 처리`에 걸리는 시간이 줄어드는 것을 확인할 수 있다.

`병렬 처리용 스레드`의 개수가 서버에 장착된 `CPU 코어 개수`를 넘게되면 **성능이 떨어질 수 있으니 유의**하자.

### 👿 ORDER BY 처리(Using filesort)

- `레코드` 1~2건을 가져오는 쿼리를 제외하면 대부분의 `SELECT 쿼리`는 정렬을 필수적으로 사용한다.
- `정렬`을 처리하는 방법은 인덱스를 이용하는 방법과 `쿼리`가 실행될 때 `“Filesort”`로 별도 처리하는 방법이 있다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/3cb99891-7a6a-4e1f-8161-8d25b8ddfff1)

`레코드`를 정렬하기 위해 항상 `Filesort` 정렬 작업을 거쳐야 하는 것은 아니다.

아래와 같은 이유로 모든 `정렬`을 `인덱스`를 이용하도록 **튜닝하기란 거의 불가능**하다.

- `정렬 기준`이 너무 많아서 요건별로 모두 `인덱스`를 생성하는 것이 `불가능`한 경우
- `GROUP BY`의 결과 또는 `DISTINCT` 같은 처리의 결과를 `정렬`해야 하는 경우
- `UNION`의 결과와 같이 `임시 테이블`의 결과를 다시 `정렬`해야 하는 경우
- `랜덤`하게 결과 레코드를 가져와야 하는 경우

`인덱스`를 이용하지 않고 별도의 `정렬 처리`를 `수행`했는지는 `실행 계획`을 통해 확인할 수 있는데,

`Extra 컬럼`에 `“Using filesort”` 메시지가 있는지 여부로 판단할 수 있다.

### 🥱 소트 버퍼

- `MySQL`은 `정렬`을 `수행`하기 위해 **별도의 메모리 공간을 할당받아 사용**한다.
- `정렬`을 수행하기 위해 할당 받은 `메모리 공간`을 `Sort Buffer` 라고 한다.

`소트 버퍼`는 `정렬`이 필요한 경우에만 할당된다.

`버퍼`의 `크기`는 **정렬해야 할 레코드의 크기에 따라 가변적으로 증가**하지만 `최대 사용`한 가능한 공간은

`sort_buffer_size` 시스템 변수로 설정이 가능하다.

**정렬의 문제점**

`정렬`해야 할 `레코드`가 아주 `소량`이라면, **소트 버퍼만으로 아주 빠르게 정렬**할 수 있다.

하지만 `정렬`해야 할 `레코드`의 건수가 `소트 버퍼`로 할당된 공간보다 클 경우 문제가 생긴다.

정렬해야 할 `레코드`를 여러 조각으로 나눠서 처리하기 위해 `임시 저장`을 위한 **디스크 I/O가 발생**하기 때문이다.

`메모리`의 `소트 버퍼 과정`을 살펴보면 아래와 같다.

1. 메모리의 `소트 버퍼`에서 `정렬`을 `수행`하고 그 결과를 임시로 `디스크`에 `기록`한다.
2. `레코드`를 가져와서 다시 `정렬`해서 반복적으로 `디스크`에 임시 저장한다.
3. 각 `버퍼 크기`만큼 정렬된 레코드를 다시 `병합`하면서 `정렬`을 `수행`한다.

여기서 일어나는 `병합 작업`을 `Multi-merge`라고 `표현`한다.

수행된 `Multi-merge` 횟수는 `Sort_merge_passes` 상태변수에 누적되서 집계된다.

`소트 버퍼`를 크게 설정하면 디스크를 사용하지 않아서 더 빨라질 거 같지만, 

실제 `벤치마크 결과`로는 큰 차이를 보이지 않는다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/81e8c4ac-c132-4a73-be1f-da2678d152ab)

더 큰 `sort_buffer_size`에서 성능이 더 빠르다는 이야기도 있지만,

어떤 `데이터`를 정렬하는지와 테스트하는 서버의 `메모리`나 `디스크`의 `특성`에 따라 `결과`가 달라질 수 있다.

`sort_buffer_size` 시스템 변수의 설정값이 무조건 크면 `메모리`에서 모두 처리되니 빨라질 것으로 예상하면

이건 아주 큰 착각이다.

특히 `리눅스` 계열의 운영체제에서는 `큰 메모리 공간 할당`으로 인해 **성능이 훨씬 떨어질 수 있다.**

`MySQL`은 글로벌 메모리 영역과 `세션(로컬) 메모리 영역`이 나뉘어 있다.

`정렬`을 위해 할당하는 `소트 버퍼`는 **세션 메모리 영역에 해당**하게 된다.

`소트 버퍼`는 여러 클라이언트가 공유해서 **사용할 수 있는 영역**이 아니다.

`커넥션`이 많으면 많을수록, `정렬 작업`이 많으면 많을수록 `소트 버퍼`로 소비되는 `메모리 공간`이 커진다.

`소트 버퍼`의 크기를 `10MB` 이상으로 설정하면 `대량`의 `레코드`를 정렬하는 `쿼리`가 여러 `커넥션`에서

동시다발적으로 실행되면 `메모리 부족 현상`을 겪을 수도 있다.

`운영체제`는 메모리가 부족할경우 `OOM-Killer`가 여유 메모리를 확보하기 위해 `프로세스`를 강제 종료하는데,

`OOM-Killer`는 메모리를 가장 많이 사용하는 `프로세스`를 `종료`하므로 `MySQL 서버`가 1순위가 된다.

### 🥱 정렬 알고리즘

- `레코드`를 정렬할 때 `레코드` 전체를 `소트 버퍼`에 담을지 또는 정렬 기준 컬럼만 `소트 버퍼`에 담을지에 따라 2가지 정렬 모드로 나뉜다.
- `Single-Pass`와 `Two-Pass` 모드가 있는데, 공식적인 명칭은 아니다.
- `정렬`을 수행하는 `쿼리`가 어떤 정렬 모드를 사용하는지는 `옵티마이저 트레이스` 기능으로 확인할 수 있다.

```sql
-- // 옵티마이저 트레이스 활성화
mysql> SET OPTIMIZER_TRACE="enabled=on", END_MARKERS_IN_JSON=on;
mysql> SET OPTIMIZER_TRACE_MAX_MEM_SIZE=1000000;

-- // 쿼리 실행
mysql> SELECT * FROM employees ORDER BY last_name LIMIT 1000000, 1;

-- // 트레이스 내용 확인
mysql> SELECT * FROM INFORMATION_SCHEMA.OPTIMIZER_TRACE \G;
```

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/a6d41a15-73d8-4a2e-91f2-7c6581d0c259)

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/87fcfd0b-bb73-495a-838f-2d4003e50762)

MySQL 서버의 `정렬 방식`은 3가지 방식이 있다.

- `<sort_key, rowid>` : 정렬 키와 레코드의 `ROWID`만 가져와서 정렬하는 방식
- `<sort_key, additional_fields>` : 정렬 키와 레코드 전체를 가져와서 정렬하는 방식으로, 레코드의 컬럼들은 고정 사이즈로 메모리 저장
- `<sort_key, packed_additional_fields>` : 정렬 키와 레코드 전체를 가져와서 정렬하는 방식으로, 레코드의 컬럼들은 **가변 사이즈로 메모리 지정**

`5.7` 버전부터 `세 번째 방식`이 도입되었는데, 정렬을 위한 `메모리 공간`의 **효율적인 사용**을 위해서 도입된 방식이다.

`옵티마이저 트레이스` 기능을 쓰다보면, 가끔 `다른 형태`로 나오는 경우가 있어서 

책에 나오지 않은 내용을 정리하면 아래와 같다.

- `varlen_sort_key`
    - 가변길이 정렬 키를 사용하는 `인덱스 타입`이다.
    - `VARCHAR, TEXT` 등 가변 길이 문자열 컬럼에 대한 인덱스에서 사용된다.
- `fixed_sort_key`
    - 고정 길이 정렬 키를 사용하는 `인덱스 타입`이다.
    - `CHAR, INT` 등 고정 길이 타입 컬럼에 대한 인덱스에서 사용된다.

- `packed_additional_fields`
    - 추가적인 필드를 압축하여 저장하는 방식이다.
    - 보다 `효율적인 저장`을 위해 여러 개의 추가 필드를 한 번에 `압축`하여 `저장`한다.
- `additional_fields`
    - 추가적인 `필드`를 별도로 `저장`하는 방식이다.
    - 각 추가 필드에 대해 별도의 `저장 공간`을 사용한다.
    - 일반적으로 `packed_additional_fields` 보다 저장 공간을 더 많이 차지한다.
    - 추가 필드에 대한 엑세스는 `packed_additional_fields` 보다 빠를 수 있다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/03d3fe33-6144-476c-8eb4-dc9af8135d05)

### 😒 싱글 패스 정렬 방식

- `소트 버퍼`에 정렬 기준 컬럼을 포함해 `SELECT` 대상이 되는 **컬럼 전부를 담아서 정렬을 수행하는 정렬 방법**이다.

```sql
mysql> SELECT emp_no, first_name, last_name
       FROM employees
       ORDER BY first_name;
```

`first_name`으로 정렬해서 `emp_no`, `first_name`, `last_name`을 SELECT 하는 쿼리를 

**싱글 패스 정렬 방식으로 처리하는 절차**를 그림으로 보면 아래와 같다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/6d1cf720-bcbc-4b7c-a0d2-c8e07986eec4)

처음 `employees` 테이블을 읽을 때 정렬에 필요하지 않은 `last_name` 컬럼까지 전부 읽어서

**소트 버퍼에 담고 정렬을 수행**하는 것을 볼 수 있다.

### 😃 투 패스 정렬 방식

- `정렬 대상 컬럼`과 `PK` 값만 **소트 버퍼에 담아서 정렬**을 수행한다.
- 정렬된 순서대로 다시 `PK`로 테이블을 읽어서 `SELECT` 할 컬럼을 가져오는 정렬 방식이다.
- `싱글 패스 정렬 방식`이 도입되기 전부터 사용하던 방식이다.

`MySQL 8.0` 버전에서도 특정 조건에서는 `투 패스 정렬 방식`을 사용한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/55cf66e0-7265-4eaf-9419-228da13665b6)

`투 패스 방식의 정렬`이 어떻게 수행되는지 순서대로 알아보자.

1. `정렬`에 필요한 `first_name` 컬럼과 PK인 `emp_no` 만 읽어서 **정렬을 수행**한다.
2. `정렬`이 완료되면 그 결과 순서대로 `employees 테이블`을 한 번 더 읽어서 `last_name`을 가져온다.
3. 최종적으로 `결과`를 클라이언트에 넘긴다.

`투 패스 정렬` 방식은 **테이블을 두 번 읽어야 하기 때문에 상당히 불합리**하다.

`싱글 패스 정렬` 방식은 이러한 불합리가 없지만, **더 많은 소트 버퍼 공간이 필요하다는 단점**이 있다.

예를 들어, `128KB`의 정렬 버퍼를 사용한다면 이 쿼리는 `투 패스 정렬` 방식에서 7,000건의 

레코드를 정렬할 수 있지만 **싱글 패스 방식은 절반 정도밖에 정렬**할 수 없다.

최신 버전의 `MySQL`에서 특정한 조건에는 `투 패스 정렬` 방식을 사용한다.

- 레코드의 크기가 `max_length_for_sort_data` 시스템 변수 값보다 클 때
- `BLOB`이나 `TEXT` 타입의 컬럼이 **SELECT 대상에 포함**될 때

`싱글 패스 방식`은 정렬 대상 **레코드의 크기나 건수가 작을 때** 빠른 성능을 보이고,

`투 패스 방식`은 **정렬 대상 레코드의 크기나 건수가 상당히 많을 때** 효율적이라고 볼 수 있다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/6d3112d9-49e2-44d8-9b55-603062f978ca)

### 🫡 정렬 처리 방법

- `쿼리`에 `ORDER BY`가 사용되면 반드시 3가지 처리 방법 중 하나로 `정렬`이 처리된다.
- `아래쪽`으로 갈 수록 **처리 속도가 떨어진다.**

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/ee0f4759-b00f-4c87-9a83-1a7b5983d946)

`옵티마이저`는 정렬 처리를 위해 어떤 판단을 할까?

`옵티마이저`는 가장 먼저 `인덱스`를 정렬처리를 위해 이용할 수 있는지 확인한다.

이용할 수 있다면 `Filesort` 과정 없이 **인덱스를 순서대로 읽어서 반환**한다.

`인덱스`를 사용할 수 없을 경우 `WHERE 조건`에 일치하는 레코드를 검색해 `정렬 버퍼`에 `저장`하면서

정렬을 `처리(Filesort)` 한다.

`옵티마이저`는 정렬 대상 레코드를 최소화하기 위해서 아래 **2가지 방법 중 하나를 선택한**다.

- `조인`의 **드라이빙 테이블만 정렬**한 다음 조인을 수행
- `조인`이 끝나고 **일치하는 레코드를 모두 가져온 후 정렬**을 수행

일반적으로 `조인`이 수행되면서, `레코드 건수`와 `레코드의 크기`는 거의 배수로 불어나게 된다.

따라서, **첫 번째 방법이 훨씬 효율적인 방법**이다.

### 🫠 인덱스를 이용한 정렬

- `인덱스`를 이용한 정렬을 사용하려면 반드시 `ORDER BY`에 명시된 `컬럼`이 제일 먼저 읽은 테이블에 속하고, `ORDER BY`의 순서대로 생성된 인덱스가 있어야 한다.
- `WHERE 절`에 첫 번째로 읽는 테이블의 `컬럼`에 대한 `조건`이 있다면 그 조건과 `ORDER BY`는 **같은 인덱스를 사용**할 수 있어야 한다.
- `해시 인덱스`나 `전문 검색 인덱스` 등에서는 **인덱스를 이용한 정렬을 사용**할 수 없다.

`R-Tree`도 `B-Tree` 계열이지만 특성상 **인덱스를 이용한 정렬**을 사용할 수 없다.

여러 테이블이 `조인`되는 경우 `Nested-loop` 방식의 `조인`에서만 이 방식을 사용할 수 있다.

`MySQL 엔진`에서는 별도의 정렬을 위한 `추가 작업`을 수행하지 않는다.

`ORDER BY`가 있든 없든 같은 `인덱스`를 **레인지 스캔해서 나온 결과는 같은 순서로 출력**된다.

```sql
mysql> SELECT * 
       FROM employees e, salaries s
       WHERE s.emp_no=e.emp_no
         AND s.emp_no BETWEEN 10002 AND 100020
       ORDER BY e.emp_no;
       

-- emp_no 컬럼으로 정렬이 필요한데, 인덱스를 사용하면서 자동으로 정렬이 된다고
-- 일부러 ORDER BY emp_no를 제거하는 것은 좋지 않은 선택이다.
mysql> SELECT *
       FROM employees e, salaries s
       WHERE s.emp_no=e.emp_no
         AND s.emp_no BETWEEN 100002 AND 100020;
```

`ORDER BY` 절이 없어도 정렬이 되는 이유는, `employees` 테이블의 `PK`를 읽고 `salaries` 테이블을 조인했기 때문이다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/222d1500-b056-47f4-b434-08b60a521787)

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/0746fd91-530c-449a-bc41-5d9effb01ffb)

`인덱스`를 사용한 정렬이 가능한 이유는 `B-Tree 인덱스`가 **키 값으로 정렬**되어 있기 때문이다.

조인이 `Nested-loop` 방식으로 실행되기 때문에 `드라이빙 테이블`의 인덱스 읽기 순서가 흐트러지지 않는다.

하지만, `조인`이 사용된 `쿼리`의 실행 계획에 `조인 버퍼`가 사용되면 **순서가 흐트러질 수 있다는 것에 주의**하자.

### 😜 조인의 드라이빙 테이블만 정렬

- `조인`이 수행되면 일반적으로 `결과 레코드`의 건수가 `몇 배`로 불어나고, `레코드`의 `크기`도 늘어난다.
- `조인`을 실행하기 전에 `첫 번째 테이블`의 **레코드를 정렬하고 조인을 실행**하는 것이 `차선책`이 될 수 있다.
- 첫 번째로 읽히는 테이블의 `컬럼`만으로 `ORDER BY 절`을 `작성`해야 **위의 방식으로 정렬이 수행**된다.

```sql
mysql> SELECT * 
       FROM employees e, salaries s
       WHERE s.emp_no=e.emp_no
         AND e.emp_no BETWEEN 100002 AND 100010
       ORDER BY e.last_name;
```

위와 같은 `쿼리`를 `수행`할 때, `WHERE 절`이 2가지 조건을 갖추고 있으므로, `옵티마이저`는 `employees` 테이블을 **드라이빙 테이블로 선택**한다.

- `WHERE` 절의 검색 조건은 `employees` 테이블의 `PK`를 이용해 검색하면 `작업량`을 줄일 수 있다.
- `드리븐 테이블(salaries`)의 조인 컬럼인 `emp_no` 컬럼에는 인덱스가 있다.

검색은 `Index Range Scan`으로 처리할 수 있지만, `ORDER BY` 절에 명시된 컬럼은 `employees` 테이블의

`PK`와 아무 연관이 없으므로 `인덱스`를 이용한 `정렬`은 `불가능`하다.

`ORDER BY` 절의 정렬 기준 컬럼이 `드라이빙 테이블(employees)`에 포함된 `컬럼`이라는 것을 알 수 있다.

`옵티마이저`는 드라이빙 테이블만 검색해서 `정렬`을 `수행`하고, 그 결과와 `salaries` 테이블을 조인한 것이다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/a03db3f9-2c9e-4314-9c4e-4a78f4b3076f)

위의 `쿼리`가 어떤 과정에 의해 `실행`되는지 확인해보자.

1. 인덱스를 이용해 `“emp_no BETWEEN 100001 AND 100010”` 조건을 만족하는 9건 검색
2. 검색 결과를 `last_name` 컬럼으로 정렬을 수행(`Filesort`)
3. 정렬된 결과를 순서대로 읽으면서 `salaries 테이블`과 조인을 수행해 `86건`의 `결과`를 가져온다.

### 😨 임시 테이블을 이용한 정렬

- `쿼리`가 여러 테이블을 `조인`하지 않고, 하나의 테이블로 `SELECT`해서 정렬하는 경우라면 `임시 테이블`이 필요하지 않다.
- `2개` 이상의 `테이블`을 `조인`해서 결과를 정렬해야 하는 경우 `임시 테이블`이 `필요`할 수 있다.

`조인`의 드라이빙 테이블만 정렬하는 예시는 `2개` 이상의 테이블이 `조인`되지만 `임시 테이블`을 사용하지 않는다.

그 외 `패턴`의 `쿼리`에서는 항상 조인의 결과를 `임시 테이블`에 저장하고 `결과`를 `정렬`하는 과정을 거치게된다.

이 방법은 `정렬`의 3가지 방법 가운데 `정렬`해야 할 레코드 건수가 가장 많기 때문에 `가장 느린 방법`이다.

```sql
mysql> SELECT * 
       FROM
       WHERE s.emp_no=e.emp_no
         AND e.emp_no BETWEEN 100002 AND 100010
       ORDER BY s.salary;
```

위의 쿼리는 `ORDER BY` 절의 정렬 기준 `컬럼`이 드라이빙 테이블이 아니 `드리븐 테이블`에 있다.

`정렬`이 `수행`되기 전에 `salaries 테이블`을 읽어야 하므로 이 쿼리는 **조인된 데이터를 가지고 정렬**해야 한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/f72a4607-773a-4337-a5cc-c2465d424cd9)

이 쿼리의 실행계획을 살펴보면, `Using Temporary; Using filesort`라는 코멘트가 있다.

`조인`의 결과를 `임시 테이블`에 저장하고 그 결과를 다시 `정렬처리` 했음을 의미한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/a68023f1-631c-4cef-b122-3d2e144e6938)

### 😀 정렬 처리 방법의 성능 비교

- 일반적으로 `LIMIT`은 `테이블`이나 처리 결과의 `일부`만 가져오기 때문에 **서버가 처리해야 할 작업량을 줄이는 역할**을 한다.
- `ORDER BY`나 `GROUP BY` 같은 작업은 `WHERE 조건`을 만족하는 레코드를 `LIMIT 건수` 만큼만 가져와서는 처리할 수 없다.

`ORDER BY`나 `GROUP BY`는 **조건을 만족하는 모든 레코드를 가져와서 정렬**을 수행하거나 `그루핑 작업`을 

실행해야만 `LIMIT`으로 건수를 제한할 수 있기 때문이다.

`WHERE 조건`이 아무리 인덱스를 잘 활용하도록 `튜닝`해도 잘못된 `ORDER BY`나 `GROUP BY`로 인해

쿼리가 느려지는 경우가 자주 발생한다.

### 👿 스트리밍 방식

- `서버` 쪽에서 처리할 `데이터`가 얼마인지에 관계없이 조건에 일치하는 `레코드`가 검색될때마다 **바로바로 클라이언트로 전송**해주는 방식이다.
- `클라이언트`는 쿼리를 요청하고 첫 번째 `레코드`를 바로 `전달` 받는다.
- 가장 `마지막 레코드`는 언제 받을지 알 수 없다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/1b943f60-3b88-4502-9245-448c25ad1538)

`쿼리`가 `스트리밍 방식`으로 처리가 가능하다면, 클라이언트는 `MySQL` 서버로부터 일치하는 레코드를 

찾는 즉시 전달받아서 `데이터 가공 작업`이 가능하다.

웹서비스 같은 `OLTP` 환경에서는 쿼리의 `요청`에서부터 첫 번째 레코드를 전달받게 되기까지의

`응답 시간`이 매우 중요하다.

따라서, `스트리밍 방식`으로 처리되는 `쿼리`는 얼마나 많은 레코드를 `조회`하느냐에 상관없이 빠른 `응답시간`을 보장해준다.

### 🥵 버퍼링 방식

- `ORDER BY`나 `GROUP BY`를 사용하면 **쿼리의 스트리밍 처리가 불가능**하다.
- `WHERE 조건`에 일치하는 모든 레코드를 가져온 후, `정렬`하거나 `그루핑`해야 하기 때문이다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/d9906cf8-8a46-4420-b265-c8dc83c669d3)

`버퍼링 방식`으로 처리되는 쿼리는 결과를 모아서 `MySQL 서버`에서 일괄 가공해야 한다.

`LIMIT`으로 결과 건수를 제한하는 조건이 있어도 `성능 향상`에 별로 도움이 되지 않는다.

`LIMIT`은 작업할 레코드의 건수를 줄이지 못하고, **네트워크로 전송되는 레코드의 건수만 줄어들 뿐**이다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/8348da77-0ce1-4f38-8854-d0d4f51859cf)

`ORDER BY`를 처리할 때 `인덱스`를 사용한 `정렬 방식`만 **스트리밍 처리가 가능**하다.

`인덱스`를 사용한 정렬 방식은 `LIMIT`으로 제한된 건수만큼만 읽으면서 `클라이언트`로 결과를 바로 `전송`해준다.

`인덱스`를 사용하지 못하는 경우의 처리는 필요한 모든 `레코드`를 `디스크`로부터 읽은 후 `정렬`한 뒤

`LIMIT`으로 제한된 건수만큼 잘라서 `클라이언트`로 `전송`해줄 수 있다.

```sql
mysql> SELECT * 
       FROM tb_test1 t1, tb_test2 t2
       WHERE t1.col1=t2.col1
       ORDER BY t1.col2
       LIMIT 10;
```

`tb_test1` 테이블의 레코드는 **100건**, `tb_test2` 테이블의 레코드를 **1000건**이라고 가정할때

`정렬 처리 방법`별로 몇 개의 레코드를 읽어야 하고 몇 건의 `정렬`이 수행될까?

- `tb_test1` 이 드라이빙되는 경우

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/db05bf42-1245-4292-a4f4-65e421bb38b0)

- `tb_test2`가 드라이빙되는 경우

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/9f41f400-ffa1-4330-90cf-bcfc309d54cc)

어느 `테이블`이 먼저 `드라이빙`되어 조인되는지도 중요하지만, 어떤 `정렬 방식`으로 `처리`되는지가 더 `중요`하다.

가능하다면 `인덱스`를 사용한 정렬로 `유도`하고, 불가능하다면 최소한 `드라이빙 테이블`만 정렬해도 되는 수준으로

유도하는 것도 `좋은 튜닝 방법`이다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/a0ee9b27-9325-4001-ad39-86e6a2b67104)

### 🫠 정렬 관련 상태 변수

- MySQL `서버`는 처리하는 `주요 작업`에 대해서는 해당 **작업의 실행 횟수를 상태 변수로 저장**한다.
- `정렬`과 관련해서 지금까지 몇 건의 `레코드`나 `정렬 처리`를 수행했는지, `멀티 머지`는 몇 번 `발생`했는지 등을 알 수 있다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/220e18dc-7210-4042-837a-2b020eafe1f5)

각 `상태 값`은 아래와 같은 의미가 있다.

- `Sort_merge_passes` : 멀티 머지 처리 횟수를 의미
- `Sort_range` : Index Range Scan을 통해 검색된 결과에 대한 정렬 작업 횟수
- `Sort_scan` : Full Table Scan을 통해 검색된 결과에 대한 정렬 작업 횟수
- `Sort_rows` : 지금까지 정렬한 전체 레코드 건수

### 🥱 GROUP BY 처리

- `GROUP BY` 또한 `ORDER BY`와 같이 쿼리가 `스트리밍`된 처리를 할 수 없게 한다.
- `GROUP BY` 절이 있는 `쿼리`에서는 `HAVING` 절을 사용할 수 있는데, `HAVING` 절은 `GROUP BY` 결과에 대한 `필터링` 역할을 한다.
- `GROUP BY`에 사용된 조건은 `인덱스`를 사용해서 처리될 수 없다.

`GROUP BY` 작업도 인덱스를 사용하는 경우와 그렇지 않은 경우로 나눌 수 있다.

**인덱스를 사용하는 경우**

- `인덱스 스캔` : 인덱스를 차례대로 읽는 스캔 방법
- `루스 인덱스 스캔` : 인덱스를 건너뛰면서 읽는 스캔 방법

**인덱스를 사용하지 못하는 경우**

- `임시 테이블 사용` : 인덱스를 사용하지 못하는 쿼리의 `GROUP BY` 작업은 임시 테이블을 이용한다.

### 🤫 인덱스 스캔을 이용하는 GROUP BY(타이트 인덱스 스캔)

- `조인`의 **드라이빙 테이블에 속한 컬럼만 이용해 그루핑**할 때 `GROUP BY` 컬럼에 인덱스가 존재한다면 그 `인덱스`를 차례대로 읽으면서 `그루핑 작업`을 수행하고 `조인`을 `처리`한다.
- `GROUP BY`가 인덱스를 사용해서 처리된다 하더라도 `그룹함수` 등의 그룹 값을 처리해야 한다면 `임시 테이블`이 필요할 수 있다.
- `GROUP BY`가 인덱스를 통해 처리되는 `쿼리`라면 이미 정렬된 `인덱스`를 읽는 것이므로 추가적인 `정렬 작업`이나 내부 `임시 테이블`은 필요하지 않다.

이런 `그루핑` 방식을 사용하는 쿼리의 실행 계획에서는 `Extra 컬럼`에 아래와 같은 메시지가 표시되지 않는다.

- `GROUP BY` 관련 코멘트 : Using Index for group-by
- `임시 테이블` 관련 코멘트 : Using temporary
- `정렬` 관련 코멘트 : Using filesort

### 😗 루스 인덱스 스캔을 이용하는 GROUP BY

- `루스 인덱스 스캔` 방식은 인덱스의 `레코드`를 건너뛰면서 필요한 부분만 읽어오는 것을 말한다.
- `루스 인덱스 스캔`을 사용하면 실행 계획의 `Extra 컬럼`에 **“Using index for group-by”** 코멘트가 표시된다.

```sql
mysql> EXPLAIN
        SELECT emp_no
        FROM salaries
        WHERE from_date='1985-03-01'
        GROUP BY emp_no;
```

`salaries` 테이블의 인덱스는 `(emp_no, from_date)`로 생성되어 있으므로

`WHERE` 조건은 `Index Range Scan` 방식으로 이용할 수 없는 쿼리다.

이 쿼리의 `실행 계획`은 아래와 같이 `Index Range Scan`을 이용했고,

`Extra 컬럼`의 메시지를 보면 `GROUP BY` 처리까지 **인덱스를 사용**했다는 것을 알 수 있다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/cb98565a-7727-47a3-a144-2ff0497ef347)

`MySQL 서버`가 이 쿼리를 어떻게 `실행`했는지 순서대로 보면 아래와 같다.

1. `(emp_no, from_date)` 인덱스를 차례대로 스캔하면서 `emp_no`의 첫 번째 유일한 값(그룹 키) `“10001”`을 찾아낸다.
2. `(emp_no, from_date)` 인덱스에서 `emp_no`가 “10001”인 것 중에서 `from_date` 값이 `‘1985-03-01’`인 레코드만 가져온다. 이 검색 방법은 1번 단계에서 알아낸 `‘10001’` 값과 쿼리의 WHERE 절에 사용된 `“from_date=’1985-03-01’”` 조건을 합쳐서 **“emp_no=10001 AND from_date=’1985-03-01’”** 조건으로 `(emp_no, from_date)` 인덱스를 검색하는 것과 거의 흡사하다.
3. `(emp_no, from_date)` 인덱스에서 `emp_no`의 그 다음 `유니크한(그룹 키)값`을 가져온다.
4. 3번 단계에서 **결과가 더 없으면 처리를 종료**하고, 결과가 있다면 2번 과정으로 돌아가서 반복 수행한다.

`MySQL`의 루스 인덱스 스캔 방식은 단일 테이블에 대해 수행되는 `GROUP BY` 처리에만 

사용할 수 있다.

`Prefix Index`(컬럼 값의 앞쪽 일부만으로 생성된 인덱스)는 `루스 인덱스 스캔`을 사용할 수 없다.

`Index Range Scan`에서는 유니크한 값의 수가 많을 수록 성능이 향상되고, 

`루스 인덱스 스캔`에서는 인덱스의 유니크한 값의 수가 적을 수록 성능이 향상된다.

`루스 인덱스 스캔`은 분포도가 좋지 않은 `인덱스`일수록 더 빠른 결과를 만들어낸다.

`루스 인덱스 스캔`으로 처리되는 쿼리는 `임시 테이블`이 필요하지 않다.

`루스 인덱스 스캔`이 사용될 수 있을지 없을지 판단하는 것은 꽤 어려운 일이다.

여러 패턴의 쿼리를 살펴보고 `루스 인덱스 스캔`을 사용할 수 있는지 없는지 판별하는 연습이 필요하다.

`(col1, col2, col3)` 컬럼으로 생성된 `tb_test` 테이블을 생성했다고 가정해보겠다.

아래는 `루스 인덱스 스캔`을 사용할 수 있는 `쿼리`다.

```sql
SELECT col1, col2 FROM tb_best GROUP BY co1, col2;
SELECT DISTINCT col1, col2 FROM tb_best;
SELECT col1, MIN(col2) FROM tb_best GROUP BY col1;
SELECT col1, col2 FROM tb_best WHERE col1 < const GROUP BY col1, col2;
SELECT MAX(col3), MIN(col3), col1, col2 FROM tb_best WHERE col2 > const GROUP BY col1, col2;
SELECT col2 FROM tb_best WHERE col1 < const GROUP BY col1, col2;
SELECT col1, col2 FROM tb_best WHERE col3 = const GROUP BY col1, col2;
```

아래의 쿼리는 `루스 인덱스 스캔`을 사용할 수 없는 쿼리다.

```sql
--// MIN()과 MAX() 이외의 집합 함수가 사용됐기 때문에 루스 인덱스 스캔은 사용 불가
SELECT col1, SUM(col2) FROM tb_best GROUP BY col1;

-- // GROUP BY에 사용된 컬럼이 인덱스 구성 컬럼의 왼쪽부터 일치하지 않기 때문에 사용 불가
SELECT col1, col2 FROM tb_best GROUP BY col2, col3;

-- // SELECT 절의 컬럼이 GROUP BY와 일치하지 않기 때문에 사용 불가
SELECT col1, col3 FROM tb_test GROUP BY col1, col2;
```

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/91037dda-d6fe-4b1d-8834-be5e2178a9bf)

### 😏 임시 테이블을 사용하는 GROUP BY

`GROUP BY` 기준 컬럼이 `드라이빙 테이블`에 있든 `드리븐 테이블`에 있든 관계없이 인덱스를 전혀 사용하지 못하면 **아래와 같은 방식**으로 처리된다.

```sql
mysql> EXPLAIN 
        SELECT e.last_name, AVG(s.salary)
        FROM employees e, salaries s
        WHERE s.emp_no=e.emp_no
        GROUP BY e.last_name;
```

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/c507bff3-7215-4f4b-82de-2c66ed747d80)

`실행 계획`을 살펴보면, `Extra` 컬럼에 `“Using temporary”` 메시지가 표시된다.

임시 테이블이 사용된 이유는 `employees` 테이블을 `풀 스캔`하기 때문이 아니라 인덱스를 전혀

사용할 수 없는 `GROUP BY`이기 때문이다.

여기서 주의 깊게 살펴봐야 할 부분이 있다. `Extra` 컬럼에 `“Using filesort”`는 표시되지 않았다는 점이다.

`8.0` 이전 버전까지는 `GROUP BY`가 사용된 쿼리는 **그루핑되는 컬럼을 기준으로 묵시적인 정렬**까지 함께 수행했다.

`GROUP BY`는 있지만 `ORDER BY` 절이 없는 **쿼리에 대해서 기본적으로 정렬된 결과를 반환**했다.

`8.0` 버전부터는 `GROUP BY`가 필요한 경우 내부적으로 `GROUP BY` 절의 컬럼들로 구성된

유니크 인덱스를 가진 `임시 테이블`을 만들어서 **중복 제거와 집합 함수 연산을 수행**한다.

`임시 테이블` 생성 후 조인의 결과를 한 건씩 가져와서 `임시 테이블`에서 중복 체크를 하고

`INSERT` 또는 `UPDATE`를 실행하는 방식으로 작업이 수행된다.

```sql
-- 임시 테이블 DDL
CREATE TEMPORARY TABLE (
   last_name VARCHAR(16),
   salary INT,
   UNIQUE INDEX ux_lastname (last_name)
);
```

하지만 8.0 버전에서도 `GROUP BY`와 `ORDER BY`가 같이 사용되면 명시적으로 `정렬 작업`을 수행한다.

```sql
mysql> EXPLAIN 
        SELECT e.last_name, AVG(s.salary)
        FROM employees e, salaries s
        WHERE s.emp_no=e.emp_no
        GROUP BY e.last_name
        ORDER BY e.last_name;
```

`ORDER BY` 절만 추가한 예제인데 `Extra` 컬럼에 `“Using filesort”`가 표시된 것을 볼 수 있다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/13703834-96ac-407a-bf0e-0592f998bb4a)

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/1dc9e0da-3a57-430b-b499-5a0b68074e95)

### 🫵🏼 DISTINCT 처리

- `특정 컬럼`의 **유니크한 값만 조회**하려면 `DISTINCT`를 사용하면 된다.
- `DISTINCT`가 사용되는 쿼리의 실행 계획에서 `DISTINCT` 처리가 인덱스를 사용하지 못하면 **항상 임시 테이블이 필요**하다.
- 하지만 `실행 계획`에서는 임시 테이블을 사용하더라도 `“Using temporary”` 메시지가 표시되지 않는다.

### 👽 SELECT DISTINCT …

- 단순히 `SELECT` 레코드 중에서 **유니크한 레코드**만 가져오려면 `SELECT DISTINCT …` 형태의 쿼리를 사용한다.
- `SELECT DISTINCT …` 형태의 쿼리는 **GROUP BY와 동일한 방식으로 처리**된다.

```sql
mysql> SELECT DISTINCT emp_no FROM salaries;
mysql> SELECT emp_no FROM salaries GROUP BY emp_no;
```

위의 쿼리는 `8.0` 버전부터 **내부적으로 같은 작업을 수행**한다.

`DISTINCT`는 **SELECT하는 레코드를 유니크하게 가져오는 것**일 뿐, `특정 컬럼`만 유니크하게

조회하는 것이 `아니라는 점`에 유의하자.

```sql
mysql> SELECT DISTINCT first_name, last_name FROM employees;
```

이 쿼리는 `first_name`만 유니크한 것을 가져오는 것이 아니라,

`(first_name, last_name)` 조합 전체가 **유니크한 레코드를 가져오는 것**이다.

```sql
mysql> SELECT DISTINCT(first_name), last_name FROM employees;
```

이 쿼리는 `에러`가 날 것 같지만 `에러`가 발생하지 않는다.

`MySQL` 서버는 `DISTINCT` 뒤의 괄호를 **의미 없이 사용된 것으로 보고 알아서 제거**하기 때문이다.

이 쿼리는 `first_name`만 유니크하게 조회하고 `last_name`은 `DISTINCT` 없이 동작할 것 같지만

괄호가 제거되면서 `(first_name, last_name)` 쌍이 **유니크한 레코드를 가져온다.**

### 😫 집합 함수와 함께 사용된 DISTINCT

- `COUNT()`, `MIN()`, `MAX()` 같은 집합 함수 내에서 **DISTINCT 키워드가 사용**될 수 있다.
- `집합 함수`를 사용하면 `SELECT DISTINCT`와 다른 형태로 해석된다.
- `집합 함수` 내에서 사용된 `DISTINCT`는 집합 함수의 인자로 전달된 **컬럼 값이 유니크한 것들만 가져온다.**

```sql
mysql> EXPLAIN SELECT COUNT(DISTINCT s.salary)
       FROM employees e, salaries s
       WHERE e.emp_no=s.emp_no
       AND e.emp_no BETWEEN 100001 AND 100100;
```

이 쿼리는 내부적으로 `“COUNT(DISTINCT s.salary)"`를 처리하기 위해 **임시 테이블을 사용**한다.

하지만 실행 계획에는 `“Using temporary”`가 표시되지 않는다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/faa73eb0-d63b-486e-a512-be810d43593b)

`employees` 테이블과 `salaries` 테이블을 조인한 결과에서 **salary 컬럼의 값만 저장**하기 위해 `임시 테이블`을 만들어 사용한다.

`임시 테이블`의 **salary 컬럼에는 유니크 인덱스가 생성**되므로 `레코드 건수`가 많아지면 `쿼리`가 상당히 느려질 수 있다.

그렇다면 아래의 예제는 어떨까?

```sql
mysql> SELECT COUNT(DISTINCT s.salary),
              COUNT(DISTINCT e.last_name)
       FROM employees e, salaries s
       WHERE e.emp_no=s.emp_no
         AND e.emp_no BETWEEN 100001 AND 100100;
```

`COUNT(DISTINCT …)`를 하나 더 추가해도 실행 계획은 같다.

하지만 각 `COUNT` 쿼리 하나마다 `임시 테이블`을 사용하므로 `2개`의 **임시 테이블을 사용**한다.

위의 쿼리는 `DISTINCT` 처리를 위해 **인덱스를 사용할 수 없어서 임시 테이블이 필요**하다.

아래의 `쿼리`들은 **인덱스가 있는 컬럼**에 `DISTINCT 처리`를 수행하므로 `임시 테이블` 없이 최적화 된 처리를 수행할 수 있다.

```sql
mysql> SELECT COUNT(DISTINCT emp_no) FROM employees;
mysql> SELECT COUNT(DISTINCT emp_no) FROM dept_emp GROUP BY dept_no;
```

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/02fb188e-d3c4-47d9-8573-edcc76005dcd)

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/a05151da-bb34-4bfe-8b07-d9e2daa08548)

### ✨ 내부 임시 테이블 활용

- `MySQL 엔진`이 **스토리지 엔진으로부터 받아온 레코드를 정렬하거나 그루핑**할 때는 `내부적인 임시 테이블`을 `사용`한다.
- `내부적인 임시 테이블`은 **CREATE TEMPORARY TABLE** 명령으로 만든 임시 테이블과는 다르다.

`MySQL 엔진`이 사용하는 임시 테이블은 처음에는 **메모리에 생성**되었다가 `테이블`의 `크기`가 커지면 **디스크로 옮겨지게 된다.**

특정 `예외 케이스`에는 메모리를 거치지 않고 바로 `디스크`에 임시 테이블이 만들어진다.

내부적인 가공을 위해 생성하는 `임시 테이블`은 다른 `세션`이나 다른 `쿼리`에서 볼 수 없고 사용도 불가능하다.

`내부적인 임시 테이블`은 사용자가 생성한 `임시 테이블`과 달리 **쿼리의 처리가 완료되면 자동으로 삭제**된다.

### 👶🏼 메모리 임시 테이블과 디스크 임시 테이블

- `8.0 이전` 버전까지는 `원본 테이블`의 스토리지 엔진과 관계없이 `임시 테이블`이 메모리를 사용할 때는 **MEMORY 스토리지 엔진**을 사용하고, `디스크 저장`시 **MyISAM 스토리지 엔진**을 이용했다.
- `8.0 버전`부터는 `메모리`는 **TempTable 스토리지 엔진**을 사용하고, `디스크에 저장`되는 임시 테이블은 **InnoDB 스토리지 엔진을 사용하도록 개선**되었다.

`MEMORY 스토리지 엔진`은 가변 길이 타입을 지원하지 못하기 때문에 `임시 테이블`이 메모리에 만들어지면 **가변 길이 타입은 최대 길이만큼 메모리를 할당해서 사용**해왔다.

`디스크`에 임시 테이블이 만들어질때 사용되는 `MyISAM 스토리지 엔진`은 **트랜잭션을 지원하지 않는다는 문제점**도 가지고 있었다.

따라서 `8.0 버전`부터 메모리 사용시 가변 타입을 지원하는 `TempTable`을 사용하고

`디스크 저장`시 트랜잭션 사용이 가능한 **InnoDB 스토리지 엔진이 사용되도록 개선**되었다.

`8.0 버전`부터는 `internal_tmp_mem_storage_engine` 변수로 메모리용 임시 테이블을

`MEMORY`와 `TempTable` 중에서 선택할 수 있도록 지원한다.

`TempTable`이 사용 가능한 메모리 공간의 최대치를 `temptable_max_ram` 시스템 변수로

제어할 수 있도록 `지원`한다.

`임시 테이블`의 크기가 **1GB보다 커지는 경우 메모리의 임시 테이블을 디스크로 기록**한다.

`MySQL 서버`는 **2가지 디스크 저장 방식 중 하나를 선택**하게 된다.

- `MMAP 파일`로 디스크에 기록
- `InnoDB 테이블`로 기록

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/ac86713b-e6ac-4c9d-9f4d-863172a1c5be)

`MySQL 서버`가 **MMAP 파일로 기록할지 InnoDB 테이블로 전환**할지는

`temptable_use_mmap` 시스템 변수로 설정할 수 있고 기본값은 `ON`이다.

`TempTable`을 `MMAP 파일`로 전환하는 것이 InnoDB 테이블로 전환하는 것보다 `오버헤드`가

적기 때문에 **temptable_use_maap 시스템 변수의 기본값이 ON으로 선택**된 것이다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/84753e15-413c-4ea3-a0e3-1e3afefe1035)

`내부 임시 테이블`이 메모리에 생성되지 않고 처음부터 `디스크 테이블`로 `생성`되는 경우도 있다.

`internal_tmp_disk_storage_engine` 시스템 변수에 설정된 `스토리지 엔진`이 사용된다.

### 😘 임시 테이블이 필요한 쿼리

아래와 같은 패턴의 `쿼리`는 **MySQL 엔진에서 별도의 데이터 가공 작업을 필요로 하는 쿼리**다.

따라서 `내부 임시 테이블`을 **생성하는 대표적인 케이스**라고 볼 수 있다.

- `ORDER BY`와 `GROUP BY`에 명시된 컬럼이 다른 쿼리
- `ORDER BY`나 `GROUP BY`에 명시된 컬럼이 **조인의 순서상 첫 번째 테이블이 아닌 쿼리**
- `DISTINCT`와 `ORDER BY`가 동시에 존재하는 경우 또는 `DISTINCT`가 인덱스로 처리되지 못하는 쿼리
- `UNION`이나 `UNION DISTINCT`가 사용된 쿼리(select_type 컬럼이 UNION RESULT인 경우)
- 쿼리의 실행 계획에서 `select_type`이 `DERIVED`인 쿼리

위의 패턴에서 마지막 3개 패턴은 `Using temporary`가 표시되지 않는다.

`첫 번째`부터 `네 번째`까지의 쿼리 패턴은 **유니크 인덱스를 가지고 마지막 패턴은 그렇지 않다.**

일반적으로 `유니크 인덱스`가 있는 내부 임시 테이블은 그렇지 않은 `쿼리`보다 **처리 성능이 느리다.**

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/38f9383f-ee24-41ce-92d4-ea2d4536d80e)

### 🥷🏼 임시 테이블이 디스크에 생성되는 경우

내부 `임시 테이블`은 기본적으로는 `메모리`상에 만들어지지만 다음과 같은 **조건을 만족하면 임시 테이블을 사용할 수 없다.**

- `UNION`이나 `UNION ALL`에서 `SELECT`되는 컬럼 중에서 길이가 `512 바이트` 이상인 크기의 컬럼이 있는 경우
- `GROUP BY`나 `DISTINCT` 컬럼에서 **512 바이트 이상인 크기의 컬럼**이 있는 경우
- 메모리 `임시 테이블`의 크기가 (MEMORY 스토리지 엔진일때) `tmp_table_size` 또는 `max_heap_table_size` 시스템 변수보다 크거나 `temptable_max_ram`(TempTable 스토리지 엔진) **시스템 변수 값보다 큰 경우**

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/0f402c8c-75aa-4461-a1f5-aaab065940d5)

### 😇 임시 테이블 관련 상태 변수

- `실행 계획 확인`만으로는 `임시 테이블`이 메모리에서 처리됐는지 디스크에서 처리됐는지 알 수 없다.
- `쿼리`를 처리하기 위해 **몇 개의 임시 테이블**이 사용됐는지 알 수 없다.

`Using temporary`가 **한 번 표시됐다고 해서 임시 테이블을 하나만 사용**했다는 것을 의미하지 않는다.

임시 테이블이 디스크에 `생성`됐는지 메모리에 생성됐는지 확인하려면 `상태 변수`를 `확인`하면 된다.

```sql
-- 현재 세션의 상태 값을 초기화
mysql> FLUSH STATUS;

mysql> SELECT first_name, last_name
       FROM employees
       GROUP BY first_name, last_name;
```

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/0076c100-7f43-47f0-9e20-a0e03a515e8e)

표시되는 `상태 변수`에 누적 된 값은 아래와 같다.

- `Created_tmp_tables` : 쿼리의 처리를 위해 만들어진 `내부 임시 테이블`의 개수를 누적하는 상태 값이다. **임시 테이블의 생성 위치와 관계없이 모두 누적**한다.
- `Created_tmp_disk_tables` : 디스크에 **내부 임시 테이블이 만들어진 개수만 누적**한다.

`상태 변수`를 확인해보았을때 `쿼리`를 처리하기 위해 `1개`의 임시 테이블이 **디스크에 생성**되었다는 것을 알 수 있다.

### 🤓 고급 최적화

- `옵티마이저`가 실행 계획을 수립할 때 **통계 정보와 옵티마이저 옵션을 결합**해서 최적의 실행 계획을 수립한다.
- `옵티마이저 옵션`은 크게 조인과 관련된 **옵티마이저 옵션**과 **옵티마이저 스위치**로 구분할 수 있다.
- `옵티마이저 스위치`는 고급 최적화 기능들의 **활성화 여부를 제어**하는 용도로 사용된다.

### 🤫 옵티마이저 스위치 옵션

- 옵티마이저 스위치 옵션은 `optimizer_switch` 시스템 변수를 이용해서 제어한다.
- `optimizer_switch` 시스템 변수는 **여러 개의 옵션을 세트로 묶어서 설정**하는 방식으로 사용한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/2eaf89b2-b38c-4056-9f0f-700ab15b7abf)

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/9853a332-960d-48d2-a95a-21553e50dbca)

`옵티마이저 스위치 옵션`은 **“default”, “on”, “off”** 중 하나의 값을 가진다.

`옵티마이저 스위치 옵션`은 **글로벌과 세션 별로 모두 설정**할 수 있다.

```sql
-- // MySQL 서버 전체적으로 옵티마이저 스위치 설정
mysql> SET GLOBAL optimizer_switch='index_merge=on, index_merge_union=on, ...';

-- // 현재 커넥션의 옵티아미어 스위치만 설정
mysql> SET SESSION optimizer_switch='index_merge=on, index_merge_union=on, ...';
```

`“SET_VAR”` **옵티마이저 힌트를 사용**해서 `현재` 쿼리에만 `적용`할 수도 있다.

```sql
mysql> SELECT /*+ SET_VAR(optimizer_switch='condition_fanout_filter=off') */
	     ...
	     FROM ...
```

### 😒 MRR과 배치 키 엑세스(mrr & batched_key_access)

- `MRR`은 `Multi-Range Read`의 약자로, 메뉴얼에서는 `DS-MRR(Disk Sweep Multi-Range Read)`라고 부른다.

`MySQL 서버`의 조인 방식은 `드라이빙 테이블`의 레코드를 한 건 읽어서 `드리븐 테이블`의 일치하는 레코드를 찾아

조인하는 방식으로 수행된다.

이를 `Nested Loop Join`이라 부르는데, MySQL 서버 구조상 조인 처리는 `MySQL 엔진`이 처리하고

실제 `레코드`를 검색하고 읽는 부분은 `스토리지 엔진`이 담당한다.

이런 방식은 `드라이빙 테이블`의 레코드 건별로 `드리븐 테이블`의 레코드를 찾고 읽는 스토리지 엔진에서는

**아무런 최적화를 수행할 수 없다는 문제점이** 있다.

`MySQL 서버`는 이러한 `단점`을 보완하기 위해 **조인 대상 테이블** 중 하나로부터 `레코드`를 읽어서 

`조인 버퍼`에 버퍼링한다. `조인`을 **즉시 실행하지 않고 조인 대상을 버퍼링**하는 것이다.

`조인 버퍼`에 레코드가 가득 차면 `MySQL 엔진`이 버퍼링 된 레코드를 `스토리지 엔진`으로 한 번에 요청한다.

`스토리지 엔진`은 읽어야 할 레코드들을 `데이터 페이지`에 정렬된 순서로 접근해서 디스크의 데이터 페이지

**읽기를 최소화**할 수 있게 되는 것이다.

위와 같은 방식을 `MRR`이라고 부른다.

`MRR`을 응용해서 실행되는 `조인 방식`을 **BKA(Batched Key Access) 조인**이라고 부른다.

`BKA 조인 최적화`는 기본적으로 `비활성화`돼 있다.

`쿼리의 특성`에 따라 큰 도움이 되는 경우도 있지만 `BKA 조인` 사용으로 인하여 **부가적인 정렬 작업**이 

필요해지면서 `성능`에 안 좋은 `영향`을 끼치는 경우도 있기 때문이다.

### 🥱 블록 네스티드 루프 조인(block_nested_loop)

- `MySQL 서버`에서 사용되는 대부분의 조인은 `Nested Loop Join` 이다.
- `조인`의 연결 조건이 되는 `컬럼`에 **모두 인덱스가 있는 경우 사용되는 조인 방식**이다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/14025a77-2e04-4961-b2e7-79517a85d5da)

이러한 형태의 `조인`은 프로그래밍 언어에서 마치 **중첩된 반복 명령을 사용하는 것과 같이 작동**한다.

```sql
for (row1 IN employees) {
  for (row2 IN salaries) {
    if (conditon_matched) return (row1, row2);
  }
}
```

`레코드`를 읽어서 다른 버퍼 공간에 저장하지 않고 즉시 `드리븐 테이블`의 레코드를 찾아서 `반환`하는 것을 볼 수 있다.

`네스티드 루프 조인`과 `블록 네스티드 루프 조인`의 가장 큰 차이는 조인 버퍼가 사용되는지 여부와

`드라이빙 테이블`과 `드리븐 테이블`이 **어떤 순서로 조인되느냐**다.

`조인 알고리즘`에서 `“Block”`이라는 단어가 사용되면 `조인`용으로 **별도의 버퍼가 사용**됐다는 것을 의미한다.

`조인 쿼리`의 실행 계획에서 `“Using Join buffer”` 문구가 표시되면 조인 버퍼가 사용된 것이다.

`조인`은 드라이빙 테이블에서 **일치하는 레코드의 건수만큼 드리븐 테이블을 검색**하면서 처리된다.

`드라이빙 테이블`은 한 번에 쭉 읽고 `드리븐 테이블`은 **여러 번 읽는 다는 것을 의미**한다.

예를 들어, `드라이빙 테이블`에서 일치하는 레코드가 `1,000건`일때 드리븐 테이블의 조인 조건이

인덱스를 이용하지 못한다면 **1,000번의 풀 테이블 스캔**을 해야 한다.

`드리븐 테이블` 검색할 때 **인덱스를 사용할 수 없는 쿼리**는 상당히 느려지기 때문에,

`옵티마이저`는 최대한 `드리븐 테이블`의 검색이 인덱스를 사용할 수 있게 `실행 계획`을 `수립`한다.

어떤 `실행 계획`을 `수립`해도 드리븐 테이블의 **풀 테이블 스캔이나 인덱스 풀 스캔**을 피할 수 없다면

드라이빙 테이블에서 읽은 레코드를 `메모리`에 `캐시`한 후 `드리븐 테이블`과 `메모리 캐시`를 조인하는

방식으로 `최적화`한다.

조인 버퍼는 `join_buffer_size`라는 시스템 변수로 크기를 제한할 수 있고 조인이 완료되면 

**조인 버퍼는 바로 해제**된다.

```sql
mysql> SELECT * 
       FROM dept_emp de, employees e
       WHERE de.from_date>'1995-01-01' AND e.emp_no<109004;
```

이 예제 쿼리는 각 `테이블`에 대한 조건만 있고 두 테이블 간의 `연결 고리` 역할을 하는 조인 조건이 없다.

`dept_emp` 테이블에서 **from_date>’1995-01-01’**인 레코드와 `employees` 테이블에서

**emp_no<109004** 조건을 만족하는 레코드는 **카테시안 조인을 수행**한다.

이 쿼리의 `실행 계획`을 살펴보면 아래와 같다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/b2dd10a7-77d3-43c9-acd6-8196f04cbe44)

`dept_emp` 테이블이 드라이빙 테이블이며, `employees` 테이블을 읽을 때는 `Join buffer`를 이용하여

**블록 네스티드 루프 조인**을 한다는 것을 알 수 있다.

이 실행 계획이 `조인 버퍼`가 어떤 단계로 사용되는지 살펴보면 아래와 같다.

1. `dept_emp` 테이블의 `ix_hiredate` 인덱스를 이용해 `from_date>’1995-01-01’` 조건을 만족하는 레코드를 검색한다.
2. `조인`에 필요한 나머지 컬럼을 모두 `dept_emp` 테이블로부터 읽어서 **조인 버퍼에 저장**한다.
3. `employees` 테이블의 `PK`를 이용해 `emp_no<109004` 조건을 만족하는 레코드를 검색한다.
4. 3번에서 검색된 결과에 2번의 **캐시된 조인 버퍼의 레코드(dept_emp)를 결합해서 반환**한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/2aafdfe5-0082-4085-9a3b-7492171ec528)

이 방식에서 중요한 점은 `조인 버퍼`가 사용되는 쿼리는 **조인의 순서가 거꾸로인 것처럼 실행**된다는 점이다.

`employees` 테이블의 결과를 기준으로 `dept_emp` 테이블의 **결과를 병합**하기 때문이다.

`실행 계획`상으로는 `dept_emp` 테이블이 드라이빙 테이블 이지만, 실제 드라이빙 테이블의 결과는

조인 버퍼에 담아두고 `드리븐 테이블`을 먼저 읽기 때문에 거꾸로인 것처럼 실행된다고 표현한다.

`조인 버퍼`가 사용되는 **조인에서는 결과의 정렬 순서가 흐트러질 수 있음**에 항상 `유의`해야 한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/45ac7185-df59-44c6-bca7-21223f5df092)

### 👿 인덱스 컨디션 푸시다운(index_condition_pushdown)

- `5.6 버전`부터 **인덱스 컨디션 푸시다운 기능이 도입**되었다.

```sql
-- // 테스트용 인덱스 생성
mysql> ALTER TABLE employees ADD INDEX ix_lastname_firstname (last_name, first_name);

-- // index condition push down 비활성화
mysql> SET optimizer_switch='index_condition_pushdown=off';
-- // index_conditoin_pushdown 비활성화 상태 확인
mysql> SHOW VARIABLES LIKE 'optimizer_switch' \G;
```

아래와 같은 `쿼리`를 실행할 때 `스토리지 엔진`이 몇 건의 레코드를 읽어올까?

```sql
mysql> SELECT * FROM employees WHERE last_name='Action' AND first_name LIKE '%sal';
```

`last_name=’Acton’` 조건은 위에서 생성한 `ix_lastname_firstname` 인덱스를 

레인지 스캔으로 사용할 수 있다. `fist_name LIKE ‘%sal’` 조건은 `레인지 스캔`으로는 검색 범위를 좁힐 수 없다.

따라서 `last_name` 조건은 **인덱스의 특정 범위만 조회할 수 있는 범위 결정 조건**이며

`first_name LIKE ‘%sal’` 조건은 **데이터를 모두 읽은 후 하나씩 비교하는 필터링 조건**이다.

이 쿼리의 `실행 계획`을 확인하면 아래와 같다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/1c5d4a3d-f92f-4ca2-8a0c-208a324f7936)

`Extra 컬럼`에 `“Using where”`가 표시된 것을 확인할 수 있다.

`“Using where”`는 **InnoDB 스토리지 엔진**이 읽어서 반환해준 레코드가 `인덱스`를 사용할 수 없는

**WHERE 조건에 일치하는지 검사**하는 과정을 의미한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/1eb23b82-d19e-4ec8-9df8-c49b0940fb75)

이 그림은 `last_name=’Acton’` 조건으로 `Index Range Scan`을 하고 테이블의 레코드를 읽고나서

`first_name LIKE ‘%sal’` 조건에 부합되는지 비교하는 과정을 나타낸 것이다.

만약 `last_name=’Acton’` 조건에 일치하는 레코드가 `10만 건`이나 되는데 그중에서 단 `1건`만 

`fist_name LIKE ‘%sal’` 조건에 부합한다면 **99,999건의 불필요한 읽기 작업**을 한 것이다.

`first_name LIKE ‘%sal’` 조건을 처리하기 위해 이미 한 번 읽은 `ix_lastname_firstname` 인덱스의

`first_name` 컬럼을 왜 사용하지 않고 **다시 레코드를 읽어서 처리 했는지 의문**이 들 것이다.

`first_name LIKE ‘%sal’` 조건을 누가 처리하느냐에 따라서 인덱스에 포함된 `first_name` 컬럼의 

**사용여부가 결정**된다.

`인덱스`를 비교하는 작업은 **InnoDB 스토리지 엔진이 수행**하지만, `테이블`의 레코드에서 `first_name` 

조건을 `비교`하는 작업은 **MySQL 엔진이 수행**한다.

`5.5 버전`까지는 인덱스를 범위 제한 조건으로 사용하지 못하는 `first_name` 조건은

`MySQL 엔진`이 **스토리지 엔진으로 아예 전달해주지 않아서 이런 문제가 발생**했다.

`5.6 버전`부터는 인덱스를 범위 제한 조건을 사용하지 못해도 `인덱스`에 포함된 컬럼의

`조건`이 있는 경우 **모두 같이 모아서 스토리지 엔진으로 전달하도록 핸들러 API가 개선**되었다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/6e734efd-1e04-4658-a653-f8433b2f4337)

`옵티마이저 옵션`을 원래대로 돌려놓고, 실행 계획을 다시 한 번 확인해보자.

```sql
mysql> SET optimizer_switch='index_condition_pushdown=on';
mysql> SHOW VARIABLES LIKE 'optimizer_switch'\G;

Variable_name : optimizer_switch
        Value : ..., index_contidtion_pushdown=on,...
```

`옵티마이저 스위치`에서 `index_condition_pushdown`을 **활성화**하면 쿼리의 실행 계획이 바뀐다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/2c654227-0baa-457b-b156-5dafd2804047)

`Extra 컬럼`에 `“Using where”`가 없어지고 `“Using index condition”`이 출력되는 것을 확인할 수 있다.

`인덱스 컨디션 푸시다운` 기능은 쿼리의 성능이 `몇 배`에서 `몇십 배`로 향상될 수 있는 **중요한 기능**이다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/63edfc3b-3b20-4096-b550-f9c2afb01978)

### 🧛🏼 인덱스 확장(use_index_extensions)

- `use_index_extensions` 옵션은 `InnoDB 스토리지 엔진`을 사용하는 테이블에서 `세컨더리 인덱스`에 자동으로 추가된 **프라이머리 키를 활용할 수 있게 할지를 결정**하는 옵션이다.

```sql
mysql> CREATE TABLE dept_emp(
         emp_no INT NOT NULL,
         dept_no CHAR(4) NOT NULL,
         from_date DATE NOT NULL,
         to_date DATE NOT NULL,
         PRIMARY KEY (dept_no, emp_no),
         KEY ix_fromdate (from_date)
       ) ENGINE=InnoDB;
```

`InnoDB 스토리지 엔진`은 **PK를 클러스터링 키로 생성**한다.

모든 `세컨더리 인덱스`는 **리프 노드에 프라이머리 키 값을 가진다.**

`세컨더리 인덱스`는 데이터 레코드를 찾아가기 위해 `PK`인 **dept_no와 emp_no 컬럼을 순서대로 포함**하고,

`ix_fromdate` 인덱스는 **(from_date, dept_no, emp_no) 조합으로 인덱스를 생성**한 것과 흡사하게

`작동`할 수 있다.

`MySQL 구버전`에서는 아래와 같은 `쿼리`가 `세컨더리 인덱스`의 마지막에 자동 추가되는 `PK`를 제대로 활용하지

못했는데 `업그레이드`되면서 `옵티마이저`가 `ix_fromdate` 인덱스의 마지막에 **(dept_no, emp_no) 컬럼**이

있다는 것을 인지하고 `실행 계획`을 `수립`하도록 `개선`했다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/2aa7bc1d-b86e-4b79-bdd7-b04e2f9efabb)

실행 계획의 `key_len 컬럼`은 이 쿼리가 `인덱스`를 구성하는 컬럼 중에서 어느 부분까지 사용했는지를

바이트 수로 보여준다.

`19 바이트`가 표시된 것을 보았을 때 `from_date 컬럼(3 바이트)`과 `dept_emp 컬럼(16 바이트)`까지

사용되었다는 것을 `실행계획`에서 알 수 있다.

`dept_no=’d001’` 조건을 제거한다면 쿼리의 실행 계획이 어떻게 바뀔까?

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/4fb9114a-2432-47be-9f3e-51b811ea60cb)

`dept_no 컬럼`을 사용하지 않기 때문에 `from_date 컬럼`을 위한 3 바이트만 `key_len 컬럼`에

표시되는 것을 확인할 수 있다.

`InnoDB`의 `PK`가 **세컨더리 인덱스에 포함**되어 있으므로 `정렬 작업`도 인덱스를 활용해서

처리된다는 `장점`도 있다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/1064623c-b936-45ce-9771-e705306d0894)

실행 계획의 `Extra 컬럼`에 `“Using Filesort”`가 표시되지 않았다는 것을 보았을 때 별도의 `정렬 작업` 없이

**인덱스 순서대로 레코드를 읽기만 했다는 것**을 알 수 있다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/af983aac-2284-4031-837b-959dd18ed231)


### 😋 인덱스 머지(index_merge)

- `인덱스`를 이용해 쿼리를 실행하면 `옵티마이저`는 대부분의 경우에 `테이블`별로 **하나의 인덱스만 사용하도록 실행계획을 수립**한다.
- `인덱스 머지 실행 계획`을 사용하면 하나의 테이블에 대해 `2개 이상`의 **인덱스를 이용해 쿼리를 처리**한다.

`쿼리`에서 한 테이블 대한 `WHERE 조건`이 여러 개 있더라도 **하나의 인덱스에 포함된 컬럼에 대한 조건**만으로

`인덱스`를 `검색`하고 나머지 조건은 읽어온 레코드에 대해서 `체크`하는 형태로만 사용되는 것이 `일반적`이다.

**하나의 인덱스**만 사용해서 `작업 범위`를 줄일 수 있다면, `테이블`별로 **하나의 인덱스만 쓰는 것이 효율적**이다.

하지만, `쿼리`에 사용된 **각각의 조건이 서로 다른 인덱스를 사용**할 수 있고 그 `조건`을 만족하는 `레코드 건수`가

**많을 것으로 예상된다면 MySQL 서버는 인덱스 머지 실행 계획을 선택**하게 된다.

`인덱스 머지 실행 계획`은 아래와 같이 `3개`의 세부 실행 계획으로 나누어 볼 수 있다.

3가지 최적화 모드 모두 **여러 개의 인덱스를 사용하는 것**은 같지만 `결과`를 `병합`하는 `방식`이 다르다.

- `index_merge_intersection`
- `index_merge_sort_union`
- `index_merge_union`

### 🤢 인덱스 머지 - 교집합(index_merge_intersection)

```sql
mysql> EXPLAIN SELECT * 
       FROM employees
       WHERE first_name='Georgi' AND emp_no BETWEEN 10000 AND 20000;
```

다름 쿼리는 `2개`의 `WHERE 조건`을 가지고, 조건절에 명시된 `컬럼`은 모두 각각의 인덱스를 가지고 있다.

`2개` 중에서 어떤 조건을 사용하더라도 `인덱스`를 `사용`할 수 있다는 이야기다.

`옵티마이저`는 `ix_firstname`과 `PRIMARY 키`를 모두 사용해서 **쿼리를 처리하도록 결정**한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/3ae218f9-afe0-4a5b-b06c-7cb1a53c9d0d)

실행 계획의 `Extra 컬럼`에 `“Using intersect”`라고 표시되어 있는데, 이 `쿼리`가 여러 개의 인덱스를 

각각 검색해서 그 결과의 `교집합`만 `반환`했다는 것을 `의미`한다.

`first_name` 컬럼의 조건과 `emp_no` 컬럼의 조건 중 **하나라도 충분히 효율적으로 쿼리를 처리**할 수 

있었다면 `옵티마이저`가 `2개`의 인덱스를 **모두 사용하지 않았을 것**이다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/57a4c239-1328-45c6-8d49-1555da5eacfd)

`인덱스 머지 실행 계획`이 아니었다면 다음과 같은 방식으로 `처리`해야 했을 것이다.

- `first_name=’Georgi’` 조건만 인덱스를 사용하면, 일치하는 레코드 `253건`을 검색한 다음 데이터 페이지에서 레코드를 찾고 `emp_no` 컬럼의 조건에 일치하는 레코드들만 반환해야 한다.
- `emp_no BETWEEN 10000 AND 20000` 조건만 인덱스를 사용하면 `PK`를 이용해 `10,000건`을 읽어서 `first_name` 조건에 일치하는 레코드들만 반환해야 한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/d2c86ef0-09e1-4f6b-b7b1-1b26fbd3bb10)

첫 번째와 두 번째 방식 모두 그다지 `나쁜 실행 계획`이 아닌 것 같지만,

실제 **두 조건을 만족하는 레코드 건수**가 `14건` 밖에 되지 않는 것을 보면 `옵티마이저`는 현명한 선택을 했다.

`index_merge_intersection` 최적화를 비활성화 하려면 세 가지 포인트로 `비활성화` 할 수 있다.

- MySQL 서버 `전체적`으로 비활성화
- 현재 `커넥션`에 대해 비활성화
- 현재 `쿼리`에서만 비활성화

```sql
-- // MySQL 서버 전체적으로 index_merge_intersection 비활성화
mysql> SET GLOBAL optimizer_switch='index_merge_intersection=off';

-- // 현재 커넥션에 대해 index_merge_intersection 비활성화
mysql> SET SESSION optimizer_switch='index_merge_intersection=off';

-- // 현재 쿼리에서만 index_merge_interserction 비활성화
mysql> EXPLAIN
       SELECT /*+ SET_VAR(optimizer_switch='index_merge_intersection=off') */ *
       FROM employees
       WHERE first_name='Georgi' AND emp_no BETWEEN 10000 AND 20000;
```

### 🤓 인덱스 머지 - 합집합(index_merge_union)

- `“Using union”`은 WHERE 절에 사용된 `2개 이상`의 조건이 **각각의 인덱스를 사용하되 OR 연산자로 연결된 경우**에 `사용`된다.

```sql
mysql> SELECT * 
       FROM employees
       WHERE first_name='Matt' OR hire_date='1987-03-31';
```

위의 예제 쿼리는 `2개`의 조건이 `OR`로 연결되어 있다.

`employees` 테이블은 `first_name` 컬럼과 `hire_date` 컬럼에 인덱스가 생성되어 있는 상태이다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/25e2c371-94b6-43d3-ab34-accdc711fe5e)

`쿼리`의 `실행 계획`에서 `Extra 컬럼`에 **“Using union(ix_fisrtname, ix_hiredate)”**가 

표시된 것을 볼 수 있다.

`인덱스 머지 최적화`가 **“Union” 알고리즘으로 검색 결과를 병합**했다는 것을 `의미`한다.

만약 `first_name=’Matt’` 이면서 `hire_date=’1987-03-31’`인 사원이 있었다면,

해당 사원의 정보는 각 `인덱스 검색 결과`에 포함되어 중복이 발생했을 것 같이 보인다.

`중복`이 `제거`된 것을 보아, 두 결과 집합을 정렬해서 `중복 레코드`를 `제거` 했을것으로 보이지만

`“Union”` 알고리즘은 정렬 없이 `중복 제거`를 한다.

`MySQL 서버`는 `first_name` 컬럼을 검색한 결과와 `hire_date` 컬럼을 검색한 결과가

`PK`로 **이미 정렬되어 있다는 사실을 알고 있기 때문**에 가능한 일이다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/3fb38b9b-b504-494f-bbf3-9d90be8e40c7)

MySQL 서버는 `두 집합`에서 하나씩 가져와서 서로 비교하면서 `PK`인 `emp_no` 컬럼의 값이

**중복된 레코드들을 정렬 없이 걸러낼 수 있는 것**이다.

정렬된 두 집합의 결과를 하나씩 가져와 `중복 제거`를 수행할 때 `Priority Queue`가 이용된다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/6eb7d8ac-3291-40b2-bbcc-55d93b364f37)

### 🙃 인덱스 머지 - 정렬 후 합집합(index_merge_sort_union)

- `인덱스 머지` 작업을 하는 도중에 결과의 `정렬`이 필요한 경우 인덱스 머지 최적화의 `‘Sort union’` 알고리즘을 사용한다.

```sql
mysql> EXPLAIN
         SELECT * FROM employees
         WHERE first_name='Matt'
            OR hire_date BETWEEN '1987-03-01' AND '1987-03-31';
```

위의 쿼리를 2개의 쿼리로 분리해서 생각해보면 `‘Sort union’` 알고리즘을 이해하기 쉽다.

```sql
mysql> SELECT * FROM employees WHERE first_name='Matt';
mysql> SELECT * FROM employees WHERE hire_date BETWEEN '1987-03-01' AND '1987-03-31';
```

첫 번째 쿼리는 `emp_no`로 `정렬`되어 출력되지만 두 번째 `쿼리`는 그렇지 않다.

따라서 `중복`을 `제거`하기 위해 `우선순위 큐`를 사용하는 것이 불가능하다.

MySQL 서버는 두 집합의 결과에서 `중복`을 제거하기 위해 각 집합을 `emp_no` 컬럼으로 정렬한 뒤

`중복 제거`를 수행한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/6999fc33-654b-4b17-bfcb-fcb92ec06683)

위의 실행계획처럼 `인덱스 머지` 최적화에서 `중복 제거`를 위해 **강제로 정렬을 수행**해야 하는 경우,

`Extra 컬럼`에 `“Using sort_union”` 문구가 표시된다.

### 🤔 세미 조인(semijoin)

- `다른 테이블`과 실제 `조인`을 수행하지 않고 다른 테이블에서 `조건`에 `일치`하는 레코드가 있는지 없는지 `체크`만 하는 형태의 쿼리를 `세미조인`이라고 한다.
- `5.7 버전`은 세미 조인 형태의 **쿼리를 최적화하는 부분이 상당히 취약**하다.

```sql
mysql> SELECT * 
       FROM employees e
       WHERE e.emp_no IN
          (SELECT de.emp_no FROM dept_emp de WHERE de.from_date='1995-01-01');
```

`세미 조인` 최적화 기능이 없었을 때는 `employees 테이블`을 풀 스캔하면서 **한 건씩 서브쿼리의 조건을 비교**했다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/085bea56-195f-4a41-9e81-31ebd798fcc0)

`실행 계획`을 보면 `57건`만 읽으면 될 쿼리를 `30만 건` 넘게 읽어서 처리하는 것을 볼 수 있다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/c4425781-02c1-4ec1-b382-9b93c13fef40)

`세미 조인` 형태의 쿼리와 `안티 세미 조인` 형태의 쿼리는 `최적화 방법`에 차이가 있다.

`= (subquery)` 형태와 `IN (subquery)` 형태의 세미 조인 쿼리에는 **3가지 최적화 방법을 적용**할 수 있다.

- `세미 조인` 최적화
- `IN-to-EXISTS` 최적화
- `MATERIALIZATION` 최적화

`<> (subquery)` 형태와 `NOT IN (subquery)` 형태의 안티 세미 조인 쿼리는 **2가지 최적화 방법**이 있다.

- `IN-to-EXISTS` 최적화
- `MATERIALIZATION` 최적화

`8.0` 부터는 세미 조인 쿼리의 성능을 개선하기 위한 `최적화 전략`들이 있다.

- Table-Pull-out
- Duplicate Weed-out
- First Match
- Loose Scan
- Materialization

`쿼리`에 사용되는 테이블과 `조인 조건`의 `특성`에 따라 `옵티마이저`는 전략을 **선별적으로 사용**한다.

`Table-Pull-out` 전략은 사용 가능하면 **항상 세미 조인보다는 좋은 성능**을 낸다.

`First Match`와 `Loose Scan` 전략은 `firstmatch`와 `loosescan` 옵티마이저 옵션으로 사용 여부를 

결정할 수 있다.

`Duplicate Weed-out`과 `Materialization` 전략은 `materialization` 옵티마이저 스위치로

사용 여부를 결정할 수 있다.

`optimizer_switch` 시스템 변수의 `semijoin` 옵티마이저 옵션은 `firstmatch`와 `loosescan`,

`materialization` 옵티마이저 **옵션을 한 번에 활성화 하거나 비활성화할 때 사용**된다.

### 😵‍💫 테이블 풀-아웃(Table Pull-out)

- `세미조인`의 `서브쿼리`에 사용된 테이블을 **아우터 쿼리로 끄집어낸 후에 쿼리를 조인 쿼리로 재작성**하는 형태의 최적화다.
- `서브쿼리` 최적화가 도입되기 이전에 **수동으로 쿼리를 튜닝**하던 대표적인 `방법`이었다.

```sql
mysql> EXPLAIN
       SELECT * FROM employees e
       WHERE e.emp_no IN (SELECT de.emp_no FROM dept_emp de WHERE de.dept_no='d009');
```

이 쿼리의 `실행 계획`을 보면 아래와 같다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/56e96c48-cce2-43fa-ad5f-5444e1983ebf)

`Table pullout` 최적화는 별도의 실행 계획의 `Extra 컬럼`에 표시되지 않는다.

`Table pullout` 최적화가 사용됐는지는 해당 `테이블`들의 `id 컬럼` 을 `동등 비교`해보는 것이 가장 간단하다.

더 정확하게 확인하려면 `EXPLAIN` 명령을 실행한 직후 `SHOW WARNINGS` 명령으로

`옵티마이저`가 **재작성한 쿼리를 살펴보는 방법**이 있다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/5f6b735d-c17f-475c-900b-4fee796bbec9)

`IN(subquery)` 형태의 쿼리를 `JOIN`으로 재작성 된 것을 볼 수 있다.

`Table pullout` 최적화는 모든 형태의 `서브쿼리`에서 사용될 수 있는 것은 아니다.

- `Table pullout` 최적화는 **세미 조인 서브쿼리에서만 사용 가능**하다.
- `Table pullout` 최적화는 `서브 쿼리` 부분이 `UNIQUE` 인덱스나 `PK 룩업`으로 결과가 `1건`인 경우에만 사용 가능하다.
- `Table pullout`이 적용된다고 하더라도 `기존 쿼리`에서 사용하는 `최적화 방법`이 사용 불가능한 것은 아니므로 가능하다면 `Table pullout` 최적화를 적용한다.
- 만약 서브쿼리의 모든 테이블이 `아우터 쿼리`로 끄집어 낼 수 있다면 **서브쿼리 자체는 없어진다.**
- `Table pullout` 최적화가 있으므로 `서브쿼리` 조인으로 풀어서 사용할 필요가 없다.

### 🥱 퍼스트 매치(firstmatch)

- `First Match` 최적화 전략은 `IN(subquery)` 형태의 세미 조인을 `EXISTS(subquery)` 형태로 튜닝한 것과 비슷하게 실행된다.

```sql
mysql> EXPLAIN SELECT * 
       FROM employees e WHERE e.first_name='Matt'
         AND e.emp_no IN (
           SELECT t.emp_no FROM titles t
           WHERE t.from_date BETWEEN '1995-01-01' AND '1995-01-30'
         );
```

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/a586eb49-c2a5-480c-a33d-9942f562156d)

`실행 계획`의 id 컬럼의 값이 모두 `“1”`로 표시 되었으므로 `titles 테이블`이 `서브쿼리 패턴`이 아닌

`조인`으로 처리됐다는 것을 알 수 있다.

`“FirstMatch(e)”`는 `employees` 테이블의 레코드에 대해 `titles` 테이블에 일치하는 레코드 1건만 찾으면

더 이상 `titles` 테이블 검색을 하지 않는다는 것을 의미한다.

의미론적으로는 `EXISTS(subquery)`와 동일하게 처리 된 것이다.

하지만 `FirstMatch`는 **서브쿼리가 아니라 조인으로 풀어서 실행**하면서 일치하는 `첫 번째 레코드`만 `검색`하도록 `최적화`를 수행한 것이다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/07b8eae4-aef7-471b-9f5e-cad5ef4c6e11)

위의 쿼리를 `FirstMatch`로 최적화 후 수행한 과정을 그림으로 나타낸 것이다.

1. `employees` 테이블에서 `first_name` 컬럼의 값이 `‘Matt’`인 사원의 정보를 `ix_firstname` 인덱스를 이용해 `Range Scan`으로 읽는다.
2. `first_name`이 `‘Matt’`이고 사원번호가 `12302`인 레코드를 **titles 테이블과 조인**해서 from_date 조건을 만족하는 레코드가 있는지 찾아본다.
3. 일치하는 레코드가 없으므로 다음 레코드인 `emp_no`가 `243075`인 사원의 레코드를 읽어서 `titles` 테이블과 조인하여 `from_date` 조건을 만족하는 레코드가 있는지 확인한다.
4. `titles 레코드` 중에서 조건을 만족하는 레코드를 찾았으므로 `243075`번 사원에 대해서는 **더 이상 titles 테이블을 탐색하지 않고 최종 결과를 반환**한다.

`First Match` 최적화는 5.5 버전의 `IN-to-EXISTS` 변환과 거의 비슷하다.

하지만 `FirstMatch` 최적화는 아래와 같은 장점이 더 있다.

- `IN-to-EXISTS` 최적화에서는 **동등 조건 전파가 서브쿼리 내에서만 가능**했지만 `First Match` 최적화에서는 **아우터 쿼리의 테이블까지 전파**될 수 있다.
- `IN-to-EXISTS` 변환 최적화 전략에서는 **아무런 조건 없이 변환이 가능할 때만 최적화를 수행**했지만 First Match 최적화는 `서브쿼리`의 **모든 테이블에 대해 최적화를 수행할 지 일부 테이블에 대해서만 수행**할지 `선택`할 수 있다.

`First Match` 최적화 또한 특정 형태의 `서브쿼리`에서 자주 사용되는 최적화 방법이다.

- `FirstMatch` 최적화에서 서브쿼리는 그 `서브쿼리`가 참조하는 **모든 아우터 테이블이 먼저 조회된 이후에 실행**된다.
- `FirstMatch` 최적화가 사용되면 실행 계획의 Extra 컬럼에는 `“FirstMatch(table-N)”` 문구가 표시된다.
- `FirstMatch` 최적화는 **상관 서브쿼리**에서도 사용될 수 있다.
- `FirstMatch` 최적화는 `GROUP BY`나 집합 함수가 사용된 `서브쿼리`의 `최적화`에는 사용될 수 없다.

`FirstMatch` 최적화는 `optimizer_switch` 시스템 변수에서 **semijoin & firstmatch 옵션이 모두 ON**으로 

활성화 된 경우에만 사용할 수 있다.

### ✌🏼루스 스캔(loose scan)

- `LooseScan`은 인덱스를 사용하는 `GROUP BY` 최적화 방법 중 **루스 인덱스 스캔과 비슷한 읽기 방식을 사용**한다.

```sql
mysql> EXPLAIN
       SELECT * FROM departments d WHERE d.dept_no IN (
          SELECT de.dept_no FROM dept_emp de );
```

이 쿼리는 `dept_emp` 테이블에 존재하는 **모든 부서 번호에 대해 부서 정보를 읽어 오기 위한 쿼리**다.

`departments` 테이블의 레코드는 `9건`이고, `dept_emp` 테이블의 레코드 건수는 `33`만건이다.

`dept_emp` 테이블에는 **(dept_no, emp_no) 컬럼**의 조합으로 `PK 인덱스`가 만들어져 있다.

그렇다면, `dept_emp` 테이블의 `PK`를 루스 인덱스 스캔으로 `유니크`한 `dept_no`만 읽으면

**중복된 레코드까지 제거**하면서 아주 효율적으로 `서브쿼리` 부분을 `실행`할 수 있다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/3eb9ba76-2e16-481b-879c-ca87193f9f00)

그림을 살펴보면 `dept_emp` 테이블이 **드라이빙 테이블로 실행**되고, `dept_emp` 테이블의 `PK`를

`dept_no` 부분에서 **유니크하게 한 건씩만 읽고 있다는 것**을 알 수 있다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/1300e750-a1fa-4abf-8809-a088fa7e59f9)

실행 계획을 보면 `Extra 컬럼`에 `“LooseScan”` 문구가 표시된 것을 알 수 있다.

또한 각 테이블에 할당된 `id 컬럼`이 `1`이라는 것을 보았을 때 조인처럼 `처리`됐다는 점도 알 수 있다.

`LooseScan`은 아래와 같은 특성을 가진다.

- `루스 인덱스 스캔`으로 **서브쿼리 테이블**을 읽고, `아우터 테이블`을 **드리븐으로 사용해서 조인을 수행**한다.
- `서브쿼리` 부분이 루스 인덱스 스캔을 사용할 수 있는 **조건이 갖춰져야 사용**할 수 있다.

`LooseScan` 최적화는 아래와 같은 `서브쿼리 패턴`에서 사용할 수 있다.

```sql
SELECT .. FROM .. WHERE expr IN (SELECT keypart1 FROM tab WHERE ...)
SELECT .. FROM .. WHERE expr IN (SELECT keypart2 FROM tab WHERE keypart1='상수' ...) 
```

`LooseScan` 최적화 사용 여부도 `optimizer_switch` 시스템 변수로 관리할 수 있다.

```sql
mysql> SET optimizer_switch='loosescan=off';
```

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/6a69a89c-103e-4fff-bc17-bdf98a0bc615)

### 😐 구체화(Materialization)

- `세미 조인`에 사용된 서**브쿼리를 통째로 구체화**해서 쿼리를 `최적화`한다는 의미이다.
- `내부 임시테이블`을 생성해서 **쿼리를 최적화**한다는 것을 의미한다.

```sql
mysql> EXPLAIN
       SELECT * 
       FROM employees e
       WHERE e.emp_no IN
          (SELECT de.emp_no FROM dept_emp de
           WHERE de.from_date='1995-01-01');
```

이 쿼리는 `FirstMatch` 최적화를 사용하면 `employees` 테이블에 대한 조건이 `서브쿼리` 이외에는

아무것도 없기 때문에 **employees 테이블을 풀 스캔**해야 한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/ad260d3e-055f-4116-9318-310ee934be4b)

이 쿼리의 `실행계획`을 보면 **서브쿼리 구체화(Subquery Materialization)** 최적화를 수행했다.

이 `쿼리`에서 사용하는 테이블은 `2개`인데 실행 계획이 `3개` 라인으로 출력된 것을 보면 `임시 테이블`을 생성해서

**최적화를 수행**했다는 것을 알 수 있다.

```sql
mysql> EXPLAIN
       SELECT * 
       FROM employees e
       WHERE e.emp_no IN
           (SELECT de.emp_no FROM dept_emp de
           WHERE de.from_date='1995-01-01'
           GROUP BY de.dept_no);
```

이 쿼리 처럼 `서브쿼리` 내에 `GROUP BY` 절이 있어도 **Materialization 최적화가 가능**하다.

`Matherialization` 최적화가 사용될 수 있는 형태의 `쿼리`도 제한 사항과 특성이 있다.

- `IN(subquery)`에서 서브쿼리는 상관 서브쿼리가 아니어야 한다.
- 서브쿼리는 `GROUP BY`나 `집합 함수`들이 **사용되어도 구체화를 사용**할 수 있다.
- `구체화`가 사용된 경우에는 `내부 임시 테이블`이 사용된다.

`Materialization` 최적화는 `optimizer_switch` 시스템 변수에서 **semijoin & materialization**

옵션이 모두 `ON`으로 활성화 되어 있어야 사용 가능하다.

`Materialization` 최적화만 비활성화 처리 하려면, `materialization` 옵션만 `OFF`로 설정하면 된다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/2ce20d58-69ef-4a6c-97b7-efd317ff115e)

### 🙃 중복 제거(Duplicated Weed-out)

- `세미 조인 서브쿼리`를 일반적인 `Inner Join` 쿼리로 바꿔서 실행하고 중복된 레코드를 제거하는 방법으로 처리하는 `최적화 알고리즘`이다.

```sql
mysql> EXPLAIN 
       SELECT * FROM employees e
       WHERE e.emp_no IN (SELECT s.emp_no FROM salaries s WHERE s.salary > 1500000);
```

`salaries` 테이블의 PK가 (emp_no + from_date) 이므로 `salary`가 150000 이상인 레코드를 조회하면

`중복`된 `emp_no`가 발생할 수 있다.

이 `쿼리`를 아래와 같이 재작성하면 `세미 조인 서브쿼리`와 동일한 결과를 얻을 수 있다.

```sql
mysql> SELECT e.*
       FROM employees e, salaries s
       WHERE e.emp_no=s.emp_no AND s.salary > 1500000
       GROUP BY e.emp_no;
```

`Duplicated Weedout` 알고리즘은 원본 쿼리를 `INNER JOIN + GROUP BY` 절로 바꿔서 실행하는 것과

**동일한 작업으로 쿼리를 처리**한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/d57817ab-3ef5-4f86-b228-ff09dea03722)

위의 그림은 `Duplicated Weedout` 알고리즘으로 예제 쿼리를 처리하는 과정이다.

1. `salaries` 테이블의 `ix_salary` 인덱스를 스캔해서 `salary`가 1500000보다 큰 사원을 검색해 **employees 테이블 조인을 실행**한다.
2. 조인된 결과를 `임시 테이블`에 저장한다.
3. 임시 테이블에 저장된 결과에서 `emp_no` 기준으로 **중복을 제거**한다.
4. `중복`을 제거하고 남은 레코드를 **최종적으로 반환**한다.

`Duplicate Weedout` 최적화를 이용한 실행 계획은 다음과 같다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/f51c8676-c937-44c4-8c47-5c9c7e959816)

Extra 컬럼에 `“Strat Temporary”`와 `“End Temporary”` 문구가 표시된 것을 볼 수 있다.

`“Duplicate Weedout”` 이라는 문구는 표시되지 않는다.

`Start/ End temporary` 문구의 구간이 **Duplicate Weedout 최적화 처리 과정**이라고 보면 된다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/b2ee4e47-e736-46c1-aa45-cf87d0a35252)

`Duplicate Weedout` 최적화 역시 제약 사항과 장점이 동시에 존재한다.

- 서브 쿼리가 `상관 서브쿼리`라고 하더라도 사용할 수 있다.
- 서브쿼리가 `GROUP BY`나 `집합 함수`가 사용된 경우에는 사용할 수 없다.
- `서브쿼리`의 테이블을 조인으로 처리하기 때문에 **최적화할 수 있는 방법이 많다.**

### 👍🏼 컨디션 팬아웃(condition_fanout_filter)

- `조인`을 실행할 때 테이블의 순서는 **쿼리의 성능에 매우 큰 영향**을 미친다.
- `옵티마이저`는 여러 테이블이 조인되는 경우 가능하다면 **일치하는 레코드 건수가 적은 순서대로 조인을 실행**한다.

```sql
mysql> SELECT * 
       FROM employees e
         INNER JOIN salaries s ON s.emp_no=e.emp_no
       WHERE e.first_name='Matt'
         AND e.hire_date BETWEEN '1985-11-21' AND '1986-11-21'
```

이 쿼리를 `condition_fanout_filter` 옵티마이저 옵션을 비활성화 하고 `실행 계획`을 보자.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/325d5e7f-b9a9-432d-9915-c61eb4288c5a)

이 `실행 계획`을 통해 이 쿼리가 어떻게 실행될지 예측할 수 있다.

1. `employees` 테이블에서 `ix_firstname` 인덱스를 이용해 **first_name=’Matt’** 조건에 일치하는 `233건`의 레코드를 검색한다.
2. 검색된 `233건`의 레코드 중에서 `hire_date`가 **‘1985-11-21’ 부터 ‘1986-11-21’일 사이**인 레코드만 걸러내는데, 실행 계획을 보면 `옵티마이저`가 **233건 모두 hire_date 컬럼의 조건을 만족할 것으로 예측**했다.
3. `employees` 테이블을 읽은 결과 `233건`에 대해 `salaries` 테이블의 `PK`를 이용해 `salaries` 테이블의 레코드를 읽는다. 옵티마이저는 `employees` 테이블의 레코드 한 건당 salaires 테이블의 레코드 10건이 일치할 것으로예상했다.

여기서 핵심은 `employees` 테이블의 `rows` 컬럼의 값이 `233`이고, `filtered` 컬럼 값이 `100%`라는 것이다.

`condition_fanout_filter` 최적화를 다시 활성화해서 실행 계획을 조회해보자.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/50f06eb7-388d-4997-9fc7-dce0b182f688)

`rows 컬럼`의 값은 `233`으로 같지만, `filtered` 컬럼의 값이 `26.03`으로 변경된 것을 알 수 있다.

`옵티마이저`는 인덱스를 사용할 수 있는 `first_name` 컬럼 조건 이외의 나머지 조건에 대해서도 얼마나

`조건`을 `충족`할지를 고려했다는 이야기가 된다.

즉, `옵티마이저`가 조건을 만족하는 `레코드 건수`를 정확하게 예측할 수 있다면 **더 빠른 실행 계획**을 만들 수 있다.

그렇다면  `filtered` 컬럼의 값을 어떻게 예측해내는 것일까?

`8.0 버전`에서는 `condition_fanout_filter` 최적화가 활성화되면 다음과 같은 조건을 만족하는 컬럼의 

조건들에 대해 조건을 만족하는 `레코드`의 **비율을 계산**할 수 있다.

1. `WHERE 조건절`에 사용된 컬럼에 대해 **인덱스가 있는 경우**
2. `WHERE 조건절`에 사용된 컬럼에 대해 **히스토그램이 존재하는 경우**

실제 쿼리를 실행하면 `ix_fristname` 인덱스만 사용한다.

하지만 `실행 계획`을 수립할 때는 `first_name` 컬럼의 인덱스를 이용해 조건에 일치하는 레코드 건수가

대략 **233건** 정도라는 것을 알아낸다.

`hire_date` 조건을 만족하는 레코드의 비율이 대략 `26.03%`일 것으로 예측했는데,

`hire_date` 컬럼에 인덱스가 없었다면 `옵티마이저`는 `first_name` 컬럼의 인덱스를 이용해 `hire_date` 

컬럼 값의 분포도를 살펴보고 **filtered 컬럼의 값을 예측**한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/b761bb1e-34e0-4192-91ad-4f7b4c4020b4)

`condition_fanout_filter` 최적화 기능은 쿼리의 실행 계획 수립에 **더 많은 시간과 컴퓨팅 자원을 소모**한다.

따라서, `쿼리` 실행 계획이 잘못된 선택을 한 적이 별로 없다면 `성능 향상`에 도움이 되지 않을 수 있다.

`MySQL 서버`가 처리하는 쿼리의 빈도가 매우 높다면 `실행 계획 수립`에 추가되는 `오버헤드`가 더 클 수 있으므로

**업그레이드 하기 전 성능 테스트**를 하는 것이 좋다.

### 🌐 파생 테이블 머지(derived_merge)

예전 버전의 `MySQL 서버`에서는 다음과 같이 `FROM` 절에 사용된 서브쿼리는 먼저 실행해서

그 결과를 **임시 테이블로 만든 다음 외부 쿼리 부분을 처리**했다.

```sql
mysql> EXPLAIN
       SELECT * FROM (
         SELECT * FROM employees WHERE first_name='Matt'
       ) derived_table
       WHERE derived_table.hire_date='1986-04-03';
```

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/daa17835-1305-494b-a580-5bfc82f7dd01)

실행 계획을 보면 `employees` 테이블을 읽는 라인의 `select_type`이 `DERIVED`라고 표시된다.

`employees` 테이블에서 `where` 조건에 맞는 레코드들만 읽어서 임시 테이블을 만든 것이다.

그 다음 임시 테이블을 다시 읽고 `hire_date` 컬럼의 값이 `‘1986-04-03’`인 레코드만 걸러내어 

반환한 것이다. 그래서 `FROM` 절에 사용된 서브 쿼리를 `파생 테이블(Derived Table)`이라고 부른다.

`임시 테이블`을 생성하고 데이터를 `INSERT` 한 뒤 다시 읽기 때문에, 레코드를 복사하고 읽는

`오버헤드`가 더 추가되게 된다.

`임시 테이블`이 메모리에 상주할 수 있다면 다행이지만 `레코드`가 많아서 디스크에 써야 하는 상황이 

온다면 `쿼리`의 `성능`이 많이 느려질 수 있다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/7a3f78d1-e175-4755-881e-3ec3c5af35ab)

`MySQL 5.7` 버전부터는 파생 테이블로 만들어지는 `서브쿼리`를 외부 쿼리와 병합해서 서브쿼리 부분을

제거하는 **최적화가 도입**되었다.

`derived_merge` 최적화 옵션은 임시 테이블 최적화를 활성화할지 여부를 결정한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/e9ff4efc-e946-4099-8249-f786a467c08e)

실행 계획을 살펴보면 `select_type` 컬럼이 `SIMPLE`로 변경되었다.

`SHOW WARNINGS` 명령으로 `옵티마이저`가 **새로 작성한 쿼리**를 살펴보면 아래와 같다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/44b53b9d-a70d-454d-a944-637d50f88faf)

구 버전의 `MySQL 서버`에서는 **서브쿼리로 작성된 쿼리를 외부 쿼리로 병합하는 작업**을 대부분 `수작업`으로 처리했었다.

`옵티마이저`가 처리 할 수 있는 쿼리를 굳이 `수작업`으로 처리해 줄 필요는 없어졌지만

모든 `쿼리`에 대해 `옵티마이저`가 서브쿼리를 외부 쿼리로 병합할 수 있는 것은 아니다.

아래와 같은 조건들은 자동으로 `서브쿼리`를 `외부 쿼리`로 병합할 수 없기 때문에, 수동으로 병합해서

작성하는 것이 **쿼리의 성능에 도움**이 될 것이다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/ea62ca0a-aa54-4988-a596-dd0e7f3f7d7d)

### 😝 인비저블 인덱스(use_invisible_indexes)

- 8.0 버전부터 `인덱스`의 **가용 상태를 제어할 수 있는 기능**이 추가 되었다.
- 8.0 이전까지는 `인덱스`가 존재하면 `옵티마이저`는 **항상 인덱스를 검토하고 사용**해왔다.
- 8.0 버전부터는 `인덱스`를 삭제하지 않고, `인덱스`를 사용하지 못하게 **제어하는 기능을 제공**한다.

`인덱스`의 **가용 상태**는 아래와 같은 `명령어`로 변경할 수 있다.

```sql
ALTER TABLE ... ALTER INDEX ... [VISIBLE | INVISIBLE]
```

`employees` 테이블의 인덱스 가용 상태를 변경해보자

```sql
-- 옵티마이저가 ix_hiredate 인덱스를 사용하지 못하게 변경
mysql> ALTER TABLE employees ALTER INDEX ix_hiredate INVISIBLE;

-- 옵티마이저가 ix_hiredate 인덱스를 사용할 수 있게 변경
mysql> ALTER TABLE employees ALTER INDEX ix_hiredate VISIBLE;
```

`use_invisible_indexes` 옵션을 사용하면 `INVISIBLE` 상태를 가진 인덱스라고 하더라도

**옵티마이저가 사용할 수 있도록 제어**할 수 있다.

```sql
-- use_invisible_indexes의 기본 값은 off 이다.
mysql> SET optimizer_switch='use_invisible_indexes=on';
```

### 😿 스킵 스캔(skip_scan)

- `인덱스`의 핵심은 값이 정렬되어 있다는 것이기 때문에, `인덱스`를 구성하는 **컬럼의 순서가 중요**하다.
- `인덱스 스킵 스캔`은 제한적이지만 인덱스의 **제약 사항을 뛰어넘을 수 있는 최적화 기법**이다.

먼저 `employees` 테이블에 인덱스를 생성해서 간단한 테스트를 해보자.

```sql
mysql> ALTER TABLE employees
        ADD INDEX ix_gender_birthdate (gender, birth_date);
```

이 인덱스를 사용하려면 `where` 조건절에 `gender` 컬럼에 대한 비교 조건이 필수적이다.

```sql
-- 인덱스를 사용하지 못하는 쿼리 -> gender 컬럼에 대한 비교 없음
mysql> SELECT * FROM employees WHERE birth_date>='1965-02-01';

-- 인덱스를 사용하는 쿼리 -> gender 컬럼에 대한 비교 있음
mysql> SELECT * FROM employees WHERE gender='M' AND birth_date>='1965-02-01';
```

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/d4f3db48-afa1-4c1a-a74b-1a164fb7e1bf)

`선행 컬럼`에 대한 비교가 없는 첫 번째 쿼리는 인덱스를 사용할 수 없으므로, 이런 경우에는

`birth_date` 컬럼부터 시작하는 **인덱스를 생성**해야 했다.

8.0 버전부터는 `인덱스 스킵 스캔` 최적화가 도입되어, 인덱스의 **선행 컬럼이 조건절에 사용되지 않아도**

`후행 컬럼`의 조건만으로도 **인덱스를 이용한 쿼리 성능 개선**이 가능하다.

`인덱스 스킵 스캔`은 첫 번째 쿼리를 `gender 컬럼`의 조건이 있는 것 처럼 쿼리를 `최적화`한다.

만약 `인덱스`의 **선행 컬럼이 매우 다양한 값을 가지는 경우에는 인덱스 스킵 스캔이 비효율적**이다.

따라서, `옵티마이저`는 인덱스의 선행 컬럼이 소수의 `유니크`한 값을 가질 때만 `인덱스 스킵 스캔` 최적화 사용한다.

```sql
-- 현재 세션에서 인덱스 스킵 스캔 최적화를 활성화
mysql> SET optimizer_switch='skip_scan=on';

-- 현재 세션에서 인덱스 스킵 스캔 최적화를 비활성화
mysql> SET optimizer_switch='skip_scan=off';

-- 특정 테이블에 대해 인덱스 스킵 스캔을 사용하도록 힌트를 사용
mysql> SELECT /*+ SKIP_SCAN(employees)*/ COUNT(*)
       FROM employees
       WHERE birth_date>='1965-02-01';
       

-- 특정 테이블과 인덱스에 대해 인덱스 스킵 스캔을 사용하도록 힌트를 사용
mysql> SELECT /*+ SKIP_SCAN(employees ix_gender_birthdate)*/ COUNT(*)
       FROM employees
       WHERE birth_date>='1965-02-01';
       

-- 특정 테이블에 대해 인덱스 스킵 스캔을 사용하지 않도록 힌트를 사용
mysql> SELECT /*+ NO_SKIP_SCAN(employees)*/ COUNT(*)
       FROM employees
       WHERE birth_date>='1965-02-01';
```

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/e546a35c-9211-484e-aa6c-0883785403e9)

`인덱스 스킵 스캔`을 사용하도록 `옵티마이저`에게 힌트를 주었을 때,

`Extra` 컬럼에 **Using index for skip scan**이 표시되는 것을 볼 수 있다.

### 😄 해시 조인(hash_join)

- `8.0.18` 버전부터 MySQL 서버도 `해시 조인`이 추가 지원되기 시작했다.

`해시 조인`이 항상 `Nested Loop Join` 보다 **빠를 거라는 기대**를 대부분의 사용자가 가지고 있다.

하지만 이는 항상 `옳은 이야기`는 아니다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/1050aa63-7fbf-437c-b4cf-60aeb3ab44ce)

`Nested Loop Join`과 해시 조인은 같은 지점에서 시작했지만, `해시 조인`이 먼저 끝나는 것을 볼 수 있다.

`조인 조건`에 `일치`하는 `마지막 레코드`를 찾았다고 해서 **항상 쿼리가 완료되는 것은 아니다.**

`해시 조인`은 첫 번째 레코드를 찾는 데 시간이 많이 걸리지만, 최종 레코드를 찾는 데는 시간이 많이 걸리지 않는다.

`Nested Loop Join`은 첫 번째 레코드를 빨리 찾고, `최종 레코드`를 찾는데 시간이 많이 걸린다.

`해시 조인 쿼리`는 **Best Throughput** 전략에 적합하고, `Nested Loop Join`은 **Best-Respose-time** 전략에 적합하다.

일반적인 `웹 서비스`는 **응답 속도**가 더 중요하고, `분석`과 같은 서비스는 **처리 소요 시간**이 중요하다.

`MySQL 서버`는 대부분 `온라인 트랜잭션 처리`를 위해 사용되기 때문에, **응답 속도에 더 집중**한다.

따라서, `조인 조건`의 `컬럼`이 `인덱스`가 없거나 조인 대상 테이블 중 일부의 **레코드 건수가 매우 적은 경우** 등에 대해서만 **해시조인 알고리즘을 사용**한다.

`MySQL 서버`의 해시 조인 최적화는 **Nested Loop Join이 사용되기에 적합하지 않은 경우**를 위한

`차선책` 정도로 생각하는 것이 좋다.

`8.0.17` 버전까지는 조인 조건이 좋지 않은 경우 **Block Nested Loop 조인 알고리즘을 사용**했다.

인덱스가 잘 설계된 `데이터베이스`에서는 **Block Nested Loop 실행 계획**은 거의 볼 수 없었다.

여기서 `“블록”`이란 `join_buffer_size`라는 시스템 변수로 크기를 조정할 수 있는 메모리 공간을 의미한다.

`조인 대상 테이블`의 레코드 크기가 `조인 버퍼`보다 크다면 **드라이빙 테이블을 여러 번 반복해서 읽어야 하는 문제가 발생**한다.

`8.0.18(8.0.19)` 버전에서는 `동등 조인`을 위해서는 **해시 조인**이 사용되고,

`안티 조인`이나 `세미 조인`을 위해서는 **블록 네스티드 루프 조인**이 사용되었다.

`8.0.20` 버전부터는 **블록 네스티드 루프 조인**은 더이상 사용되지 않고, `네스티드 루프 조인`을 사용될 수 없는 경우 항상 **해시 조인이 사용**되도록 바뀌었다.

`8.0.20` 버전부터는 `block_nested_loop` 옵티마이저 설정이나, `BNL` 또는 `NO_BNL` 힌트 등으로

`블록 네스티드 루프`가 아닌 **해시 조인을 유도하는 목적으로 사용**한다.

`INDEX 힌트`로 인덱스를 사용하지 못하게 하면 `쿼리`가 어떻게 `실행`되는지 확인해보자

```sql
mysql> EXPLAIN
       SELECT * 
       FROM employees e IGNORE INDEX(PRIMARY, ix_hiredate)
        INNER JOIN dept_emp de IGNORE INDEX(ix_empno_fromdate, ix_fromdate)
         ON de.emp_no=e.emp_no AND de.from_date=e.hire_date;
```

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/8621153a-d824-46ec-9926-77d6af433381)

`옵티마이저`는 적절한 인덱스가 없으므로 `해시 조인`을 사용하는 것을 볼 수 있다.

일반적으로 `해시 조인`은 **빌드 단계**와 **프로브 단계**로 나뉘어 처리된다.

`빌드 단계`에서는 `조인 대상 테이블` 중에서 **레코드 건수가 적어서 해시 테이블로 만들기 용이한 테이블**을

골라서 `메모리`에 **해시 테이블을 생성**하는 작업을 수행한다.

`프로브 단계`는 나머지 테이블의 레코드를 읽어서 `해시 테이블`의 **일치 레코드를  찾는 과정**을 의미한다.

이 `실행 계획`으로는 어느 테이블이 `빌드 테이블`이고 `프로브 테이블`인지 식별하기 어렵다.

이런 경우에는 `EXPLAIN FORMAT`을 **TREE**로 주거나 `EXPLAIN ANALYZE` **명령을 사용**하면 좋다.

```sql
mysql> EXPLAIN FORMAT=TREE
       SELECT * 
       FROM employees e IGNORE INDEX(PRIMARY, ix_hiredate)
        INNER JOIN dept_emp de IGNORE INDEX(ix_empno_fromdate, ix_fromdate)
         ON de.emp_no=e.emp_no AND de.from_date=e.hire_date;
```

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/152cf271-2006-4f2e-9b87-97629c3d98b3)

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/deac5533-5047-4454-8d92-0816abff68a2)

`TREE 포맷`의 실행 계획 기준으로 보면, `최하단` 제일 안쪽의 `dept_emp` 테이블이 **빌드 테이블로 선택** 된 것을 볼 수 있다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/05e8888f-2ac1-48ff-9ffb-8b62bb79b9bd)

이 그림은 `조인 버퍼`의 공간이 적당한 **일반적인 해시 조인의 처리 과정**을 보여준다.

`해시 테이블`을 메모리에 저장할 때 `조인 버퍼`의 공간이 부족한 경우, `빌드 테이블`과 `프로브 테이블`을

적당한 크기의 **청크로 분리한 다음, 청크별로 해시 조인을 처리**한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/41462d5b-8cb9-4d06-96bf-93057bc0435e)

이 그림을 보면, `해시 조인` 처리 방법이 복잡해보인다.

만들어질 `해시 테이블`이 설정 된 `메모리 크기`보다 큰지를 알 수 없기 때문에 이와 같이 처리된다.

`MySQL 서버`는 `dept_emp` 테이블을 읽으면서 **메모리의 해시 테이블을 준비**하다가 지정된 메모리 크기를 넘어서면 `dept_emp 테이블`의 **나머지 레코드를 디스크에 청크로 구분해서 저장**한다.

그리고 `employees` 테이블의 `emp_no` 값을 이용해 메모리의 `해시 테이블`을 검색해서 1차 조인 결과를 생성한다.

동시에 `employees` 테이블에서 **읽은 레코드를 디스크에 저장**한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/a25e57f5-5163-41ec-9fdd-29809a44a37a)

이 그림을 보면, `디스크`에 **2개의 그룹으로 구분된 청크 목록**이 표현되어 있다.

`빌드 테이블 청크`는 **dept_emp의 레코드를 저장**하고 `프로브 테이블 청크`는 **employees의 레코드를 저장**하는 공간이다.

`1차 조인`이 완료되면, 디스크에 저장된 `빌드 테이블 청크`에서 첫 번째 청크를 읽어서 `메모리 해시 테이블`을 구축하고, `프로브 테이블 청크`에서 새로 구축된 메모리 `해시 테이블`과 `조인`을 수행 후 **2차 결과를 반환**한다.

`MySQL 서버`는 청크 단위로 조인을 수행하기 위해서 `2차 해시 함수`를 이용한다.

`옵티마이저`는 **빌드 테이블의 크기에 따라서 다양한 해시 조인 알고리즘을 사용**한다.

`메모리`에서 모두 처리 `가능`한 경우에는 **클래식 해시 조인 알고리즘**을 사용하고,

`메모리`에서 모두 처리가 `불가능`하면 **그레이스 해시 조인 알고리즘**을 **하이브리드**하게 활용한다.

`해시 조인`에서 해시 키를 만들 때 `xxHash64` 함수를 사용하는데, 이 함수는 매우 빠르고 해시된 값의 분포도도 훌륭한 `알고리즘`이다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/3254effe-32aa-415a-a70e-dab125af64bf)

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/91002a87-d15d-4bee-947d-e85da845e7f9)

### 🤩 인덱스 정렬 선호(prefer_ordering_index)

- `MySQL 옵티마이저`는 **ORDER BY 또는 GROUP BY**를 `인덱스`를 `사용`해 처리할 수 있다면 쿼리의 실행 계획에서 **인덱스의 가중치를 높이 설정**해서 `실행`한다.

```sql
mysql> EXPLAIN
       SELECT *
       FROM employees
       WHERE hire_date BETWEEN '1985-01-01' AND '1985-02-01'
       ORDER BY emp_no;
```

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/3a90b085-c9b8-4aee-a9e0-3b6b43ceaac2)

이 쿼리는 대표적으로 `2가지` 실행 계획이 `선택 가능`하다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/1a2617c4-381c-42b7-bfbd-d5aa7397bef5)

상황에 따라 1번이 효율적일 수도 있고 2번이 효율적일 수도 있다.

`hire_date` 컬럼의 **조건에 부합되는 레코드 건수**가 많지 않다면 `1번`이 `효율`적일 것이다.

`옵티마이저`는 현재 2번 실행계획을 선택하고 있다.

`옵티마이저`가 체크해야 하는 레코드 건수가 많음에도 불구하고 **잘못된 실행 계획을 선택**했다.

`옵티마이저`가 `ORDER BY`를 위한 인덱스에 너무 가중치를 줘서 실수를 자주 한다면

**두 가지 방법으로 개선**할 수 있다.

**MySQL 8.0.20 이전 버전**

- 특정 인덱스를 사용하지 못하도록 `“IGNORE INDEX”` 힌트 사용

**MySQL 8.0.21 버전 ~**

- `prefer_ordering_index` 옵티마이저 옵션 **비활성화**

```sql
-- 현재 커넥션에서만 prefer_ordering_index 옵션 비활성화
mysql> SET SESSION optimizer_switch='prefer_ordering_index=OFF';

-- 현재 쿼리에 대해서만 prefer_ordering_index 옵션 비활성화
mysql> SELECT /*+ SET_VAR(optimizer_switch='prefer_ordering_index=OFF') */
```

### 🤓 조인 최적화 알고리즘

- `테이블`의 개수가 많아지면 최적화된 `실행 계획`을 찾는 것이 상당히 어려워진다.
- 하나의 `쿼리`에서 조인되는 테이블의 개수가 많아지면 `실행 계획`을 수립하는 데만 몇 분이 걸릴 수 있다.

`MySQL`에는 최적화된 조인 실행 계획 수립을 위한 `2가지` 알고리즘이 있다.

```sql
mysql> SELECT *
       FROM t1, t2, t3, t4
       WHERE ...
```

이 `쿼리`가 `알고리즘`에 따라서 어떻게 처리되는지 확인해보자.

### 😅 Exhaustive 검색 알고리즘

- `MySQL 5.0` 버전과 그 이전 버전에서 사용되던 `조인 최적화 기법`이다.
- `FROM 절`에 명시된 모든 테이블의 조합에 대해 **실행 계획의 비용을 계산해서 최적의 조합 1개를 찾는 방법**이다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/7c5782a0-c464-43bb-9b41-e02b703b7edd)

예를 들어 테이블이 `20`개라면 이 방법으로 처리했을 경우 가능한 조인 조합이 `20 factorial`에 달한다.

이전 버전에서 사용되던 `알고리즘`은 테이블이 10개만 넘어도 **실행 계획 수립에 많은 시간이 걸린다.**

### 😱 Greedy 검색 알고리즘

- `Exhaustive` 알고리즘의 **시간 소모적인 문제를 해결**하기 위해 `5.0 버전` 부터 도입되었다.
- `Exhaustive` 알고리즘보다는 **조금 복잡한 형태로 최적의 조인 순서를 결정**한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/0efc31b2-1cd8-4b74-a5fd-34b0ce3c01ea)

`Greedy` 검색 알고리즘으로 `t1~t4` 테이블의 조인을 처리하면 아래와 같은 방식으로 처리된다.

1. 전체 `N개`의 테이블 중에서 `optimizer_search_depth` 시스템 변수에 정의된 개수의 테이블로 가능한 조인 조합을 생성한다.
2. 1번에서 생성된 `조인 조합` 중에서 **최소 비용의 실행 계획을 하나 선정**한다.
3. 선정된 `실행 계획`의 첫 번째 테이블을 부분 실행 계획의 `첫 번째 테이블`로 선정한다.
4. 전체 `N-1개`의 테이블 중 `optimizer_search_depth` 시스템 변수에 정의된 개수의 테이블로 가능한 조인 조합을 생성한다.
5. 생성된 `조인 조합`들을 하나씩 3번에서 생성된 **부분 실행 계획에 대입해 실행 비용을 계산**한다.
6. 최적의 `실행 계획`에서 두 번째 테이블을 부분 실행 계획의 `두 번째 테이블`로 선정한다.
7. 남은 테이블이 모두 없어질 때까지 `4~6번` 과정을 반복하면서 **부분 실행 계획에 테이블 조인 순서를 기록**한다.
8. 최종적인 `부분 실행 계획`이 테이블의 `조인 순서`로 결정된다.

`Greedy` 검색 알고리즘은 `optimizer_search_depth` 시스템 변수에 설정된 값에 따라 조인 최적화의 비용이 상당히 줄어들 수 있다. 

`MySQL`에서는 조인 최적화를 위한 시스템 변수로 `optimizer_prune_level`과 `optimizer_search_depth`를 제공된다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/725d6002-7f2b-4c8d-af20-1884aa2062d4)

`테이블 조인`이 많은 쿼리의 `실행 계획` 수립이 얼마나 느려질 수 있는지,

`조인 최적화`를 위한 `시스템 변수`를 조정해서 얼마나 빨라질 수 있는지 살펴보자.

`테스트`를 위해 아래와 같은 `스펙`의 테이블을 **tab01~tab30** 까지 `30개` 생성한다.

이 중 몇 개의 테이블은 `INT`와 `BIGINT` 타입의 컬럼을 가지고 몇 개의 테이블은 `보조 인덱스`를 제거했다.

테이블당 `2000개` 정도의 데이터를 `INSERT` 한 상태라고 가정하자.

```sql
mysql> CREATE TABLE tab01(
         fd1 char(20) NOT NULL,
         fd2 char(20) DEFAULT NULL,
         PRIMARY KEY (fd1),
         KEY ix_fd2 (fd2)
       );
```

이제 `쿼리`의 실행 계획을 확인해보자.

```sql

mysql> SET SESSION optimizer_prune_level = {0 | 1};
mysql> SET SESSION optimizer_search_depth = { 1 | 5 | 10 | 15 | 20 | 25 | 30 | 35 | 40 | 62 };
```

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/47372ae0-3cb4-4f03-98aa-995271ffd7fb)

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/a54ed627-2159-4ff6-94de-173fd1c8a81c)


위 쿼리의 `실행 계획 수립`에 걸린 시간은 `0.01`초 정도 소요되었다.

`MySQL` 5.1 버전에서는 조인 순서 결정에 `Heuristic` 최적화를 적용해도 `optimizer_search_depth` 변수 값이 증가하면 `실행 계획 수립`에 `1초` 넘는 시간이 걸렸다.

결과적으로, `MySQL` 서버의 **조인 최적화나 딕셔너리 정보 검색 성능이 버전이 올라감에 따라 많이 개선**되었다.

`optimizer_prune_leve`l 세션 변수의 값을 `0`으로 고정하고, `optimizer_search_depth` 변수의 값을 **1부터 5씩 증가시키면서 실행 계획 수립에 걸리는 시간을 확인**해봤다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/971d5c8b-2060-487c-ac29-87dd44c4cff2)

`MySQL 8.0` 버전의 조인 최적화는 많이 개선되어 `optimizer_search_depth` 변수의 값에는 큰 영향을 받지 않는다. 하지만 `optimizer_prune_level`을 `0`으로 설정하면 소요되는 시간이 급증한다.

즉, `8.0` 버전에서는 조인 최적화와 관련된 `휴리스틱`을 비활성화 할 필요가 거의 없어졌다는 뜻이다.

`optimizer_prune_level`의 기본 값은 `1`이므로 시스템 변수의 조정은 더 이상 필요하지 않을 것이다.

### 🙃 쿼리 힌트

- `MySQL` 서버는 우리가 서비스하는 비즈니스를 **100% 이해하지 못한다.**
- 서비스 개발자나 `DBA`보다 `MySQL` 서버가 **부족한 실행 계획을 수립할 때가 있다.**

MySQL 서버에서 사용 가능한 `쿼리 힌트`는 **두 가지**가 있다.

- `인덱스 힌트`
- `옵티마이저 힌트`

인덱스 힌트는 `“USE INDEX”` 같은 힌트를 말하며, `옵티마이저 힌트`는 **5.6 버전부터 새롭게 추가**된

힌트들을 의미한다. 

### 😛 인덱스 힌트

- `“STRAIGHT_JOIN”`과 `“USE INDEX”` 등은 **옵티마이저 힌트가 도입되기 전 사용되던 기능**들이다.
- `인덱스 힌트`는 모두 SQL의 문법에 맞게 사용해야 하기 때문에 `ANSI-SQL 표준`에 **어긋난다는 단점**이 있다.

`옵티마이저` 힌트들은 다른 `RDBMS`에서 주석으로 인식한다.

따라서 `ANSI-SQL` 표준을 준수한다고 할 수 있으므로, 가능하다면 `옵티마이저 힌트`를 사용하는 것이 좋다.

인덱스 힌트는 `SELECT` 명령과 `UPDATE` 명령에서만 사용이 가능하다.

### 😡 STRAIGHT_JOIN

- `STRAIGHT_JOIN`은 옵티마이저 힌트인 동시에 **조인 키워드**이기도 하다.
- SELECT, UPDATE, DELETE 쿼리에서 여러 개의 테이블이 조인되는 경우 `조인 순서`를 `고정`하는 `역할`을 한다.

```sql
mysql> EXPLAIN
       SELECT * 
       FROM employees e, dept_emp de, departments d
       WHERE e.emp_no=de.emp_no AND d.dept_no=de.dept_no;
```

이런 `쿼리`를 실행하면 어느 테이블이 `드라이빙 테이블`이 되고 어느 테이블이 `드리븐 테이블`일지 알 수 없다.

`옵티마이저`가 각 테이블의 통계 정보와 `쿼리`의 `조건`을 기반으로 **조인 순서**를 정할 것이기 때문이다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/5ff5dbc0-02ca-473d-9087-c84ef14aab46)

이 쿼리의 실행 계획을 보면, `departments` 테이블을 드라이빙 테이블로 선택했다.

두 번째로 `dept_emp` 테이블을 읽고 마지막으로 `employees` 테이블을 읽었다.

일반적으로 `조인`을 하기 위한 `컬럼`들의 인덱스 여부로 조인 순서가 결정되므로, `레코드`가 가장 적은 테이블인

`departments` 테이블이 **드라이빙 테이블로 선택**되었다.

이 쿼리의 조인 순서를 변경하려면 `STRAIGHT_JOIN` 힌트를 사용할 수 있다.

`인덱스 힌트`는 사용해야 하는 위치가 이미 결정되었으므로 **다른 위치에서 사용하면 안된다.**

```sql
mysql> SELECT STRAIGHT_JOIN
         e.first_name, e.last_name, d.dept_name
       FROM employees e, dept_emp de, departments d
       WHERE e.emp_no=de.emp_no
         AND d.dept_no=de.dept_no;
         

mysql> SELECT /*! STRAIGHT_JOIN */
         e.first_name, e.last_name, d.dept_name
       FROM employees e, dept_emp de, departments d
       WHERE e.emp_no=de.emp_no
         AND d.dept_no=de.dept_no;
```

`STRAIGHT_JOIN` 힌트는 `옵티마이저`가 **FROM 절에 명시된 테이블의 순서대로 조인을 수행하도록 유도**한다.

**employees → dept_emp → departments** 순서로 조인을 수행한다는 것을 알 수 있다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/c78b774d-c21a-4c24-b91a-9b75f855ea41)

주로 아래의 기준에 맞게 `조인 순서`가 결정되지 않는 경우에만 `STRAIGHT_JOIN` 힌트를 사용하여

조인 순서를 조정해주는 것이 좋다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/e170807f-0b0e-4240-85c4-6912ee718e37)

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/1181f9d7-fe18-44af-9a76-2c83a84d067a)

여기서 `레코드 건수`라는 것은 인덱스를 사용할 수 있는 `WHERE 조건`까지 포함해서 조건을 만족하는 

레코드의 수를 의미하는 것이지, **테이블 전체의 레코드 수를 의미하는 것은 아니다.**

예를 들어, `employees` 테이블의 레코드 건수가 훨씬 많지만 조건을 만족하는 레코드의 건수는 적다면

실행계획을 살펴보고 `employees` 테이블을 `드라이빙`되게 하는 것이 좋다.

```sql
mysql> SELECT 
         e.first_name, e.last_name, d.dept_name
       FROM employees e, departments d, dept_emp de
       WHERE e.emp_no=de.emp_no
         AND d.dept_no=de.dept_no
         AND e.emp_no=10001;
```

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/d5ea406e-33a1-4a7e-8ec6-833a9ab16b5a)

`STRAIGHT_JOIN` 힌트를 적용했을 때와 하지 않았을 때 모두 `employees` 테이블을 드라이빙 테이블로

선택하는 것을 볼 수 있다.

`STRAIGHT_JOIN` 힌트와 비슷한 역할을 하는 `옵티마이저 힌트`는 아래와 같은 것들이 있다.

- **JOIN_FIXED_ORDER**
- **JOIN_ORDER**
- **JOIN_PREFIX**
- **JOIN_SUFFIX**

`JOIN_FIXED_ORDER` 옵티마이저 힌트는 `STRAIGHT_JOIN`과 완전히 같은 효과를 낸다.

나머지 3개의 `옵티마이저 힌트`는 **일부 테이블의 조인 순서에 대해서만 제안하는 힌트**다.

### 😾 USE INDEX / FORCE INDEX / IGNORE INDEX

- 조인의 순서를 변경하는 것 다음으로 **자주 사용**되는 것이 `인덱스 힌트`이다.
- `STRAIGHT_JOIN` 힌트와는 달리 인덱스 힌트는 사용하려는 `인덱스`를 가지는 **테이블 뒤에 힌트를 명시**해야 한다.

`MySQL 옵티마이저`는 어떤 인덱스를 사용해야 할지를 **무난하게 잘 선택하는 편**이다.

간혹 `3~4개` 이상의 컬럼을 포함하는 **비슷한 인덱스가 여러 개 존재하는 경우**에는 `옵티마이저`도 가끔 실수를 한다.

이런 경우에는 강제로 `특정 인덱스`를 사용하도록 `힌트`를 `추가`해야 한다.

`인덱스 힌트`는 크게 아래와 같이 3가지 종류가 있다. `인덱스 힌트`는 모두 키워드 뒤에 사용할 인덱스의 이름을

괄호로 묶어서 사용하며 `힌트`의 **문법 오류**는 `쿼리 문법 오류`로 `처리`된다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/e1fef9f9-2541-42a4-80a5-e0184b9b76cd)

별도로 사용자가 부여한 이름이 없는 PK는 `“PRIMARY”`라고 `명시`하면 된다.

`인덱스 힌트`를 사용할 때 모두 용도를 명시해 줄 수 있고, 이는 `선택 사항`이다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/47d532b0-f5c9-4c80-8218-958a248cf8a7)

`ORDER BY`나 `GROUP BY` 작업에서 인덱스를 사용할 수 있다면 나은 성능을 보장한다.

용도는 대부분의 경우에 `옵티마이저`가 대부분 최적으로 선택하기 때문에 고려하지 않아도 괜찮다.

```sql
mysql> SELECT * FROM employees WHERE emp_no=10001;
mysql> SELECT * FROM employees FORCE INDEX(primary) WHERE emp_no=10001;
mysql> SELECT * FROM employees USE INDEX(primary) WHERE emp_no=10001;

mysql> SELECT * FROM employees IGNORE INDEX(primary) WHERE emp_no=10001;
mysql> SELECT * FROM employees FORCE INDEX(ix_firstname) WHERE emp_no=10001;
```

`첫 번째` 부터 `세 번째`까지의 쿼리는 **동일한 실행 계획으로 쿼리를 처리**한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/cb3d59df-08f8-4a98-9e25-8cf8a5acc517)

`옵티마이저`도 인덱스 힌트가 없어도 `조건절`을 보고 `PK`를 사용하는 것이 `최적`이라는 것을 `인식`하기 때문이다.

`네 번째` 쿼리는 `인덱스`를 사용하지 못하도록 강제해서 **풀 테이블 스캔 실행계획이 수립**되는 것을 볼 수 있다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/6a15ecba-2a42-477c-b408-617510fc7e36)

`다섯 번째` 쿼리도 **전혀 관계 없는 인덱스를 사용**하도록 했더니 `풀 테이블 스캔` 실행계획이 `수립`된다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/64807d81-6238-4e5c-8015-acb220fc631b)

여기서 알 수 있는 것은, 터무니 없는 힌트를 주어도 `옵티마이저`가 이대로 실행 계획을 수립한다는 것이다.

`전문 검색 인덱스`가 있는 경우에는 다른 일반 `보조 인덱스`를 사용할 수 있는 경우에도

`옵티마이저`는 전문 검색 인덱스를 사용하는 경우가 많다.

`인덱스`의 사용법이나 좋은 `실행 계획`이 어떤 것인지 판단하기 힘든 상황에서는 `힌트`를 사용해서

강제로 `옵티마이저`의 실행 계획에 영향을 미치는 것은 좋지 않다.

최적의 `실행 계획`은 데이터의 성격에 따라서 변하므로, 가능하다면 `옵티마이저`가 당시 `통계 정보`를 가지고

**선택할 수 있도록 두는 것**이 가장 좋다.

`쿼리`를 서비스에서 없애 버리거나 `튜닝`할 필요가 없게 **데이터를 최소화하는 것**이 `힌트`를 주는 것보다

더 휼륭한 `최적화` 방법이다.

마지막 방법은 `데이터 모델`의 단순화를 통해 `쿼리`를 간결하게 만드는 것인데, 이 세 가지 방법은

`상당한 시간`과 `작업 능력`이 필요해서 **힌트에 의존하는 경우가 실무에서는 꽤 많다.**

### 😛 SQL_CALC_FOUND_ROWS

- `LIMIT`을 사용하면 `LIMIT`에 명시된 수만큼 만족하는 `레코드`를 찾으면 `검색 작업`을 멈춘다.
- `SQL_CALC_FOUND_ROWS` 힌트가 포함된 쿼리는 `LIMIT`에 명시된 수 만큼 만족하는 레코드를 찾아도, 끝까지 검색을 수행한다.

`SQL_CALC_FOUND_ROWS` 힌트가 사용된 쿼리가 실행된 경우, `FOUND_ROWS()` 함수를 이용해

`LIMIT`을 제외한 **조건을 만족하는 레코드**가 전체 몇 건이었는지 알아 낼 수 있다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/11877662-75d3-4374-ac09-16457a769772)

`페이징`에서 이 기능을 사용한다고 했을때 `적합한 방법`일지 검토하기 위해서

`SQL_CALC_FOUND_ROWS`를 이용한 페이징 처리와 `COUNT(*)` 쿼리를 사용하는 예제를 비교해보자.

- `SQL_CALC_FOUND_ROWS` 사용법

```sql
mysql> SELECT SQL_CALC_FOUND_ROWS * FROM employees WHERE first_name='Georgi' LIMIT 0, 20;
mysql> SELECT FOUND_ROWS() AS total_record_count;
```

이 경우는 한 번의 `쿼리 실행`으로 필요한 정보 `2가지`를 가져오는 것 처럼 보이지만, `2번`의 `쿼리`가 나간다.

`LIMIT 조건`이 처음 `20건`만 가져오도록 했지만 **힌트로 인해 조건에 만족하는 레코드 전부를 읽어야 한다.**

현재 이 `조건`을 만족하는 레코드는 `253건`이므로 실제 `데이터 레코드`를 찾아가는 작업을 `253번` 수행해야 하며,

`랜덤 I/O`가 **253번 일어난다.**

- 기존 `2개`의 `쿼리`로 쪼개어 실행하는 방법

```sql
mysql> SELECT COUNT(*) FROM employees WHERE first_name='Georgi';
mysql> SELECT * FROM employees WHERE first_name='Georgi' LIMIT 0, 20;
```

이 방식 또한 `쿼리`가 `2번` 나간다는 것은 같다.

첫 번째 쿼리는 `실제 레코드 데이터`가 필요한 것이 아니므로 **랜덤 I/O는 발생하지 않는다.**

두 번째 쿼리는 `ix_firstname` 인덱스를 `레인지 스캔`으로 접근한 뒤, 실제로 레코드를 읽어야 하기 때문에

`랜덤 I/O`가 발생한다.

이 쿼리는 `LIMIT 0, 20`의 제한이 있기 때문에 `20번`만 `랜덤 I/O`가 발생한다.

이렇게 비교해 보았을 때 `SQL_CALC_FOUND_ROWS`를 사용하면 **성능에 좋지 않다는 것**을 알 수 있다.

`SELECT` 쿼리 문장이 `UNION`으로 연결 된 경우에는 **정확한 레코드 건수를 가져올 수 없다는 문제점**도 존재한다.

`SQL_CALC_FOUND_ROWS` 힌트는 성능이 아닌 **개발자의 편의를 위해 만들어진 힌트**다.

따라서 `페이징 처리`는 적절한 `인덱스 튜닝`만 되어 있다면 **일반적인 방법이 더 성능이 좋다.**

`인덱스`가 준비되지 않은 경우에는 `SQL_CALC_FOUND_ROWS`로 처리하는 것이 더 빠를 수 있으나,

`쿼리`나 `인덱스`를 `튜닝` 하는 것이 훨씬 **더 빠른 결과를 만들어 낼 수 있을 것이다.**

따라서, `SQL_CALC_FOUND_ROWS` 보다는 **레코드 카운터용 쿼리**와 **데이터를 조회하는 쿼리**를

`분리`하는 것이 더 `효율적`이다.

### 😛 옵티마이저 힌트 힌트 종류

`옵티마이저 힌트`는 영향 범위에 따라 `4개 그룹`으로 나눠볼 수 있다.

- `인덱스` : 특정 인덱스의 이름을 사용할 수 있는 옵티마이저 힌트
- `테이블` : 특정 테이블의 이름을 사용할 수 있는 옵티마이저 힌트
- `쿼리 블록` : 특정 쿼리 블록에서 사용할 수 있는 옵티마이저 힌트로서, 특정 쿼리 블록의 이름을 명시하는 것이 아니라 **힌트가 명시된 쿼리 블록에 대해서만 영향을 미치는 옵티마이저 힌트**
- `글로벌(쿼리 전체)` : 전체 쿼리에 대해서 영향을 미치는 힌트

이 `구분`으로 인해서 `힌트`의 `사용 위치`가 **달라지는 것은 아니다.**

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/cb2f1ef7-5e6c-4161-a168-58f11fffbac1)

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/b1dcc84b-ec07-4f16-b9e5-33e95c476c4a)

모든 `인덱스` 수준의 힌트는 **반드시 테이블 명이 선행**되어야 한다.

```sql
mysql> EXPLAIN
       SELECT /*+ INDEX(employees ix_firstname) */ *
       FROM employees
       WHERE first_name='Matt';
       

mysql> EXPLAIN
       SELECT /*+ NO_INDEX(employees ix_firstname) */ *
       FROM employees
       WHERE first_name='Matt';
```

`옵티마이저 힌트`가 문법에 맞지 않게 잘못 사용된 경우에는 `경고 메시지`가 `표시`된다.

익숙하지 않은 힌트 사용시 `EXPLAIN` 명령으로 힌트 `문법 오류`를 확인해보자.

```sql
mysql> EXPLAIN
       SELECT /*+ NO_INDEX(ix_fristname) */ *
       FROM employees
       WHERE first_name='Matt';
```

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/8b82487a-2643-4ec5-be78-cfe8803d09d1)

어떤 `경고`가 있는지 확인해보자.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/ae3e3ce9-efc1-46d8-b0a5-486152bc5a98)

`인덱스`를 가진 `테이블명`을 **명시하지 않아서 생긴 경고**이다.

하나의 `SQL 문장`에서 `SELECT` 키워드는 **여러 번 사용될 수 있다.**

이때 각 `SELECT` 키워드로 시작하는 `서브쿼리 영역`을 **쿼리 블록**이라고 한다.

`특정 쿼리 블록`에 영향을 미치는 `옵티마이저 힌트`는 **외부 쿼리 블록에서 사용**할 수 있다.

`특정 쿼리 블록`을 **외부 쿼리 블록**에서 사용하려면 `QB_NAME()` 힌트를 사용해서 **이름을 부여**해야 한다.

```sql
mysql> EXPLAIN
       SELECT /*+ JOIN_ORDER(e, s@subq1) */
         COUNT(*)
       FROM employees e
       WHERE e.first_name='Matt'
         AND e.emp_no IN (SELECT /*+ QB_NAME(subq1) */ s.emp_no
         FROM salaries s
         WHERE s.salary BETWEEN 50000 AND 50500);
```

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/1662ff23-2c47-4c7c-b7ae-0a5d696a10d5)

이 쿼리는 `서브쿼리`에 사용된 `salaries` 테이블이 `세미 조인 최적화`를 통해 **조인으로 처리될 것을 예상**하고

`JOIN_ORDER` 힌트를 사용해서, 조인의 순서로 `외부 쿼리` 블록의 **employees 테이블**과 `서브쿼리` 블록의 

**salaries** 테이블을 순서대로 `조인`하게 `힌트`를 `사용`한 `쿼리`이다.

### 👿 MAX_EXECUTION_TIME

- `옵티마이저 힌트` 중에서 유일하게 **쿼리의 실행 계획에 영향을 미치지 않는 힌트**이다.
- `쿼리`의 **최대 실행 시간을 설정**하는 힌트로 사용되며, `밀리초 단위`로 **시간을 설정**한다.

```sql
mysql> SELECT /*+ MAX_EXECUTION_TIME(100) */ *
       FROM employees
       ORDER BY last_name LIMIT 1;
```

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/85383ae5-42f2-4fd1-bb06-3f10ef87a59f)

`쿼리`가 지정된 `시간`을 초과하면 **쿼리는 실패**하게 된다.

### 🤣 SET_VAR

- `MySQL 서버`의 **시스템 변수들 또한 쿼리의 실행 계획에 상당한 영향**을 준다.
- `옵티마이저 힌트`로 `부족`한 경우 `optimizer_switch` 시스템 변수를 **제어해야 하는 경우도 생길 수 있다.**

```sql
mysql> EXPLAIN
       SELECT /*+ SET_VAR(optimizer_switch='index_merge_intersection=off') */ *
       FROM employees
       WHERE first_name='Georgi' AND emp_no BETWEEN 10000 AND 20000;
```

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/5cf10559-d244-4176-95ed-528696136e0c)

`SET_VAR` 힌트는 실행 계획을 바꾸는 용도뿐만 아니라 `조인 버퍼`나 `정렬용 버퍼`의 크기를 일시적으로 

증가시켜서 `대용량 처리` 쿼리의 **성능을 향상** 시킬 수도 있다.

하지만 모든 `시스템 변수`를 `SET_VAR` 힌트로 조정할 수 없다.

### ✌🏼 SEMIJOIN & NO_SEMIJOIN

- `SEMIJOIN` 힌트는 **어떤 세부 전략을 사용할지 제어**할 수 있다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/04e92edc-aa35-4530-b2d0-501e87dd0d51)

`Table Pull-out` 최적화 전략은 별도로 **힌트 사용이 불가능**하다.

선택만 가능하다면 항상 `더 나은 성능`을 `보장`하기 때문이다. 

```sql
mysql> EXPLAIN
       SELECT * 
       FROM departments d
       WHERE d.dept_no IN
           (SELECT de.dept_no FROM dept_emp de);
```

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/9a659f76-00f9-4c7d-b40d-bafdd3cf60b3)

현재 이 `쿼리`는 `Loose Scan` **최적화가 적용**되어 있는데 ,

이 예제 `쿼리`가 다른 **세미 조인 최적화**를 사용하도록 변경해보자.

**세미조인 최적화 힌트**는 외부 쿼리가 아니라 `서브 쿼리`에 `명시`해야 한다. 

```sql
mysql> EXPLAIN
       SELECT * 
       FROM departments d
       WHERE d.dept_no IN
           (SELECT /*+ SEMIJOIN(MATERIALIZATION) */ de.dept_no 
           FROM dept_emp de);
```

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/7588483e-23dc-44ee-a624-7dab56ddfe39)

`서브쿼리`에 쿼리 블록 이름을 명시하고 `세미 조인 힌트`는 **외부 쿼리 블록에 명시**하는 방법도 있다.

```sql
mysql> EXPLAIN
       SELECT /*+ SEMIJOIN(@subq1 MATERIALIZATION) */ *
       FROM departments d
       WHERE d.dept_no IN
           (SELECT /*+ QB_NAME(subq1) */ de.dept_no
            FROM dept_emp de);
```

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/d7993e8b-d461-408d-8eed-754db0cfe2c0)

특정 `세미 조인 최적화 전략`을 사용하지 않도록 `NO_SEMIJOIN` 힌트를 명시할 수 있다.

```sql
mysql> EXPLAIN
       SELECT * 
       FROM departments d
       WHERE d.dept_no IN
           (SELECT /*+ NO_SEMIJOIN(DUPSWEEDOUT, FIRSTMATCH) */ de.dept_no 
           FROM dept_emp de);
```

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/58cdf669-8092-4198-b1ee-ab79f42325fe)

### 😡 SUBQUERY

- `서브쿼리 최적화`는 **세미 조인 최적화가 사용되지 못할 때 사용하는 최적화 방법**이다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/bd52c324-cb83-44ac-949a-1d83710e3ff3)

`세미 조인 최적화`는 주로 `IN(subquery)` 형태의 쿼리에서 사용할 수 있다.

`안티 세미 조인 최적화`에는 주로 위의 2가지 최적화가 사용된다.

`서브쿼리 최적화 힌트`는 세미 조인 최적화 힌트와 비슷한 형태로 사용된다.

`서브쿼리`에 힌트를 사용하거나 외부 **쿼리 블록에서 최적화 방법을 명시**하면 된다.

### 🤔 BNL & NO_BNL & HASH_JOIN & NO_HASHJOIN

- `BNL` 힌트와 `NO_BNL` 힌트는 MySQL `8.0.20`과 그 이후의 버전에서도 **여전히 사용 가능**하다.
- `8.0.20` 버전부터는 `BNL` 힌트를 사용하면 **해시 조인을 사용하도록 유도하는 용도로 변경**되었다.
- `HASH_JOIN`과 `NO_HASHJOIN` 힌트는 8.0.18 버전에서만 유효하다.

```sql
mysql> EXPLAIN
       SELECT /*+ BNL(e, de) */ *
       FROM employees e
       INNER JOIN dept_emp de ON de.emp_no=e.emp_no;
```

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/6b2d953d-0162-4cc7-bcd1-5e5e15beedf0)

MySQL `8.0.20`과 그 이후 버전에서는 `해시 조인`을 `유도`하거나 사용하지 않게 하려면 **BNL과 NO_BNL**

힌트를 `사용`해야 한다. 

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/be7ece71-9380-405a-97b3-b00fc8814744)

### 😑 JOIN_FIXED_ORDER & JOIN_ORDER & JOIN_PRERIX & JOIN_SUFFIX

- `MySQL` 서버에서는 조인의 순서를 결정하기 위해 전통적으로 `STRAIGHT_JOIN` 힌트를 사용해왔다.
- `STRAIGHT_JOIN` 힌트는 우선 쿼리의 `FROM 절`에 사용된 테이블의 순서를 **조인 순서에 맞게 변경**해야 하는 번거로움이 있었다.

`STRAIGHT_JOIN`은 한 번 사용되면 `FROM 절`에 명시된 모든 테이블의 순서가 결정된다.

따라서 일부는 `조인 순서`를 `강제`하고 나머지는 `옵티마이저`에게 **순서를 결정하게 맞기는 것이 불가능**했다.

`옵티마이저 힌트`에서는 `STRAIGHT_JOIN`과 **동일한 힌트까지 포함해서 4가지를 제공**한다.

- `JOIN_FIXED_ORDER` : **STRAIGHT_JOIN** 힌트와 동일하게 **FROM 절**의 테이블 순서대로 조인을 실행하게 하는 힌트
- `JOIN_ORDER` : **FROM 절**에 사용된 테이블의 순서가 아니라 `힌트`에 명시된 **테이블의 순서대로 조인**을 실행하는 힌트
- `JOIN_PREFIX` : 조인에서 드라이빙 테이블만 강제하는 힌트
- `JOIN_SUFFIX` : 조인에서 드리븐 테이블(**가장 마지막에 조인돼야 할 테이블들**)만 강제하는 힌트

`조인 순서`와 관련된 `옵티마이저` 힌트의 사용법은 아래와 같다.

```sql
-- FROM 절에 나열된 테이블의 순서대로 조인 실행
mysql> SELECT /*+ JOIN_FIXED_ORDER() */ *
       FROM employees e
         INNER JOIN dept_emp de ON de.emp_no=e.emp_no
         INNER JOIN departments d ON d.dept_no=de.dept_no;
         

-- 일부 테이블에 대해서만 조인 순서를 나열
mysql> SELECT /*+ JOIN_ORDER(d, de) */ *
       FROM employees e
         INNER JOIN dept_emp de ON de.emp_no=e.emp_no
         INNER JOIN departments d ON d.dept_no=de.dept_no;
        

-- 조인의 드라이빙 테이블에 대해서만 조인 순서를 나열
mysql> SELECT /*+ JOIN_PREFIX(e, de) */ *
       FROM emplyoees e
         INNER JOIN dept_emp de ON de.emp_no=e.emp_no
         INNER JOIN departments d ON d.dept_no=de.dept_no;
         

-- 조인의 드리븐 테이블에 대해서만 조인 순서를 나열
mysql> SELECT /*+ JOIN_SUFFIX(de, e) */ *
       FROM emplyoees e
         INNER JOIN dept_emp de ON de.emp_no=e.emp_no
         INNER JOIN departments d ON d.dept_no=de.dept_no;
```

### 🧑🏼‍🎄 MERGE & NO_MERGE

- 예전 버전의 `MySQL` 서버에서는 `FROM 절`에 사용된 **서브쿼리를 항상 내부 임시테이블로 생성**했다.
- MySQL `5.7`과 `8.0` 버전에서는 가능하면 `임시 테이블`을 사용하지 않게 `FROM 절`의 서브쿼리를 외부 쿼리와 **병합하는 최적화를 도입**했다.

때로는 `내부 임시 테이블`을 생성하는 것이 더 나은 선택일 수 있는데, `옵티마이저`는 최적의 방법을 선택하지 못할 수 있다. 

이럴때는 `MERGE` 또는 `NO_MERGE` **옵티마이저 힌트를 사용**하면 된다.

```sql
mysql> EXPLAIN
       SELECT /*+ MERGE(sub) */ *
       FROM (SELECT * 
             FROM employees
             WHERE first_name='Matt') sub LIMIT 10;
```

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/031d0757-405e-4ae3-9999-2fdd00fc5441)

```sql
mysql> EXPLAIN
       SELECT /*+ NO_MERGE(sub) */ *
       FROM (SELECT * 
             FROM employees
             WHERE first_name='Matt') sub LIMIT 10;
```

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/9d2e0af1-fe19-43cd-9de1-9d7d2b4e0ab3)

### 😎 INDEX_MERGE & NO_INDEX_MERGE

- `MySQL` 서버는 가능하다면 테이블당 `하나`의 `인덱스`만을 이용해 쿼리를 처리하려고 한다.
- `인덱스 머지` 실행 계획은 때로는 `성능 향상`에 도움이 되지만 항상 그렇지는 않을 수 있다.
- `인덱스 머지` 실행 계획의 사용 여부를 제어하려면 `옵티마이저 힌트`를 사용하면 된다.

```sql
mysql> EXPLAIN SELECT * 
       FROM employees
       WHERE first_name='Georgi' AND emp_no BETWEEN 10000 AND 20000;
```

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/fbafcf84-3ee3-4e63-bd98-f34ab914306f)

```sql
mysql> EXPLAIN 
       SELECT /*+ NO_INDEX_MERGE(employees PRIMARY) */ *
       FROM employees
       WHERE first_name='Georgi' AND emp_no BETWEEN 10000 AND 20000;
```

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/fc9d90b3-0191-4822-88e9-e93cb424b41c)

```sql
mysql> EXPLAIN 
       SELECT /*+ INDEX_MERGE(employees ix_firstname, PRIMARY) */ *
       FROM employees
       WHERE first_name='Georgi' AND emp_no BETWEEN 10000 AND 20000;
```

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/7487dc0e-d525-4343-ba61-74df7473a05d)

### 👿 NO_ICP

- `인덱스 컨디션 푸시다운` 최적화는 사용 가능하다면 항상 성능 향상에 도움이 된다 .
- `인덱스 컨디션 푸시다운`으로 인해 여러 실행 계획의 `비용 계산`이 `잘못`된다면 결과적으로 잘못된 실행 계획을 `수립`하게 될 수 있다.

`A` 인덱스와 `B` 인덱스 둘 중에서 하나만 **선택해야 하는 상황이 있다고 가정**한다.

`A` 인덱스에서는 `인덱스 컨디션 푸시다운`이 가능해서 A 인덱스를 사용하는 것이 **비용이 더 낮게 예측**되었다.

그렇다면 `옵티마이저`는 A 인덱스를 사용하는 `실행 계획`을 선택할 것이다.

하지만 `실제 서비스`에서는 B 인덱스를 선택하는 것이 더 효율적일 때가 있다. 

그렇다고 해서 `A` 인덱스를 **완전히 사용하지 못하게** 하거나 `B` 인덱스를 **선호하게 하는 것**은 좋지 않다.

`테이블`의 데이터 분포가 항상 `균등`한 것이 아니기 때문에 `쿼리 검색 범위`에 따라 **A 인덱스가 더 효율적**일 수 있기 때문이다.

이런 경우에는 `인덱스 컨디션 푸시다운` 최적화만 `비활성화`해서 조금 **더 유연한 실행 계획을 선택**하게 할 수 있다.

```sql
-- ICP 테스트를 위한 임시 인덱스 생성
mysql> ALTER TABLE employees ADD INDEX ix_lastname_firstname (last_name, first_name);

-- Extra 컬럼의 "Using index condition" 문구를 보면
-- 기본적으로 MySQL 옵티마이저는 인덱스 컨디션 푸시다운 최적화를 선택한 것을 알 수 있다.
mysql> EXPLAIN
       SELECT * 
       FROM employees
       WHERE last_name='Action' AND first_name LIKE '%sal';
```

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/de4ccf54-7d0f-44e7-9f67-b6ac44ad388c)

```sql
-- NO_ICP 옵티마이저 힌트를 이용해 인덱스 컨디션 푸시다운 최적화를 비활성화한 후
-- Extra 컬럼에는 "Using where"만 표시됨
mysql> EXPLAIN
       SELECT /*+ NO_ICP(employees ix_lastname_firstname) */ * 
       FROM employees
       WHERE last_name='Action' AND first_name LIKE '%sal';
```

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/5888585a-e33e-40c8-bf9b-bc6e95218209)

### 😍 SKIP_SCAN & NO_SKIP_SCAN

- 조건이 누락된 `선행 컬럼`이 가지는 **유니크한 값의 개수**가 많아진다면 `인덱스 스킵 스캔`의 성능은 오히려 떨어진다.
- `옵티마이저`가 인덱스 스킵 스캔을 선택함으로 인해서 성능이 떨어진다면 `NO_SKIP_SCAN` 옵티마이저 힌트를 이용할 수 있다.

```sql
-- 인덱스 스킵 스캔 테스트를 위한 임시 인덱스 생성
mysql> ALTER TABLE employees
         ADD INDEX ix_gender_birthdate (gender, birth_date);
         

mysql> EXPLAIN
       SELECT gender, birth_date
       FROM employees
       WHERE birth_date >= '1965-02-01';
```

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/f0a26f91-1f48-4209-8e40-62621039ccc4)

```sql
-- NO_SKIP_SCAN 힌트를 이용해 인덱스 스킵 스캔을 비활성화
mysql> EXPLAIN
       SELECT /*+ NO_SKIP_SCAN(employees ix_gender_birthdate) */ gender, birth_date
       FROM employees
       WHERE birth_date >= '1965-02-01';
```

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/2584c850-1c9b-4def-9733-9273fe97e252)

### 🙃 INDEX & NO_INDEX

- `INDEX`와 `NO_INDEX` 옵티마이저 힌트는 예전 `MySQL` 서버에서 사용되던 **인덱스 힌트를 대체하는 용도**로 `제공`된다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/72148b49-c602-45bf-be67-e273906c0a4d)

`인덱스 힌트`는 특정 테이블 뒤에 사용했기 때문에 별도로 `테이블명` 명시 없이 `인덱스 이름`만 나열했지만,

`옵티마이저 힌트`에는 테이블명과 인덱스 이름을 함께 `명시`해야 한다.

```sql
-- 인덱스 힌트 사용
mysql> EXPLAIN
       SELECT * 
       FROM employees USE INDEX(ix_firstname)
       WHERE first_name='Matt';
       

-- 옵티마이저 힌트 사용
mysql> EXPLAIN
       SELECT /*+ INDEX(employees ix_firstname) */ *
       FROM employees
       WHERE first_name='Matt';
```

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/29d0a7a1-7c79-40b8-b5a3-f73f3a2e88bb)
