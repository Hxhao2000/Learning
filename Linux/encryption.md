## 混淆加密

### 业务场景

将用户`id`混淆为8位以内的数字码`code`，并且可以通过`code`还原对应的`id`，其中`id`为从1开始递增的正整数。

### 实现思路

我们可以选择混淆加密的方式来实现这一功能。

其中，我选择公式`code = (id * a + b) mod c`作为我的混淆函数映射，由于`code`为8位以内的数字码，所以`c`应选择8位以内最大的素数`99999989`，`a`和`b`可以为任意小于`c`的素数。

### 数学证明

> 由于我们的`c`选择了99999989，因此为保证唯一性，我们的id最大为99999989，以做到{ id }域和{ code }域中的元素一一对应。

$定理一：不存在id_1, id_2 \in[1, c], id_1 \neq id_2使得code_1 == code_2，其中code_1为id_1的映射结果，code_2为id_2的映射结果。$

**证：**

我们可以使用反证法。

$我们假设存在id_1, id_2 \in[1, c], id_1 \neq id_2使得code_1 == code_2，则$

1. $(id_1 \times a + b) \bmod c == (id_2 \times a + b) \bmod c$

2. $id_1 \times a + b - (id_2 \times a + b) = n \times c,\,n\in N $
3. $id_1 - id_2 = n \times c \div a$
4. $由于a和c都是素数，且id_1-id_2 \in N，因此n = m \times a, m \in N，则n \times c \div a = n \times m \times c$
5. $id_1 - id_2 = n \times c \div a = n \times m \times c，由于n,m \in N且id_1 - id_2 \neq 0，则\left| id_1-id_2 \right| \geq c$
6. $又因id_1, id_2 \in[1, c]，则\left| id_1-id_2 \right| < c$
7. `5`与`6`的结论冲突，因此假设不成立，故**定理一成立**

$定理二：任取id \in[1, c]，都有唯一的code \in [0, c-1]为id的映射结果$。

**证：**

1. $由code = (id \times a + b) \bmod c，可得code \in [0, c-1]$

2. $由定理一，任取id_1, id_2 \in[1, c], id_1 \neq id_2，必有code_1 \neq code_2$

3. $由 id \in [1, c]，则id域的大小为c，由code \in [0, c-1]，则code域的大小也为c$
4. 结合`2`和`3`，可得**定理二成立**

### 实验

```go
package experiment

import "testing"

const (
	a int64 = 9967
	b int64 = 9973
	c int64 = 99999989
)

func IdHash(id int) int64 {
	return (a*int64(id) + b) % c
}

func TestIdHash(t *testing.T) {
	codeSet := map[int64]int{}
	for id := 1; id < int(c); id++ {
		code := idHash(id)
		if old_id, ok := codeSet[code]; !ok {
			codeSet[code] = id
		} else {
			t.Fatalf("old_id: %v, new_id: %v", old_id, id)
		}
	}
}
```

### 参考

[为什么要用素数作为模](https://flat2010.github.io/2018/04/19/%E6%A8%A1%E8%BF%90%E7%AE%97%E4%B8%AD%E4%B8%BA%E4%BD%95%E8%A6%81%E7%94%A8%E7%B4%A0%E6%95%B0%E4%BD%9C%E4%B8%BA%E6%A8%A1/)