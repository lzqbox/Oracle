# Oracle实验一分析实验代码文档
### 查询1：
#### 实验源代码：
```
     SELECT d.department_name，count(e.job_id)as "部门总人数"，
     avg(e.salary)as "平均工资"
     from hr.departments d，hr.employees e
     where d.department_id = e.department_id
     and d.department_name in ('IT'，'Sales')
     GROUP BY department_name;
```
#### 查询1图片
![查询1结果](/1.png)
#### 查询1指导和优化
![查询2结果](./photo/2.png)
#### 查询1详细查询
```
GENERAL INFORMATION SECTION
-------------------------------------------------------------------------------
Tuning Task Name   : staName18194
Tuning Task Owner  : HR
Tuning Task ID     : 269
Workload Type      : Single SQL Statement
Execution Count    : 1
Current Execution  : EXEC_951
Execution Type     : TUNE SQL
Scope              : COMPREHENSIVE
Time Limit(seconds): 1800
Completion Status  : COMPLETED
Started at         : 10/09/2018 11:15:08
Completed at       : 10/09/2018 11:15:08

-------------------------------------------------------------------------------
Schema Name   : HR
Container Name: PDBORCL
SQL ID        : 69cx0cq340p0k
SQL Text      : SELECT d.department_name，count(e.job_id)as "部门总人数"，
                avg(e.salary)as "平均工资"
                from hr.departments d，hr.employees e
                where d.department_id = e.department_id
                and d.department_name in ('IT'，'Sales')
                GROUP BY department_name

-------------------------------------------------------------------------------
FINDINGS SECTION (1 finding)
-------------------------------------------------------------------------------

1- Index Finding (see explain plans section below)
--------------------------------------------------
  通过创建一个或多个索引可以改进此语句的执行计划。

  Recommendation (estimated benefit: 59.99%)
  ------------------------------------------
  - 考虑运行可以改进物理方案设计的访问指导或者创建推荐的索引。
    create index HR.IDX$$_010D0001 on HR.DEPARTMENTS("DEPARTMENT_NAME","DEPARTM
    ENT_ID");

  Rationale
  ---------
    创建推荐的索引可以显著地改进此语句的执行计划。但是, 使用典型的 SQL 工作量运行 "访问指导"
    可能比单个语句更可取。通过这种方法可以获得全面的索引建议案, 包括计算索引维护的开销和附加的空间消耗。

-------------------------------------------------------------------------------
EXPLAIN PLANS SECTION
-------------------------------------------------------------------------------

1- Original
-----------
Plan hash value: 3808327043

 
---------------------------------------------------------------------------------------------------
| Id  | Operation                     | Name              | Rows  | Bytes | Cost (%CPU)| Time     |
---------------------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT              |                   |     1 |    23 |     5  (20)| 00:00:01 |
|   1 |  HASH GROUP BY                |                   |     1 |    23 |     5  (20)| 00:00:01 |
|   2 |   NESTED LOOPS                |                   |    19 |   437 |     4   (0)| 00:00:01 |
|   3 |    NESTED LOOPS               |                   |    20 |   437 |     4   (0)| 00:00:01 |
|*  4 |     TABLE ACCESS FULL         | DEPARTMENTS       |     2 |    32 |     3   (0)| 00:00:01 |
|*  5 |     INDEX RANGE SCAN          | EMP_DEPARTMENT_IX |    10 |       |     0   (0)| 00:00:01 |
|   6 |    TABLE ACCESS BY INDEX ROWID| EMPLOYEES         |    10 |    70 |     1   (0)| 00:00:01 |
---------------------------------------------------------------------------------------------------
 
Query Block Name / Object Alias (identified by operation id):
-------------------------------------------------------------
 
   1 - SEL$1
   4 - SEL$1 / D@SEL$1
   5 - SEL$1 / E@SEL$1
   6 - SEL$1 / E@SEL$1
 
Predicate Information (identified by operation id):
---------------------------------------------------
 
   4 - filter("D"."DEPARTMENT_NAME"='IT' OR "D"."DEPARTMENT_NAME"='Sales')
   5 - access("D"."DEPARTMENT_ID"="E"."DEPARTMENT_ID")
 
Column Projection Information (identified by operation id):
-----------------------------------------------------------
 
   1 - (#keys=1) "DEPARTMENT_NAME"[VARCHAR2,30], COUNT("E"."SALARY")[22], COUNT(*)[22], 
       SUM("E"."SALARY")[22]
   2 - (#keys=0) "D"."DEPARTMENT_ID"[NUMBER,22], "DEPARTMENT_NAME"[VARCHAR2,30], 
       "E"."SALARY"[NUMBER,22]
   3 - (#keys=0) "D"."DEPARTMENT_ID"[NUMBER,22], "DEPARTMENT_NAME"[VARCHAR2,30], 
       "E".ROWID[ROWID,10]
   4 - "D"."DEPARTMENT_ID"[NUMBER,22], "DEPARTMENT_NAME"[VARCHAR2,30]
   5 - "E".ROWID[ROWID,10]
   6 - "E"."SALARY"[NUMBER,22]
 
Note
-----
   - this is an adaptive plan

2- Using New Indices
--------------------
Plan hash value: 778337127

 
---------------------------------------------------------------------------------------------------
| Id  | Operation                     | Name              | Rows  | Bytes | Cost (%CPU)| Time     |
---------------------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT              |                   |     2 |    46 |     2   (0)| 00:00:01 |
|   1 |  SORT GROUP BY NOSORT         |                   |     2 |    46 |     2   (0)| 00:00:01 |
|   2 |   NESTED LOOPS                |                   |    19 |   437 |     2   (0)| 00:00:01 |
|   3 |    NESTED LOOPS               |                   |    20 |   437 |     2   (0)| 00:00:01 |
|   4 |     INLIST ITERATOR           |                   |       |       |            |          |
|*  5 |      INDEX RANGE SCAN         | IDX$$_010D0001    |     2 |    32 |     1   (0)| 00:00:01 |
|*  6 |     INDEX RANGE SCAN          | EMP_DEPARTMENT_IX |    10 |       |     0   (0)| 00:00:01 |
|   7 |    TABLE ACCESS BY INDEX ROWID| EMPLOYEES         |    10 |    70 |     1   (0)| 00:00:01 |
---------------------------------------------------------------------------------------------------
 
Query Block Name / Object Alias (identified by operation id):
-------------------------------------------------------------
 
   1 - SEL$1
   5 - SEL$1 / D@SEL$1
   6 - SEL$1 / E@SEL$1
   7 - SEL$1 / E@SEL$1
 
Predicate Information (identified by operation id):
---------------------------------------------------
 
   5 - access("D"."DEPARTMENT_NAME"='IT' OR "D"."DEPARTMENT_NAME"='Sales')
   6 - access("D"."DEPARTMENT_ID"="E"."DEPARTMENT_ID")
 
Column Projection Information (identified by operation id):
-----------------------------------------------------------
 
   1 - (#keys=1) "DEPARTMENT_NAME"[VARCHAR2,30], COUNT("E"."SALARY")[22], COUNT(*)[22], 
       SUM("E"."SALARY")[22]
   2 - (#keys=0) "DEPARTMENT_NAME"[VARCHAR2,30], "E"."SALARY"[NUMBER,22]
   3 - (#keys=0) "DEPARTMENT_NAME"[VARCHAR2,30], "E".ROWID[ROWID,10]
   4 - "D"."DEPARTMENT_ID"[NUMBER,22], "DEPARTMENT_NAME"[VARCHAR2,30]
   5 - "D"."DEPARTMENT_ID"[NUMBER,22], "DEPARTMENT_NAME"[VARCHAR2,30]
   6 - "E".ROWID[ROWID,10]
   7 - "E"."SALARY"[NUMBER,22]

-------------------------------------------------------------------------------
```
#### 查询1结果分析
通过创建一个或多个索引可以改进此语句的执行计划。考虑运行可以改进物理方案设计的访问指导或者创建推荐的索引。创建推荐的索引可以显著地改进此语句的执行计划。但是, 使用典型的 SQL 工作量运行 "访问指导"。可能比单个语句更可取。通过这种方法可以获得全面的索引建议案, 包括计算索引维护的开销和附加的空间消耗。
缺失关键字。

### 查询2：
#### 实验源代码：
```
    SELECT d.department_name，count(e.job_id)as "部门总人数"，
    avg(e.salary)as "平均工资"
    FROM hr.departments d，hr.employees e
    WHERE d.department_id = e.department_id
    GROUP BY department_name
    HAVING d.department_name in ('IT'，'Sales');
    
```
#### 查询2图片
![查询2结果](./photo/3.png)
#### 查询2指导和优化
![查询2结果](./photo/4.png)
#### 查询2详细查询
```
GENERAL INFORMATION SECTION
-------------------------------------------------------------------------------
Tuning Task Name   : staName20279
Tuning Task Owner  : HR
Tuning Task ID     : 271
Workload Type      : Single SQL Statement
Execution Count    : 1
Current Execution  : EXEC_953
Execution Type     : TUNE SQL
Scope              : COMPREHENSIVE
Time Limit(seconds): 1800
Completion Status  : COMPLETED
Started at         : 10/09/2018 11:17:01
Completed at       : 10/09/2018 11:17:01

-------------------------------------------------------------------------------
Schema Name   : HR
Container Name: PDBORCL
SQL ID        : 4q3mr4gt7rkph
SQL Text      : SELECT d.department_name，count(e.job_id)as "部门总人数"，
                avg(e.salary)as "平均工资"
                FROM hr.departments d，hr.employees e
                WHERE d.department_id = e.department_id
                GROUP BY department_name
                HAVING d.department_name in ('IT'，'Sales')

-------------------------------------------------------------------------------
There are no recommendations to improve the statement.

-------------------------------------------------------------------------------
EXPLAIN PLANS SECTION
-------------------------------------------------------------------------------

1- Original
-----------
Plan hash value: 2128232041

 
----------------------------------------------------------------------------------------------
| Id  | Operation                      | Name        | Rows  | Bytes | Cost (%CPU)| Time     |
----------------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT               |             |     1 |    23 |     7  (29)| 00:00:01 |
|*  1 |  FILTER                        |             |       |       |            |          |
|   2 |   HASH GROUP BY                |             |     1 |    23 |     7  (29)| 00:00:01 |
|   3 |    MERGE JOIN                  |             |   106 |  2438 |     6  (17)| 00:00:01 |
|   4 |     TABLE ACCESS BY INDEX ROWID| DEPARTMENTS |    27 |   432 |     2   (0)| 00:00:01 |
|   5 |      INDEX FULL SCAN           | DEPT_ID_PK  |    27 |       |     1   (0)| 00:00:01 |
|*  6 |     SORT JOIN                  |             |   107 |   749 |     4  (25)| 00:00:01 |
|   7 |      TABLE ACCESS FULL         | EMPLOYEES   |   107 |   749 |     3   (0)| 00:00:01 |
----------------------------------------------------------------------------------------------
 
Query Block Name / Object Alias (identified by operation id):
-------------------------------------------------------------
 
   1 - SEL$1
   4 - SEL$1 / D@SEL$1
   5 - SEL$1 / D@SEL$1
   7 - SEL$1 / E@SEL$1
 
Predicate Information (identified by operation id):
---------------------------------------------------
 
   1 - filter("DEPARTMENT_NAME"='IT' OR "DEPARTMENT_NAME"='Sales')
   6 - access("D"."DEPARTMENT_ID"="E"."DEPARTMENT_ID")
       filter("D"."DEPARTMENT_ID"="E"."DEPARTMENT_ID")
 
Column Projection Information (identified by operation id):
-----------------------------------------------------------
 
   1 - "DEPARTMENT_NAME"[VARCHAR2,30], COUNT("E"."SALARY")[22], COUNT(*)[22], 
       SUM("E"."SALARY")[22]
   2 - (#keys=1) "DEPARTMENT_NAME"[VARCHAR2,30], COUNT("E"."SALARY")[22], 
       COUNT(*)[22], SUM("E"."SALARY")[22]
   3 - (#keys=0) "D"."DEPARTMENT_NAME"[VARCHAR2,30], "E"."SALARY"[NUMBER,22]
   4 - "D"."DEPARTMENT_ID"[NUMBER,22], "D"."DEPARTMENT_NAME"[VARCHAR2,30]
   5 - "D".ROWID[ROWID,10], "D"."DEPARTMENT_ID"[NUMBER,22]
   6 - (#keys=1) "E"."DEPARTMENT_ID"[NUMBER,22], "E"."SALARY"[NUMBER,22]
   7 - "E"."SALARY"[NUMBER,22], "E"."DEPARTMENT_ID"[NUMBER,22]

-------------------------------------------------------------------------------

```
##### 查询2源代码无指导优化
#### 查询2结果分析
查询源代码2的查询时间是查询1源代码的查询时间的25%，极大程度的优化了查询语句的查询时间。但由于没有实现做出相应的预处理，大量的数据并没有进行过滤，所以查询2语句并没有做到最优。
##查询1与查询2的比较
####查询1中的cost =5,rows=20,谓词信息中有1次索引搜索access,1次全表搜索filter。查询2中的索access,2次全表搜索filter。其中：总体来看，查询1比查询2更优，查询1的参数都优于查询2，从分析两个SQL语句可以看出，查询1是先过滤后汇总。参与汇总与计算的数据量最少。而查询2是先汇总后过滤(having子句)，参与汇总与计算的数据最多。cost =7,rows=106,谓词信息中有1次索引搜
