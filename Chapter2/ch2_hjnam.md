## 2.1 MySQL 서버 설치

다양한 형태로 설치가 가능하다.

- Tar/Zip 압축 버전
- Linux RPM 설치 버전(권장)
- 소스코드 빌드

### 2.1.1 버전과 에디션(엔터프라이즈와 커뮤니티) 선택

제약 사항이 없다면 `최신 버전 설치`가 가장 좋다. 

버전 업그레이드를 할 경우 최소 `패치 버전이 15-20번 이상 릴리스`된 버전이 안정적인 서비스에 도움된다.

갓 출시된 메이저 버전은 버그가 많을 수 있기 때문에 선택에 위험이 따를 수 있다.

MySQL은 오픈 코어 모델 상용화 방식을 가져가는 중이다. 특정 부가 기능들만 사용버전에 포함시키고 나머지는 동일하게 제공하는 것이다. 

### 2.1.2  MySQL 설치

운영체제별로 설치 프로그램을 이용하는 방법을 살펴본다.

**2.1.2.1 리눅스 서버의 Yum 인스톨러 설치**

1. MySQL 다운로드 페이지에서 RPM 설치 파일 다운로드 및 설치
2. MySQL 소프트웨어 리포지토리 등록
    
    ```sql
    linux> sudo rpm -Uvh mysql80-community-release-el7-3.noarch.rpm
    ```
    
3. Yum 인스톨러 명령으로 버전별로 설치 가능한 MySQL 목록 확인
    
    ```sql
    // RPM 패키지 확인
    linux> sudo yum search mysql-community
    
    // 설치 가능한 모든 버전 확인
    linux> sudo yum --showduplicates list mysql-community-server
    ```
    
4. MySQL 8.0.21 설치
    
    ```sql
    linux> sudo yum install mysql-community-server-8.0.21
    ```
    

**2.1.2.2 리눅스 서버에서 Yum 인스톨러 없이 RPM 파일로 설치**

1. RPM 패키지 파일 다운로드
2. 의존 관계 순서대로 설치

**2.1.2.3 macOS용 DMG 패키지 설치**

1. DMG 패키지 파일 다운로드
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e1cdd83a-bb4e-43bb-b8ac-f046170c6a41/cbb38f59-6bee-44fd-860f-0e54caf5c260/Untitled.png)
    
2. 사용자 인증 방식 선택
    - 사설 네트워크에서만 사용 시 `Use Legacy Password Encryption`
    - 인터넷을 경유하여 접속 시 `Use Strong Password Encryption`
3. 비밀번호 설정
4. 서버 실행 확인: `ps -ef | grep mysqld`

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e1cdd83a-bb4e-43bb-b8ac-f046170c6a41/07cc66d0-cf24-4aa7-82b9-e8bccd0d0941/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e1cdd83a-bb4e-43bb-b8ac-f046170c6a41/f5a2f59b-9f07-4c24-8d0a-3645f82ee6ef/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e1cdd83a-bb4e-43bb-b8ac-f046170c6a41/151415be-8373-4a80-b517-6f98c1df0fae/Untitled.png)

***절대 삭제하면 안되는 디렉토리***

MacOS에서 MySQL 서버 시작/종료

```sql
# MySQL 서버 시작
macos> sudo /usr/local/mysql/support-files/;mysql.server start

# MySQL 서버 종료
macos> sudo /usr/local/mysql/support-files/;mysql.server stop
```

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e1cdd83a-bb4e-43bb-b8ac-f046170c6a41/a6d2310c-8bea-481a-b748-bad9fb44fff0/Untitled.png)

/usr/local/mysql 디렉토리에 설정 파일을 만들고 등록한다. 

**2.1.2.4 윈도우 MSI 인스톨러 설치**

## 2.2 MySQL 서버의 시작과 종료

리눅스 운영체제에서의 사용방법을 중심으로 살펴본다.

### 2.2.1 설정 파일 및 데이터 파일 준비

MySQL 서버에 필요한 프로그램과 디렉토리 중 로그 파일과 시스템 테이블이 준비되지 않아 아직 MySQL 서버 시작이 불가능하다. 

우선 다음과 같이 초기 데이터 파일과 트랙잭션 로그 파일을 생성한다.

```sql
linux> mysqld --defaults-file=/etc/my.cnf --initialize-insecure
```

`—initialize-insecure` 옵션을 사용하면 필요한 데이터 파일과 로그 파일 생성과 함께 `비밀번호가 없는 관리자 계정 root 유저를 생성`한다. 

`비밀번호`를 가지게 하려면 `—initialize` 를 사용하면 된다. 이 때 비밀번호는 에러 로그 파일에서 확인할 수 있다. 

### 2.2.2 시작과 종료

원격에서 MySQL 서버를 셧다운하려면 로그인한 상태에서 `SHUTDOWN` 명령을 실행하면 된다. 대신 셧다운 권한을 가지고 있어야 한다. 

MySQL 서버에서는 실제 트랜잭션이 정상적으로 커밋돼도 데이터 파일에 변경된 내용이 기록되지 않고 로그 파일(리두 로그)에만 기록돼 있을 수 있다. 

MySQL 서버가 종료될 때 모든 커밋된 내용을 데이터 파일에 기록하고 종료하게 할 수도 있는데, 이 경우에는 다음과 같이 MySQL 서버의 옵션을 변경하고 MySQL 서버를 종료하면 된다.

```sql
## linux에서 MySQL 서버 종료 시
mysql> SET GLOBAL innodb_fast_shutdown=0;
linux> systemctl stop mysqld.service

## 원격으로 MySQL 서버 종료 시
mysql> SET GLOBAL innodb_fast_shutdown=0;
mysql> SHUTDOWN;
```

이렇게 모든 커밋된 데이터를 데이터 파일에 적용하고 종료하는 것을 `클린 셧다운`이라고 한다. 이러면 서버가 다시 기동할 때 별도 트랜잭션 복구 과정이 생략되어 빠르게 시작할 수 있다.

### 2.2.3 서버 연결 테스트

MySQL 서버에 접속하려면 MySQL 기본 클라이언트인 mysql을 실행하면 된다. 

```sql
# MySQL 소켓 파일을 이용한 접속
linux> mysql -uroot -p --host=localhost --socket=/tmp/mysql.sock

# TCP/IP를 통해 127.0.0.1(로컬 호스트)로 접속
linux> mysql -uroot -p --host=127.0.0.1 --port==3306

# 호스트 주소와 포트 명시 없이 접속
linux> mysql -uroot -p
```

1. MySQL 소켓 파일을 이용한 접속
2. TCP/IP를 통해 127.0.0.1(로컬 호스트)로 접속
    - 포트를 명시하는 것이 일반적
    - 원격 호스트에 있는 MySQL 서버 접속 시 반드시 이 방법 사용
    - `127.0.0.1` vs. `localhost`
        - `localhost`: 소켓 파일을 통해 MySQL 서버에 접속(Unix domain socket 이용) - IPC의 일종
        - `127.0.0.1`: 루프백 IP를 사용한 TCP/IP 통신 방식
3. 호스트 주소와 포트 명시 없이 접속
    - 호스트 기본값: localhost → 소켓 파일 사용(MySQL 서버의 설정 파일에서 읽어서 사용)

`SHOW DATABASES` 명령으로 데이터베이스 목록을 확인할 수 있다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e1cdd83a-bb4e-43bb-b8ac-f046170c6a41/d5751bdd-b87b-4896-9a8b-806842b889d2/Untitled.png)

MySQL 서버 접속 여부만 확인하려면 nc(NetCat)이나 telnet 명령어를 사용하면 된다. 

## 2.3 MySQl 서버 업그레이드

`In-Place Upgrade`

- MySQL 서버의 데이터 파일을 그대로 두고 업그레이드
- 여러 제약 사항이 존재하지만 업그레이드 시간 단축

`Logical Upgrade`

- MySQL 서버의 데이터를 SQL 문장이나 텍스트 파일로 덤프한 후 새 버전에 재적재
- mysqldump 등의 프로그램을 이용
- 제약 사항이 거의 없지만 업그레이드에 많은 시간 소요

### 2.3.1 인플레이스 업그레이드 제약 사항

마이너 버전 간 업그레이드와 메이저 버전 간 업그레이드가 있다. 

마이너 버전 간 업데이트는 대부분 데이터 파일의 변경 없이 진행하고, 여러 버전을 건너뛰는 것도 허용한다.

하지만 메이저 버전 간 업그레이드는 반드시 직전 버전에서만 업그레이드가 허용된다. 데이터 파일의 변경이 필요하기 때문이다.

여러 버전을 건너뛰고 업그레이드를 해야한다면 mysqldump 등을 활용한 논리적 업그레이드가 더 나은 방법일 수 있다.

특정 마이너 버전에서만 메이저 버전 업그레이드가 가능한 경우도 있다. 결국 메뉴얼 정독이 중요하다. 

### 2.3.2 MySQL 8.0 업그레이드 시 고려 사항

8.0에서는 많은 기능이 개선되고 변경되었으므로 업그레이드 전 아래 내용의 영향을 검토해 보아야 한다.

- 사용자 인증 방식 변경
    - Caching SHA-2 Authentication 인증 방식이 기본으로 변경
- MySQL 8.0과의 호환성
    - mysqlcheck 유틸리티로 확인 권장
    
    ```sql
    linux> mysqlcheck -u root -p --all-databases --check-upgrade
    ```
    
- 외래키 이름의 길이
    - 외래키 이름이 64글자로 제한됨
    
    ```sql
    -- // 외랰키 이름의 길이 체크
    mysql> SELECT TABLE_SCHEMA, TABLE_NAME
    			 FROM information_schema.TABLES
    			 WHERE TABLE_NAME IN
    				 (SELECT LEFT(SUBSTR(ID,INSTR(ID,'/')+1),
    											INSTR(SUBSTR(ID,INSTR(ID,'/')+1),'_ibfk_')-1)
    					FROM information_schema.INNODB_SYS_FOREIGN
    					WHERE LENGTH(SUBSTR(ID,INSTR(ID,'/')+1))>64);
    ```
    
- 인덱스 힌트
    - 성능 향상에 도움이 되던 인덱스 힌트가 성능 저하를 유발할 수도 있음
- GROUP BY에 사용된 정렬 옵션
    - `GROUP BY` 절의 칼럼 뒤에 ‘ASC‘나 ‘DESC‘를 사용하고 있다면 제거하거나 다른 방식으로 변경 권장
- 파티션을 위한 공용 테이블스페이스
    - 파티션의 각 테이블스페이스를 공용에 저장 불가
    - `ALTER TABLE . . . REORGANIZE` 명령을 실행해 개별 테이블스페이스를 사용하도록 변경
    
    ```sql
    -- // 공용 테이블스페이스에 저장된 파티션이 있는지 체크
    mysql> SELECT DISTINCT NAME, SPACE, SPACE_TYPE
    			 FROM information_schema.INNODB_SYS_TABLES
    			 WHERE NAME LIKE '%#P#%' AND SPACE_TYPE NOT LIKE '%Single%';
    ```
    

### 2.3.3 MySQL 8.0 업그레이드

8.0부터는 시스템 테이블의 정보와 데이터 딕셔너리 정보의 포맷이 완전히 바뀌었다.

5.7에서 8.0으로의 업그레이드는 두 단계로 나뉘어서 처리된다.

1. 데이터 딕셔너리 업그레이드
    - 5.7까지는 데이터 딕셔너리 정보가 FRM 확장자를 가진 파일로 별도 보관
    - 8.0부터는 데이터 딕셔너리 정보가 트랜잭션이 지원되는 InnoDB 테이블로 저장
    - 기존 FRM 파일의 내용을 InnoDB 시스템 테이블로 저장
    - 딕셔너리 데이터의 버전 간 호환성 관리를 위해 테이블 생성 시 사용된 MySQL 서버의 버전 정보도 기록
2. 서버 업그레이드
    - MySQL 서버의 시스템 데이터베이스의 테이블 구조를 8.0에 맞게 변경

8.0.15까지는 데이터 딕셔너리 업그레이드를 MySQL 서버 프로그램(mysqld)이, 서버 업그레이드는 mysql_upgrade 프로그램이 실행했다. 하지만 8.0.16부터는 mysql_upgrade가 없어지고 mysqld가 모두 진행한다.

업그레이드 시 —upgrade 옵션으로 데이터 딕셔너리 업그레이드 수행 여부를 제어할 수 있다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e1cdd83a-bb4e-43bb-b8ac-f046170c6a41/3da0b63f-6c89-4ecb-bb86-a81980464f0c/Untitled.png)

## 2.4 서버 설정

MySQL 서버는 `my.cnf`라는 이름을 가진 단 하나의 설정 파일을 사용한다. 설정 파일 경로가 고정이 아니라서 지정된 여러 개의 디렉토리를 순차 탐색하여 처음 발견한 파일을 사용한다. 

my.cnf의 위치가 궁금하다면 아래 명령어를 사용하자.

```sql
shell> mysqld --verbose --help
```

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e1cdd83a-bb4e-43bb-b8ac-f046170c6a41/84b3f0c1-8414-4619-8d6f-4ed30164cd51/Untitled.png)

1, 2, 4번 파일은 어느 MySQL이나 동일하며, 3번은 컴파일될 때 MySQL 프로그램에 내장된 경로다.

주로 1, 2번을 사용한다.

### 2.4.1 설정 파일의 구성

하나의 파일에 여러 개의 설정 그룹을 담을 수 있다. 대체로 실행 프로그램 이름을 그룹명으로 한다. 

```sql
[mysqld_safe]
malloc-lib = /opt/lib/libtcmalloc_minimal.so

[mysqld]
socket = /usr/local/mysql/tmp/mysql.sock
port = 3306

[mysql]
default-character-set = utf8mb4
socket = /usr/local/mys이1/tmp/mysql.sock
port = 3304

[mysqldump]
defualt-character-set = utf8mb4
socket = /usr/local/mysql/tmp/mysql.sock
port = 3305
```

MySQL 서버만을 위한 설정 파일이라면 [mysqld]만 명시해도 무방하다.

설정 파일을 공용으로 사용하고 싶다면 다른 그룹도 함께 사용하면 된다.

### 2.4.2 MySQL 시스템 변수의 특징

MySQL 서버는 기동하면서 설정 파일의 내용을 읽어 `메모리나 작동 방식을 초기화`하고, 접속된 사용자를 제어하기 위해 이러한 값을 `시스템 변수(System Variables)`로 저장해 둔다. 

시스템 변수는 `SHOW VARIABLES` 또는 `SHOW GLOBAL VARIABLES` 명령으로 확인할 수 있다.

시스템 변수의 5가지 속성은 다음과 같다.

- Cmd-Line
    - MySQL 서버의 명령행 인자로 설정될 수 있는지 여부
- Option file
    - my.cnf로 제어 가능한지 여부
- System Var
    - 시스템 변수 인지 여부
    - 변수명의 하이픈/언더스코어 구분에 주의(언더스코어로 거의 통일 중)
- Var Scope
    - 시스템 변수의 적용 범위
    - Global vs. Session vs. Both
- Dynamic
    - 동적 변수인지 정적 변수인지

### 2.4.3 글로벌 변수와 세션 변수

시스템 변수는 글로벌 변수와 세션 변수가 있다. 이는 적용 범위에 따라 나뉘어진다. 

일반적으로 세션별로 적용되는 시스템 변수의 경우 글로벌 변수뿐만 아니라 세션 변수에도 동시에 존재한다.
이러한 경우 MySQL 매뉴얼의 `Var Scope`에는 `Both`라고 표시된다.

- 글로벌 변수
    - MySQL 서버 인스턴스 전체적으로 영향을 미치는 시스템 변수
    - 주로 MySQL 서버 자체에 관련된 설정
    - ex) innodb_buffer_pool_size, key_buffer_size 등
- 세션 변수
    - MySQL 클라이언트가 서버에 접속할 때 기본으로 부여되는 옵션의 기본값을 제어
    - 클라이언트의 필요에 따라 개별 커넥션 단위로 값을 변경 가능
    - 기본값은 글로벌 변수이고, 각 클라이언트의 값은 세션 변수
    - ex) autocommit - 쿼리 단위로 자동 커밋을 수행할지 여부

`순수하게 범위가 세션`(Session)이라고 명시된 시스템 변수는 MySQL 서버의 `설정 파일에 초깃값을 명시할 수 없`으며, 커넥션이 만들어지는 순간부터 `해당 커넥션에서만 유효한 설정 변수`를 의미한다.

### 2.4.4 정적 변수와 동적 변수

둘을 나누는 기준은 `MySQL 서버가 기동 중인 상태에서 변경 가능한지 여부`이다. 

- 디스크에 저장된 설정 파일을 변경하는 경우
    - 서버 재시작 전까지 적용되지 않음
- 이미 기동 중인 MySQL 서버의 메모리에 있는 변수를 변경하는 경우
    - SET 명령으로 값을 변경 가능 → 현재 인스턴스에만 유효
    - 영구히 적용하려면 설정 파일도 변경해주어야 함

8.0부터는 `SET PERSIST` 명령으로 동시에 변경이 가능하다. 

SHOW나 SET에서 `GLOBAL`을 사용하면 글로벌 변수의 목록과 내용을 읽고 변경할 수 있다.

GLOBAL을 빼면 세션 변수를 조회하고 변경하게 된다.

동적으로 시스템 변숫값을 변경하는 경우 SET 명령으로 시스템 변수를 변경하면 my.cnf 설정 파일에는 변경 내용이 기록되지 않는다. SET PERSIST를 사용해야 한다. 이때 변경된 시스템 변수는 my.cnf 가 아닌 별도 파일에 기록된다.

시스템 변수의 범위가 `Both`인 경우, 글로벌 변수의 값을 변경해도 세션 변수는 그대로 유지된다. 

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e1cdd83a-bb4e-43bb-b8ac-f046170c6a41/4ee56798-d51b-4393-91bb-2cd653b18cbe/Untitled.png)

SET 명령으로 새로운 값을 설정할 때 수식 사용이 가능하다. (ex. `2*1024*1024`)

### 2.4.5 SET PERSIST

동적 변수의 경우 MySQL 서버에서 SET GLOBAL 명령으로 변경하면 즉시 MySQL 서버에 반영된다. 하지만 이렇게 변경한 후 MySQL 서버의 설정 파일에서도 이 내용을 적용해야 하는데, 적용하는 것을 잊어버릴 때가 생기기 마련이다. 

이러한 문제점을 보완하기 위해 `SET PERSIST`를 도입했다. 

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e1cdd83a-bb4e-43bb-b8ac-f046170c6a41/01718b76-544a-48fe-804f-99c5ce942232/Untitled.png)

변경하면 `즉시 적용`되고 `별도의 설정 파일(mysqld_auto.cnf)에 변경 내용`을 추가로 `기록`한다. 

다시 서버가 시작될 때 my.cnf 뿐만 아니라 mysqld_auto.cnf도 같이 참조해서 시스템 변수를 적용한다. 덕분에 자동으로 영구 변경이 되는 것이다. 

SET PERSIST는 세션 변수에는 적용되지 않으며, SET PERSIST 명령으로 시스템 변수를 변경하면 MySQL 서버는 자동으로 GLOBAL 시스템 변수의 변경으로 인식하고 변경한다.

현재 서버에 적용하지 않고 `다음 재시작을 위한 변경 기록`이 필요하다면 `SET PERSIST_ONLY` 를 사용하면 된다. 

정적인 변수의 값을 영구히 변경하고자 할 때도 사용할 수 있다. 

SET PERSIST / SET PERSIST_ONLY로 시스템 변수를 변경하면 `JSON 포맷의 mysqld-auto.cnf` 파일이 생성된다. 

- 변경된 시스템 변수의 이름 및 설정값
- 언제, 누가 변경했는지에 대한 정보

변경된 시스템 변수의 메타데이터는 `performance_schema.variables_info 뷰`와 `performance_schma.persisted_variables 테이블`을 통해 참조할 수 있다. 

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e1cdd83a-bb4e-43bb-b8ac-f046170c6a41/e705b500-99b4-4237-bfaa-b11bdcb0d216/Untitled.png)

`RESET PERSIST` 명령으로 변경한 내용을 삭제할 수도 있다. 

```sql
mysql> RESET PERSIST max_connections;
mysql> RESET PERSIST IF EXISTS max_connections;
mysql> RESET PERSIST;
```

### 2.4.6 my.cnf 파일

8.0에서 시스템 변수는 약 570개 수준이고, 플러그인이나 컴포넌트에 따라 늘어날 수 있다.
