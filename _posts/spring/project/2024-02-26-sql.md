---
layout: single
title: "sql"
categories: sql
tag: sql
toc: true
---

## sql


~~~

join - groupby

-----------------------------------------------------------------------------------------------------------------------------------------------------



/************************************
   조인 실습 - 1
*************************************/

-- 직원 정보와 직원이 속한 부서명을 가져오기
select a.*, b.dname 
from hr.emp a
	join hr.dept b on a.deptno = b.deptno;

-- job이 SALESMAN인 직원정보와 직원이 속한 부서명을 가져오기. 
select a.*, b.dname 
from hr.emp a
	join hr.dept b on a.deptno = b.deptno
where job = 'SALESMAN';
	
-- 부서명 SALES와 RESEARCH의 소속 직원들의 부서명, 직원번호, 직원명, JOB 그리고 과거 급여 정보 추출 
select a.dname, b.empno, b.ename, b.job, c.fromdate, c.todate, c.sal 
from hr.dept a
	join hr.emp b on a.deptno = b.deptno
	join hr.emp_salary_hist c on b.empno = c.empno
where a.dname in('SALES', 'RESEARCH')
order by a.dname, b.empno, c.fromdate;

-- 부서명 SALES와 RESEARCH의 소속 직원들의 부서명, 직원번호, 직원명, JOB 그리고 과거 급여 정보중 1983년 이전 데이터는 무시하고 데이터 추출 
select a.dname, b.empno, b.ename, b.job, c.fromdate, c.todate, c.sal 
from hr.dept a
	join hr.emp b on a.deptno = b.deptno
	join hr.emp_salary_hist c on b.empno = c.empno
where  a.dname in('SALES', 'RESEARCH')
and fromdate >= to_date('19830101', 'yyyymmdd')
order by a.dname, b.empno, c.fromdate;

-- 부서명 SALES와 RESEARCH 소속 직원별로 과거부터 현재까지 모든 급여를 취합한 평균 급여
with 
temp_01 as 
(
select a.dname, b.empno, b.ename, b.job, c.fromdate, c.todate, c.sal 
from hr.dept a
	join hr.emp b on a.deptno = b.deptno
	join hr.emp_salary_hist c on b.empno = c.empno
where  a.dname in('SALES', 'RESEARCH')
order by a.dname, b.empno, c.fromdate
)
select empno, max(ename) as ename, avg(sal) as avg_sal
from temp_01 
group by empno; 



-- 직원명 SMITH의 과거 소속 부서 정보를 구할 것. 
select a.ename, a.empno, b.deptno, c.dname, b.fromdate, b.todate  
from hr.emp a
	join hr.emp_dept_hist b on a.empno = b.empno
	join hr.dept c on b.deptno = c.deptno
where a.ename = 'SMITH';

/************************************
   조인 실습 - 2
*************************************/

-- 고객명 Antonio Moreno이 1997년에 주문한 주문 정보를 주문 아이디, 주문일자, 배송일자, 배송 주소를 고객 주소와 함께 구할것.  
select a.contact_name, a.address, b.order_id, b.order_date, b.shipped_date, b.ship_address
from nw.customers a
	join nw.orders b on a.customer_id = b.customer_id
where a.contact_name = 'Antonio Moreno'
and b.order_date between to_date('19970101', 'yyyymmdd') and to_date('19971231', 'yyyymmdd')
;

-- Berlin에 살고 있는 고객이 주문한 주문 정보를 구할것
-- 고객명, 주문id, 주문일자, 주문접수 직원명, 배송업체명을 구할것. 
select a.customer_id, a.contact_name, b.order_id, b.order_date
	, c.first_name||' '||c.last_name as employee_name, d.company_name as shipper_name  
from nw.customers a
	join nw.orders b on a.customer_id = b.customer_id
	join nw.employees c on b.employee_id = c.employee_id
	join nw.shippers d on b.ship_via = d.shipper_id
where a.city = 'Berlin';

--Beverages 카테고리에 속하는 모든 상품아이디와 상품명, 그리고 이들 상품을 제공하는 supplier 회사명 정보 구할것 
select a.category_id, a.category_name, b.product_id, b.product_name, c.supplier_id, c.company_name 
from nw.categories a
	join nw.products b on a.category_id = b.category_id
	join nw.suppliers c on b.supplier_id = c.supplier_id
where category_name = 'Beverages';


-- 고객명 Antonio Moreno이 1997년에 주문한 주문 상품정보를 고객 주소, 주문 아이디, 주문일자, 배송일자, 배송 주소 및
-- 주문 상품아이디, 주문 상품명, 주문 상품별 금액, 주문 상품이 속한 카테고리명, supplier명을 구할 것. 
select a.contact_name, a.address, b.order_id, b.order_date, b.shipped_date, b.ship_address
	, c.product_id, d.product_name, c.amount, e.category_name, f.contact_name as supplier_name
from nw.customers a
	join nw.orders b on a.customer_id = b.customer_id
	join nw.order_items c on b.order_id = c.order_id
	join nw.products d on c.product_id = d.product_id
	join nw.categories e on d.category_id = e.category_id
	join nw.suppliers f on d.supplier_id = f.supplier_id
where a.contact_name = 'Antonio Moreno'
and b.order_date between to_date('19970101', 'yyyymmdd') and to_date('19971231', 'yyyymmdd')
;

/************************************
   조인 실습 - Outer 조인. 
*************************************/	

-- 주문이 단 한번도 없는 고객 정보 구하기. 
select *
from nw.customers a
	left join nw.orders b on a.customer_id = b.customer_id
where b.customer_id is null;

-- 부서정보와 부서에 소속된 직원명 정보 구하기. 부서가 직원을 가지고 있지 않더라도 부서정보는 표시되어야 함. 
select a.*, b.empno, b.ename
from hr.dept a
	left join hr.emp b on a.deptno = b.deptno; 

-- Madrid에 살고 있는 고객이 주문한 주문 정보를 구할것.
-- 고객명, 주문id, 주문일자, 주문접수 직원명, 배송업체명을 구하되, 
-- 만일 고객이 주문을 한번도 하지 않은 경우라도 고객정보는 빠지면 안됨. 이경우 주문 정보가 없으면 주문id를 0으로 나머지는 Null로 구할것. 
select a.customer_id, a.contact_name, coalesce(b.order_id, 0) as order_id, b.order_date
	, c.first_name||' '||c.last_name as employee_name, d.company_name as shipper_name  
from nw.customers a
	left join nw.orders b on a.customer_id = b.customer_id
	left join nw.employees c on b.employee_id = c.employee_id
	left join nw.shippers d on b.ship_via = d.shipper_id
where a.city = 'Madrid';

-- 만일 아래와 같이 중간에 연결되는 집합을 명확히 left outer join 표시하지 않으면 원하는 집합을 가져 올 수 없음.  
select a.customer_id, a.contact_name, coalesce(b.order_id, 0) as order_id, b.order_date
	, c.first_name||' '||c.last_name as employee_name, d.company_name as shipper_name  
from nw.customers a
	left join nw.orders b on a.customer_id = b.customer_id
	join nw.employees c on b.employee_id = c.employee_id
	join nw.shippers d on b.ship_via = d.shipper_id
where a.city = 'Madrid';

-- orders_items에 주문번호(order_id)가 없는 order_id를 가진 orders 데이터 찾기 
select *
from nw.orders a
	left join nw.order_items b on a.order_id = b.order_id
where b.order_id is null;

-- orders 테이블에 없는 order_id가 있는 order_items 데이터 찾기. 
select * 
from nw.order_items a 
	left join nw.orders b on a.order_id = b.order_id
where b.order_id is null;


/************************************
   조인 실습 - Full Outer 조인. 
*************************************/	

-- dept는 소속 직원이 없는 경우 존재. 하지만 직원은 소속 부서가 없는 경우가 없음. 
select a.*, b.empno, b.ename
from hr.dept a
	left join hr.emp b on a.deptno = b.deptno; 

-- full outer join 테스트를 위해 소속 부서가 없는 테스트용 데이터 생성. 
drop table if exists hr.emp_test;

create table hr.emp_test
as
select * from hr.emp;

select * from hr.emp_test;

-- 소속 부서를 Null로 update
update hr.emp_test set deptno = null where empno=7934;

select * from hr.emp_test;

-- dept를 기준으로 left outer 조인시에는 소속직원이 없는 부서는 추출 되지만. 소속 부서가 없는 직원은 추출할 수 없음.  
select a.*, b.empno, b.ename
from hr.dept a
	left join hr.emp_test b on a.deptno = b.deptno; 

-- full outer join 하여 양쪽 모두의 집합이 누락되지 않도록 함. 
select a.*, b.empno, b.ename
from hr.dept a
	full outer join hr.emp_test b on a.deptno = b.deptno; 


/************************************
   조인 실습 - Non Equi 조인과 Cross 조인. 
*************************************/
-- 직원정보와 급여등급 정보를 추출. 
select a.*, b.grade as salgrade
from hr.emp a 
	join hr.salgrade b on a.sal between b.losal and b.hisal;


-- 직원 급여의 이력정보를 나타내며, 해당 급여를 가졌던 시점에서의 부서번호도 함께 가져올것. 
select * 
from hr.emp_salary_hist a
	join hr.emp_dept_hist b on a.empno = b.empno and a.fromdate between b.fromdate and b.todate;

-- cross 조인
with
temp_01 as (
select 1 as rnum 
union all
select 2 as rnum
)
select a.*, b.*
from hr.dept a 
	cross join temp_01 b;

/************************************
   Group by 실습 - 01 
*************************************/	

-- emp 테이블에서 부서별 최대 급여, 최소 급여, 평균 급여를 구할것. 
select deptno, max(sal) as max_sal, min(sal) as min_sal, round(avg(sal), 2) as avg_sal
from hr.emp
group by deptno
;

-- emp 테이블에서 부서별 최대 급여, 최소 급여, 평균 급여를 구하되 평균 급여가 2000 이상인 경우만 추출. 
select deptno, max(sal) as max_sal, min(sal) as min_sal, round(avg(sal), 2) as avg_sal
from hr.emp
group by deptno
having avg(sal) >= 2000
;

-- emp 테이블에서 부서별 최대 급여, 최소 급여, 평균 급여를 구하되 평균 급여가 2000 이상인 경우만 추출(with 절을 이용)
with
temp_01 as (
select deptno, max(sal) as max_sal, min(sal) as min_sal, round(avg(sal), 2) as avg_sal
from hr.emp
group by deptno
)
select * from temp_01 where avg_sal >= 2000;


-- 부서명 SALES와 RESEARCH 소속 직원별로 과거부터 현재까지 모든 급여를 취합한 평균 급여
select b.empno, max(b.ename) as ename, avg(c.sal) as avg_sal --, count(*) as cnt
from hr.dept a
	join hr.emp b on a.deptno = b.deptno
	join hr.emp_salary_hist c on b.empno = c.empno
where  a.dname in('SALES', 'RESEARCH')
group by b.empno
order by 1;

-- 부서명 SALES와 RESEARCH 소속 직원별로 과거부터 현재까지 모든 급여를 취합한 평균 급여(with 절로 풀기)
with 
temp_01 as 
(
select a.dname, b.empno, b.ename, b.job, c.fromdate, c.todate, c.sal 
from hr.dept a
	join hr.emp b on a.deptno = b.deptno
	join hr.emp_salary_hist c on b.empno = c.empno
where  a.dname in('SALES', 'RESEARCH')
order by a.dname, b.empno, c.fromdate
)
select empno, max(ename) as ename, avg(sal) as avg_sal
from temp_01 
group by empno; 

-- 부서명 SALES와 RESEARCH 부서별 평균 급여를 소속 직원들의 과거부터 현재까지 모든 급여를 취합하여 구할것. 
select a.deptno, max(a.dname) as dname, avg(c.sal) as avg_sal, count(*) as cnt 
from hr.dept a
	join hr.emp b on a.deptno = b.deptno
	join hr.emp_salary_hist c on b.empno = c.empno
where  a.dname in('SALES', 'RESEARCH')
group by a.deptno
order by 1;

-- 부서명 SALES와 RESEARCH 부서별 평균 급여를 소속 직원들의 과거부터 현재까지 모든 급여를 취합하여 구할것(with절로 풀기)
with 
temp_01 as 
(
select a.deptno, a.dname, b.empno, b.ename, b.job, c.fromdate, c.todate, c.sal 
from hr.dept a
	join hr.emp b on a.deptno = b.deptno
	join hr.emp_salary_hist c on b.empno = c.empno
where  a.dname in('SALES', 'RESEARCH')
order by a.dname, b.empno, c.fromdate
)
select deptno, max(dname) as dname, avg(sal) as avg_sal
from temp_01 
group by deptno
order by 1; 

/************************************
   Group by 실습 - 02(집계함수와 count(distinct))
*************************************/
-- 추가적인 테스트 테이블 생성. 
drop table if exists hr.emp_test;

create table hr.emp_test
as
select * from hr.emp;

insert into hr.emp_test
select 8000, 'CHMIN', 'ANALYST', 7839, TO_DATE('19810101', 'YYYYMMDD'), 3000, 1000, 20
;

select * from hr.emp_test;

-- Aggregation은 Null값을 처리하지 않음. 
select deptno, count(*) as cnt
	, sum(comm), max(comm), min(comm), avg(comm)
from hr.emp_test
group by deptno;

select mgr, count(*), sum(comm)
from hr.emp
group by mgr;

-- max, min 함수는 숫자열 뿐만 아니라, 문자열,날짜/시간 타입에도 적용가능. 
select deptno, max(job), min(ename), max(hiredate), min(hiredate) --, sum(ename) --, avg(ename)
from hr.emp
group by deptno;

-- count(distinct 컬럼명)은 지정된 컬럼명으로 중복을 제거한 고유한 건수를 추출
select count(distinct job) from hr.emp_test;

select deptno, count(*) as cnt, count(distinct job) from hr.emp_test group by deptno;


/************************************
   Group by 실습 - 03(Group by절에 가공 컬럼 및 case when 적용)
*************************************/
-- emp 테이블에서 입사년도별 평균 급여 구하기.  
select to_char(hiredate, 'yyyy') as hire_year, avg(sal) as avg_sal --, count(*) as cnt
from hr.emp
group by to_char(hiredate, 'yyyy')
order by 1;


-- 1000미만, 1000-1999, 2000-2999와 같이 1000단위 범위내에 sal이 있는 레벨로 group by 하고 해당 건수를 구함. 
select floor(sal/1000)*1000, count(*) from hr.emp
group by floor(sal/1000)*1000;

select *, floor(sal/1000)*1000 as bin_range --, sal/1000, floor(sal/1000)
from hr.emp; 

-- job이 SALESMAN인 경우와 그렇지 않은 경우만 나누어서 평균/최소/최대 급여를 구하기. 
select case when job = 'SALESMAN' then 'SALESMAN'
		      else 'OTHERS' end as job_gubun
	   , avg(sal) as avg_sal, max(sal) as max_sal, min(sal) as min_sal --, count(*) as cnt
from hr.emp
group by case when job = 'SALESMAN' then 'SALESMAN'
		      else 'OTHERS' end ;

/************************************
   Group by 실습 - 04(Group by와 Aggregate 함수의 case when 을 이용한 pivoting)
*************************************/

select job, sum(sal) as sales_sum
from hr.emp a
group by job;


-- deptno로 group by하고 job으로 pivoting 
select sum(case when job = 'SALESMAN' then sal end) as sales_sum
	, sum(case when job = 'MANAGER' then sal end) as manager_sum
	, sum(case when job = 'ANALYST' then sal end) as analyst_sum
	, sum(case when job = 'CLERK' then sal end) as clerk_sum
	, sum(case when job = 'PRESIDENT' then sal end) as president_sum
from emp;


-- deptno + job 별로 group by 		     
select deptno, job, sum(sal) as sal_sum
from hr.emp
group by deptno, job;


-- deptno로 group by하고 job으로 pivoting 
select deptno, sum(sal) as sal_sum
	, sum(case when job = 'SALESMAN' then sal end) as sales_sum
	, sum(case when job = 'MANAGER' then sal end) as manager_sum
	, sum(case when job = 'ANALYST' then sal end) as analyst_sum
	, sum(case when job = 'CLERK' then sal end) as clerk_sum
	, sum(case when job = 'PRESIDENT' then sal end) as president_sum
from emp
group by deptno;

-- group by Pivoting시 조건에 따른 건수 계산 유형(count case when then 1 else null end)
select deptno, count(*) as cnt
	, count(case when job = 'SALESMAN' then 1 end) as sales_cnt
	, count(case when job = 'MANAGER' then 1 end) as manager_cnt
	, count(case when job = 'ANALYST' then 1 end) as analyst_cnt
	, count(case when job = 'CLERK' then 1 end) as clerk_cnt
	, count(case when job = 'PRESIDENT' then 1 end) as president_cnt
from emp
group by deptno;

-- group by Pivoting시 조건에 따른 건수 계산 시 잘못된 사례(count case when then 1 else null end)
select deptno, count(*) as cnt
	, count(case when job = 'SALESMAN' then 1 else 0 end) as sales_cnt
	, count(case when job = 'MANAGER' then 1 else 0 end) as manager_cnt
	, count(case when job = 'ANALYST' then 1 else 0 end) as analyst_cnt
	, count(case when job = 'CLERK' then 1 else 0 end) as clerk_cnt
	, count(case when job = 'PRESIDENT' then 1 else 0 end) as president_cnt
from emp
group by deptno;

-- group by Pivoting시 조건에 따른 건수 계산 시 sum()을 이용
select deptno, count(*) as cnt
	, sum(case when job = 'SALESMAN' then 1 else 0 end) as sales_cnt
	, sum(case when job = 'MANAGER' then 1 else 0 end) as manager_cnt
	, sum(case when job = 'ANALYST' then 1 else 0 end) as analyst_cnt
	, sum(case when job = 'CLERK' then 1 else 0 end) as clerk_cnt
	, sum(case when job = 'PRESIDENT' then 1 else 0 end) as president_cnt
from emp
group by deptno;

/************************************
   Group by rollup 
*************************************/

--deptno + job레벨 외에 dept내의 전체 job 레벨(결국 dept레벨), 전체 Aggregation 수행. 
select deptno, job, sum(sal)
from hr.emp
group by rollup(deptno, job)
order by 1, 2;


-- 상품 카테고리 + 상품별 매출합 구하기
select c.category_name, b.product_name, sum(amount) 
from nw.order_items a
	join nw.products b on a.product_id = b.product_id
	join nw.categories c on b.category_id = c.category_id
group by c.category_name, b.product_name
order by 1, 2
;

-- 상품 카테고리 + 상품별 매출합 구하되, 상품 카테고리 별 소계 매출합 및 전체 상품의 매출합을 함께 구하기 
select c.category_name, b.product_name, sum(amount) 
from nw.order_items a
	join nw.products b on a.product_id = b.product_id
	join nw.categories c on b.category_id = c.category_id
group by rollup(c.category_name, b.product_name)
order by 1, 2
;

-- 년+월+일별 매출합 구하기
-- 월 또는 일을 01, 02와 같은 형태로 표시하려면 to_char()함수, 1, 2와 같은 숫자값으로 표시하려면 date_part()함수 사용. 
select to_char(b.order_date, 'yyyy') as year
	, to_char(b.order_date, 'mm') as month
	, to_char(b.order_date, 'dd') as day
	, sum(a.amount) as sum_amount
from nw.order_items a
	join nw.orders b on a.order_id = b.order_id
group by to_char(b.order_date, 'yyyy'), to_char(b.order_date, 'mm'), to_char(b.order_date, 'dd')
order by 1, 2, 3;

-- 년+월+일별 매출합 구하되, 월별 소계 매출합, 년별 매출합, 전체 매출합을 함께 구하기
with 
temp_01 as (
select to_char(b.order_date, 'yyyy') as year
	, to_char(b.order_date, 'mm') as month
	, to_char(b.order_date, 'dd') as day
	, sum(a.amount) as sum_amount
from nw.order_items a
	join nw.orders b on a.order_id = b.order_id
group by rollup(to_char(b.order_date, 'yyyy'), to_char(b.order_date, 'mm'), to_char(b.order_date, 'dd'))
)
select case when year is null then '총매출' else year end as year
	, case when year is null then null
		else case when month is null then '년 총매출' else month end
	  end as month
	, case when year is null or month is null then null
		else case when day is null then '월 총매출' else day end
	  end as day
	, sum_amount
from temp_01
order by year, month, day
;


/************************************
   Group by cube
*************************************/

-- deptno, job의 가능한 결합으로 Group by 수행. 
select deptno, job, sum(sal)
from hr.emp
group by cube(deptno, job)
order by 1, 2;

-- 상품 카테고리 + 상품별 + 주문처리직원별 매출
select c.category_name, b.product_name, e.last_name||e.first_name as emp_name, sum(amount) 
from nw.order_items a
	join nw.products b on a.product_id = b.product_id
	join nw.categories c on b.category_id = c.category_id
	join nw.orders d on a.order_id = d.order_id
	join nw.employees e on d.employee_id = e.employee_id
group by c.category_name, b.product_name, e.last_name||e.first_name
order by 1, 2, 3
;

--상품 카테고리, 상품별, 주문처리직원별 가능한 결합으로 Group by 수행
select c.category_name, b.product_name, e.last_name||e.first_name as emp_name, sum(amount) 
from nw.order_items a
	join nw.products b on a.product_id = b.product_id
	join nw.categories c on b.category_id = c.category_id
	join nw.orders d on a.order_id = d.order_id
	join nw.employees e on d.employee_id = e.employee_id
group by cube(c.category_name, b.product_name, e.last_name||e.first_name)
order by 1, 2, 3
;

------

date 

-----------------------------------------------------------------------------------------------------------------------------------------------------



/******************************************************
to_date, to_timestamp 로 문자열을 Date, Timestamp로 변환.
to_char로 Date, Timestamp를 문자열로 변환. 
*******************************************************/

-- 문자열을 formating에 따라 Date, Timestamp로 변환. 
select to_date('2022-01-01', 'yyyy-mm-dd');

select to_timestamp('2022-01-01', 'yyyy-mm-dd');

select to_timestamp('2022-01-01 14:36:52', 'yyyy-mm-dd hh24:mi:ss')

-- Date를 Timestamp로 변환
select to_date('2022-01-01', 'yyyy-mm-dd')::timestamp;

-- Timestamp를 Text로 변환
select to_timestamp('2022-01-01', 'yyyy-mm-dd')::text;

-- Timestamp를 Date로 변환. 
select to_timestamp('2022-01-01 14:36:52', 'yyyy-mm-dd hh24:mi:ss')::date


-- to_date, to_timestamp, to_char 실습-1
with
temp_01 as (
select a.*
	  , to_char(hiredate, 'yyyy-mm-dd') as hiredate_str
from hr.emp a
)
select empno, ename, hiredate, hiredate_str
	, to_date(hiredate_str, 'yyyy-mm-dd') as hiredate_01
	, to_timestamp(hiredate_str, 'yyyy-mm-dd') as hiretime_01
	--, to_timestamp(hiredate_str, 'yyyy-mm-dd hh24:mi:ss') as hiretime_02
	, to_char(hiredate, 'yyyymmdd hh24:mi:ss') as hiredate_str_01
	, to_char(hiredate, 'month dd yyyy') as hiredate_str_02
	, to_char(hiredate, 'MONTH dd yyyy') as hiredate_str_03
	, to_char(hiredate, 'yyyy month') as hiredate_str_04
	-- w 는 해당 달의 week, d는 일요일(1) 부터 토요일(7)
	, to_char(hiredate, 'MONTH w d') as hiredate_str_05
	-- day는 요일을 문자열로 나타냄. 
	, to_char(hiredate, 'Month, Day') as hiredate_str_06
from temp_01;

-- to_date, to_timestamp, to_char 실습-2 
with
temp_01 as (
select a.*
	  , to_char(hiredate, 'yyyy-mm-dd') as hire_date_str
	  , hiredate::timestamp as hiretime
from hr.emp a
)
select empno, ename, hiredate, hire_date_str, hiretime
	, to_char(hiretime, 'yyyy/mm/dd hh24:mi:ss') as hiretime_01
	, to_char(hiretime, 'yyyy/mm/dd PM hh12:mi:ss') as hiretime_02
	, to_timestamp('2022-03-04 22:10:15', 'yyyy-mm-dd hh24:mi:ss') as timestamp_01
	, to_char(to_timestamp('2022-03-04 22:10:15', 'yyyy-mm-dd hh24:mi:ss'), 'yyyy/mm/dd AM hh12:mi:ss') as timestr_01
from temp_01;  


/********************************************************************
extract와 date_part를 이용하여 Date/Timestamp에서 년,월,일/시간,분,초 추출
*********************************************************************/

-- extract와 date_part를 이용하여 년, 월, 일 추출
select a.* 
	, extract(year from hiredate) as year
	, extract(month from hiredate) as month
	, extract(day from hiredate) as day
from hr.emp a;

select a.*
	, date_part('year', hiredate) as year
	, date_part('month', hiredate) as month
	, date_part('day', hiredate) as day
from hr.emp a;

-- extract와 date_part를 이용하여 시간, 분, 초 추출. 
select date_part('hour', '2022-02-03 13:04:10'::timestamp) as hour
	, date_part('minute', '2022-02-03 13:04:10'::timestamp) as minute
	, date_part('second', '2022-02-03 13:04:10'::timestamp) as second
;

select extract(hour from '2022-02-03 13:04:10'::timestamp) as hour
	, extract(minute from '2022-02-03 13:04:10'::timestamp) as minute
	, extract(second from '2022-02-03 13:04:10'::timestamp) as second


/******************************************************
날짜와 시간 연산. interval의 활용. 
*******************************************************/

-- 날짜 연산 
-- Date 타입에 숫자값을 더하거나/빼면 숫자값에 해당하는 일자를 더해거나/빼서 날짜 계산. 
select to_date('2022-01-01', 'yyyy-mm-dd') +  2 as date_01;

-- Date 타입에 곱하기나 나누기는 할 수 없음. 
select to_date('2022-01-01', 'yyyy-mm-dd') * 10 as date_01;

-- Timestamp 연산. +7을 하면 아래는 오류를 발생. 
select to_timestamp('2022-01-01 14:36:52', 'yyyy-mm-dd hh24:mi:ss') + 7;

-- Timestamp는 interval 타입을 이용하여 연산 수행. 
select to_timestamp('2022-01-01 14:36:52', 'yyyy-mm-dd hh24:mi:ss') + interval '7 hour' as timestamp_01;

select to_timestamp('2022-01-01 14:36:52', 'yyyy-mm-dd hh24:mi:ss') + interval '2 days' as timestamp_01;

select to_timestamp('2022-01-01 14:36:52', 'yyyy-mm-dd hh24:mi:ss') + interval '2 days 7 hours 30 minutes' as timestamp_01;

-- Date 타입에 interval을 더하면 Timestamp로 변환됨. 
select to_date('2022-01-01', 'yyyy-mm-dd') + interval '2 days' as date_01;

-- interval '2 days'와 같이 ' '내에는 days나 day를 혼용해도 되지만 interval '2' day만 허용. 
select to_date('2022-01-01', 'yyyy-mm-dd') + interval '2' day as date_01;

-- 날짜 간의 차이 구하기. 차이값은 정수형.  
select to_date('2022-01-03', 'yyyy-mm-dd') - to_date('2022-01-01', 'yyyy-mm-dd') as interval_01
	, pg_typeof(to_date('2022-01-03', 'yyyy-mm-dd') - to_date('2022-01-01', 'yyyy-mm-dd')) as type ;

-- Timestamp간의 차이 구하기. 차이값은 interval 
select to_timestamp('2022-01-01 14:36:52', 'yyyy-mm-dd hh24:mi:ss') 
     - to_timestamp('2022-01-01 12:36:52', 'yyyy-mm-dd hh24:mi:ss') as time_01
     , pg_typeof(to_timestamp('2022-01-01 08:36:52', 'yyyy-mm-dd hh24:mi:ss') 
     - to_timestamp('2022-01-01 12:36:52', 'yyyy-mm-dd hh24:mi:ss')) as type
;

-- date + date는 허용하지 않음. 
select to_date('2022-01-03', 'yyyy-mm-dd') +  to_date('2022-01-01', 'yyyy-mm-dd')

-- now(), current_timestamp, current_date, current_time 
-- interval을 년, 월, 일로 표시하기. justify_interval와 age 사용 차이
with 
temp_01 as (
select empno, ename, hiredate, now(), current_timestamp, current_date, current_time
	, date_trunc('second', now()) as now_trunc
	, now() - hiredate as 근속기간
from hr.emp
)
select *
	, date_part('year', 근속기간)
	, justify_interval(근속기간)
	, age(hiredate)
	, date_part('year', justify_interval(근속기간))||'년 '||date_part('month', justify_interval(근속기간))||'월' as 근속년월
	, date_part('year', age(hiredate))||'년 '||date_part('month', age(hiredate))||'월' as 근속년월_01
from temp_01;




/******************************************************
date_trunc 함수를 이용하여 년/월/일/시간/분/초 단위 절삭 
*******************************************************/

select trunc(99.9999, 2);

--date_trunc는 인자로 들어온 기준으로 주어진 날짜를 절삭(?),
select date_trunc('day', '2022-03-03 14:05:32'::timestamp)

-- date타입을 date_trunc해도 반환값은 timestamp타입임. 
select date_trunc('day', to_date('2022-03-03', 'yyyy-mm-dd')) as date_01;

-- 만약 date 타입을 그대로 유지하려면 ::date로 명시적 형변환 
select date_trunc('day', '2022-03-03'::date)::date as date_01

-- 월, 년으로 절단. 
select date_trunc('month', '2022-03-03'::date)::date as date_01;

-- week의 시작 날짜 구하기. 월요일 기준.
select date_trunc('week', '2022-03-03'::date)::date as date_01;

-- week의 마지막 날짜 구하기. 월요일 기준(일요일이 마지막 날짜)
select (date_trunc('week', '2022-03-03'::date) + interval '6 days')::date as date_01;

-- week의 시작 날짜 구하기. 일요일 기준.
select date_trunc('week', '2022-03-03'::date)::date -1 as date_01;

-- week의 마지막 날짜 구하기. 일요일 기준(토요일이 마지막 날짜)
select (date_trunc('week', '2022-03-03'::date)::date - 1 + interval '6 days')::date as date_01;

-- month의 마지막 날짜 
select (date_trunc('month', '2022-03-03'::date) + interval '1 month' - interval '1 day')::date;

-- 시분초도 절삭 가능. 
select date_trunc('hour', now());

--date_trunc는 년, 월, 일 단위로 Group by 적용 시 잘 사용됨.
drop table if exists hr.emp_test;

create table hr.emp_test
as
select a.*, hiredate + current_time
from hr.emp a;

select * from hr.emp_test;

-- 입사월로 group by
select date_trunc('month', hiredate) as hire_month, count(*)
from hr.emp_test
group by date_trunc('month', hiredate);

-- 시분초가 포함된 입사일일 경우 시분초를 절삭한 값으로 group by 
select date_trunc('day', hiredate) as hire_day, count(*)
from hr.emp_test
group by date_trunc('day', hiredate);





------

anylast

-----------------------------------------------------------------------------------------------------------------------------------------------------



/* 0. 강의 SQL */

-- order_items 테이블에서 order_id 별 amount 총합까지 표시
select order_id, line_prod_seq, product_id, amount
	, sum(amount) over (partition by order_id) as total_sum_by_ord from nw.order_items 

-- order_items 테이블에서시 rder_id별 line_prod_seq순으로  누적 amount 합까지 표시
select order_id, line_prod_seq, product_id, amount
	, sum(amount) over (partition by order_id) as total_sum_by_ord 
	, sum(amount) over (partition by order_id order by line_prod_seq) as cum_sum_by_ord
from nw.order_items;

-- order_items 테이블에서 order_id별 line_prod_seq순으로  누적 amount 합 - partition 또는 order by 절이 없을 경우 windows. 
select order_id, line_prod_seq, product_id매 amount
	, sum(amount) over (partition by order_id) as total_sum_by_ord 
	, sum(amount) over (partition by order_id order by line_prod_seq) as cum_sum_by_ord_01
	, sum(amount) over (partition by order_id order by line_prod_seq rows between unbounded preceding and current row) as cum_sum_by_ord_02
	, sum(amount) over ( ) as total_sum
from nw.order_items where order_id between 10248 and 10250;

-- order_items 테이블에서 order_id 별 상품 최대 구매금액, order_id별 상품 누적 최대 구매금액
select order_id, line_prod_seq, product_id, amount
	, max(amount) over (partition by order_id) as max_by_ord 
	, max(amount) over (partition by order_id order by line_prod_seq) as cum_max_by_ord
from nw.order_items;

-- order_items 테이블에서 order_id 별 상품 최소 구매금액, order_id별 상품 누적 최소 구매금액
select order_id, line_prod_seq, product_id, amount
	, min(amount) over (partition by order_id) as min_by_ord 
	, min(amount) over (partition by order_id order by line_prod_seq) as cum_min_by_ord
from nw.order_items;

-- order_items 테이블에서 order_id 별 상품 평균 구매금액, order_id별 상품 누적 평균 구매금액
select order_id, line_prod_seq, product_id, amount
	, avg(amount) over (partition by order_id) as avg_by_ord 
	, avg(amount) over (partition by order_id order by line_prod_seq) as cum_avg_by_ord
from nw.order_items;


/* 1. aggregation analytic 실습 */ 

-- 직원 정보 및 부서별로 직원 급여의 hiredate순으로 누적 급여합. 
select empno, ename, deptno, sal, hiredate, sum(sal) over (partition by deptno order by hiredate) cum_sal from hr.emp; 

--직원 정보 및 부서별 평균 급여와 개인 급여와의 차이 출력
select empno, ename, deptno, sal, avg(sal) over (partition by deptno) dept_avg_sal
	, sal - avg(sal) over (partition by deptno) dept_avg_sal_diff
from hr.emp;

-- analytic을 사용하지 않고 위와 동일한 결과 출력
with 
temp_01 as (
	select deptno, avg(sal) as dept_avg_sal 
	from hr.emp group by deptno
)
select a.empno, a.ename, a.deptno, b.dept_avg_sal,
	a.sal - b.dept_avg_sal as dept_avg_sal_diff
from hr.emp a 
	join temp_01 b
		on a.deptno = b.deptno
order by a.deptno
;

-- 직원 정보및 부서별 총 급여 대비 개인 급여의 비율 출력(소수점 2자리까지로 비율 출력)
select empno, ename, deptno, sal, sum(sal) over (partition by deptno) as dept_sum_sal
	, round(sal/sum(sal) over (partition by deptno), 2) as dept_sum_sal_ratio
from hr.emp;


-- 직원 정보 및 부서에서 가장 높은 급여 대비 비율 출력(소수점 2자리까지로 비율 출력)
select empno, ename, deptno, sal, max(sal) over (partition by deptno) as dept_max_sal
	, round(sal/max(sal) over (partition by deptno), 2) as dept_max_sal_ratio
from hr.emp;


-- product_id 총 매출액을 구하고, 전체 매출 대비 개별 상품의 총 매출액 비율을 소수점2자리로 구한 뒤 매출액 비율 내림차순으로 정렬
with 
temp_01 as (
	select product_id, sum(amount) as sum_by_prod
	from order_items
	group by product_id
)
select product_id, sum_by_prod
	, sum(sum_by_prod) over () total_sum
	, round(1.0 * sum_by_prod/sum(sum_by_prod) over (), 2) as sum_ratio
from temp_01
order by 4 desc;

-- 직원별 개별 상품 매출액, 직원별 가장 높은 상품 매출액을 구하고, 직원별로 가장 높은 매출을 올리는 상품의 매출 금액 대비 개별 상품 매출 비율 구하기
with 
temp_01 as (
	select b.employee_id, a.product_id, sum(amount) as sum_by_emp_prod
	from order_items a
		join orders b on a.order_id = b.order_id
	group by b.employee_id, a.product_id
)
select employee_id, product_id, sum_by_emp_prod
	, max(sum_by_emp_prod) over (partition by employee_id) as sum_by_emp
	, sum_by_emp_prod/max(sum_by_emp_prod) over (partition by employee_id) as sum_ratio
from temp_01
order by 1, 5 desc;



-- 상품별 매출합을 구하되, 상품 카테고리별 매출합의 5% 이상이고, 동일 카테고리에서 상위 3개 매출의 상품 정보 추출. 
-- 1. 상품별 + 상품 카테고리별 총 매출 계산. (상품별 + 상품 카테고리별 총 매출은 결국 상품별 총 매출임)
-- 2. 상품 카테고리별 총 매출 계산 및 동일 카테고리에서 상품별 랭킹 구함
-- 3. 상품 카테고리 매출의 5% 이상인 상품 매출과 매출 기준 top 3 상품 추출.  
with
temp_01 as (
	select a.product_id, max(b.category_id) as category_id , sum(amount) sum_by_prod
	from  order_items a
		join products b 
			on a.product_id = b.product_id 
	group by  a.product_id
), 
temp_02 as (
select product_id, category_id, sum_by_prod
	, sum(sum_by_prod) over (partition by category_id) as sum_by_cat
	, row_number() over (partition by category_id order by sum_by_prod desc) as top_prod_ranking
from temp_01
)
select * from temp_02 where sum_by_prod >= 0.05 * sum_by_cat and top_prod_ranking <=3;





--lag() 현재 행보다 이전 행의 데이터를 가져옴. 동일 부서에서 hiredate순으로 이전 ename을 가져옴. 
select empno, deptno, hiredate, ename
	, lag(ename) over (partition by deptno order by hiredate) as prev_ename
from hr.emp;

-- lead( ) 현재 행보다 다음 행의 데이터를 가져옴. 동일 부서에서 hiredate순으로 다음 ename을 가져옴. 
select empno, deptno, hiredate, ename, 
	lead(ename) over (partition by deptno order by hiredate) as next_ename
from hr.emp;

-- lag() over (order by desc)는 lead() over (order by asc)와 동일하게 동작하므로 혼돈을 방지하기 위해 order by 는 asc로 통일하는것이 좋음. 
select empno, deptno, hiredate, ename
	, lag(ename) over (partition by deptno order by hiredate desc) as lag_desc_ename
	, lead(ename) over (partition by deptno order by hiredate) as lead_desc_ename
from hr.emp; 

-- lag 적용 시 windows에서 가져올 값이 없을 경우 default 값을 설정할 수 있음. 이 경우 반드시 offset을 정해 줘야함. 
select empno, deptno, hiredate, ename
	, lag(ename, 1, 'No Previous') over (partition by deptno order by hiredate) as prev_ename 
from hr.emp;

-- Null 처리를 아래와 같이 수행할 수도 있음. 
select empno, deptno, hiredate, ename
	, coalesce(lag(ename) over (partition by deptno order by hiredate), 'No Previous') as prev_ename 
from hr.emp;

-- 현재일과 1일전 매출데이터와 그 차이를 출력. 1일전 매출 데이터가 없을 경우 현재일 매출 데이터를 출력하고, 차이는 0
with
temp_01 as (
select date_trunc('day', b.order_date)::date as ord_date, sum(amount) as daily_sum
from nw.order_items a
	join nw.orders b on a.order_id = b.order_id
group by date_trunc('day', b.order_date)::date 
)
select ord_date, daily_sum
	, coalesce(lag(daily_sum) over (order by ord_date), daily_sum) as prev_daily_sum
	, daily_sum - coalesce(lag(daily_sum) over (order by ord_date), daily_sum) as diff_prev
from temp_01;





-- 부서별로 가장 hiredate가 오래된 사람의 sal 가져오기.
select empno, ename, deptno, hiredate, sal 
	, first_value(sal) over (partition by deptno order by hiredate) as first_hiredate_sal 
from emp;

-- 부서별로 가장 hiredate가 최근인 사람의 sal 가져오기. windows절이 rows between unbounded preceding and unbounded following이 되어야 함. 
select empno, ename, deptno, hiredate, sal 
, last_value(sal) over (partition by deptno order by hiredate
						rows between unbounded preceding and unbounded following) as last_hiredate_sal_01
, last_value(sal) over (partition by deptno order by hiredate 
						rows between unbounded preceding and current row) as last_hiredate_sal_02
from emp;

-- last_value() over (order by asc) 대신 first_value() over (order by desc)를 적용 가능. 
select empno, ename, deptno, hiredate, sal 
, last_value(sal) over (partition by deptno order by hiredate rows between unbounded preceding and unbounded following) as last_hiredate_sal
, first_value(sal) over (partition by deptno order by hiredate desc) as last_hiredate_sal
from emp;

-- first_value()와 min() 차이
select empno, ename, deptno, hiredate, sal 
	, first_value(sal) over (partition by deptno order by hiredate) as first_hiredate_sal 
	, min(sal) over (partition by deptno order by hiredate) as min_sal
from emp;

-- 연속된 데이터 흐름에서 값이 Null일 경우 바로 값이 있는 바로 위의 데이터를 가져 오기. 
with ref_days
as (
	select generate_series('1996-07-04'::date , '1996-07-23'::date, '1 day'::interval)::date as ord_date
), 
temp_01 as (
	select date_trunc('day', b.order_date)::date as ord_date, sum(amount) as daily_sum
	from nw.order_items a
		join nw.orders b on a.order_id = b.order_id
	group by date_trunc('day', b.order_date)::date 
),
temp_02 as (
	select a.ord_date, b.daily_sum as daily_sum
	from ref_days a
		left join temp_01 b on a.ord_date = b.ord_date
), 
temp_03 as 
(
select *
, first_value(daily_sum) over (order by ord_date)
, case when daily_sum is null then 0
	else row_number() over () end as rnum
from temp_02
),
temp_04 as (
select *
	, max(lpad(rnum::text, 6, '0')||daily_sum) over (order by ord_date rows between unbounded preceding and current row) as temp_str
from temp_03 order by ord_date
)
select * 
	, substring(temp_str, 7)::float as inherited_daily_sum
from temp_04;



/* 0. 강의 SQL */

-- rank, dense_rank, row_number 사용하기 - 1
select a.empno, ename, job, sal 
	, rank() over(order by sal desc) as rank 
	, dense_rank() over(order by sal desc) as dense_rank
	, row_number() over (order by sal desc) as row_number 
from hr.emp a;

-- rank, dense_rank, row_number 사용하기 - 2
select a.empno, ename, job, deptno, sal 
	, rank() over(partition by deptno order by sal desc) as rank 
	, dense_rank() over(partition by deptno order by sal desc) as dense_rank
	, row_number() over (partition by deptno order by sal desc) as row_number 
from hr.emp a

/* 1. 순위 함수 실습 */

-- 회사내 근무 기간 순위(hiredate) : 공동 순위가 있을 경우 차순위는 밀려서 순위 정함
select a.*
	, rank() over (order by hiredate) as hire_rank 
from hr.emp a;

-- 부서별로 가장 급여가 높은/낮은 순으로 순위: 공동 순위 시 차순위는 밀리지 않음.
select a.*
	, dense_rank() over (partition by deptno order by sal desc) as sal_rank_desc
	, dense_rank() over (partition by deptno order by sal ) as sal_rank_asc
from hr.emp a;

-- 부서별 가장 급여가 높은 직원 정보:  공동 순위는 없으며 반드시 고유 순위를 정함.  
select * 
from 
(
	select a.*
		, row_number() over (partition by deptno order by sal desc) as sal_rn
	from hr.emp a
) a where sal_rn = 1;

-- 부서별 급여 top 2 직원 정보: 공동 순위는 없으며 반드시 고유 순위를 정함. 
select * 
from 
(
	select a.*
		, row_number() over (partition by deptno order by sal desc) as sal_rn
	from hr.emp a
) a where sal_rn <=2;

-- 부서별 가장 급여가 높은 직원과 가장 급여가 낮은 직원 정보. 공동 순위는 없으며 반드시 고유 순위를 정함
select a.*
	, case when sal_rn_desc=1 then 'top'
	       when sal_rn_asc=1 then 'bottom'
	       else 'middle' end as gubun
from (
	select a.*
		, row_number() over (partition by deptno order by sal desc) as sal_rn_desc
		, row_number() over (partition by deptno order by sal asc) as sal_rn_asc
	from hr.emp a
) a where sal_rn_desc = 1 or sal_rn_asc=1;

-- 부서별 가장 급여가 높은 직원과 가장 급여가 낮은 직원 정보 그리고 두 직원값의 급여차이도 함께 추출. 공동 순위는 없으며 반드시 고유 순위를 정함
with
temp_01 as (
	select a.*
		, case when sal_rn_desc=1 then 'top'
		       when sal_rn_asc=1 then 'bottom'
		       else 'middle' end as gubun
	from (
		select a.*
			, row_number() over (partition by deptno order by sal desc) as sal_rn_desc
			, row_number() over (partition by deptno order by sal asc) as sal_rn_asc
		from hr.emp a
	) a where sal_rn_desc = 1 or sal_rn_asc=1
),
temp_02 as (
	select deptno
		, max(sal) as max_sal, min(sal) as min_sal
	from temp_01 group by deptno
)
select a.*, b.max_sal - b.min_sal as diff_sal 
from temp_01 a 
	join temp_02 b on a.deptno = b.deptno
order by a.deptno, a.sal desc;

-- 회사내 커미션 높은 순위. rank와 row_number 모두 추출. 
select a.*
	, rank() over (order by comm desc) as comm_rank
	, row_number() over (order by comm desc) as comm_rnum
from hr.emp a;

/* 2. 순위 함수에서 null 처리 실습 */

-- null을 가장 선두 순위로 처리
select a.*
	, rank() over (order by comm desc nulls first ) as comm_rank
	, row_number() over (order by comm desc nulls first) as comm_rnum
from hr.emp a;

-- null을 가장 마지막 순위로 처리
select a.*
	, rank() over (order by comm desc nulls last ) as comm_rank
	, row_number() over (order by comm desc nulls last) as comm_rnum
from hr.emp a;

-- null을 전처리하여 순위 정함. 
select a.*
	, rank() over (order by COALESCE(comm, 0) desc ) as comm_rank
	, row_number() over (order by COALESCE(comm, 0) desc) as comm_rnum
from hr.emp a;



/************************************************
cume_dist, percent_rank, ntile 실습
 *************************************************/

-- cume_dist는 percentile을 파티션내의 건수로 적용하고 0 ~ 1 사이 값으로 변환. 
-- 파티션내의 자신을 포함한 이전 로우수/ 파티션내의 로우 건수로 계산될 수 있음. 
select a.empno, ename, job, sal
	, rank() over(order by sal desc) as rank 
	, cume_dist() over (order by sal desc) as cume_dist
	, cume_dist() over (order by sal desc)*12.0 as xxtile
from hr.emp a;

select * from nw.order_items;

select a.order_id 
	, rank() over(order by amount desc) as rank 
	, cume_dist() over (order by amount desc) as cume_dist
from nw.order_items a;


-- percent_rank는 rank를 0 ~ 1 사이 값으로 정규화 시킴. 
-- (파티션내의 rank() 값 - 1) / (파티션내의 로우 건수 - 1)
select a.empno, ename, job, sal
    , rank() over(order by sal desc) as rank 
	, percent_rank() over (order by sal desc) as percent_rank
	, 1.0 * (rank() over(order by sal desc) -1 )/11 as percent_rank_calc
from hr.emp a;

-- ntile은 지정된 숫자만큼의 분위를 정하여 그룹핑하는데 사용. 
select a.empno, ename, job, sal
	, ntile(5) over (order by sal desc) as ntile
from hr.emp a;


-- 상품 매출 순위 상위 10%의 상품 및 매출액
with 
temp_01 as ( 
	select product_id, sum(amount) as sum_amount
	from nw.orders a 
		join nw.order_items b 
			on a.order_id = b.order_id
	group by product_id
)
select * from (
	select a.product_id, b.product_name, a.sum_amount
		, cume_dist() over (order by sum_amount) as percentile_norm
		, 1.0 * row_number() over (order by sum_amount)/count(*) over () as rnum_norm
	from temp_01 a
		join nw.products b on a.product_id = b.product_id
) a where percentile_norm >= 0.9;

/************************************************
percentile_disc/percentile_cont 실습
 *************************************************/

-- 4분위별 sal 값을 반환. 
select percentile_disc(0.25) within group (order by sal) as qt_1
	, percentile_disc(0.5) within group (order by sal) as qt_2
	, percentile_disc(0.75) within group (order by sal) as qt_3
	, percentile_disc(1.0) within group (order by sal) as qt_4
from hr.emp;  

-- percentile_disc는 cume_dist의 inverse 값을 반환. 
-- percentile_disc는 0 ~ 1 사이의 분위수값을 입력하면 해당 분위수 값 이상인 것 중에서 최소 cume_dist 값을 가지는 값을 반환
with
temp_01 as 
(
	select percentile_disc(0.25) within group (order by sal) as qt_1
	, percentile_disc(0.5) within group (order by sal) as qt_2
	, percentile_disc(0.75) within group (order by sal) as qt_3
	, percentile_disc(1.0) within group (order by sal) as qt_4
from hr.emp
)
select a.empno, ename, sal
	, cume_dist() over (order by sal) as cume_dist
	, b.qt_1, b.qt_2, b.qt_3, b.qt_4
from hr.emp a
	cross join temp_01 b
order by sal;


-- products 테이블에서 category별 percentile_disc 구하기 
with
temp_01 as 
(
	select a.category_id, max(b.category_name) as category_name 
	, percentile_disc(0.25) within group (order by unit_price) as qt_1
	, percentile_disc(0.5) within group (order by unit_price) as qt_2
	, percentile_disc(0.75) within group (order by unit_price) as qt_3
	, percentile_disc(1.0) within group (order by unit_price) as qt_4
from nw.products a
	join nw.categories b on a.category_id = b.category_id
group by a.category_id
)
select * from temp_01;

-- percentile_disc와 cume_dist 비교하기 
with
temp_01 as 
(
	select a.category_id, max(b.category_name) as category_name 
	, percentile_disc(0.25) within group (order by unit_price) as qt_1
	, percentile_disc(0.5) within group (order by unit_price) as qt_2
	, percentile_disc(0.75) within group (order by unit_price) as qt_3
	, percentile_disc(1.0) within group (order by unit_price) as qt_4
from nw.products a
	join nw.categories b on a.category_id = b.category_id
group by a.category_id
)
select product_id, product_name, a.category_id, b.category_name
	, unit_price
	, cume_dist() over (partition by a.category_id order by unit_price) as cume_dist_by_cat
	, b.qt_1, b.qt_2, b.qt_3, b.qt_4
from nw.products a 
	join temp_01 b on a.category_id = b.category_id;



--입력 받은 분위수가 특정 로우를 정확하게 지정하지 못하고, 두 로우 사이일때 
--percentile_cont는 보간법을 이용하여 보정하며, percentile_cont는 두 로우에서 작은 값을 반환
select 'cont' as gubun 
	, percentile_cont(0.25) within group (order by sal) as qt_1
	, percentile_cont(0.5) within group (order by sal) as qt_2
	, percentile_cont(0.75) within group (order by sal) as qt_3
	, percentile_cont(1.0) within group (order by sal) as qt_4
from hr.emp
union all
select 'disc' as gubun 
	, percentile_disc(0.25) within group (order by sal) as qt_1
	, percentile_disc(0.5) within group (order by sal) as qt_2
	, percentile_disc(0.75) within group (order by sal) as qt_3
	, percentile_disc(1.0) within group (order by sal) as qt_4
from hr.emp;

-- percentile_cont와 percentile_disc를 cume_dist와 비교. 
with 
temp_01 as ( 
select 'cont' as gubun 
	, percentile_cont(0.25) within group (order by sal) as qt_1
	, percentile_cont(0.5) within group (order by sal) as qt_2
	, percentile_cont(0.75) within group (order by sal) as qt_3
	, percentile_cont(1.0) within group (order by sal) as qt_4
from hr.emp
union all
select 'disc' as gubun 
	, percentile_disc(0.25) within group (order by sal) as qt_1
	, percentile_disc(0.5) within group (order by sal) as qt_2
	, percentile_disc(0.75) within group (order by sal) as qt_3
	, percentile_disc(1.0) within group (order by sal) as qt_4
from hr.emp
)
select a.empno, ename, sal
	, cume_dist() over (order by sal)
	, b.qt_1, b.qt_2, b.qt_3, b.qt_4
from hr.emp a
	cross join temp_01 b
where b.gubun = 'disc'
;



-----------------------------------------------------------------------------------------------------------------------------------------------------

window

-----------------------------------------------------------------------------------------------------------------------------------------------------

/************************************************
강의 실습 - windows 절
 *************************************************/

/* rows between unbounded preceding and current row */
select *, sum(unit_price) over (order by unit_price) as unit_price_sum from products;

select *, sum(unit_price) over (order by unit_price rows between unbounded preceding and current row) as unit_price_sum from products;

select *, sum(unit_price) over (order by unit_price rows unbounded preceding) as unit_price_sum from products;

/* 중앙합, 중앙 평균(Centered average) */
select product_id, product_name, category_id, unit_price
	, sum(unit_price) over (partition by category_id order by unit_price rows between 1 preceding and 1 following) as unit_price_sum 
	, avg(unit_price) over (partition by category_id order by unit_price rows between 1 preceding and 1 following) as unit_price_avg 
from products;

/* rows between current row and unbounded following */
select product_id, product_name, category_id, unit_price
, sum(unit_price) over (partition by category_id order by unit_price rows between current row and unbounded following) as unit_price_sum 
from products;


/* range와 rows의 차이 */
with
temp_01 as (
select c.category_id, date_trunc('day', b.order_date) as ord_date, sum(a.amount) sum_by_daily_cat
from order_items a
	join orders b on a.order_id = b.order_id 
	join products c on a.product_id = c.product_id 
group by c.category_id, date_trunc('day', b.order_date) 
order by 1, 2
)
select *
	, sum(sum_by_daily_cat) over (partition by category_id order by ord_date 
	                              rows between 2 preceding and current row)
	, sum(sum_by_daily_cat) over (partition by category_id order by ord_date 
	                              range between interval '2' day preceding and current row)
from temp_01;

/************************************************
이동평균 실습
 *************************************************/

-- 3일 이동 평균 매출
with
temp_01 as (
select date_trunc('day', b.order_date)::date as ord_date, sum(amount) as daily_sum
from order_items a
	join orders b on a.order_id = b.order_id
group by date_trunc('day', b.order_date)::date 
)
select ord_date, daily_sum
	, avg(daily_sum) over (order by ord_date 
	                              rows between 2 preceding and current row) as ma_3days
from temp_01;

-- 3일 중앙 평균 매출
with
temp_01 as (
select date_trunc('day', b.order_date)::date as ord_date, sum(amount) as daily_sum
from order_items a
	join orders b on a.order_id = b.order_id
group by date_trunc('day', b.order_date)::date 
)
select ord_date, daily_sum
	, avg(daily_sum) over (order by ord_date 
	                              rows between 1 preceding and 1 following) as ca_3days
from temp_01;

-- N 이동 평균에서 맨 처음 N-1 개의 데이터의 경우 정확히 N이동 평균을 구할 수 없을 때 Null 처리 하기. 
with
temp_01 as (
select date_trunc('day', b.order_date)::date as ord_date, sum(amount) as daily_sum
from order_items a
	join orders b on a.order_id = b.order_id
group by date_trunc('day', b.order_date)::date 
)
select ord_date, daily_sum
	, avg(daily_sum) over (order by ord_date 
	                              rows between 2 preceding and current row) as ma_3days_01
	, case when  row_number() over (order by ord_date) <= 2 then null 
	             else avg(daily_sum) over (order by ord_date 
	                              rows between 2 preceding and current row) 
	             end as ma_3days_02
from temp_01;

-- 또는 아래와 같이 작성
with
temp_01 as (
select date_trunc('day', b.order_date)::date as ord_date, sum(amount) as daily_sum
from order_items a
	join orders b on a.order_id = b.order_id
group by date_trunc('day', b.order_date)::date 
), 
temp_02 as (
select ord_date, daily_sum
	, avg(daily_sum) over (order by ord_date 
	                              rows between 2 preceding and current row) as ma_3days_01
	, row_number() over (order by ord_date) as rn
from temp_01
)
select ord_date, daily_sum
	, ma_3days_01
	, case when rn <= 2 then null 
		   else ma_3days_01 end as ma_3days_02
from temp_02;

-- 연속된 매출 일자에서 매출이 Null일때와 그렇지 않을 때의 Aggregate Analytic 결과 차이. 
with ref_days
as (
	select generate_series('1996-07-04'::date , '1996-07-23'::date, '1 day'::interval)::date as ord_date
), 
temp_01 as (
	select date_trunc('day', b.order_date)::date as ord_date, sum(amount) as daily_sum
	from order_items a
		join orders b on a.order_id = b.order_id
	group by date_trunc('day', b.order_date)::date 
),
temp_02 as (
	select a.ord_date, b.daily_sum as daily_sum
	from ref_days a
		left join temp_01 b on a.ord_date = b.ord_date
)
select ord_date, daily_sum
	, avg(daily_sum) over (order by ord_date rows between 2 preceding and current row) as ma_3days
from temp_02;

/************************************************
range와 rows 적용 시 유의 사항
 *************************************************/
-- range와 rows의 차이: order by 시 동일 row 처리 차이 - 1
select empno, deptno, sal
	, avg(sal) over (partition by deptno order by sal) as avg_default
	, avg(sal) over (partition by deptno order by sal range between unbounded preceding and current row) as avg_range
	, avg(sal) over (partition by deptno order by sal rows between unbounded preceding and current row) as avg_rows
	, sum(sal) over (partition by deptno order by sal) as sum_default
	, sum(sal) over (partition by deptno order by sal rows between unbounded preceding and current row) as sum_rows
from hr.emp;

-- range와 rows의 차이: order by 시 동일 row 처리 차이 - 2
select empno, deptno, sal, date_trunc('month', hiredate)::date as hiremonth
	, avg(sal) over (partition by deptno order by date_trunc('month', hiredate)) as avg_default
	, avg(sal) over (partition by deptno order by date_trunc('month', hiredate) range between unbounded preceding and current row) as avg_range
	, avg(sal) over (partition by deptno order by date_trunc('month', hiredate) rows between unbounded preceding and current row) as avg_rows
	, sum(sal) over (partition by deptno order by date_trunc('month', hiredate)) as sum_default
	, sum(sal) over (partition by deptno order by date_trunc('month', hiredate) rows between unbounded preceding and current row) as sum_rows
from hr.emp;





-----------------------------------------------------------------------------------------------------------------------------------------------------

서브쿼리

-----------------------------------------------------------------------------------------------------------------------------------------------------

/************************************************
                  서브쿼리 유형 기본
 *************************************************/

-- 평균 급여 이상의 급여를 받는 직원
select * from hr.emp where sal >= (select avg(sal) from hr.emp);

-- 가장 최근 급여 정보
select * from hr.emp_salary_hist a where todate = (select max(todate) from hr.emp_salary_hist x where a.empno = x.empno);


-- 스칼라 서브쿼리
select ename, deptno, 
	(select dname from hr.dept x where x.deptno=a.deptno) as dname
from hr.emp a;

-- 인라인뷰 서브쿼리
select a.deptno, b.dname, a.sum_sal
from
(
	select deptno, sum(sal) as sum_sal 
	from hr.emp 
	group by deptno
) a 
join hr.dept b 
on a.deptno = b.deptno;

/************************************************
                 where 절 서브쿼리 이해
 *************************************************/
-- ok
select a.* from hr.dept a where a.deptno in (select deptno from hr.emp x where x.sal > 1000);

-- 수행 안됨. 
select a.*, x.ename from hr.dept a where a.deptno in (select deptno from hr.emp x where x.sal > 1000 );

--ok
select a.* from hr.dept a where exists (select deptno from hr.emp x where x.deptno=a.deptno and x.sal > 1000)

-- 서브쿼리의 반환값은 무조건 중복이 제거된 unique한 값 - 비상관 서브쿼리
select * from nw.orders a where order_id in (select order_id from nw.order_items where amount > 100);

-- 서브쿼리의 반환값은 메이쿼리의 개별 레코드로 연결된 결과값에서 무조건 중복이 제거된 unique한 값 - 상관 서브쿼리
select * from nw.orders a where exists (select order_id from nw.order_items x where a.order_id = x.order_id and x.amount > 100);


/************************************************
              비상관(non-correlated) 서브쿼리
 *************************************************/

-- in 연산자는 괄호내에 한개 이상의 값을 상수값 또는 서브쿼리 결과 값으로 가질 수 있으며 개별값의 = 조건들의 or 연산을 수행
select * from hr.emp where deptno in (20, 30);

select * from hr.emp where deptno = 20 or deptno=30;

-- 여러개의 중복된 값을 괄호 내에 가질 경우 중복을 제거하고 unique한 값을 가짐. 
select * from hr.dept where deptno in (select deptno from hr.emp where sal < 1300);

-- 단일 컬럼 뿐 아니라 여러컬럼을 가질 수 있음. 
select * from hr.dept where (deptno, loc) in (select deptno, 'DALLAS' from hr.emp where sal < 1300);

-- 고객이 가장 최근에 주문한 주문 정보 추출
select * from nw.orders where (customer_id, order_date) in (select customer_id, max(order_date) 
from nw.orders group by customer_id);

-- 메인쿼리-서브쿼리의 연결 연산자가 단순 비교 연산자일 경우 서브쿼리는 단 한개의 값을 반환해야 함. 
select * from hr.emp where sal <= (select avg(sal) from hr.emp);

-- 메인쿼리-서브쿼리의 연결 연산자가 = 인데 서브쿼리의 반환값이 여러개이므로 수행 안됨
select * from hr.dept where deptno = (select deptno from hr.emp where sal < 1300);

-- 단순 비교 연산자로 서브쿼리를 연결하여도 여러 컬럼 조건을 가질 수 있음.
select * from nw.orders where (customer_id, order_date) = (select customer_id, max(order_date)
from nw.orders where customer_id='VINET' group by customer_id);



/************************************************
              상관(correlated) 서브쿼리
 *************************************************/

-- 상관 서브쿼리, 주문에서 상품 금액이 100보다 큰 주문을 한 주문 정보 
select * from nw.orders a 
where exists (select order_id from nw.order_items x where a.order_id = x.order_id and x.amount > 100);

-- 비상관 서브쿼리, 상품 금액이 100보다 큰 주문을 한 주문 정보
select * from nw.orders a where order_id in (select order_id from nw.order_items where amount > 100);


-- 2건 이상 주문을 한 고객 정보
select * from nw.customers a 
where exists (select 1 from nw.orders x where x.customer_id = a.customer_id 
              group by customer_id having count(*) >=2);


-- 1997년 이후에 한건이라도 주문을 한 고객 정보
select * from nw.customers a 
where exists (select 1 from nw.orders x where x.customer_id = a.customer_id
                                        and x.order_date >= to_date('19970101', 'yyyymmdd'));

--1997년 이후에 단 한건도 주문하지 않은 고객 정보
select * from nw.customers a 
where not exists (select 1 from nw.orders x where x.customer_id = a.customer_id
                                        and x.order_date >= to_date('19970101', 'yyyymmdd'));
-- 조인으로 변환
select * 
from nw.customers a
	left join (select customer_id from nw.orders 
	      where order_date >= to_date('19970101', 'yyyymmdd') 
	      group by customer_id
	      ) b on a.customer_id = b.customer_id 
where b.customer_id is null;


-- 직원의 급여이력에서 가장 최근의 급여이력
select * from hr.emp_salary_hist a where todate = (select max(todate) from hr.emp_salary_hist x 
						   where x.empno = a.empno);

-- 아래는 메인쿼리의 개별 레코드 별로 empno 연결조건으로 단 한건이 아닌 여러건을 반환하므로 수행 오류
select * from hr.emp_salary_hist a where todate = (select todate from hr.emp_salary_hist x
						   where x.empno = a.empno);



/************************************************
         서브쿼리 실습 - 가장 높은 sal을 받는 직원 정보
 *************************************************/

-- 가장 높은 sal을 받는 직원정보
select * from hr.emp where sal = (select max(sal) from hr.emp);

-- 조인
select a.* 
from hr.emp a
	join (select max(sal) sal from hr.emp) b
on a.sal = b.sal;

-- Analytic SQL
select * from
(
select *,
	row_number() over (order by sal desc) as rnum
from hr.emp
) a where rnum = 1;


/************************************************
     서브쿼리 실습 - 부서별로 가장 높은 sal을 받는 직원 정보
 *************************************************/
-- 부서별로 가장 높은 sal을 받는 직원 상세 정보. 비상관 서브쿼리
select * from hr.emp where (deptno, sal) in (select deptno, max(sal) as sal from hr.emp group by deptno) order by empno;

-- 상관서브쿼리 
select * from hr.emp a
where sal = (select max(sal) as sal from hr.emp x 
             where x.deptno = a.deptno);

-- Analytic SQL
select * from
(
select *,
	row_number() over (partition by deptno order by sal desc) as rnum
from hr.emp
) a where rnum = 1 
order by empno;


/************************************************
     서브쿼리 실습 - 직원의 가장 최근 부서 근무이력 조회
 *************************************************/

drop table if exists hr.emp_dept_hist_01;

-- todate가 99991231가 아닌 경우를 한개 레코드로 생성하기 위해 임시 테이블 생성
create table hr.emp_dept_hist_01
as
select * from hr.emp_dept_hist;


update hr.emp_dept_hist_01
set todate=to_date('1983-12-24', 'yyyy-mm-dd') 
where empno = 7934 and todate=to_date('99991231', 'yyyymmdd');

select * from hr.emp_dept_hist_01;

-- 직원의 가장 최근 부서 근무이력 조회. 비상관 서브쿼리
select * from hr.emp_dept_hist_01 a where (empno, todate) in (select empno, max(todate) from hr.emp_dept_hist_01 x
group by empno);

-- 상관 서브쿼리
select * from hr.emp_dept_hist_01 a where todate = (select max(todate) from hr.emp_dept_hist_01 x where x.empno=a.empno);

-- Analytic SQL
select *
from (
select * 
	, row_number() over (partition by empno order by todate desc) as rnum
from hr.emp_dept_hist_01
)a where rnum = 1;


/************************************************
 서브쿼리 실습 - 고객의 첫번째 주문일의 주문정보와 고객 정보를 함께 추출
 *************************************************/

-- 고객의 첫번째 주문일의 order_id, order_date, shipped_date와 함께 고객명(contact_name), 고객거주도시(city) 정보를 함께 추출
select a.order_id, a.order_date, a.shipped_date, b.contact_name, b.city 
from nw.orders a
	join nw.customers b on a.customer_id = b.customer_id
where a.order_date = (select min(order_date) from nw.orders x where x.customer_id = a.customer_id);

-- Analytic SQL
select order_id, order_date, shipped_date, contact_name, city
from
(
select a.order_id, a.order_date, a.shipped_date, b.contact_name, b.city,
	row_number() over (partition by a.customer_id order by a.order_date) as rnum
from nw.orders a
	join nw.customers b on a.customer_id = b.customer_id
)a where rnum = 1;



/************************************************
 서브쿼리 실습 - 고객별 주문 상품 평균 금액보다 더 큰 금액의 주문 상품명, 주문번호, 주문 상품금액을 구하되 고객명과 고객도시명을 함께 추출
 *************************************************/

-- 고객별 주문상품 평균 금액 
select a.customer_id, avg(b.amount) avg_amount from nw.orders a
join nw.order_items b
on a.order_id = b.order_id
group by customer_id;

-- 상관 서브쿼리로 구하기
select a.customer_id, a.contact_name, a.city, b.order_id, c.product_id, c.amount, d.product_name
from nw.customers a
	join nw.orders b on a.customer_id = b.customer_id
	join nw.order_items c on b.order_id = c.order_id
	join nw.products d on c.product_id = d.product_id
where c.amount >= (select avg(y.amount) avg_amount 
					from nw.orders x
						join nw.order_items y on x.order_id = y.order_id
					where x.customer_id =a.customer_id
					group by x.customer_id
					)
order by a.customer_id, amount;
				
-- Analytic SQL로 구하기 				
select customer_id, contact_name, city, order_id, product_id, amount, product_name
from (
	select a.customer_id, a.contact_name, a.city, b.order_id, c.product_id, c.amount, d.product_name
	, avg(amount) over (partition by a.customer_id rows between unbounded preceding and unbounded following) as avg_amount
	from nw.customers a
		join nw.orders b on a.customer_id = b.customer_id
		join nw.order_items c on b.order_id = c.order_id
		join nw.products d on c.product_id = d.product_id
) a 
where a.amount >= a.avg_amount
order by customer_id, amount;

/************************************************
 Null값이 있는 컬럼의 not in과 not exists 차이 실습
 *************************************************/

select * from hr.emp where deptno in (20, 30, null);

select * from hr.emp where deptno = 20 or deptno=30 or deptno = null;


-- 테스트를 위한 임의의 테이블 생성. 
drop table if exists nw.region;

create table nw.region
as
select ship_region as region_name from nw.orders 
group by ship_region 
;

-- 새로운 XX값을 region테이블에 입력. 
insert into nw.region values('XX');

commit;

select * from nw.region;

-- null값이 포함된 컬럼을 서브쿼리로 연결할 시 in과 exists는 모두 동일. 
select a.*
from nw.region a 
where a.region_name in (select ship_region from nw.orders x);

select a.*
from nw.region a 
where exists (select ship_region from nw.orders x where x.ship_region = a.region_name
             );

-- null값이 포함된 컬럼을 서브쿼리로 연결 시 not in과 not exists의 결과는 서로 다름. 
select a.*
from nw.region a 
where a.region_name not in (select ship_region from nw.orders x);

select a.*
from nw.region a 
where not exists (select ship_region from nw.orders x where x.ship_region = a.region_name
                 );
;

-- true
select 1=1;

-- false
select 1=2;

-- null
select null = null;

-- null
select 1=1 and null;

-- null
select 1=1 and (null = null);

-- true
select 1=1 or null;

-- false
select not 1=1;

-- null
select not null;

-- not in을 사용할 경우 null인 값은 서브쿼리내에서 is not null로 미리 제거해야 함. 
select a.*
from nw.region a 
where a.region_name not in (select ship_region from nw.orders x where x.ship_region is not null);

-- not exists의 경우 null 값을 제외하려면 서브쿼리가 아닌 메인쿼리 영역에서 제외
select a.*
from nw.region a 
where not exists (select ship_region from nw.orders x where x.ship_region = a.region_name --and a.region_name is not null
                 );
--and a.region_name is not null


/************************************************
               스칼라 서브쿼리 이해 
 *************************************************/

-- 직원의 부서명을 스칼라 서브쿼리로 추출
select a.*,
	(select dname from hr.dept x where x.deptno=a.deptno) as dname
from hr.emp a;

-- 아래는 수행 오류 발생. 스칼라 서브쿼리는 단 한개의 결과 값만 반환해야 함. 
select a.*
	, (select ename from hr.emp x where x.deptno=a.deptno) as ename
from hr.dept a;

-- 아래는 수행 오류 발생. 스칼라 서브쿼리는 단 한개의 열값만 반환해야 함. 
select a.*,
	(select dname, deptno from hr.dept x where x.deptno=a.deptno) as dname
from hr.emp a;


-- case when 절에서 스칼라 서브쿼리 사용
select a.*,
	(case when a.deptno = 10 then (select dname from hr.dept x where x.deptno=20)
		  else (select dname from hr.dept x where x.deptno=a.deptno)
		  end
	) as dname
from hr.emp a;

-- 스칼라 서브쿼리는 일반 select와 동일하게 사용. group by 적용 무방. 
select a.*,
	(select avg(sal) from hr.emp x where x.deptno = a.deptno) dept_avg_sal
from hr.emp a;

-- 조인으로 변경. 
select a.*, b.avg_sal 
from hr.emp a
	join (select deptno, avg(sal) as avg_sal from hr.emp x group by deptno) b 
	on a.deptno = b.deptno;

/************************************************
               스칼라 서브쿼리 실습 
 *************************************************/

-- 직원 정보와 해당 직원을 관리하는 매니저의 이름 추출
select a.*,
	(select ename from hr.emp x where x.empno=a.mgr) as mgr_name
from hr.emp a;

select a.*, b.ename as mgr_name
from hr.emp a
	left join hr.emp b on a.mgr=b.empno;

-- 주문정보와 ship_country가 France이면 주문 고객명을, 아니면 직원명을 new_name으로 출력 
select a.order_id, a.customer_id, a.employee_id, a.order_date, a.ship_country
	, (select contact_name from nw.customers x where x.customer_id = a.customer_id) as customer_name
	, (select first_name||' '||last_name from nw.employees x where x.employee_id = a.employee_id) as employee_name
	, case when a.ship_country = 'France' then 
	            (select contact_name from nw.customers x where x.customer_id = a.customer_id)
	       else (select first_name||' '||last_name from nw.employees x where x.employee_id = a.employee_id)
	  end as new_name
from nw.orders a;

-- 조인으로 변경. 
select a.order_id, a.customer_id, a.employee_id, a.order_date, a.ship_country
	, b.contact_name, c.first_name||' '||c.last_name
	, case when a.ship_country = 'France' then b.contact_name
		   else c.first_name||' '||c.last_name end as new_name
from nw.orders a
	left join nw.customers b on a.customer_id = b.customer_id
	left join nw.employees c on a.employee_id = c.employee_id
;

-- 고객정보와 고객이 처음 주문한 일자의 주문 일자 추출.
select a.customer_id, a.contact_name
	, (select min(order_date) from nw.orders x where x.customer_id = a.customer_id) as first_order_date
from nw.customers a;

-- 조인으로 변경 
select a.customer_id, a.contact_name, b.last_order_date
from nw.customers a
	left join (select customer_id, min(order_date) as first_order_date from nw.orders x group by customer_id) b
	on a.customer_id = b.customer_id;

-- 고객정보와 고객이 처음 주문한 일자의 주문 일자와 그때의 배송 주소, 배송 일자 추출
select a.customer_id, a.contact_name
	, (select min(order_date) from nw.orders x where x.customer_id = a.customer_id) as first_order_date
	, (select x.ship_address from nw.orders x where x.customer_id=a.customer_id and x.order_date = 
	          (select min(order_date) from nw.orders y where y.customer_id = x.customer_id)
	  ) as first_ship_address
	, (select x.shipped_date from nw.orders x where x.customer_id=a.customer_id and x.order_date = 
	          (select min(order_date) from nw.orders y where y.customer_id = x.customer_id)
      ) as first_shipped_date
from nw.customers a
order by a.customer_id;

-- 조인으로 변경.
select a.customer_id, a.contact_name
	, b.order_date, b.ship_address, b.shipped_date
from nw.customers a
	left join nw.orders b on a.customer_id = b.customer_id
	and b.order_date = (select min(order_date) from nw.orders x 
						  where a.customer_id = x.customer_id
						  )
order by a.customer_id;


-- 고객정보와 고객이 마지막 주문한 일자의 주문 일자와 그때의 배송 주소, 배송 일자 추출
-- 현재 데이터가 고객이 하루에 주문을 두번한 경우가 있음. max(order_date) 시 고객이 하루에 주문을 두번한 일자가 나오고 있음
-- 때문에 반드시 1개의 값만 스칼라 서브쿼리에서 반환하도록 limit 1 추가
select a.customer_id, a.contact_name
	, (select max(order_date) from nw.orders x where x.customer_id = a.customer_id) as last_order_date
	, (select x.ship_address from nw.orders x where x.customer_id=a.customer_id and x.order_date = 
	          (select max(order_date) from nw.orders y where y.customer_id = x.customer_id)
	          limit 1
	  ) as last_ship_address
	, (select x.shipped_date from nw.orders x where x.customer_id=a.customer_id and x.order_date = 
	          (select max(order_date) from nw.orders y where y.customer_id = x.customer_id)
	          limit 1) as last_shipped_date
from nw.customers a
order by a.customer_id;

-- 조인으로 변경
select a.customer_id, a.contact_name
	, b.order_date, b.ship_address, b.shipped_date
	--, row_number() over (partition by a.customer_id order by b.order_date desc) as rnum
from nw.customers a
	left join nw.orders b on a.customer_id = b.customer_id
	and b.order_date = (select max(order_date) from nw.orders x 
						  where a.customer_id = x.customer_id
					   )
	--where a.customer_id = 'ALFKI'
	--limit 1
;





~~~