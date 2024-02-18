mysql의 사용자 계정은 단순히 사용자의 아이디뿐 아니라 해당 사용자가 어느 IP에서 접속하고 있는지도 확인한다. 8.0버전부터 권한을 묶어서 관리하는 역할(Role) 개념이 도입되었기 때문이다. 

## 3.1 사용자 식별

MySQL의 사용자는 사용자의 계정과 사용자의 접속 지점으로 이루어진다.

접속 지점은 클라이언트가 실행된 호스트명이나 도메인 또는 IP주소를 의미한다.

아래와 같이 계정을 언급할 때는 아이디와 호스트를 함께 명시해야 한다. 

```kotlin
`svc_id`@`127.0.0.1`

// -> MySQL 서버가 가동 중인 로컬 호스트에서 svc_id라는 아이디로 접속할 때만 사용가능한 계정
// -> 다른 컴퓨터에서는 접속 불가
```

모든 외부 컴퓨터에서 접속이 가능한 사용자 계정을 생성하고 싶다면 호스트 부분에 `%`를 사용하면 된다.

`%`는 모든 IP 또는 모든 호스트명을 뜻한다. 

동일한 아이디가 존재할 때 MySQL 서버는 범위가 가장 작은 것을 항상 먼저 선택한다. 

```sql
'svc_id'@'192.168.0.10' (비밀번호 123)
'svc_id'@'%' (비밀번호 abc)
```

비밀번호가 123인 것이 범위가 좁으므로 이를 항상 먼저 선택한다. 따라서 192.168.0.10인 PC에서 비밀번호 abc로 로그인하면 비밀번호 불일치로 접속이 거절될 것이다. 

## 3.2 사용자 계정 관리

### 3.2.1 시스템 계정과 일반 계정

`SYSTEM_USER` 권한을 가지고 있느냐에 따라 `시스템 계정(System Account)`과 `일반 계정(Regular Account)`로 나뉜다. 

여기서의 시스템 계정은 MySQL 서버 내부적으로 실행되는 백그라운드 스레드와는 무관하며, 시스템 계정도 일반 계정과 같이 사용자를 위한 계정이다. 

시스템 계정은 DB 서버 관리자를 위한 계정, 일반 계정은 응용 프로그램이나 개발자를 위한 계정으로 생각하면 된다.

시스템 계정이 수행 가능한 작업은 다음과 같다.

- 시스템 계정과 일반 계정을 관리(생성, 삭제, 변경)할 수 있다.
- 다른 세션(Connection) 또는 그 세션에서 실행 중인 쿼리를 강제 종료
- 스토어드 프로그램 생성 시 DEFINER를 타 사용자로 설정

MySQL 서버에는 내장된 계정들이 존재한다. 삭제되지 않도록 주의해야 한다.

- ‘mysql.sys’@’localhost’
    - 8.0부터 기본으로 내장된 sys 스키마의 객체들의 DEFINER로 사용되는 계정
- ‘mysql.session’@’localhost’
    - MySQL 플러그인이 서버로 접근할 때 사용되는 계정
- ‘mysql.infoschema’@’localhost’
    - information_schema에 정의된 뷰의 DEFINER로 사용되는 계정

위 3개의 계정은 처음부터 잠겨있는 상태(`account_locked` 칼럼)이므로 의도적으로 풀지 않는 한 악의적인 용도로 사용이 불가능해 보안 걱정은 넣어두어도 된다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e1cdd83a-bb4e-43bb-b8ac-f046170c6a41/84dbe359-582f-4a67-8e27-c26d289901fc/Untitled.png)

### 3.2.2 계정 생성

MySQL 5.7버전까지는 GRANT 명령으로 권한의 부여와 계정 생성이 가능했다. 
하지만 8.0부터 `계정 생성`은 `CREATE USER` 명령, `권한 부여`는 `GRANT` 명령으로 구분해서 실행해야 한다. 

계정 생성 시 다양한 옵션을 설정할 수 있다.

- 계정의 인증 방식과 비밀번호
- 비밀번호 관련 옵션(비밀번호 유효 기간, 비밀번호 이력 개수, 비밀번호 재사용 불가 기간)
- 기본 역할(Role)
- SSL 옵션
- 계정 잠금 여부

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e1cdd83a-bb4e-43bb-b8ac-f046170c6a41/034fc5a8-e991-4041-ae70-3b1586f7173b/Untitled.png)

<일반적으로 많이 사용되는 옵션을 가진 CREATE USER 명령>

**3.2.2.1 IDENTIFIED WITH**

사용자의 인증 방식과 비밀번호를 설정한다. `IDENTIFIED WITH 뒤에는 반드시 인증 방식`(인증 플러그인의 이름)을 명시해야 한다. 

예를 들어 기본 인증 방식을 사용하고자 한다면 `IDENTIFIED BY ‘password’` 로 사용한다. 

인증의 대표적인 방식에는 4가지가 있다.

- `Native Pluggable Authentication`
    - MySQL 5.7 버전까지 기본 방식
    - 비밀번호에 대한 `해시(SHA-1)` 값 저장 및 클라이언트 값과 비교하는 방식
- `Caching SHA-2 Pluggable Authentication`
    - MySQL 5.6에 도입, 8.0에 보완된 인증 방식 → 8.0부터의 기본 인증 방식
    - 암호화 해시값 생성을 위해 `SHA-2 알고리즘`을 사용
    - 내부적으로 `Salt키`를 활용(Salted Challenge Response Authentication Mechanism, SCRAM)
    - 동일한 키 값에 대해서도 결과가 달라짐
    - 계산에 상당한 시간 소모 → `해시 결과값을 메모리에 캐싱`해서 사용하여 해결 → 이름에 Caching이 포함된 이유
    - `SSL/TLS 또는 RSA 키페어 필수` → 클라이언트에서 접속할 때 SSL 옵션 활성화 필요
- PAM Pluggable Authentication
    - 유닉스, 리눅스 패스워드 또는 LDAP 같은 외부 인증을 사용할 수 있게 해주는 인증 방식
    - MySQL 엔터프라이즈 에디션에서만 사용 가능
- LDAP Pluggable Authentication
    - LDAP를 이용한 외부 인증을 사용할 수 있게 해주는 인증 방식
    - MySQL 엔터프라이즈 에디션에서만 사용 가능

보안 수준이 낮아지더라도 기존 버전과의 호환을 생각한다면 Native Authentication을 선택할 수 있다. 이 경우 아래와 같이 MySQL 설정을 변경하거나 my.cnf 설정 파일에 추가하면 된다.

```sql
-- // Native Authentication을 기본 인증 방식으로 설정
mysql> SET GLOBAL default_authentication_plugin="mysql_native_password"
```

CREATE USER / ALTER USER 명령으로 계정 생성/변경 시 연결 방식과 비밀번호 옵션, 자원 사용과 관련된 여러 옵션을 설정할 수 있다. 

**3.2.2.2 REQUIRE**

MySQL 서버에 접속할 때 암호화된 SSL/TLS 채널을 사용할지 여부를 설정한다. 별도 설정이 없다면 비암호화 채널로 연결한다. 물론 Caching SHA-2 Authentication 인증 방식을 사용한다면 암호화된 채널만 연결 가능하다. 

**3.2.2.3 PASSWORD EXPIRE**

비밀번호의 `유효 기간`을 설정하는 옵션이다. 별도 설정이 없다면 `default_password_lifetime` 시스템 변수 값을 사용한다. 설정 가능한 옵션은 다음과 같다.

- PASSWORD EXPIRE
    - 계정 생성과 동시에 비밀번호 만료
- PASSWORD EXPIRE NEVER
    - 계정 비밀번호 만료 기간 없음
- PASSWORD EXPIRE DEFAULT
    - `default_password_lifetime` 시스템 변수 값을 사용
- PASSWORD EXPRIE INTERVAL `n` DAY
    - 오늘부터 n일자로 비밀번호 유효 기간을 설정

**3.2.2.4 PASSWORD HISTORY**

한 번 사용했던 비밀번호를 `재사용하지 못하게` 설정하는 옵션이다. 비밀번호 이력을 기억하기 위해 MySQL 서버는 mysql DB의 password_history 테이블을 사용한다. 다음은 설정 가능한 옵션이다. 

- PASSWORD HISTORY DEFAULT
    - password_history 시스템 변수에 저장된 개수만큼 비밀번호 이력을 저장
    - 해당 이력의 비밀번호는 사용 불가
- PASSWORD HISTORY `n`
    - 비밀번호 이력을 n개까지만 저장
    - 해당 이력의 비밀번호는 사용 불가

**3.2.2.5 PASSWORD REUSE INTERVAL**

한 번 사용한 비밀번호의 `재사용 금지 기간`을 설정하는 옵션이다. 기본적으로 `password_reuse_interval` 시스템 변수 값을 사용한다. 

- PASSWORD REUSE INTERVAL DEFAULT
    - `password_reuse_interval` 시스템 변수 값을 사용
- PASSWORD REUSE INTERVAL `n` DAY
    - n일자 이후에 비밀번호 재사용 가능

**3.2.2.6 PASSWORD REQUIRE**

새 비밀번호로 `변경할 때 현재 비밀번호를 필요로 할지` 결정하는 옵션이다. 기본 설정은 `password_require_current` 시스템 변수 값이다. 

- PASSWORD REQUIRE CURRENT
    - 현재 비민번호 필요
- PASSWORD REQUIRE OPTIONAL
    - 현재 비밀번호 불필요
- PASSWORD REQUIRE DEFAULT
    - `password_require_current` 시스템 변수 값 사용

**3.2.2.7 ACCOUNT LOCK / UNLOCK**

계정 생성/변경 시 계정을 사용하지 못하게 잠글지 여부를 결정하는 옵션이다. 

## 3.3 비밀번호 관리

### 3.3.1 고수준 비밀번호

MySQL의 비밀번호는 유효기간과 재사용 금지 기능뿐만 아니라 글자 조합 강제나 금칙어 설정 기능이 있다. 

비밀번호 유효성 체크 규칙을 적용하려면 `validate_password` 컴포넌트를 이용하면 되는데, 이는 설치가 필요하다. 

```sql
## validate_password 컴포넌트 설치
mysql> INSTALL COMPONENT 'file://component_validate_password';

## 설치된 컴포넌트 확인
mysql> SELECT * FROM mysql.component;
```

설치되면 컴포넌트에서 제공하는 시스템 변수를 확인할 수 있다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e1cdd83a-bb4e-43bb-b8ac-f046170c6a41/630a1c6e-70a5-4436-89bc-e92bc8869781/Untitled.png)

비밀번호 정책은 크게 3가지 중에서 선택이 가능하다. `기본값은 MEDIUM`이다.

- LOW
    - 비밀번호 `길이만` 검증
- MEDIUM
    - 비밀번호의 `길이`, `숫자와 대소문자`, `특수문자` 배합을 검증
- STRONG
    - `MEDIUM + 금칙어` 포함 여부 검증

| 시스템 변수 | 내용 |
| --- | --- |
| validate_password.length | 길이 |
| validate_password.mixed_case_count | 대소문자 |
| validate_password.number_count | 숫자 |
| validate_password.special_cahr_count | 특수문자 |
| validate_password.dictionary_file | 금칙어 사전 파일 |

금칙어 파일은 한 줄에 하나씩 기록해서 저장한 텍스트 파일로 작성하면 된다. 다음은 그 예시이다. 

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e1cdd83a-bb4e-43bb-b8ac-f046170c6a41/e31c005f-7587-4730-9361-c128f6014f80/Untitled.png)

금칙어 파일을 사용하기 위해 정책을 STRONG으로 변경하는 방법은 다음과 같다.

```sql
mysql> SET GLOBAL validate_password.dictionary_file='prohibitive_word.data';
mysql> SET GLOBAL validate_password.policy='STRONG';
```

### 3.3.2 이중 비밀번호

데이터베이스 서버의 계정 정보는 응용 프로그램 서버로부터 공용으로 사용되는 경우가 많다. 이로 인해 데이터베이스 서버의 계정 정보는 쉽게 변경하기가 어렵다. 

특히, 데이터베이스 계정의 비밀번호는 서비스가 실행 중인 상태에서 변경이 불가능했다. 이를 해결하기 위해 MySQL 8.0부터는 계정의 비밀번호로 2개의 값을 동시에 사용할 수 있도록 기능을 만들었다. 이 기능이 `이중 비밀번호(Dual Password)`이다. 

하나는 프라이머리(Primary), 다른 하나는 세컨더리(Secondary)로 구분된다. 최근 설정 비밀번호가 프라이머리이고 이전 비밀번호가 세컨더리이다. 이중 비밀번호를 사용하려면 다음과 같이 옵션을 추가해야 한다. 

```sql
-- // 비밀번호를 "ytrewq"로 설정
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'old_password';

-- // 비밀번호를 "qwerty"로 변경하면서 기존 비밀번호를 세컨더리 비밀번호로 설정
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password' RETAIN CURRENT PASSWORD;
```

이렇게 설정된 상태에서 데이터베이스에 연결하는 응용 프로그램의 소스코드나 설정 파일의 비밀번호를 새로운 비밀 번호인 `new—password`로 변경하고 배포 및 재시작을 순차적으로 실행한다.
MySQL 서버에 접속하는 모든 응용 프로그램의 재시작이 완료되면 이제 다음 명령으로 세컨더리 비밀
번호는 삭제한다. 세컨더리 비밀번호를 꼭 삭제해야 하는 것은 아니지만 계정의 보안을 위해 세컨더리
비밀번호는 삭제하는 것이 좋다.

```sql
mysql> ALTER USER 'root'@'localhost' DISCARD OLD PASSWORD;
```

## 3.4 권한(Privilege)

MySQL 5.7까지 권한은 `글로벌 권한`과 `객체 단위의 권한`으로 구분된다. 

객체 권한은 데이터베이스나 테이블 객체를 제어하는데 필요한 권한이고, 글로벌 권한은 이외의 객체에 적용되는 권한이다. 

객체 권한은 GRANT 명령으로 권한을 부여할 때 반드시 특정 객체를 명시해야 하고, 

글로벌 권한은 특정 객체를 명시하지 말아야 한다. 

예외적으로 ALL(ALL PRIVILEGES)은 글로벌과 객체 권한 두 가지 용도로 사용된다.

- 특정 객체에 ALL 권한 부여 → 해당 객체에 적용될 수 있는 모든 객체 권한을 부여
- 글로벌로 ALL 사용 → 글로벌 수준에서 가능한 모든 권한을 부여

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e1cdd83a-bb4e-43bb-b8ac-f046170c6a41/2133221f-6e1d-44f8-8cb5-41e3cda6f062/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e1cdd83a-bb4e-43bb-b8ac-f046170c6a41/69f20b66-ce52-4da9-bf37-f491c5f2c652/Untitled.png)

MySQL 8.0부터는 `동적 권한`이 추가되었다.

위 표의 권한은 정적 권한이며, 소스코드에 고정적으로 명시돼 있는 권한을 의미하는 반면, 

동적 권한은 MySQL 서버가 시작되면서 동적으로 생성하는 권한을 의미한다. 

ex) MySQL 서버의 컴포넌트나 플러그인이 설치될 때 등록되는 권한

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e1cdd83a-bb4e-43bb-b8ac-f046170c6a41/6245083a-64e6-40f9-ad4a-22d7870e1b9c/Untitled.png)

이전까지는 `SUPER`라는 권한이 데이터베이스 관리를 위해 꼭 필요했지만, 8.0부터는 SUPER가 잘게 쪼개어져 동적 권한으로 분산되었다. 덕분에 백업 관리자와 복제 관리자 개별로 필요한 권한만 부여 가능하게 되었다. 

권한 부여는 GRANT 명령을 사용한다. 

```sql
mysql> GRANT privilege_list ON db.table TO 'user'@'host';
```

각 권한의 특성(범위)에 따라 GRANT 명령의 ON 절에 명시되는 객체의 내용이 바뀌어야 한다. 

8.0부터는 존재하지 않는 사용자에 대한 GRANT 명령은 에러를 발생시키므로 사용자 생성이 꼭 선행되어야 한다.

GRANT OPTION 권한은 GRANT 명령의 마지막에 WITH GRANT OPTION을 명시해서 부여한다. 

privilege_list에는 구분자(,)를 써서 권한 여러 개를 동시에 명시할 수 있다. 

TO 키워드 뒤에는 권한을 부여할 대상 사용자를 명시하고, ON 키워드 뒤에는 어떤 DB의 어떤 오브젝트에 권한을 부여할지를 결정한다. 이는 권한의 범위에 따라 사용법이 다르다. 

- 글로벌 권한
    
    ```sql
    mysql> GRANT SUPER ON *.* TO 'user'@'localhost';
    ```
    
    글로벌 권한은 특정 DB, 테이블에 부여될 수 없으므로 `ON 절`에는 항상 `*.*`를 사용한다. 
    
    `*.*`는 모든 DB의 모든 오브젝트를 포함, 즉 `MySQL 서버 전체`를 의미한다. 
    
- DB 권한
    
    ```sql
    mysql> GRANT EVENT ON *.* TO 'user'@'localhost';
    mysql> GRANT EVENT ON employees.* TO 'user'@'localhost';
    ```
    
    DB 권한은 특정 DB 혹은 서버에 존재하는 모든 DB에 대해서 권한 부여가 가능하므로 ON 절에 `*.*`, `employees.*` 모두 사용 가능하다. 하지만 DB 권한은 `테이블에 대해 부여가 불가`하므로 employees.department 처럼 테이블은 명시할 수 없다. 
    
- 테이블 권한
    
    ```sql
    mysql> GRANT SELECT, INSERT, UPDATE, DELETE ON *.* TO 'user'@'localhost';
    mysql> GRANT SELECT, INSERT, UPDATE, DELETE ON employees.* TO 'user'@'localhost';
    mysql> GRANT SELECT, INSERT, UPDATE, DELETE ON employees.department TO 'user'@'localhost';
    ```
    
    테이블 권한은 다음이 모두 가능하다.
    
    - `*.*`: `모든 DB`에 대한 권한 부여
    - `employees.*`: `특정 DB`의 오브젝트에 대해 권한 부여
    - `employees.department`: `특정 DB의 특정 테이블`에 대해서만 권한 부여
    
    테이블의 특정 칼럼에 대해서만 권한을 부여하는 경우에는 GRANT 문법이 달라져야 한다. 
    
    칼럼에 부여할 수 있는 권한은 DELETE 제외 INSERT, UPDATE, SELECT 3가지이며, 각 권한 뒤에 칼럼을 명시해야 한다. 
    
    ```sql
    mysql> GRANT SELECT, INSERT, UPDATE**(dept_name)** ON employees.department TO 'user'@'localhost';
    ```
    
    - SELECT, INSERT는 모든 칼럼에 대해, UPDATE는 dept_name 칼럼에 대해서만 수행 가능
    
    여러 가지 레벨이나 범위로 권한을 설정하는 것이 가능하지만 테이블이나 칼럼 단위의 권한은 잘 사용하지 않는다. 
    
    `칼럼 단위의 권한이 하나라도 설정`되면 나머지 모든 테이블의 `모든 칼럼에 대해서도 권한
    체크`를 하기 때문에 칼럼 하나에 대해서만 권한을 설정하더라도 `전체적인 성능에 영향`을 미칠 수 있다.
    
    → GRANT 명령보다는 별도의 뷰(VIEW)를 만들어 사용하는 방법을 생각해볼 수 있다. 
    
    `뷰도 하나의 테이블로 인식`되기 때문에 뷰를 만들어 두면 뷰의 칼럼에 대해 권한을 체크하지 않고 `뷰 자체에 대한 권한만 체크`하게 된다.
    

`SHOW GRANTS` 명령으로 각 계정이나 권한에 부여된 권한, 역할을 확인할 수 있다. 또한, mysql DB의 권한 관련 테이블을 참조하여 표 형태로 깔끔하게 볼 수도 있다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e1cdd83a-bb4e-43bb-b8ac-f046170c6a41/119d6053-faff-4fc2-ad30-b5b57db87c75/Untitled.png)

## 3.5 역할(Role)

8.0부터 `권한을 묶어 역할을 사용`할 수 있다. 내부적으로 역할은 계정과 똑같은 모습을 하고 있다. 

`CREATE ROLE` 명령으로 역할을 정의한다. 

```sql
mysql> CREATE ROLE
					role_emp_read,
					role_emp_write;
```

아직은 빈 껍데기만 있는 역할이므로 GRANT 명령으로 역할에 실질적인 권한을 부여해야 한다. 

```sql
mysql> GRANT SELECT ON employees.* TO role_emp_read;
mysql> GRANT INSERT, UPDATE, DELETE ON employees.* TO role_emp_write;
```

역할은 그 자체로 사용되지 못하고 계정에 부여해야 한다. 

`CREATE USER`로 계정을 만들고, `GRANT`로 역할을 부여한다. 

```sql
// 계정 생성 - 권한 없는 상태
mysql> CREATE USER reader@'127.0.0.1' IDENTIFIED BY 'qwerty';
mysql> CREATE USER writer@'127.0.0.1' IDENTIFIED BY 'qwerty';

// 각 계정에 역할 부여
mysql> GRANT role_emp_read TO reader@'127.0.0.1';
mysql> GRANT role_emp_read, role_emp_write TO writer@'127.0.0.1';
```

`SHOW GRANTS` 명령으로 계정 권한을 확인하면 잘 부여된 것을 확인할 수 있다. 

```sql
mysql> SHOW GRANTS;
+-------------------------------------------+
| Grants for reader@%                       |
| GRANT USAGE ON *.* TO 'reader'@'%'        |
| GRANT 'role_emp_read'@'%' TO 'reader'@'%' |
+-------------------------------------------+
```

하지만 이 상태로 로그인해서 데이터를 조회하거나 변경하려고 하면 `권한이 없다는 에러`를 만나게 된다. 

```sql
linux> mysql -hi27.0.0.1 -ureader -p
mysql> SELECT * FROM employees.employees LIMIT 10；

ERROR 1142 (42000): 
			SELECT command denied to user 'reader'@'localhost' for table 'employees'
```

역할이 부여되긴 했지만 활성화되지 않았기 때문에 발생하는 문제이다.  

`SELECT current_role();` 로 활성화된 역할을 조회해보면 NONE으로 나올 것이다.

따라서 `SET ROLE` 명령으로 해당 `역할을 활성화`해야 한다.

```sql
mysql> SET ROLE 'role_emp_read';
```

`계정이 로그아웃`되면 계정 `활성화 상태도 초기화`된다. 이는 MySQL 서버의 역할이 자동으로 활성화되지 않게 설정되어 있기 때문이다. 이를 관리하는 시스템 변수는 `activate_all_roles_on_login`이다. 

```sql
mysql> SET GLOBAL activate_all_roles_on_login=ON;
```

위와 같이 설정하면 로그인과 동시에 부여된 역할이 자동으로 활성화된다. 

MySQL 서버 내부적으로 역할과 계정은 동일한 객체로 취급된다. 단지 하나의 사용자 계정에 다른 사용자 계정이 가진 권한을 병합해서 권한 제어가 가능해졌을 뿐이다.

mysql DB의 user 테이블을 살펴보면 실제 `권한과 사용자 계정 구분 없이 저장`되어 있다.

```sql
mysql> SELECT user, host, account_locked FROM mysql.user;
+-------------------+-----------+----------------+
| user              | host      | account_locked |
+-------------------+-----------+----------------+
| role_emp_read     | %         | Y              |     // 권한
| role_emp_write    | %         | Y              |     // 권한
| reader            | 127.0.0.1 | N              |     // 계정
| writer            | 127.0.0.1 | N              |     // 계정
| root              | localhost | N              |     // 계정
+-------------------+-----------+----------------+
```

역할과 계정의 차이는 account_locked 칼럼의 값 뿐이다. 

MySQL 서버는 이 둘을 구분할 필요가 없다. 하나의 계정에 다른 계정의 권한을 병합하기만 하면 되기 때문이다. 

CREATE ROLE 명령은 호스트 부분을 명시하지 않았다. 이 경우 호스트 부분에는 ‘모든 호스트(%)’가 자동으로 추가되기 때문에 아래 두 명령은 같다. 

```sql
mysql> CREATE ROLE
					role_emp_read,
					role_emp_write;

mysql> CREATE ROLE
					role_emp_read@'%',
					role_emp_write@'%';
```

아래처럼 계정과 역할의 호스트 부분이 다름에도 불구하고 역할을 계정에 부여하면 어떻게 될까?

```sql
mysql> CREATE ROLE role_emp_local_read@localhost;

mysql> CREATE USER reader@'127.0.0.1' IDENTIFIED BY 'qwerty';

mysql> GRANT SELECT ON employees.* TO role_emp_local_read@'localhost';

mysql> GRANT role_emp_local_read@'localhost' TO reader@'127.0.0.1';
```

정답은 `아무런 영향이 없다`이다. 정확히 말하면 호스트 부분은 아무런 영향을 주지 못하고 SELECT 권한이 잘 부여된다. 호스트 부분은 역할을 직접 로그인하는 용도로 사용할 때 중요해지는 부분이다.

역할과 계정은 내부적으로 동일한 객체인데 `왜 굳이 CREATE ROLE과 CREATE USER 명령을 구분`해 놓았을까?

데이터베이스 관리의 직무를 분리할 수 있게 하여 보안을 강화하기 위해서이다.
