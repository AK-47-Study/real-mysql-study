# 06 데이터 압축

디스크에 저장된 `데이터 파일의 크기`는 `쿼리 처리 성능` 뿐만 아니라 `백업 및 복구 시간`과도 밀접한 관계가 있다.

데이터 파일이 클수록 쿼리 처리를 위해 `더 많은 데이터 페이지를 InnoDB 버퍼 풀`로 읽어야 할 수도 있고, 새로운 데이터 페이지가 버퍼 풀로 적재되기 때문에 그만큼 `더티 페이지가 더 자주 디스크로 기록`돼야 한다. `백업 시간`도 오래 걸리고, `복구`도 마찬가지다. 저장 공간은 `비용`과도 직결된다.

이 문제를 해결하기 위해 제공하는 기능이 `데이터 압축`이다. MySQL 서버는 `테이블 압축`과 `페이지 압축`을 사용할 수 있다.

## 6.1 페이지 압축

페이지 압축은 `Transparent Page Compression`이라고도 한다. 

MySQL 서버가 `디스크에 저장하는 시점`에 `데이터 페이지가 압축되어 저장`되고, 디스크에 데이터 페이지를 `읽어올 때 압축이 해제`된다. 즉 버퍼 풀에 데이터 페이지가 적재되면 InnoDB 스토리지 엔진은 압축 해제된 상태로만 데이터 페이지를 관리한다. 그래서 MySQL 서버의 내부 코드에서는 압축 여부와 관계없이 `투명하게(Transparent)` 작동한다. 

여기서 문제는 16KB 데이터 페이지를 압축한 결과의 용량 예측이 불가능하다는 점이다. 적어도 `하나의 테이블`은 `동일한 크기의 페이지`로 통일되어야 한다. 

이런 문제 때문에 페이지 압축 기능은 `펀치 홀(Punch Hole)`이라는 기능을 사용한다. 페이지 압축 작동 방식은 다음과 같다.

1. 16KB 페이지를 `압축`(결과: 7KB 로 가정)
2. MySQL 서버는 디스크에 압축된 `결과(7KB)를 기록`
    - 이 때 MySQL 서버는 압축 데이터 7KB에 9KB의 빈 데이터를 기록
3. 7KB 이후의 공간 `9KB에 대해 펀치 홀`을 생성
4. 파일 시스템은 7KB만 남기고 나머지 디스크의 9KB는 `운영체제로 반납`

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e1cdd83a-bb4e-43bb-b8ac-f046170c6a41/6330e5b9-4f17-4f8d-9b08-fe5a1c39ecf4/Untitled.png)

MySQL 서버의 페이지 압축의 문제는 펀치 홀 기능이 운영체제뿐만 아니라 하드웨어 자체에서도 기능을 지원해야 한다는 것이다. 또한, 아직 파일 시스템 관련 명령어가 펀치 홀을 지원하지 못한다. 

게다가 펀치 홀이 적용된 파일을 일반적인 파일 복사 명령(**`cp`**)이나 XtraBackup 같은 백업 도구를 사용하여 복사할 경우, 펀치 홀로 인해 비워진 공간이 다시 채워지게 된다. 결과적으로, 복사된 데이터 파일의 크기는 펀치 홀을 적용하기 전의 원본 크기인 10GB로 확대될 수 있다. 이는 펀치 홀을 통해 얻은 저장 공간 효율성이 복사 과정에서 상실될 수 있음을 의미한다. 

이런 이유로 `페이지 압축은 많이 사용되지 않는다.` 

페이지 압축을 이용하려면 테이블 생성, 변경 시 다음과 같이 설정하면 된다.

```java
-- // 테이블 생성 시
mysql> CREATE TABLE t1 (c1 INT) COMPRESSION="zlib";

-- // 테이블 변경 시
mysql> ALTER TABLE t1 COMPRESSION="zlib";
mysql> OPTIMIZE TABLE t1;
```

## 6.2 테이블 압축

테이블 압축은 운영체제나 하드웨어 제약 없이 사용 가능하여 활용도가 높은 편이다. 데이터 파일 크기를 줄일 수 있는 이득이 있지만 단점도 존재한다.

- `낮은 버퍼 풀 공간 활용률`
- `낮은 쿼리 처리 성능`
- 빈번한 데이터 변경 시 `낮은 압축률`

단점이 발생하는 이유를 이해하기 위해서 압축 과정과 버퍼 풀 적재 과정을 알아야 한다.

### 6.2.1 압축 테이블 생성

테이블 압축의 전제 조건은 `테이블이 별도 테이블 스페이스를 사용`해야 한다. 

이를 위해 `innodb_file_per_table` 시스템 변수가 `ON`으로 설정된 상태에서 테이블을 생성해야 하고, 생성 시 `ROW_FORMAT=COMPRESSED` 옵션을 명시해야 한다. 또한 `KEY_BLOCK_SIZE` 옵션으로 `압축된 페이지의 타깃 페이지 크기를 명시`해야 한다. 

InnoDB 스토리지 엔진의 페이지 크기（`innodb_page_size`）가 `16KB`라면 `KEY_BLOCK_SIZE`는 `4KB` 또는 `8KB`만 설정할 수 있다. 그리고 페이지 크기가 `32KB` 또는 `64KB`인 경우에는 `테이블 압축 적용이 불가`하다.

```java
mysql> SET GLOBAL innodb_file_per_table=ON;

-- // ROW_FORMAT 옵션과 KEY_BLOCK_SIZE 옵션을 모두 명시
mysql> CREATE TABLE compressed_table(
				c1 INT PRIMARY KEY
				)
				ROW_FORMAT=COMPRESSED
				KEY_BLOCK_SIZE=8;
				
		
-- // KEY_BLOCK_SIZE 옵션만 명시
-- // ROW_FORMAT 옵션이 생략되면 자동으로 ROW_FORMAT=COMPRESSED 옵션이 추가됨
mysql> CREATE TABLE compressed_table (
				c1 INT PRIMARY KEY
				)
				KEY_BLOCK_SIZE=8;
```

InnoDB 스토리지 엔진이 압축을 적용하는 방법은 다음과 같다.

1. 데이터 페이지(16KB)를 압축
    1. 압축 결과가 KEY_BLOCK_SIZE(8KB) 이하면 그대로 디스크에 저장
    2. 압축 결과가 KEY_BLOCK_SIZE(8KB) 초과면 원본 페이지를 스플릿해서 2개의 페이지에 8KB씩 저장
2. 나뉜 페이지에 대해 1번을 반복

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e1cdd83a-bb4e-43bb-b8ac-f046170c6a41/02f0806d-89b8-4713-bbb4-2a77666f3b06/Untitled.png)

테이블 압축에서는 InnoDB 스토리지 엔진의 I/O 레이어에서는 아무런 역할을 하지 않는다.

중요한 점은 원본 데이터 페이지의 압축 결과가 KEY_BLOCK_SIZE보다 작거나 같을 때까지 반복해서 스플릿을 진행한다는 것이다. 목표 크기가 잘못 설정되면 처리 성능으로 이어질 수 있으니 주의해야한다.

### 6.2.2 KEY_BLOCK_SIZE 결정

테이블 압축에서 가장 중요한 부분이다. `KEY_BLOCK_SIZE`는 압축된 결과 크기를 예측해서 결정해야 한다. 

압축을 적용하기 전 샘플 데이터를 통해 판단해보는 것이 좋다. 샘플은 데이터 페이지가 최소 10개 이상이 되도록 데이터를 INSERT하는 걸 추천한다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e1cdd83a-bb4e-43bb-b8ac-f046170c6a41/32896d93-8dce-45be-8c50-7c6a2ac8511f/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e1cdd83a-bb4e-43bb-b8ac-f046170c6a41/5bd49b06-b4f8-4735-8a4f-a487edf433d0/Untitled.png)

실제로 해보지는 못했지만, 예제를 살펴보면 압축된 테이블의 PRIMARY 키는 전체 18635번 압축을 실행했는데, 13478번 성공했다. 즉 5175(18635-13478)번 압축했는데, 압축 결과가 KEY_BLOCK_SIZE를 초과해서 데이터 페이지 스플릿을 통해 다시 압축을 실행했다는 의미이다. 

일반적으로 압축 실패율은 3~5% 미만으로 유지할 수 있게 KEY_BLOCK_SIZE를 선택하는 것이 좋다.

압축 실패율이 높다면 압축 과정에 오랜 시간이 걸릴 것을 예측할 수 있다. 

압축 실패율이 높다고 해서 실제 디스크의 데이터 파일 크기가 줄어들지 않는다는 뜻은 아니다.

압축 실패율이 높다고 압축을 사용하면 안된다는 의미는 아니다. INSERT만 빈번한 테이블은 압축 시도가 실패해도 크기가 큰 폭으로 줄어든다면 손해는 아니다. 

반대로 압축 실패율이 낮더라도 조회와 변경이 빈번하다면 압축은 비효율적이다. 

압축 알고리즘(zlib)은 CPU 자원을 많이 소모한다. 

### 6.2.3 압축된 페이지의 버퍼 풀 적재 및 사용

InnoDB 스토리지 엔진은 버퍼 풀에 데이터 페이지를 적재 시 `압축 상태`, `압축 해제 상태` `2개 버전을 관리`한다.

각각 디스크에서 읽은 상태 그대로의 데이터 페이지 목록을 관리하는 `LRU 리스트`와 

압축된 페이지의 해제 버전인 `Unzip_LRU 리스트`로 관리한다. 

MySQL 서버는 압축/압축 해제 테이블이 공존하므로 `LRU 리스트`는 `두 종류의 페이지를 모두` 가질 수 있다.

`Unzip_LRU 리스트`는 압축이 적용되지 않은 테이블의 데이터 페이지는 가지지 않으며, `압축이 적용된 테이블에서 읽은 것`만 압축을 해제한 상태의 데이터 페이지 목록으로 관리한다. 

InnoDB 스토리지 엔진은 압축된 테이블에 대해 버퍼 풀 공간을 이중으로 사용하여 메모리를 낭비하는 문제가 있다. 그리고 압축된 페이지에서 데이터를 읽거나 변경할 때 압축을 해제하면서 CPU를 상대적으로 많이 소모한다.

이 두 문제를 해결하기 위해 Unzip_LRU 리스트를 별도로 관리하고 있다가 요청 패턴에 따라 적절히(`Adaptive`) 처리한다. 

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e1cdd83a-bb4e-43bb-b8ac-f046170c6a41/c3f1452f-394b-49cc-beb0-63fa7920c5da/Untitled.png)

InnoDB 스토리지 엔진은 버퍼 풀에서 압축 해제 데이터 페이지를 적절히 유지하기 위해 `어댑티브 알고리즘`을 사용한다. 

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e1cdd83a-bb4e-43bb-b8ac-f046170c6a41/9f779d54-5958-47f0-9895-646119c2d126/Untitled.png)

### 6.2.4 테이블 압축 관련 설정

테이블 압축과 연관된 시스템 변수는 모두 페이지의 `압축 실패율을 낮추기` 위해 필요한 튜닝 포인트를 제공한다. 

- innodb_cmp_pre_index_enabled
- innodb_compression_level
- innodb_compression_failure_threshold_pct
- innodb_compression_pad_pct_max
- innodb_log_compressed_pages

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/e1cdd83a-bb4e-43bb-b8ac-f046170c6a41/9fd174da-a523-4ca8-b005-25b6c2ee0d5b/Untitled.png)
