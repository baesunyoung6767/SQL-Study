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

