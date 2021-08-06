[서브쿼리]



1. 개요

   - 쿼리 안에 포함된 쿼리를 의미

   > where절에 조건을 추가해준다. 비교할 수 있는 값을 하나 더 추가할 수 있다.

   - select 문에 삽입된 select문을 의미

     ```shell
     select *
     from table_name
     where column_name = (subquery)
     ```

   - 구문

     ```shell
     select column_name
     from table_name
     where column_name 연산자 (select column_name
     						 from table_name
     						 where ___  )
     ```

   - subquery는 ()를 삽입하여 작업하기

   - 서브쿼리도 일반 쿼리 처럼 group by, join 모두 사용 가능

   - 서브쿼리는 메인쿼리(혹은 기본쿼리) 실행 전에 한 번만 실행된다.

   - where절에 서브쿼리를 삽입하는 경우 비교조건의 오른쪽에 쿼리를 삽입

   - 보통 서브쿼리의 결과가 행 하나 컬럼 하나인 경우가 대부분이다.

   - select, from, where, having, group by절에 서브쿼리를 사용할 수 있다.

   - insert문, delete문, update문, select문 모두 서브쿼리를 사용할 수 있다.

[실습1] 10번부서의 평균 급여보다 급여가 많은 사원의 목록을 조회

```
select ename
from emp
where sal > (select avg(sal)
                   from emp
                   where deptno = 10);
```

[실습2] SMITH와 같은 부서에 근무하는 사원의 목록을 조회

```
select ename
from emp
where deptno = (select deptno
                   from emp
                   where ename = 'SMITH');
```

[실습3] 사원번호 7566인 사원보다 급여를 많이 받는 사원의 이름과 급여를 조회

```
select ename, sal
from emp
where sal > (select sal
                   from emp
                   where empno = 7566);
```

[단행 서브쿼리 실습, scott 계정]

1. 이름에 'T'를 포함하고 있는 사원과 같은 부서에서 근무하는 사원의 이름과 사원번호 출력

   ```
   select ename, empno
   from emp
   where deptno = (select deptno
                      from emp
                      where ename like '%T%');
                      
   where deptno = (select deptno
                   *
   ERROR at line 3:
   ORA-01427: single-row subquery returns more than one row
   
   SQL> (select deptno
     2      from emp
     3      where ename like '%T%');
   
       DEPTNO
   ----------
           20
           30
           20
           30
           
   <possible answer>
   select ename, empno
   from emp
   where deptno in (select deptno
   from emp
   where ename like '%T%');
   ```

   

2. 20번 부서의 최고 급여보다 급여가 많은 사원들의 사원명, 부서코드, 급여 목록 출력하기

   ```
   select ename, deptno, sal
   from emp
   where sal > (select max(sal)
                      from emp
                      where deptno = 20);
   ```

   

3. 1982년에 입사한 직원의 평균 급여보다 급여가 높은 사원들의 사원명, 입사일, 급여의 목록을  출력하기

   ```
   select ename, hiredate, sal
   from emp
   where sal > (select avg(sal)
                      from emp
                      where to_char(hiredate, 'YYYY') = 1982);
   ```



2. 서브쿼리의 종류

    1. 단일행 서브쿼리

        - 서브쿼리의 결과가 1행 1열인 쿼리

   	2) 다중행 서브쿼리

       * 서브쿼리의 결과가 다중 행인 경우

       * 비교연산자를 사용할 수 없다.(비교 연산자도 any나 all과 함께 사용은 가능)

       * in, any, all

         > in : 괄호안의 값과 일치하는 결과가 반영
         >
         > any 
         >
         > >< any : 어떤거 보다도 작다 = 최대값보다 작은 값을 조회
         > >
         > >\> any : 어떤거 보다도 크다 = 최소값보다 큰 값을 조회
         >
         > all : 
         >
         > > < all - 최소값보다 작은 값을 조회
         > >
         > > \> all - 최대값보다 큰 값을 조회

   3. 다중 컬럼 서브쿼리

      두 개 이상의 컬럼을 리턴하는 서브쿼리

      where (컬럼1, 컬럼2...) in (비교할값1, 비교할값2...)

​       [실습] 각 부서에서 최소 급여를 받는 사용자의 사원번호, 부서번호, 급여를 출력

```
select empno, deptno, sal
from emp
where (deptno, sal) in (select deptno, min(sal)
					from emp
					group by deptno);
```



4. 상호 연관 서브쿼리

   > 서브쿼리를 실행할때 메인쿼리 테이블의컬럼을 참조해서 작업해야 하는 서브쿼리

   * 메인쿼리의 비교 값이 계속 바뀌는 경우 메인의 값을 참조해서 사용
     * 각각의 행에 입력된 값에 따라 결과가 달라지는 경우!
   * 메인 쿼리의 테이블에 alias를 추가하고 참조해서 사용
   * 메인쿼리의 한 row에 대해서 서브쿼리가 한번씩 실행.
   * 서브쿼리에서는 메인쿼리의 컬럼을 사용할 수 있으나 메인 쿼리에서는 서브쿼리의 컬럼을 사용할 수 없다.

   [실행흐름]

   (1) 메인쿼리에서 참조할 값을 가져온다.

   (2) 메인쿼리에서 참조할 값을 이용해 서브쿼리를 실행

   (3) 서브쿼리의 결과를 이용해서 메인쿼리를 실행

   (4) 1번부터 3번까지를 전체 레코드에 반복해서 실행

   

   [실습] 급여가 본인 부서의 평균 급여보다 많은 사원의 사원번호, 사원명, 부서번호, sal

   ```
   select empno, ename, deptno, sal
   from emp e1
   where sal > (select avg(sal)
   				from  emp e2
   				where e1.deptno = e2.deptno);
   ```

   

   