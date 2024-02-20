### 💟 MySQL 서버 설치(Mac 기준)

`MySQL 다운로드`는 https://dev.mysql.com/downloads/mysql/ 에 접속해서 받을 수 있다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/1fb78da8-fb93-4bc6-8846-3745bc75c5b7)

책에서 사용하는 버전은 `8.0.21 버전`이므로 `구버전`을 다운로드 할 수 있는 `Archives` 탭을 클릭한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/0573c37f-6422-4038-a06c-eb5e1f7ab9ad)

`8.0.21` 버전을 선택하고 `DMG Archive`를 다운로드 한다.

- 다운받은 파일을 열고 `.pkg 확장자` 파일을 실행한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/78cfef9d-5171-4c2d-8569-bcd9b1306227)

- 설치를 클릭한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/ef6bcc1f-0013-47a1-aebd-1278000e15e3)

- 사용자 인증 방식을 선택하는 화면에서, `Use String Password Encryption`을 선택한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/2688baf7-aeb0-4321-a250-9653b015ff5d)

- `root` 계정의 비밀번호를 입력한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/e17c45e1-6d5a-4ad1-8aa7-71bae00930fa)

설치가 완료되면 `MySQL` 서버가 자동으로 실행된다.

`MySQL` 실행 확인을 위해서는 아래의 명령어로 확인할 수 있다.

```kotlin
macos> ps -f | grep mysqld
```

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/3b28ddee-214f-4083-992b-077a5b45ed09)

`MySQL` 서버가 설치된 서버의 디렉토리는 `/usr/local/mysql` 이다.

각 `디렉토리`는 여러 개가 있지만, 적어도 아래의 `디렉토리`는 절대 삭제하면 안된다.

- `bin` : `MySQL` 서버와 클라이언트 프로그램, `유틸리티`를 위한 디렉터리
- `data` : 로그파일과 데이터 파일들이 저장되는 디렉터리
- `include` : C/C++ 헤더 파일들이 저장된 디렉터리
- `lib` : 라이브러리 파일들이 저장된 디렉터리
- `share` : 다양한 지원 파일들이 저장돼 있으며. 에러 메시지나 샘플 설정 파일(`my .cnf`)이 있는 디렉터리

`macOS`에 설치된 MySQL 서버의 설정 파일(`my.cnf`) 등록 및 시작과 종료는 `시스템 환경설정`에서 할 수 있다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/29b82264-c306-4f2c-b05d-f52c352707de)

터미널로 `MySQL 서버`를 `실행`하거나 `종료`할 수도 있다.

```bash
## MySQL 서버 시작
macos> sudo /usr/local/mysql/support-files/mysql. server start

## MySQL 서버 종료
macos> sudo /usr/local/mysql/support-files/mysql. server stop
```

`Configuration` 탭을 클릭하면 `MySQL` 서버의 설정을 변경할 수 있는 화면이 표시된다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/afb8f10f-143a-440d-ae90-0af3a8f38690)

`MySQL` 서버의 설정 파일은 아직 준비되어 있지 않은 것을 확인할 수 있다.

`/usr/local/mysql` 디렉토리에 `my.cnf` 파일을 생성하고, `Configuration File` 항목에 넣어줘야 한다.

- 아래의 명령어로 비어있는 `my.cnf` 파일을 만들어준다.

```bash
macos> sudo touch /usr/local/mysql/my.cnf
```

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/d291a1e2-9c03-4f62-b77d-9afb4720d563)

- 생성된 파일을 `Configuration File`의 경로로 넣어주고, `Apply`를 눌러 적용해주면 된다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/5f298159-d377-45ab-a718-e8cc3a6d2e45)

### 👋🏼 MySQL 종료

- `원격`으로 `MySQL 서버`를 종료하려면, `MySQL` 서버에 로그인한 상태에서 `SHUTDOWN` 명령을 실행하면 된다.
- `원격`으로 서버를 종료하려면 `SHUTDOWN 권한(Privileges)`를 가지고 있어야 한다.

```bash
mysql> SHUTDOWN;
```

`MySQL` 서버에서는 실제 `트랜잭션`이 정상적으로 `커밋`돼도 데이터 파일에 변경된 내용이 기록되지 않고 로그 파일(`리두 로그`)에만 기록되어 있을 수 있다.

사용량이 많은 `MySQL 서버`에서는 이런 현상이 일반적이며, `비정상적인 상황`이 아니다.

`MySQL 서버`가 시작되거나 종료될 때는 `MySQL 서버`의 버퍼 풀 내용을 백업하고 복구하는 과정이 내부적으로 실행된다.

`버퍼 풀`에 적재돼 있던 데이터 파일의 `데이터 페이지`에 대한 메타 정보를 백업하기 때문에 용량은 크지 않고, 백업 자체도 매우 빠르게 완료된다.

- `MySQL` 서버가 종료될 때 모든 `커밋`된 내용을 `데이터 파일`에 기록하고 `종료`하게 하려면, 아래와 같이 서버의 옵션을 변경하고 서버를 종료하면 된다.
- 모든 `커밋`된 데이터를 데이터 파일에 적용하고 종료하는 것을 `클린 셧다운`이라고 한다.

```bash
mysql> SET GLOBAL innodb_fast_shutdown=0;
linux> systemctl stop mysqld.service

## 또는 원격으로 MySQL 서버 종료 시
mysql> SET GLOBAL innodb_fast_shutdown=0;
mysql> SHUTDOWN; 
```

### 😀 MySQL 서버 연결

`MySQL 서버`에 접속하는 방법은 `MySQL` 기본 클라이언트 프로그램인 `mysql`을 `실행`하면 된다.

여러 가지 형태의 `명령행 인자`를 넣어 접속을 시도할 수 있다.

```bash
## MySQL 소켓 파일을 이용해 접속
linux> mysql -u root -p --host=localhost --socket=/tmp/mysql.sock

## 127.0.0.1은 루프백 IP 이지만, TCP/IP 통신 방식을 이용하고 있다.
## 원격지에 접속하기 위해서는 반드시 이 방식을 사용해야 한다.
linux> mysql -u root -p --host=127.0.0.1 --port=3306

## 호스트 주소와 포트를 명시하지 않으면 기본값으로 호스트는 localhost가 되고 소켓 파일을 사용하게 된다.
## 소켓 파일의 위치는 MySQL 서버의 설정 파일에서 읽어서 사용한다.
linux> mysql -u root -p
```

`MySQL` 서버에 접속하여 `SHOW DATABASE` 명령으로 데이터베이스의 `목록`을 조회할 수 있다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/f1e5891b-ce16-4792-841f-fd81abcb5426)

`MySQL 서버`에 직접 로그인 하지 않고, 접속 가능 여부만 확인하고 싶다면 `두 가지 방법`을 시도할 수 있다.

`telnet` 명령이나 `nc(Netcat)` 명령을 사용하면 된다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/82d1b0a6-f42d-4588-b3a7-20eea8d1f88f)

`MySQL 서버`가 보내준 메시지를 출력한다면 `네트워크 수준`의 연결은 `정상`이라고 `판단`할 수 있다.

서버의 `버전 정보`를 제대로 출력하는 상태에서도 응용 프로그램이 `MySQL 서버`에 접속하지 못한다면 계정 비밀번호가 맞지 않거나 `host` 부분이 허용되지 않을 `가능성`이 높다.

## 😄 MySQL 서버 업그레이드

`MySQL 서버`를 업그레이드 하는 방법은 두 가지 방법이 있다.

**인플레이스 업그레이드(In-Place Upgrade)**

- `MySQL 서버`의 데이터 파일을 그대로 두고 업그레이드하는 방법이다.
- 여러 가지 제약 사항이 있지만 업그레이드 시간을 크게 단축할 수 있다.

**논리적 업그레이드(Logical Upgrade)**

- `mysqldump` 도구 등을 이용해 MySQL 서버의 데이터를 `덤프`한 후, 새로 업그레이드 된 버전의 `MySQL` 서버에서 `덤프된 데이터`를 `적재`하는 방법이다.
- `버전 간 제약 사항`이 거의 없지만 업그레이드 시간이 `매우 많이 소요`될 수 있다.

## 🤓 인플레이스 버전 업그레이드 제약 사항

업그레이드는 `마이너(패치)` 버전 간 업그레이드와 `메이저 버전` 간 업그레이드가 있을 수 있다.

동일 `메이저 버전`에서 `마이너 버전` 간 업그레이드는 대부분 데이터 파일의 변경 없이 진행된다.

대부분의 경우 `여러 버전`을 건너뛰어서 `업그레이드` 하는 것도 허용된다.

예를 들면, `8.0.16` → `8.0.21` 버전으로 업그레이드 하는 경우에는 `MySQL 서버 프로그램`만 재설치하면 끝난다.

문제는 `메이저 버전` 간 업그레이드는 대부분의 경우에서 크고 작은 `데이터 파일 변경`이 필요하기 때문에 반드시 `직전 버전`에서만 업그레이드가 허용된다.

`MySQL 5.5` 버전에서 `MySQL 5.6` 버전으로는 업그레이드가 가능하지만, `5.7`이나 `8.0` 버전을로 업그레이드는 지원하지 않는다.

메이저 버전 업그레이드는 `데이터 파일`의 `패치`가 필요하기 때문이다.

`인플레이스 버전 업그레이드`의 제약사항으로 인해, `두 단계` 이상 업그레이드를 진행해야 하는 경우, `mysqldump` 프로그램을 이용한 `논리적 업그레이드`가 더 나은 방법일 수 있다.

`인플레이스 업그레이드`에서 주의해야 할 마지막 포인트는 `메이저 버전` 업그레이드가 특정 `마이너 버전`에서만 가능한 경우가 있다는 점이다.

이런 문제가 생길 수 있으므로, `MySQL 서버`를 선택할 때는 최소 `GA` 버전은 지나서 `15~20`번 이상의 마이너 버전을 `선택`하는 것이 좋다.

### 💪🏼 MySQL 8.0 업그레이드 시 고려 사항

`MySQL 8.0`에서는 많은 부분이 개선되거나 변경 되었다.

`MySQL 5.7` 버전과 `8.0` 버전의 차이점과 `8.0` 버전에서는 `사용할 수 없는 기능`들도 몇 가지 있다.

따라서 `버전 업그레이드` 이전에, 아래의 사항을 고려해보는 것이 좋다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/161da392-b968-4e19-8fd9-b778556bdde0)

### 🤦🏼 MySQL 8.0 업그레이드

- MySQL `5.7` → `8.0` 업그레이드는 단순하지 않다.
- 사용자에게 복잡하게 보이지는 않겠지만, `MySQL` 서버가 내부적으로 진행하는 `업그레이드 과정`은 상당히 복잡하다.

`MySQL 8.0` 부터는 **시스템 테이블**의 정보와 `데이터 딕셔너리(Data Dictionary)` 정보의 포맷이 완전히 바뀌었다.

MySQL `5.7` → MySQL `8.0` 업그레이드는 아래의 두 가지 단계로 나뉘어서 처리된다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/8e44543c-f355-4f02-af4e-53fdf7286af7)

`MySQL 8.X` 버전에서도 버전에 따라 업그레이드 방식의 `차이`가 있다.

### MySQL 8.0.15 버전까지의 업그레이드

- `데이터 딕셔너리` 업그레이드 작업은 `MySQL 서버(mysqld)` 프로그램이 실행한다.
- `서버 업그레이드` 작업은 `mysql_upgrade` 프로그램이 실행한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/ed7894b3-8155-4eca-abff-3d7f3723cac3)

### MySQL 8.0.16 버전까지의 업그레이드

- `mysql_upgrade` 유틸리티가 없어지고, `MySQL 서버 프로그램(mysqld)`이 시작된다.
- 업그레이드 작업을 `데이터 딕셔너리` 업그레이드와 `서버` 업그레이드 순서로 진행한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/542c9102-b484-4305-858e-0c5a3c9784cd)

`MySQL 서버`의 버전을 업그레이드 하고 `mysql_upgrade` 유틸리티를 실행하지 않아서 `이슈`가 발생하곤 했었는데, `사용자 실수`를 더 줄일 수 있게 개선되었다.

- `MySQL 8.0.16` 이상 버전에서도 `—upgrade` 옵션을 이용해서 제어할 수 있다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/4fc084d0-672c-4a30-ad3f-6717d3cb1d28)

### 👈🏼 서버 설정

- 일반적으로 `MySQL 서버`는 단 하나의 설정 파일을 사용한다.
- Unix 계열에서는 `my.cnf`라는 이름을 사용하고, 윈도우 계열에서는 `my.ini`라는 이름을 사용한다.

`MySQL 서버`는 시작될때만 이 `설정 파일`을 참조하는데 경로가 `고정`되어 있지 않다.

`MySQL 서버`는 여러 개의 `디렉토리`를 순차적으로 탐색하면서 `처음 발견`된 `my.cnf 파일`을 사용하게 된다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/8f8908c0-4787-4cb7-b2a6-42441b1e9877)

나열된 순서대로 `my.cnf` 파일을 `탐색`하고, 가장 먼저 발견된 `my.cnf` 파일을 사용하게 된다.

터미널에 출력된 정보에 의하면 순서는 아래와 같다.

1. /etc/my.cnf
2. /etc/mysql/my.cnf
3. /opt/homebrew/etc/my.cnf
4. ~/.my.cnf

`MySQL 서버` 설정은 주로 1번이나 2번을 사용하고, 3번 파일은 **컴파일 될 때 MySQL 프로그램이 저장된 경로**다.

보통 2개 이상의 `MySQL 서버`를 실행하는 형태로 서비스하는 경우는 거의 없다.

### 🥶 설정 파일의 구성

- `MySQL 설정 파일`은 하나의 `my.cnf`나 `my.ini` 파일에 여러개의 설정 그룹을 담을 수 있다.
- `실행 프로그램 이름`은 대체로 `그룹명`으로 사용한다.
- 예를 들면 `mysqldump` 프로그램은 `[mysqldump]` 설정 그룹으로 사용한다.

```bash
[mysqld_safe]
malloc-lib = /opt/lib/libtcmalloc_minimal.so

[mysqld]
socket = /usr/local/mysql/tmp/mysql. sock port = 3306

[mysql]
default-character-set = utf8mb4
socket = /usr/local/mys이1/tmp/mysql.sock port = 3304

[mysqldump]
default-character-set = utf8mb4
socket = /usr/local/mysql/tmp/mysql. sock port = 3305
```

`MySQL 서버`만을 위한 설정파일이라면 `[mysqld]` 그룹만 명시해도 아무런 문제가 없다.

이 설정 파일이 다른 프로그램이 실행될때도 참조하길 원한다면 `다른 그룹`을 `같이 설정`해 둘 수 있다.

### 🥺 MySQL 시스템 변수의 특징

- MySQL 서버는 기동하면서 `설정 파일`의 내용을 읽어 메모리나 `작동 방식`을 초기화한다.
- `접속된 사용자`를 제어하기 위해서 값을 별도로 저장해둔다.
    - `MySQL 서버`에서는 이렇게 저장된 값을 `시스템 변수`라고 한다.

각 시스템 변수는 `MySQL 서버`에 접속해 아래의 명령어로 확인할 수 있다.

```bash
mysql> SHOW GLOBAL VARIABLES;

mysql> SHOW VARIABLES;
```

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/268a9183-fa80-44fc-9182-b901a2ecd740)

- `시스템 변수` 값이 어떻게 `MySQL 서버`와 `클라이언트`에 영향을 미치는지 판단하려면 각 변수가 글로벌 변수인지 세션 변수인지 구분할 수 있어야 한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/211114b4-65ea-44ff-9cf6-bb05478306fa)

`시스템 변수`가 가지는 5가지 속성의 의미는 아래와 같다.

- `Cmd-Line` : MySQL 서버의 `명령행 인자`로 설정될 수 있는지의 여부를 나타낸다.
- `Option file` : MySQL의 설정 파일인 `my.cnf(my.ini)`로 제어할 수 있는지 여부를 나타낸다.
- `System Var` : 시스템 변수인지 아닌지를 나타낸다.
    - `MySQL 8.0`에서는 모든 시스템 변수를 **‘_’**를 구분자로, 명령행 옵션으로 사용 가능한 설정들을 **‘-’**을 구분자로 사용한다.
- `Var Scope` : 시스템 변수의 적용 범위를 나타낸다. `Session`, `Global`, `Both` 속성을 가질 수 있다.
- `Dynamic` : 시스템 변수가 `동적`인지 `정적`인지 구분하는 변수이다.

### 🤘🏼글로벌 변수와 세션 변수

- `MySQL 시스템 변수`는 적용 범위에 따라 **글로벌 변수**와 **세션 변수**로 나뉜다.
- 세션별로 적용되는 `시스템 변수`의 경우 **글로벌 변수**와 **세션 변수**에도 동시에 존재한다.
    - MySQL 메뉴얼의 `Var Scope`에는 `Both`로 표시된다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/15300c6b-9bc2-45b2-bf65-f474fe36e486)

### 🤔 정적 변수와 동적 변수

- `MySQL 서버`의 시스템 변수는 MySQL 서버가 `기동 중인 상태`에서 `변경 가능`한지에 따라 동적 변수와 정적 변수로  구분된다.
- `시스템 변수`는 디스크에 저장되어 있는 `설정파일`을 변경하는 경우와 이미 기동 중인 `MySQL 서버`의 메모리에 있는 `시스템 변수`를 변경하는 경우 두 가지로 구분할 수 있다.
- `변수명`을 정확히 모른다면 `LIKE 검색`을 통해 검색을 하는 것도 가능하다.

`max_connections`라는 키워드를 포함한 시스템 변수를 검색하고, 값을 바꾸면 어떻게 될까?

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/067269d9-7dea-4c9a-88a8-19462ade4eb4)

값이 `151`에서 `500`으로 변경되어 조회되는 것을 볼 수 있다.

하지만 `SET 명령`을 통해 변경되는 값은 `설정 파일(my.cnf)`에 반영되는 것은 아니기 때문에 현재 기동중인 `MySQL 서버`에 대해서만 유효하다.

변경한 설정을 `영구히 적용`하려면 `my.cnf` 파일도 반드시 `변경`해야한다.

MySQL 8.0 버전부터는 `SET PERSIST` 명령을 이용하면 `설정 파일`로도 기록된다.

`SET PERSIST` 명령을 사용하는 경우 `my.cnf` 파일이 아닌 `별도의 파일`에 기록된다.

`SHOW`나 `SET` 명령에서 `GLOBAL` 키워드를 사용하면 **글로벌 시스템 변수**의 목록과 내용을 읽고 변경할 수 있다.

`GLOBAL` 키워드를 빼면 `세션 변수`를 조회하고 변경한다.

`글로벌 시스템 변수` 중에서도 실시간으로 변경할 수 있는 것도 있다.

`시스템 변수`의 범위가 `Both`인 경우 **글로벌 시스템 변수**의 값을 변경해도 이미 존재하는 `커넥션`의 설정값은 변경되지 않고 `그대로 유지`된다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/ca7f9553-169f-439c-ad4b-c4571882a9be)

- `join_buffer_size`의 글로벌 변수 값을 `524288`로 변경한 뒤 세션 변수 값을 조회하면 결과는 아래와 같다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/f933185d-7a01-453c-aaa5-80f15374c82f)

`글로벌 변수` 값은 변경된 값을 유지하고 있지만, `세션 변수` 값은 예전의 값을 유지하고 있다.

`SET 명령`으로 새로운 값을 설정할 때는 설정 파일에서 처럼 `MB`나 `GB`와 같은 표기법을 사용할 수 없지만, `2*1024*1024`와 같은 `수식 사용`은 가능하다.

### 💬 SET PERSIST

- `동적 변수`의 경우 `SET GLOBAL` 명령으로 변경하면 즉시 `MySQL 서버`에 반영된다.
- `동적 변수`를 변경하는 응급 조치를 한 경우에는, 설정 파일에 변경 내용을 적용하는 것을 잊어버려서 `장애`가 발생하는 경우가 많다.
- 따라서, MySQL 8.0 부터는 `SET PERSIST` 명령을 사용하는 것이 좋다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/66a61004-f770-4e08-8a05-1d7c032f21c3)

`SET PERSIST` 명령으로 `시스템 변수`를 변경하면, 변경된 값을 즉시 적용하고 별도의 

설정 파일(`mysqld-auto.cnf`)에 변경 내용을 `추가`로 `기록`해둔다.

`MySQL 서버`가 다시 시작될 때 기본 설정 파일뿐만 아니라 `mysqld-auto.cnf`을 같이 참조하기 때문에 자동으로 `영구 변경`이 된다.

`SET PERSIST` 명령은 세션 변수에는 적용되지 않으며, `GLOBAL 시스템 변수`의 변경으로 인식하고 변경한다.

실행중인 `인스턴스`에는 변경 내용을 적용하지 않고, 다음 `재시작`을 위해 `mysqld-auto.cnf` 파일에만 기록하고자 한다면 `SET PERSIST_ONLY` 명령을 사용하면 된다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/05a8f17e-5f6f-492d-81aa-ac0c88f5714e)

`SET PERSIST_ONLY` 명령은 `정적`인 `변수`의 값을 변경하고자 할 때도 사용가능하다.

정적 변수는 `SET PERSIST 명령`으로는 변경이 불가능하다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/8d444af3-4bc4-439d-ac53-b05a4c6c78de)

`SET PERSIST` 명령이나 `SET PERSIST_ONLY` 명령으로 시스템 변수를 변경하면 JSON 포맷의 

`mysqld-auto.cnf` 파일이 생성된다.

`실제 파일`의 내용은 `들여쓰기`나 `라인 구분`이 되어있지 않다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/b4802ec4-026b-4954-81cf-ac382eeaf12d)

`SET PERSIST` 또는 `SET PERSIST_ONLY` 명령으로 변경된 시스템 변수의 `메타데이터`는 아래와 같이 조회할 수 있다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/e1bf3591-7e27-40e4-94ad-1c38bc199263)

`SET PERSIST` 또는 `SET PERSIST_ONLY` 명령으로 추가된 시스템 변수의 내용을 삭제하는 경우, 파일의 내용을 직접 변경하는 것보다는 `RESET PERSIST` 명령을 사용하는 것이 안전하다.

```bash
## 특정 시스템 변수만 삭제
mysql> RESET PERSIST max_connections;
mysql> RESET PERSIST IF EXISTS max_connections;

## mysqld-auto.cnf 파일의 모든 시스템 변수를 삭제 
mysql> RESET PERSIST;
```

`RESET PERSIST` 명령 입력 후 다시 시스템 변수의 `메타데이터`를 조회하면 아래와 같이 결과가 나온다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/5c821ec5-0673-4213-bf4e-c7557972737b)
