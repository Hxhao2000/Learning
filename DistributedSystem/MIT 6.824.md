# 6.824

## Lesson 1

- 并行 => 性能

- 错误容忍

- 分布式事务

- 安全与隔离

### 挑战

- 并行

- 局部错误

- 性能

### Infrastructure

- 存储
- 通信
- 计算

### weak consistency

- allow to get older value

- high performance

### strong consistency

- guarantee that `get` response must be the newest value

- cost more resource during replica communicating to ensure above feature

## Lab1 MapReduce

异步模型 <=> 事件循环模型



## Lab2 Raft for tolerance



## Lab3 K/V Server



## Lab4 Shared K/V Service