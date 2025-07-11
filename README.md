# 파티셔닝 전 / 후 메모리 5mb일때 소요 시간.

실제 운영 환경에서는 훨씬 더 큰 자원으로 서비스를 운영하지만, 제한된 자원으로 운영하는 상황이 발생하고, 파티셔닝 전/후 비교를 명확히 확인하기 위해서, mysql에 할당되는 메모리 크기를 하한선까지 설정한 후, 결과 비교를 진행했습니다.

mysql에서는 메모리 하한선은 5MB이며, 그 이하로 설정해도 Mysql에서 확인 시, 5MB는 확보되는 것을 확인할 수 있습니다. 

### OS에서 mysql의 메모리 값을 5MB 미만으로 설정 후, mysql 내부에서 메모리 리소스 확인 결과

OS에서 mysql의 메모리 사이즈를 1M로 설정해도 mysql에서 약 5M로 나오는 것을 확인할 수 있습니다. 

**(innodb_buffer_pool_size =  MySQL 서버의 RAM(메모리)에 할당되는 공간)**

![image.png](image.png)

![image.png](image%201.png)

## OS에서 mysql 메모리 값 설정 확인

![image.png](image%202.png)

### 메모리 5MB ,파티셔닝 전

![image.png](image%203.png)

**파티셔닝을 하기 전에는 최대 2.11초까지 소요가 된다.**

![image.png](image%204.png)

## 메모리 5MB, 파티셔닝 후

데이터베이스에서 적용된 메모리 크기 확인을 위한 명령어.

![image.png](image%203.png)

파티셔닝을 진행한 후, 최대 0.7초까지 확인했으며, 평균 0.3~0.4초 정도 소요되는 것을 확인할 수 있었습니다.

<img width="875" height="205" alt="image" src="https://github.com/user-attachments/assets/e4f2d9ad-8529-48ec-b904-2565c90d8435" />


파티션 테이블 생성 및 데이터 삽입

```sql
CREATE TABLE survey_partitioned (
  id INT NOT NULL AUTO_INCREMENT,
  Age VARCHAR(512),  -- ← 수정됨
  Country VARCHAR(512),
  CompTotal double,
  LanguageHaveWorkedWith varchar(512),
  DatabaseHaveWorkedWith varchar(512),
  RemoteWork VARCHAR(100),
  MainBranch VARCHAR(128),
  DevType TEXT,
  PRIMARY KEY (id, Country)
)
PARTITION BY LIST COLUMNS(Country) (
  PARTITION p_USA VALUES IN ('United States of America'),
  PARTITION p_India VALUES IN ('India'),
  PARTITION p_Germany VALUES IN ('Germany'),
  PARTITION p_Others VALUES IN ('Others')
);			-- 파티션 생성

UPDATE merged_survey
SET Country = 'Others'
WHERE Country NOT IN (
  'United States of America', 'India', 'Germany'
);			-- 미국, 인도, 독일 제외한 다른 나라들 Others로 설정

INSERT INTO survey_partitioned (
  Age, Country, CompTotal, LanguageHaveWorkedWith,
  DatabaseHaveWorkedWith, RemoteWork, MainBranch, DevType
)
SELECT
  Age, Country, CompTotal, LanguageHaveWorkedWith,
  DatabaseHaveWorkedWith, RemoteWork, MainBranch, DevType
FROM merged_survey;	-- 파티션에 데이터 삽입
```

원본 테이블을 조회 후 성능 확인

```sql
-- 원본
EXPLAIN ANALYZE
SELECT *
FROM merged_survey
WHERE Country = 'United States of America' and age = 'Under 18 years old';
```

파티션이 적용된 테이블의 Country를 기준으로 조회 후 성능 확인 

```sql

-- 파티션
EXPLAIN ANALYZE
SELECT *
FROM survey_partitioned
WHERE Country = 'United States of America' and age = 'Under 18 years old';

```
