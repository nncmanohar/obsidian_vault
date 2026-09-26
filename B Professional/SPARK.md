

### Apache Spark — Interview Scenario Questions

#### Spark Fundamentals

1. A Spark job works fine with 10 GB of data but becomes extremely slow with 1 TB. How would you investigate?
2. Explain what happens internally when you execute an action such as `count()`.
3. A Spark job has 1,000 tasks for processing a relatively small dataset. What could cause this?
4. When would you use `repartition()` vs `coalesce()`?
5. Why can increasing the number of executors sometimes make a Spark job slower?

#### Data Skew

6. One Spark task runs for 40 minutes while all other tasks finish in 2 minutes. What would you investigate?
7. How would you identify data skew from a Spark UI?
8. How would you handle a heavily skewed join?
9. What is **salting**, and when would you use it?
10. A join key has one value representing 40% of the entire dataset. How would you design the join?

#### Joins

11. You have a 2 TB fact table and a 50 MB dimension table. How would you optimize the join?
12. Spark is performing a Sort-Merge Join when you expected a Broadcast Hash Join. What would you investigate?
13. A broadcast join causes executors to run out of memory. What could be happening?
14. Two large datasets need to be joined, but neither can be broadcast. How would you optimize the join?
15. How would you detect whether a join is causing a shuffle?

#### Shuffle

16. What exactly happens during a Spark shuffle?
17. A job spends 80% of its time in shuffle. How would you troubleshoot it?
18. Why can `groupBy()` be expensive in Spark?
19. How would you reduce unnecessary shuffles in a multi-stage Spark pipeline?
20. What is the difference between **narrow and wide transformations**, and why does it matter?

#### Partitions & Files

21. You have 100,000 small Parquet files. A Spark job takes hours before actual processing starts. Why?
22. How would you solve the small-files problem?
23. What happens if you have too few partitions?
24. What happens if you have too many partitions?
25. How would you determine an appropriate number of partitions for a 5 TB dataset?

#### Spark Memory

26. An executor is frequently getting `OutOfMemoryError`. How would you troubleshoot it?
27. What is the difference between **heap memory, execution memory, storage memory, and overhead memory**?
28. Increasing executor memory doesn't solve an OOM problem. What would you investigate next?
29. A Spark application has many executors but very low memory utilization. What might be wrong?
30. How can caching cause an application to fail?

#### Caching & Persistence

31. When would you use `cache()`?
32. When would you use `persist()` instead?
33. You cached a DataFrame, but the next action still takes almost as long as the first. Why?
34. When should you **not** cache a DataFrame?
35. How would you determine whether caching actually improved your job?

#### Spark SQL / DataFrames

36. Why is Spark SQL generally preferred over RDD APIs for many data-engineering workloads?
37. What is **Catalyst Optimizer**?
38. What is **Tungsten**?
39. What is **predicate pushdown**, and how does it improve performance?
40. What is **column pruning**?
41. You filter a Parquet dataset on a partition column, but Spark still reads many files. What would you investigate?

#### Execution & Debugging

42. A Spark job has 20 stages. How would you determine which stage is the bottleneck?
43. How do you use the **Spark UI** to troubleshoot a slow job?
44. What would you look for in the **SQL/DataFrame execution plan**?
45. A job has high CPU utilization but low I/O. What might that indicate?
46. A job has low CPU utilization but extremely high disk/network I/O. What might that indicate?

#### Production Scenarios

47. Your daily Spark job normally finishes in 30 minutes but suddenly takes 3 hours. Walk through your troubleshooting process.
48. Yesterday's data volume was 500 GB; today's is 2 TB. The job failed. How would you make the pipeline resilient to volume spikes?
49. A Spark job succeeds in development but fails in production with the same code. What would you investigate?
50. Your Spark pipeline processes 10 TB daily and the business wants it reduced from 2 hours to 30 minutes. How would you approach the optimization?

#### Architecture / Lead-Level

51. How would you design a Spark platform for **100 TB/day** processing?
52. How would you decide between **Spark, BigQuery, and Dataproc** for a particular workload?
53. How would you design Spark jobs to handle **late-arriving data**?
54. How would you make a Spark pipeline **idempotent**?
55. How would you design **incremental processing** instead of reprocessing the entire dataset?
56. How would you handle schema evolution in a large Spark pipeline?
57. How would you monitor Spark jobs in production?
58. What metrics would you establish as **Spark pipeline SLAs/SLOs**?
59. How would you prevent a single skewed customer/account from repeatedly causing production failures?
60. A Spark application is optimized at the code level, but infrastructure costs remain very high. How would you approach **cost-performance optimization**?







**Broadcast**

ANALYZE TABLE small_table COMPUTE STATISTICS;
spark.sql.autoBroadcastJoinThreshold

from pyspark.sql.functions import broadcast

result = large.join(
    **broadcast**(small),
    "id"
)

SELECT /*+ **BROADCAST(s)** */
       ...
FROM large l
JOIN small s
ON l.id = s.id;


A broadcast join that causes executor OOM is worse than a Sort-Merge Join.


**Adaptive query execution**

spark.sql.adaptive.enabled

spark.conf.set("spark.sql.adaptive.enabled", "true")
spark.conf.set("spark.sql.adaptive.skewJoin.enabled", "true")



| Join strategy                         | How it works                                                           | Best situation                                       | Main drawback            |
| ------------------------------------- | ---------------------------------------------------------------------- | ---------------------------------------------------- | ------------------------ |
| **Broadcast Hash Join (BHJ)**         | Broadcasts small table to every executor; hashes join keys             | One side is small                                    | Memory pressure          |
| **Sort-Merge Join (SMJ)**             | Shuffle both sides by key, sort, then merge                            | Both sides large                                     | Expensive shuffle + sort |
| **Shuffle Hash Join (SHJ)**           | Shuffle both sides, build hash table on smaller side of each partition | One side smaller, but not small enough to broadcast  | Memory + shuffle         |
| **Broadcast Nested Loop Join (BNLJ)** | Broadcast one side and compare rows without normal hash-key matching   | Non-equi / complex joins where hash join can't apply | Can be very expensive    |
| **Cartesian Product**                 | Every row from A paired with every row from B                          | Rare; intentional cross join                         | Potentially enormous     |