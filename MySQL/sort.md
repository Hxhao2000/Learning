## 排序函数

官方文档在[这里](https://dev.mysql.com/doc/refman/8.0/en/window-function-descriptions.html).

> MySQL8.0以上可用

### rank()

按照某字段的排序结果添加排名，但它是跳跃的、间断的排名，例如两个并列第一名后，下一个是第三名，1、1、3、4.

```sql
SELECT Score, rank() over(ORDER BY Score desc) as 'Rank' FROM score;
# 分组排序
SELECT Score, rank() over(partition by xxx ORDER BY Score desc) as 'Rank' FROM score;
```

### row_number()

它是将某字段按照顺序依次添加行号。如1、2、3、4

```sql
SELECT Score, row_number() over(ORDER BY Score desc) as 'Rank' FROM score；
# 分组排序
SELECT Score, row_number() over(partition by xxx ORDER BY Score desc) as 'Rank' FROM score；
```

### dense_rank()

dense 英语中指“稠密的、密集的”。dense_rank()是的排序数字是连续的、不间断。当有相同的分数时，它们的排名结果是并列的，例如，1、2、2、3、4。

```sql
SELECT Score, dense_rank() over(ORDER BY Score desc) as 'Rank' FROM score；
# 分组排序
SELECT Score, dense_rank() over(partition by xxx ORDER BY Score desc) as 'Rank' FROM score；
```

4.总结

| 函数名       | 关键词   | 作用                                                         |
| ------------ | -------- | ------------------------------------------------------------ |
| rank()       | **跳跃** | 按顺序排名，相同值排名相同，但下一个排名将向后跳跃           |
| dense_rank() | **连续** | 按顺序排名，相同值排名相同，下一个排名保持连续               |
| row_number() | **全值** | 按顺序排名，相同值排名递增，最后排名数即为总个数             |
| ntile()      | **分组** | 步骤A：尝试按总组数进行平均，如果不能均分，则先分出一个平均数向下取整+1的组，然后继续步骤A，直到可以均分 |