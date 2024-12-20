## 논리적인 SQL 개념 용어

### 서브쿼리 위치에 따른 SQL 용어
---
> 서브쿼리란 쿼리 안의 보조쿼리를 가리키는 용어이다.<br>
가장 바깥쪽의 SELECT 문인 메인쿼리를 기준으로 내부에 SELECT 문을 추가로 작성해서 서브쿼리를 만든다. <br>

이처럼 SELECT 문 안쪽에 위치한 SELECT 문은 어느 위치에 작성되었는지에 따라 부르는 용어가 달라진다.
<br>
작성하는 위치는 크게 ```SELECT 절, FROM 절, WHERE 절```로 나뉜다. 

<br>

```
SELECT (SELECT ... FROM ...) ①
FROM (SELECT ... FROM ...) ②
WHERE 컬럼명 IN (SELECT ... FROM ...) ③
```
- ①은 주로 메인 쿼리의 SELECT 절 내부에 하나의 숫자나 문자, 기호 등을 출력하는 SELECT 문으로 ```스칼라 서브쿼리```라고 한다.
- ②는 메인쿼리의 FROM 절 내부에 작성한 SELECT 문으로 ```인라인 뷰```라고 한다.
- ③은 메인쿼리의 WHERE 절 내부에 작성한 SELECT 문으로 ```중첩 서브쿼리```라고 부른다.

<br>

**스칼라 서브 쿼리** <br>
메인 쿼리의 SELECT 절에는 최종 출력하려는 열들이 나열되므로, <u>출력 데이터 1건과 스칼라 서브쿼리의 결과 건수가 일치</u>해야 한다. <br>

<br>

**인라인 뷰** <br>
FROM 절 내부에서 일시적으로 뷰를 생성하는 방식이므로 인라인 뷰라고 부른다. <br>
인라인 뷰의 결과는 내부적으로 <u>메모리 또는 디스크에 임시 테이블을 생성</u>하여 활용한다. <br>
```→ 많이 선호되는 방식```

<br>

**중첩 서브쿼리** <br>
WHERE 절에서 단순한 값을 비교 연산하는 대신 서브쿼리를 추가하여 비교 연산하기 위해 중첩 서브쿼리를 사용한다. <br>
중첩 서브쿼리와 비교할 때는 보툥 비교 연산자(=, <, > ... )를 비롯해 IN, EXISTS, NOT IN, NOT EXISTS 문을 많이 사용한다. <br>

<br>

### 메인쿼리와 관계성에 따른 SQL 용어
---
**비상관 서브쿼리** <br>
메인쿼리와 서브쿼리 간에 <u>관계성이 없음</u>을 의미한다. <br>
서브쿼리가 독자적으로 실행된 뒤 메인쿼리에게 그 결과를 던져주는 형태이다. <br><br>

✅ 서브쿼리가 먼저 실행된 뒤에 그 결과를 메인쿼리가 활용한다. <br>
즉, ```서브 쿼리 → 메인 쿼리``` 순서로 실행된다.

<br><br>

**상관 서브쿼리** <br>
메인쿼리와 서브쿼리 간에 <u>관계성이 있음</u>을 의미한다. <br>
서브쿼리가 수행되려면 메인쿼리의 값을 받아야 하므로 서브쿼리와 메인쿼리는 서로 끈끈한 관계를 유지한다. <br>
→ SELECT 절에 작성하는 **스칼라 서브쿼리**와 WHERE 절에 작성하는 **중첩 서브쿼리**일 때 발생한다. <br>

```
SELECT ...
FROM 학생
WHERE ... IN (SELECT ...
            FROM 지도 교수
            WHERE 학생.학번 = ...)
```
1. 메인쿼리에서 학생 테이블의 학번 결과를 서브쿼리로 전달
2. 지도교수 테이블의 학번과 비교(지도교수.학번 = 학생.학번)
3. 지도교수테이블의 학번과 학생 테이블의 학번이 동일할 때만 서브쿼리의 결과로서 도출
4. 도출된 서브쿼리의 학번 결과를 메인쿼리의 학번과 비교해 최종 결과를 출력

<br>

### 반환 결과에 따른 SQL 용어
---
**단일행 서브쿼리** <br>
서브쿼리 결과가 1건의 행으로 반환되는 쿼리이다. <br>
그에 따라 메인쿼리의 조건절에서는 ```=, <, >``` 등의 연산자와 비교한다. <br><br>

**다중행 서브쿼리**<br>
서브쿼리 결과가 여러 건의 행으로 반환되는 쿼리이다. <br>
그에 따라 메인쿼리의 조건절에서는 ```IN``` 구문으로 서브쿼리에서 반환되는 값들을 받는다. <br><br>

**다중열 서브쿼리** <br>
서브쿼리 결과가 여러 개의 열과 행으로 반환된다. <br>
그에 따라 메인쿼리의 조건절에서는 IN 구문과 함께 서브쿼리에서 반환될 열들을 동일하게 나열해 서브쿼리 결과를 받는다. <br>
```
SELECT ...
FROM ...
WHERE (이름, 전공코드) IN (SELECT 이름, 전공코드
                            FROM 학생
                            WHERE 이름 LIKE '김%')
```

<br><br>

### 조인 연산방식 용어
---
**내부 조인** <br>
말 그대로 교집합에 해당하는 방식으로, 양쪽에 모두 존재하는 데이터만 반환한다. <br>
```
SELECT 학생.학번, 학생.이름, 지도교수.교수명
FROM 학생
JOIN 지도교수
ON 학생.학번 = 지도교수.학번

-- 암시적 조인
SELECT 학생.학번, 학생.이름, 지도교수.교수명
FROM 학생, 지도교수
WHERE 학생.학번 = 지도교수.학번
```
- ```JOIN``` 키워드에는 조인 대상 테이블 작성
- ```ON``` 절에는 조인할 비교 조건 작성

<br>

**왼쪽 외부 조인** <br>
왼쪽 테이블 기준으로 오른쪽 테이블과 조인을 수행하지만 조인 조건과 일치하지 않더라도 <u>왼쪽 테이블의 결과는 최종 결과에 포함</u>된다. <br>
키워드로 ```LEFT OUTER JOIN``` 또는 ```LEFT JOIN``` 사용 <br><br>

**오른쪽 외부 조인** <br>
오른쪽 테이블 기준으로 왼쪽 테이블과 조인을 하지만, 조인 조건과 일치하지 않더라도 <u>오른쪽 테이블의 결과는 최종 결과에 포함</u>된다. <br>
키워드로 ```RIGHT OUTER JOIN```과 ```RIGHT JOIN``` 사용 <br><br>

> 사람의 인지적 특성상 보통 **왼쪽 → 오른쪽**을 정방향으로 인식하므로, <br>
쿼리에서 왼쪽에 위치한 테이블 기준으로 조인을 수행하는 <u>왼쪽 외부 조인을 주로 사용!</u> <br><br>
따라서 오른쪽 외부 조인으로 작성된 SQL문은 왼쪽 외부 조인으로 변경해서<br>
일관성 있는 SQL 문으로 작성하는 편이 ***유지보수나 관리 편의성 측면에서 유리하다***<br>

<br>

**교차 조인** <br>
데카르트 곱이라고 하는 **곱집합 개념**으로, 조인에 참여하는 테이블에서 발생할 수 있는 <u>모든 조합을 찾아내어 반환한다.</u> <br>
※ 오버헤드가 발생할 수 있기 때문에 주의!
```
SELECT 학생.학번, 학생.이름, 지도교수.학번, 지도교수.교수명
FROM 학생
CROSS JOIN 지도교수

-- 암시적으로 작성
SELECT 학생.학번, 학생.이름, 지도교수.학번, 지도교수.교수명
FROM 학생, 지도교수
```
- ```CROSS JOIN``` 키워드만으로 두 테이블을 조건 없이 연결하는 조인이 수행

<br>

**자연 조인**<br>
2개 테이블에 동일한 열명이 있을 때 조인 조건절을 따로 작성하지 않아도 자동으로 조인을 수행한다. <br>
조인이 제대로 성사되면 내부 조인과 동일한 결과가 출력 <br>
```
SELECT 학생.*, 지도교수.*
FROM 학생
NATURAL JOIN 지도교수
```
- ```ON 학생.학번 = 지도교수.학번```과 같은 구문을 입력하면 에러가 발생
- <u>조인 조건절을 알아서 찾아준다</u>
- **동일한 열명이 없다면** 발생 가능한 경우의 수를 모두 조합하는 **교차 조인**으로 수행
- 열명이 달라지면 의도치 않은 결과가 출력될 가능성이 높기 때문에 잘 활용하지 않는다.

<br>

### 조인 알고리즘 용어
---
**드라이빙 테이블과 드리븐 테이블** <br>
- **드라이빙 테이블** : 먼저 접근하는 테이블
- **드리븐 테이블** : 드라이빙 테이블의 검색 결과를 통해 뒤늦게 데이터를 검색하는 테이블

<br>

> ✅ ***적은 결과가 반환될 것으로 에상되는 드라이빙 테이블을 선정하고, 조인 조건절의 열이 인덱스로 설정되도록 구성해야 한다!***

<br>

**중첩 루프 조인** <br>
드라이빙 테이블의 데이터 1건당 드리븐 테이블을 반복해 검색하며 최종적으로는 양쪽 테이블에 공통된 데이터를 출력 <br>

<br>

**블록 중첩 루프 조인**<br>
중첩 루프 조인의 효율성을 높이고자 탄생! <br>
드라이빙 테이블에 대해 **조인 버퍼**라는 개념을 도입하여 조인 성능의 향상을 꾀할 수 있다 <br>
<br>
***BNL 조인 수행 절차 (예시)***
1. 드라이빙 테이블인 학생 테이블에서 학번 1과 100에 해당하는 데이터를 검색 
2. 검색된 데이터를 조인버퍼에 가득 채워질 때까지 적재
3. 조인 버퍼와 비상연락망 테이블의 데이터를 비교한다 <br>
    → 조인 버퍼와 데이터를 조인하고 다시 조인 버퍼와 데이터를 조인하는 식으로 반복하여 모든 비상연락망 데이터에 접근! 


✅ 이처럼 조인 버퍼의 데이터들과 비상연락망 데이틀의 한 번의 데이터 풀 스캔으로 원하는 데이터를 모두 찾을 수 있다!

<br>

**배치 키 액세스 조인** <br>
접근할 데이터를 미리 예상하고 가져오는 데 착안한 조인 알고리즘 <br>
BKA 조인은 블록 중첩 루프 조인에서 활용한 드라이빙 테이블의 조인 버퍼 개념을 그대로 사용한다 <br>
그리고 **드리븐 테이블**에 필요한 데이터를 <u>미리 예측하고 정렬된 상태로 담는 **랜덤 버퍼**</u>의 개념을 도입한다 <br><br>

> ***다중 범위 읽기*** <br>
드리븐 테이블의 데이터를 예측하고 정렬된 상태로 버퍼에 적재하는 기능

✅ 시퀀셜 액세스를 수행하는 방식! 

<br>

**해시 조인** <br>
조인에 참여하는 각 테이블의 데이터를 내부적으로 해시값으로 만들어 내부 조인을 수행한다 <br>
결과는 조인 버퍼에 저장되므로 조인 열의 인덱스를 필수로 요구하지 않아도 된다 <br>
```ex. 학생 테이블의 학생 1 데이터와 비상연락망 테이블의 학번 1의 해시값을 비교한 뒤 서로 동일한 경우에만 조인 버퍼에 저장```