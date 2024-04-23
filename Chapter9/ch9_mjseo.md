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
