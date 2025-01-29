## 악성 SQL 튜닝으로 전문가 되기

### 처음부터 모든 데이터를 가져오는 나쁜 SQL문
---
**튜닝 전 수행 결과** <br>
```
SELECT 사원.사원번호,
		급여.평균연봉,
		급여.최고연봉,
        급여.최저연봉
FROM 사원,
	(SELECT 사원번호,
			ROUND(AVG(연봉), 0) 평균연봉,
			ROUND(MAX(연봉), 0) 최고연봉,
            ROUND(MIN(연봉), 0) 최저연봉
	FROM 급여
    GROUP BY 사원번호
	) 급여
WHERE 사원.사원번호 = 급여.사원번호
AND 사원.사원번호 BETWEEN 10001 AND 10100;

100 rows in set (3.843sec)
```
→ 사원 테이블의 전체 데이터는 약 30만 건 수준인 데 비해 BETWEEN 구문으로 추출하는 데이터는 100건 뿐이다.

**튜닝 후 수행 결과** <br>
```
SELECT 사원.사원번호,
		(SELECT ROUND(AVG(연봉), 0)
			FROM 급여 AS 급여1
            WHERE 사원번호 = 사원.사원번호
		) AS 평균연봉,
        (SELECT ROUND(MAX(연봉), 0)
			FROM 급여 AS 급여2
            WHERE 사원번호 = 사원.사원번호
		) AS 최고연봉,
        (SELECT ROUND(MIN(연봉), 0)
			FROM 급여 AS 급여3
            WHERE 사원번호 = 사원.사원번호
		) AS 최저연봉
	FROM 사원
    WHERE 사원.사원번호 BETWEEN 10001 AND 10100;

100 rows in set (0.000sec)
```
→ 전체 사원 데이터가 아닌 <u>필요한 사원정보에만 접근</u>하도록 튜닝 <br><br>
> 💡 급여 테이블에 SELECT를 3번이나 하고 있지만 **WHERE 절에서 추출하려는 사원 테이블의 데이터가 전체 데이터 대비 극히 소량**에 불과하므로 인덱스를 활용해서 수행하는 3번의 스칼라 서브쿼리는 많은 리소스를 소모하지않는다.

✅ 다만 위 쿼리의 실행 계획을 살펴보면 ```DEPENDENT SUBQUERY```가 출력되는데 이는 호출을 반복해 일으키므로 자주 반복 호출될 경우에는 지양해야 한다! <br><br>

### 비효율적인 페이징을 수행하는 나쁜 SQL 문
---
**튜닝 전 수행 결과** <br>
```
SELECT 사원.사원번호, 사원.이름, 사원.성, 사원.입사일자
FROM 사원, 급여
WHERE 사원.사원번호 = 급여.사원번호
AND 사원.사원번호 BETWEEN 10001 AND 50000
GROUP BY 사원.사원번호
ORDER BY SUM(급여.연봉) DESC
LIMIT 150, 10;

10 rows in set (0.844sec)
```
→ 전체 데이터를 가져온 뒤 마지막으로 10건의 데이터만 조회하고 있는 것은 비효율적일 수 있다. 

**튜닝 후 수행 결과** <br>
```
SELECT 사원.사원번호, 사원.이름, 사원.성, 사원.입사일자
FROM (SELECT 사원번호
		FROM 급여
        WHERE 사원번호 BETWEEN 10001 AND 50000
        GROUP BY 사원번호
        ORDER BY SUM(급여.연봉) DESC
        LIMIT 150, 10) 급여,
		사원
WHERE 사원.사원번호 = 급여.사원번호;

10 rows in set (0.313sec)
```

<br><br>

### 필요 이상으로 많은 정보를 가져오는 나쁜 SQL 문
---
**튜닝 전 수행 결과** <br>
```
SELECT COUNT(사원번호) AS  카운트
FROM (
	SELECT 사원.사원번호, 부서관리자.부서번호
    FROM (SELECT *
			FROM 사원
            WHERE 성별='M'
            AND 사원번호 > 300000
            ) 사원
		LEFT JOIN 부서관리자
        ON 사원.사원번호 = 부서관리자.사원번호
	) 서브쿼리;

1 rows in set (0.406sec)
```
→ 부서관리자 테이블의 데이터를 보면 사원번호가 300000 초과인 데이터는 존재하지 않는다. 즉, 불필요하게 부서관리자와 조인을 수행하고 있다. <br>

**튜닝 후 수행 결과** <br>
```
SELECT COUNT(사원번호) AS 카운트
FROM 사원
WHERE 성별 = 'M'
AND 사원번호 > 300000;

1 rows in set (0.094sec)
```

<br><br>

### 대량의 데이터를 가져와 조인하는 나쁜 SQL문
---
**튜닝 전 수행 결과** <br>
```
SELECT DISTINCT 매핑.부서번호
FROM 부서관리자 관리자,
	부서사원_매핑 매핑
WHERE 관리자.부서번호 = 매핑.부서번호
ORDER BY 매핑.부서번호;

9 rows in set (1.656sec)
```
→ DISTINCT를 수행하기 전에 가져오는 데이터의 개수가 897570개이다. <u>조인 전에 미리 중복 제거를 할 수 없을지</u> 고민해봐야 한다. <br>
또한, 매핑 테이블의 경우 331603개의 데이터가 저장되어 있다. 많은 수의 데이터를 조회하여 일일이 부서번호가 같은지 확인하기 보다는 둘 중 하나의 테이블은 <u>단순히 부서번호가 존재하는지 여부만 판단해도 충분하지 않을지</u> 고민해봐야 한다. <br>

**튜닝 후 수행 결과**<br>
```
SELECT 매핑.부서번호
FROM (SELECT DISTINCT 부서번호
		FROM 부서사원_매핑 매핑
	) 매핑
WHERE EXISTS (SELECT 1
				FROM 부서관리자 관리자
                WHERE 부서번호 = 매핑.부서번호)
ORDER BY 매핑.부서번호;

9 rows in set (0.000sec)
```
