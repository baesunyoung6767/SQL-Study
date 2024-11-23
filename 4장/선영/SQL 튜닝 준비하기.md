## SQL 튜닝 준비하기

SQL문의 구성요소는 크게 두 가지로 구분할 수 있다. <br>
**가시적**으로는 ```테이블 현황과 조건절, 그루핑 열, 정렬되는 열, SELECT 절의 열```이며 **비가시적**으로는 ```실행 계획, 인덱스 현황, 조건절 열들의 데이터 분포, 데이터의 적재 속도, 업무 특성``` 등이다. <br><br>

> ***💡 실무적인 SQL 튜닝 절차*** <br>
① SQL문 실행 결과 & 현황 파악 <br>
② 가시적 / 비가시적 파악 <br>
③ 튜닝 방향 판단 & 개선/적용

<br>

### 기본 키를 변형하는 나쁜 SQL 문
---
**튜닝 전 수행 결과** <br>
```
SELECT *
FROM 사원
WHERE SUBSTRING(사원번호,1,4) = 1100
AND LENGTH(사원번호) = 5;

10 rows in set (0.25sec)
```
→ SUBSTRING이나 LENGTH를 사용하게 된다면 기본 키를 사용하지 않고 테이블 풀 스캔을 수행하게 된다. <br><br>

**튜닝 후 수행 결과** <br>
```
SELECT *
FROM 사원
WHERE 사원번호 BETWEEN 11000 AND 11009

10 rows in set (0.00sec)
```
→ BETWEEN, 비교 연산자를 사용하면 사원번호가 변형되지 않아 기본키나 인덱스를 활용할 수 있게 된다. <br><br>

### 사용하지 않는 함수를 포함하는 나쁜 SQL 문
---
**튜닝 전 수행 결과** <br>
```
SELECT IFNULL(성별, 'NO DATA') AS 성별, COUNT(1) 건수
FROM 사원
GROUP BY IFNULL(성별, 'NO DATA');

2 rows in set (0.391sec)
```
→ 인덱스 풀 스캔 방식 사용 / 성별 칼럼에는 NULL이 존재하지 않음 (=IFNULL 불필요) <br><br>

**튜닝 후 수행 결과** <br>
```
SELECT 성별, COUNT(1) 건수
FROM 사원
GROUP BY 성별

2 rows in set (0.063sec)
```
→ IFNULL() 함수를 처리하려고 DB 내부에 ***별도 임시 테이블을 만들 필요가 없음***

### 형변환으로 인덱스를 활용하지 못하는 나쁜 SQL 문
---
