### 🤞🏼 사용자 식별

- `MySQL`의 사용자 계정은 사용자의 아이디뿐 아니라 **해당 사용자가 어느 IP에서 접속**하고 있는지도 확인한다.
- `MySQL 8.0` 버전부터는 권한을 묶어서 관리하는 `Role 개념`이 도입되었다.

`MySQL`의 사용자는 다른 `DBMS`와는 다르게, 사용자의 접속 지점(`클라이언트`가 실행된 `호스트명`이나 `도메인` 또는 `IP 주소`)도 계정의 일부가 된다.

```bash
## 로컬 호스트에서 접속할 수 있는 계정 svc_id
'svc_id'@'127.0.0.1'
```

- `로컬 호스트`에서만 접속할 수 있는 `svc_id` 계정은 다른 컴퓨터에서 `접속`이 `불가능`하다.
- 모든 `외부 컴퓨터`에서 접속이 가능한 `사용자 계정`을 `생성`하고 싶다면 사용자 계정의 호스트 부분을 `%` 문자로 대체하면 된다.

```bash
## 192.168.0.10 호스트에 접속할 수 있는 계정
'svc_id'@'192.168.0.10' (비밀번호 123)

## 모든 호스트에 접속할 수 있는 계정
'svc_id'@'%' (비밀번호 abc)

```

- `아이디`가 같은 경우 `MySQL` 서버가 어떤 `계정`을 선택해서 인증하는지 `이슈`가 생길 수 있다.
- `MySQL`은 권한이나 계정 정보에 대해서 `범위가 가장 작은 것`을 `선택`한다.

이런 문제가 생길 수 있으므로,  `중첩된 계정`을 생성하지 않도록 `주의`해야 한다.

### 😐 사용자 계정 관리

**시스템 계정과 일반 계정**

- `MySQL 8.0`부터 계정은 `SYSTEM_USER` 권한 보유 여부에 따라 `시스템 계정`과 `일반 계정`으로 구분된다.
- 시스템 계정은 `MySQL` 서버 내부적으로 실행되는 `백그라운드 스레드`와는 무관하다.
- `시스템 계정`은 **DBA**를 위한 계정이며, `일반 계정`은 **개발자**를 위한 계정이라고 보면 이해가 쉽다.

`시스템 계정`은 **시스템 계정**과 **일반 계정**을 관리할 수 있고, `일반 계정`은 **시스템 계정**을 관리할 수 없다.

아래와 같은 중요 작업은 `시스템 계정`으로만 `수행`할 수 있다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/fee3ef9b-6036-444e-a19c-33448b4466bc)

**사용자와 계정의 의미**

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/d33817e3-a3c2-49bc-ba98-9beec79eaa38)

**MySQL 서버의 내장된 계정**

`MySQL 서버`에는 내장된 계정들이 있는데, `‘root’@’localhost’`를 제외한 `3`개의 계정은 `삭제`되지 않도록 주의해야 한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/828f542c-a256-4ecb-8c48-dacd5f2b7d07)

위의 `3`개의 계정은 처음부터 잠겨 있는 상태이므로 `의도적`으로 잠긴 계정을 풀지 않는 한 이용이 불가능하다.

따라서 `보안적인 문제`는 없다고 볼 수 있다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/db96998c-b4cf-4992-b052-6b6a0653bdde)

### ➡️ 계정 생성

`MySQL 5.7` 버전까지는 `GRANT` 명령으로 권한 부여와 동시에 계정 생성이 가능했다.

`8.0` 버전에서는 `CREATE USER` 명령으로 계정을 생성하고, `GRANT 명령`으로 권한을 부여하도록 

명령어가 분리되었다.

`계정`을 생성할 때는 `다양한 옵션`을 설정할 수 있다.

- 계정의 인증 방식과 비밀번호
- 비밀번호 관련 옵션(유효기간, 이력 개수, 재사용 불가 기간)
- 기본 역할(ROLE)
- SSL 옵션
- 계정 잠금 여부

```sql
## 일반적으로 많이 사용되는 옵션을 가진 CREATE_USER 명령

mysql> CREATE USER 'user'©'%'
IDENTIFIED WITH 'mysql_native_password' BY 'password' REQUIRE NONE
PASSWORD EXPIRE INTERVAL 30 DAY
ACCOUNT UNLOCK
PASSWORD HISTORY DEFAULT
PASSWORD REUSE INTERVAL DEFAULT
PASSWORD REQUIRE CURRENT DEFAULT;
```

### 🚣🏼 IDENTIFIED WITH

- `사용자`의 `인증 방식`과 `비밀번호`를 설정하는 옵션이다.
- `IDENTIFIED WITH` 뒤에는 반드시 `인증 방식`(인증 플러그인 이름)을 명시해야 한다.
    - 기본 `인증 방식`을 사용하려면, 명시하지 않아도 된다.
    

`MySQL`은 다양한 `인증 방식`을 플러그인 형태로 제공한다.

- `Native Pluggable Authentication` : 5.7 버전까지 사용되던 방식으로, 비밀 번호에 대한                  `해시(SHA-1 알고리즘)` 값을 저장해두고 클라이언트가 보낸 값과 해시값의 일치 여부를 비교한다.
- `PAM Pluggable Authentication` : 유닉스나 리눅스 패스워드 또는 LDAP 같은 외부 인증을 사용할 수 있게 해주는 인증 방식이다. 엔터프라이즈 버전에서만 이용이 가능하다.
- `LDAP Pluggable Authentication` : LDAP를 이용한 외부 인증을 사용할 수 있게 해주는 인증 방식이다. 엔터프라이즈 버전에서만 이용이 가능하다.

`Caching SHA-2 Pluggable Authentication`

- 5.6 버전에 도입되었고, 8.0 버전에서는 조금 더 보완된 인증 방식이다. 암호화 해시 값 생성을 위해 `SHA-2(256 비트)알고리즘`을 사용한다.
- `Native Authentication` 플러그인은 입력이 `동일한 해시값`을 출력하지만, 이 방식은 내부적으로 `Salt` 키를 활용하고 수천 번의 `해시 계산`을 수행해서 결과를 만들기 때문에 동일한 키 값도 다른 결과를 만든다.
- `해시 값`을 매번 로그인때마다 계산하는 방식은 `많은 리소스`가 들기 때문에, 해시 연산 결과를 `메모리`에 캐시해서 사용한다.
- 이 `인증 방식` 사용을 위해서는 `SSL/TLS` 또는 `RSA 키페어`를 반드시 사용해야 한다.

`MySQL 5.7` 버전 까지는 `Native Authentication`이 기본 인증 방식으로 사용되었으나. 8.0 버전 부터는 `Caching SHA-2 Pluggable Authentication` 방식이 기본 인증 방식이다.

`Caching SHA-2 Pluggable Authentication` 방식은 `SSL/TLS` 또는 `RSA 키페어`를 필요로 하기 때문에 기존 `5.7` 버전 까지의 연결 방식과는 `다른 방식`으로 접속해야 한다.

8.0 버전에서도 `Native Authentication`을 기본 인증 방식으로 설정하는 것을 지원한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/335b00af-7135-4377-99e7-2e62028cf87e)

### 😀 REQUIRE

- MySQL 서버 접속시 암호화된 `SSL/TLS` 채널을 사용할지 여부를 설정한다.
- `REQUIRE` 옵션을 `SSL`로 설정하지 않아도, `Caching SHA-2 Pluggable Authentication` 인증 방식을 사용중이라면, `암호화된 채널` 만으로 서버에 접속할 수 있다.

### 👌🏼PASSWORD EXPIRE

- `비밀번호`의 유효 기간을 설정하는 옵션이다.
- 별도로 명시하지 않으면 `default_password_lifetime` 시스템 변수에 저장된 기간으로 설정된다.
- `응용 프로그램 접속용 계정`에 유효 기간을 설정하는 것은 위험할 수 있다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/90752a7f-8113-4a56-b1a7-48e36554cf27)

`PASSWORD EXPIRE` 절에 설정 가능한 옵션은 아래와 같다.

- `PASSWORD EXPIRE` : 계정 생성과 동시에 비밀번호의 만료 처리
- `PASSWORD EXPIRE NEVER` : 계정 비밀번호의 만료 기간 없음
- `PASSWORD EXPIRE DEFAULT` : `default_password_lifetime` 시스템 변수에 저장된 기간으로 비밀번호의 유효 기간 설정
- `PASSWORD EXPIRE INTERVAL n DAY` : 비밀번호의 유효 기간을 오늘부터 N일 뒤로 설정

### 😐 PASSWORD HISTORY

- 한 번 `사용했던 비밀번호`를 `재사용`하지 못하게 설정하는 옵션이다.
- `PASSWORD HISTORY` 절에 설정 가능하다.

`PASSWORD_HISTORY` 절에 설정 가능한 옵션은 아래와 같다.

- `PASSWORD HISTORY DEFAULT` : **password_history** 시스템 변수에 저장된 개수만큼 비밀번호의 이력을 저장하고, 저장된 이력에 남아있는 비밀번호는 사용이 불가능하다.
- `PASSWORD HISTORY n` : 비밀번호의 이력을 최근 n개 까지만 저장하고, 저장된 이력에 남아있는 비밀번호는 사용이 불가능하다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/2e357b8c-51b3-4247-8498-4681ae143649)

이전에 사용했던 비밀번호는 `password_history` 테이블을 사용해 저장한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/056d9bd2-a08f-4660-8c10-43d819352c78)

### 🤢 PASSWORD REUSE INTERVAL

- 한 번 사용했던 `비밀번호`의 `재사용 금지 기간`을 설정하는 옵션이다.
- 별도로 명시하지 않으면 `password_reuse_interval` 시스템 변수에 저장된 기간으로 설정된다.

`PASSWORD REUSE INTERVAL` 절에 설정 가능한 옵션은 아래와 같다.

- `PASSWORD REUSE INTERVAL DEFAULT` : `password_reuse_interval` 변수에 저장된 기간으로 설정
- `PASSWORD REUSE INTERVAL n DAY` : n일자 이후에 비밀번호를 재사용할 수 있게 설정

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/131651b7-8cd7-4a0d-afa3-315c37665616)

### 😊 PASSWORD REQUIRE

- `비밀번호`가 만료되어 `새로운 비밀번호`로 변경할 때 현재 비밀번호를 필요로 할지 여부를 결정하는 옵션이다.
- 별도로 명시되지 않으면 `password_require_current` 시스템 변수의 값으로 설정된다.

`PASSWORD REQUIRE` 절에 설정 가능한 옵션을 아래와 같다.

- `PASSWORD REQUIRE CURRENT` : 비밀번호를 변경할 때 현재 비밀번호를 먼저 입력하도록 설정
- `PASSWORD REQUIRE OPTIONAL` : 비밀번호를 변경할 때 현재 비밀번호를 입력하지 않아도 되도록 설정
- `PASSWORD REQUIRE DEFAULT` : `password_require_current` 시스템 변수의 값으로 설정

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/37fee959-cd27-4270-8c0c-454639f7ad12)

### 🐙 ACCOUNT_LOCK / UNLOCK

- `계정 생성`시 또는 `ALTER USER 명령`을 사용해 계정 정보를 변경할 때 `계정`을 사용하지 못하게 잠글지 여부를 결정하는 옵션이다.

설정 가능한 옵션은 아래와 같다.

- `ACCOUNT LOCK` : 계정을 사용하지 못하게 잠금
- `ACCOUNT UNLOCK` : 잠긴 계정을 다시 사용 가능 상태로 잠금 해제

### 🙏🏼 고수준 비밀번호

- `MySQL 서버`는 유효기간이나 이력 관리를 통한 `재사용 금지` 기능뿐만 아니라 `비밀번호`를 쉽게 유추할 수 있는 단어들이 사용되지 않게 설정하는 기능도 있다.
- `MySQL 서버`에서 비밀번호의 유효성 체크 규칙을 적용하려면 `validate_password` 컴포넌트를 이용하면 된다.
- `validate_password` 컴포넌트는 `MySQL 서버` 프로그램에 내장되어 있기 때문에 별도의 파일 경로를 지정하지 않아도 된다.

```sql
## validate_password 컴포넌트 설치
mysql> INSTALL COMPONENT 'file://component_validate_password';
```

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/0d902d82-6019-4d37-9717-f57f7dcc7141)

`validate_password` 컴포넌트가 설치되면 `컴포넌트`에서 제공하는 `시스템 변수`를 확인할 수 있다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/512753ed-732f-42b5-a29e-08cdca49d1af)

`비밀번호 정책`은 아래의 3가지 옵션에서 선택이 가능하며, 기본값은 `MEDIUM`이다.

- `LOW` : 비밀번호의 길이만 검증
- `MEDIUM` : 비밀번호의 길이를 검증하며, 숫자와 대소문자, 특수문자의 배합을 검증
- `STRONG` : MEDIUM 레벨의 검증 + 금칙어 포함 여부 검증

`validate_password` 컴포넌트가 제공하는 시스템 변수들의 역할은 아래와 같다.

- `validate_password.length` : 비밀번호의 길이를 설정
- `validate_password.mixed_case_count` : 대소문자 배합을 검증
- `validate_password.number_count` : 숫자 포함 여부를 검증
- `validate_password.special_char_count` : 특수문자 포함 여부를 검증
- `validate_password.dictionary_file` : 시스템 변수에 설정된 사전 파일에 명시된 금칙어 포함 여부를 검증

`높은 수준`의 보안이 필요한 서비스에서는 `금칙어`들이 저장된 사전 파일을 등록해서 사용할 수 있다.

```sql
## 금칙어 사전 파일 등록
mysql> SET GLOBAL validate_password.dictionary_file='prohibitive_word.data'; 
mysql> SET GLOBAL validate_password.policy='STRONG';
```

`MySQL 5.7` 버전 까지는 `validate_password`가 플러그인 형태로 제공되었다.

`plugin`과 `component`는 거의 동일한 기능일 제공하지만, `plugin`의 단점을 보완하기 위해 `component`가

`8.0` 버전부터 출시 되었기 때문에 `component`를 선택하는 것이 더 나을 수 있다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/36dc0293-c25f-407a-8a33-f2dde79601d2)

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/8311377e-f6ad-48d4-bba3-523756779466)

### 🏀 이중 비밀번호

- `데이터베이스 계정`의 비밀번호는 서비스가 `실행 중인 상태`에서 변경이 불가능하다.
- `8.0 버전` 부터는 계정의 비밀번호로 2개 값을 동시에 사용할 수 있는 기능이 추가되어, `이중 비밀번호 사용`이 가능하다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/27f5a82d-1655-4909-9554-8e06ecc88a9e/529e2405-7f26-4d04-9e70-f8c6068d7a30/Untitled.png)

`이중 비밀번호` 기능은 하나의 계정에 대해 `2`개의 비밀번호를 동시에 설정할 수 있다.

두 개의 비밀번호는 `Primary`와 `Secondary`로 구분된다. 

최근에 설정된 비밀번호는 `Primary 비밀번호`, 이전 비밀번호가 `Secondary 비밀번호`가 된다.

`이중 비밀번호`를 사용하려면 기존 비밀번호 변경 구문에 `RETAIN CURRENT PASSWORD` 옵션만 주면 된다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/279869fe-3fe7-444e-b1e1-07cd926a1765)

첫 번째 `ALTER USER` 명령이 실행되면 root 계정의 `Primary 비밀번호`는 `password1`로 변경된다.

두 번째 `ALTER USER` 명령이 실행되면, `password1`이 `Secondary 비밀번호`가 되고, `password2`가 

`Primary 비밀번호`가 된다.

`Secondary 비밀번호`를 꼭 삭제해야 하는 것은 아니지만, `보안`을 위해 `삭제`하는 것이 좋다.

```sql
mysql> ALTER USER 'root'@'localhost' DISCARD OLD PASSWORD;
```

### 🌈 권한(Privilege)

- `MySQL 5.7` 버전까지 권한은 `Global 권한`과 `객체 단위의 권한`으로 구분되었다.
    - `Global 권한` : 데이터베이스나 테이블 이외의 객체에 적용되는 권한
    - `객체 권한` : 데이터베이스나 테이블을 제어하는 데 필요한 권한

- `객체 권한`은 GRANT 명령으로 권한을 부여할 때 반드시 `특정 객체를 명시`해야 한다.
- `Global 권한`은 GRANT 명령에서 `특정 객체`를 명시하지 말아야 한다.

예외적으로 `ALL(ALL PRIVILEGES)`은 `Global`과 `객체` 권한 두 가지 용도로 사용될 수 있다.

특정 객체에 `ALL 권한`이 부여되면 `해당 객체`에 적용될 수 있는 `모든 객체 권한`을 부여하고, 

Global로 `ALL`이 사용되면 `Global 수준`에서 `가능한 모든 권한`이 부여된다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/9f2d0f17-fb23-4fb6-b0c6-b4b0fa193863)

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/ef6d263f-bbe5-4c85-91a4-f39e8b515391)

`MySQL 8.0` 버전부터는 `동적 권한`이 추가 되었다.

`정적 권한`은 소스코드에 **고정적으로 명시**돼 있는 권한이며, `동적 권한`은 **서버가 시작되면서** 생성한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/579d0f9b-bfc8-4ddc-a656-650cefa944d4)

`5.7 버전`까지는 `SUPER`라는 권한이 `데이터베이스 관리`를 위해 꼭 필요한 `권한`이였다.

`8.0 버전` 부터는 `SUPER` 권한을 잘게 쪼개서 `동적 권한`으로 `분산`시켰다.

`모든 권한`을 전부 주지 않고 `역할`에 따라서 `서버 관리`, `백업 관리` 등 꼭 필요한 권한만 제한하여 부여함으로서 

`데이터베이스 관리`의 `안정성`을 가져갈 수 있게 되었다.

`사용자`에게 `권한`을 부여할 때는 `GRANT` 명령을 사용하면 된다.

`권한`의 특성에 따라 `GRANT` 명령의 `ON 절`에 명시되는 `오브젝트`의 내용이 바뀐다.

```sql
mysql> GRANT privilege_list ON db.table TO 'user'@'host';
```

`privilege_list`에는 구분자(`,`)를 써서 앞의 표에 `명시된 권한` 여러 개를 동시에 명시할 수 있다.

`ON 키워드` 뒤에는 어떤 `DB`의 어떤 `오브젝트`에 `권한`을 부여할지 `결정`할 수 있다.

- `Global 권한 부여`

```sql
mysql> GRANT SUPER ON *.* TO 'user'@'localhost';
```

`Global 권한`은 특정 `DB`나 `테이블`에 부여될 수 없기 때문에 `ON 절`에는 항상 `*.*` 를 사용하게 된다.

`DB`의 모든 오브젝트(`테이블`과 `스토어드 프로시저` 등)를 포함해서 `MySQL 서버 전체`를 의미한다.

- `DB 권한`

```sql
mysql> GRANT EVENT ON *.* TO 'user'@'localhost';
mysql> GRANT EVENT ON employees.* TO 'user'@'localhost';
```

`DB 권한`은 `특정 DB` 또는 서버에 존재하는 `모든 DB`에 대해 권한을 부여할 수 있다.

`DB`는 테이블 뿐만 아니라 `스토어드 프로그램`도 모두 포함한다.

`DB 권한`은 테이블에 대해 부여할 수 없기 때문에 `테이블`까지 명시할 수는 없다.

`특정 DB`에 대해서만 권한을 부여하려면 `db.*`로 대상을 `설정`할 수 있다.

- `테이블 권한`

```sql
## 서버의 모든 DB에 대해 권한을 부여
mysql> GRANT SELECT, INSERT, UPDATE, DELETE ON *.* TO 'user'@'localhost';

## 특정 DB의 오브젝트에 대해서만 권한을 부여
mysql> GRANT SELECT, INSERT, UPDATE, DELETE ON employees.* TO 'user'@'localhost';

## 특정 테이블에 대해서만 권한을 부여
mysql> GRANT SELECT, INSERT, UPDATE, DELETE ON employees.department TO 'user'@'localhost';
```

`테이블`의 `특정 컬럼`에 대해서만 `권한`을 부여할 수도 있다. 

각 `권한` 뒤에 `컬럼`을 명시하는 형태로 부여할 수 있다.

```sql
mysql> GRANT SELECT, INSERT, UPDATE(dept_name) ON employees.department TO 'user'@'localhost';
```

`테이블`이나 `컬럼 단위`의 권한은 잘 사용하지 않는다.

특히 `컬럼 단위`의 권한이 하나라도 설정되면 나머지 모든 테이블의 `모든 컬럼`에 대해서도 `권한 체크`를 하기    때문에 `전체적인 성능`에 `악영향`을 끼칠 수 있다.

`컬럼 단위`의 접근 권한이 꼭 필요하다면 `GRANT 명령`으로 해결하는 것보다 테이블에서 권한을 허용하고자 하는 컬럼 만으로 `별도의 VIEW`를 만들어 사용하는 방법도 생각해 볼 수 있다.

`VIEW`도 하나의 테이블로 인식하기 때문에 `VIEW` 자체에 대한 `권한`만 `체크`하면 된다.

각 `계정`이나 `권한`에 부여된 권한이나 역할을 확인하려면 `SHOW GRANTS` 명령을 사용하면 된다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/a2b38734-5d46-4f80-a576-caa43b21904f)

`권한`을 `표`로 깔끔하게 보고 싶다면, `MySQL`의 `권한 관리 테이블`을 참고하면 된다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/e9425930-d41b-4820-9c3a-42f1db2958a5)

### 🫠 역할(Role)

- `MySQL 8.0` 버전부터 권한을 묶어서 `역할(Role)`을 사용할 수 있다.
- `MySQL 서버` 내부적으로는 `역할(Role)`은 `계정`과 똑같은 모습을 하고 있다.

**역할 생성하기**

- `CREATE ROLE` 명령을 이용해 역할을 정의한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/f53d9f95-150e-4471-abf4-b83941bb0857)

**실질적인 권한 부여하기**

- `CREATE ROLE` 명령은 `빈 껍데기`만 있는 역할을 정의한 것이다.
- `GRANT` 명령으로 각 역할에 대해 `실질적인 권한`을 `부여`해야 한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/4ab761c4-cd06-450c-880c-b15f5dbf1796)

`role_emp_read`는 employees DB에 대한 `읽기` 권한을 부여받았다.

`role_emp_write`는 employees DB에 대한 `쓰기`, `수정`, `삭제` 권한을 부여받았다.

`역할`은 그 자체로 사용할 수 없기 때문에 `계정`에 부여해야 한다.

**권한 부여 테스트 빈계정 생성**

- `읽기 권한`을 가질 `reader`와 `쓰기 권한`을 가질 `writer` 계정을 생성한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/b774c641-0320-4be9-98bd-a0f0c3af3b46)

**계정에 권한 부여**

- `GRANT` 명령어로 각 계정에 맞는 `권한`을 `부여`한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/def97a1b-be74-4f31-9db9-a62999f39125)

**부여한 권한 확인**

- `권한`이 잘 부여되었는지 확인하려면 `SHOW GRANTS` 명령을 사용하면 된다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/90016307-d5a1-44ab-8299-42e34bf0006e)

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/2e6765db-99ed-4eed-b6e1-1f652f5cf739)

각 `계정`에 알맞은 `권한`이 부여된 것을 확인할 수 있다.

**권한 활성화**

`역할`은 부여한다고 바로 `권한`이 부여되어 `사용`할 수 있는 것은 아니다.

`SELECT CURRENT_ROLE()` 명령어로 역할을 조회하면 아무것도 나오지 않는다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/d2a76f99-bda4-4403-ac16-b9b97b7f631e)

`계정`이 `역할`을 사용할 수 있게 하려면 `SET ROLE` 명령을 실행해 해당 `역할`을 `활성화`해야 한다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/de94e5a7-8b5e-4051-b5ed-20af87abf121)

`역할`이 활성화되면 `역할`이 가진 `권한`은 사용할 수 있는 상태지만, 계정이 `로그아웃` 되었다가 다시 `로그인`하면

역할이 `활성화`되지 않은 상태가 된다.

이 문제는 `시스템 변수`인 `activate_all_roles_on_login`의 값을 `ON`으로 주면 해결할 수 있다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/053b619a-9734-40f8-a77a-f892f5f44634)

### 😒 권한의 은밀한 비밀

- `MySQL` 서버 내부적으로 `역할`과 `계정`은 `동일한 객체`로 취급된다.
- 하나의 사용자 계정에 다른 사용자 계정이 가진 `권한`을 `병합`해서 `제어`가 가능한 것 뿐이다.

`MySQL DB`의 `User 테이블`을 살펴보면 실제 권한과 사용자 계정이 `구분 없이 저장`되어 있다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/cb946b5e-4762-4775-8e80-90dacfa0affc)

`역할`과 `계정`의 차이는 `account_locked` 컬럼의 값만 다를 뿐 차이가 없다.

하나의 계정에 다른 계정의 `권한을 병합`하기만 하면 되기 때문에 `역할`과 `계정`을 **구분할 필요가 없다.**

**호스트 부분을 가진 역할**

```sql
mysql> CREATE ROLE role_emp_local_read@localhost;

mysql> CREATE USER reader©'127.0.0.1' IDENTIFIED BY 'qwerty';

mysql> GRANT SELECT ON employees.* TO role_emp_local_read@'localhost'; 

mysql> GRANT role_emp_local_read@'localhost' TO reader©'127.0.0.1';
```

`role_emp_local_read@’localhost’` 역할을 `reader@’127.0.0.1’` 계정에 부여하면 문제가 생길까?

정답은 아무런 문제가 생기지 않는다. 역할의 호스트 부분은 아무런 영향이 없다.

`역할`을 로그인하는 용도로 사용한다면 `호스트 부분`이 `중요`해질 것이다.

그렇다면 `MySQL 서버`에서 왜 `CREATE ROLE` 명령과 `CREATE USER` 명령을 구분했을까?

데이터베이스 관리의 직무를 분리하여 보안을 강화하는 용도로 활용될 수 있기 때문이다.

`CREATE USER` 명령의 권한은 없어도 `CREATE ROLE` 명령만 사용할 수 있는 계정을 만들 수 있다.

즉, `역할`은 `계정`과 `동일`한 객체이지만 `account_locked` 컬럼 값이 `‘Y’` 이므로 **로그인 할 수 없다.**

계정의 `기본 역할` 또는 `역할`에 부여된 역할을 `표 형태`로 깔끔하게 보려면 

`MySQL DB`의 `권한 테이블`을 참조하면 된다.

![image](https://github.com/AK-47-Study/real-mysql-study/assets/91787050/d0b623e1-36c0-4d74-bcd8-0d9539d46d65)
