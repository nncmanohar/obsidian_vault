
For a **Lead Data Engineer / Technology Manager** interview, I’d organize Advanced SQL into these areas:

### Advanced SQL — Interview Checklist

1. **Window Functions**
    
    - `ROW_NUMBER`, `RANK`, `DENSE_RANK`
    - `LAG`, `LEAD`
    - Running totals, moving averages
    - Window frames: `ROWS` vs `RANGE`
2. **CTEs**
    
    - Recursive CTEs
    - Multiple CTE chains
    - CTE vs subquery vs temporary table
3. **Advanced Joins**
    
    - Self joins
    - Semi/anti joins
    - `EXISTS` vs `IN`
    - Many-to-many joins
    - Join explosion
4. **Set Operations**
    
    - `UNION` vs `UNION ALL`
    - `INTERSECT`
    - `EXCEPT/MINUS`
5. **Subqueries**
    
    - Correlated subqueries
    - Scalar subqueries
    - `EXISTS` / `NOT EXISTS`
6. **Aggregation**
    
    - `GROUPING SETS`
    - `ROLLUP`
    - `CUBE`
    - Conditional aggregation
7. **Query Execution & Optimization**
    
    - Execution plans
    - Predicate pushdown
    - Partition pruning
    - Join strategies
    - Broadcast/hash/nested-loop joins
    - Statistics and cardinality estimation
8. **NULL & Three-Valued Logic**
    
    - `NULL` behavior
    - `COALESCE`, `NULLIF`
    - `NOT IN` + `NULL` trap
9. **Data Modeling in SQL**
    
    - Slowly Changing Dimensions
    - Deduplication
    - Surrogate keys
    - Fact/dimension design
    - Snapshot vs transaction tables
10. **Advanced Patterns**
    
    - Gaps and islands
    - Top-N per group
    - Deduplication
    - Sessionization
    - As-of joins
    - Change detection
    - Pivot/unpivot
11. **Transactions & Concurrency**
    - ACID
    - Isolation levels
    - Dirty/non-repeatable/phantom reads
    - Locks and deadlocks
12. **Distributed SQL / BigQuery**
    - Partitioning vs clustering
    - Slot consumption
    - Shuffle
    - Broadcast joins
    - Approximate aggregation
    - `QUALIFY`
    - Nested/repeated fields

### ⭐ Highest-priority interview topics

If you have limited time, focus first on:

**Window functions → Join optimization → CTEs → Execution plans → Deduplication → Gaps & Islands → SCD → NULL logic → Partitioning/Clustering → Distributed query execution.**


----------
**CTE (Common Table Expression)** is a **temporary named result set defined within a single SQL statement**, using `WITH`.

Primarily to make complex SQL **modular and readable**, and to reuse an intermediate result within the same statement.


```
WITH high_salary AS (
    SELECT employee_id, salary
    FROM employees
    WHERE salary > 100000
)
SELECT *
FROM high_salary;
```

Recursive CTE 
(Anchor query → recursive query → repeat until no more rows.)

```
WITH RECURSIVE employee_tree AS (
    -- Anchor: start with the CEO
    SELECT employee_id, name, manager_id, 1 AS level
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive part: find employees under them
    SELECT e.employee_id, e.name, e.manager_id, t.level + 1
    FROM employees e
    JOIN employee_tree t
        ON e.manager_id = t.employee_id
)
SELECT *
FROM employee_tree;
```


Multiple CTE Chains
You can define **multiple CTEs**, where one CTE uses the previous one.

