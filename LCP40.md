[Problem: LCP 40. 心算挑战](https://leetcode.cn/problems/uOAnQW/description/)

### 方法：排序+前缀和+分组/合并

目标得分是偶数，而偶数是由 `奇+奇` 得到或者 `偶(+偶)` 得到。

### 分组

既然是奇数与奇数相加，偶数与偶数相加，那将奇偶进行分组。

为了得到最大的偶数和，那就可以将奇偶两组数都进行 **排序**。使用贪心的思路，想要让和最大，那就尽量让每次挑选的数都最大。

那么，奇数组和偶数组应该各取几个数？

不好想的话，那就用 **暴力**。不管取几个数，奇数组一定是取偶数个，不然累加和不会是偶数。那就枚举所有可能的情况，假设奇数组取 $odd$ 个，那偶数组一定取 $cnt-odd$ 个，最终统计出一个最大即可。

对于数组求和来说，会联想到 **前缀和**。前缀和的原理简单，就是从左到右不断累加从 $[0,i]$ 遍历到的元素的和。

这里使用数组 $[1,2,3,4,5]$ 举例，解释前缀思想。定义一个前缀数组 $pre$，其中 $pre[i]$ 表示数组 $nums$ 的区间 $[0,i]$ 的累加和。

- $pre[0]$ 对应 $nums[0,0]$，所以 $pre[0]=nums[0]=1$
- $pre[1]$ 对应 $nums[0,1]$，所以 $pre[1]=pre[0]+nums[1]=3$
- $pre[2]$ 对应 $nums[0,2]$，所以 $pre[2]=pre[1]+nums[2]=6$
- ...

对于任意一个数组片段 $[0,i]$ 的求和，只需要 $O(1)$ 时间即可得到。

毕竟每次都是取大的数，从大到小排序后都在数组前面部分，所以用前缀和优化后，就无需在每次取 $odd$ 个数时，还去遍历求和。

前缀和是一种非常常用的技巧，只要是连续子数组都可以使用。如果数组过长，那就不太适合。

```Python
# python
class Solution:
    def maxmiumScore(self, cards: List[int], cnt: int) -> int:
        # 按照从大到小排序卡片
        cards.sort(reverse=True)

        # 初始化奇数和偶数前缀和
        odd_prefix = [0]
        even_prefix = [0]
        
        # 奇偶分组
        for num in cards:
            if num % 2 == 0:
                even_prefix.append(num)
            else:
                odd_prefix.append(num)

        # 计算前缀和，原地修改
        for i in range(1, len(even_prefix)):
            even_prefix[i] += even_prefix[i - 1]
        
        for i in range(1, len(odd_prefix)):
            odd_prefix[i] += odd_prefix[i - 1]

        ans = 0
        
        # 假定奇数的个数，结合偶数前缀和找出最大得分
        for odd_count in range(0, min(len(odd_prefix), cnt + 1), 2):
            even_count = cnt - odd_count
            # 偶数不够
            if even_count > len(even_prefix) - 1:
                continue
            ans = max(ans, odd_prefix[odd_count] + even_prefix[even_count])

        return ans
```

```java
// java
class Solution {
    public int maxmiumScore(int[] cards, int cnt) {
        // 按照从大到小排序卡片
        Arrays.sort(cards);
        for (int i = 0; i < cards.length / 2; i++) {
            int temp = cards[i];
            cards[i] = cards[cards.length - 1 - i];
            cards[cards.length - 1 - i] = temp;
        }

        // 初始化奇数和偶数前缀和
        List<Integer> oddPrefix = new ArrayList<>();
        List<Integer> evenPrefix = new ArrayList<>();
        oddPrefix.add(0);
        evenPrefix.add(0);
        
        // 奇偶分组
        for (int num : cards) {
            if (num % 2 == 0) {
                evenPrefix.add(evenPrefix.get(evenPrefix.size() - 1) + num);
            } else {
                oddPrefix.add(oddPrefix.get(oddPrefix.size() - 1) + num);
            }
        }

        int ans = 0;

        // 假定奇数的个数，结合偶数前缀和找出最大得分
        for (int oddCount = 0; oddCount <= Math.min(oddPrefix.size() - 1, cnt); oddCount += 2) {
            int evenCount = cnt - oddCount;
            // 偶数不够
            if (evenCount > evenPrefix.size() - 1) {
                continue;
            }
            ans = Math.max(ans, oddPrefix.get(oddCount) + evenPrefix.get(evenCount));
        }

        return ans;
    }
}
```

- 时间复杂度： $O(nlogn)$，其中 $n$ 为数组 $nums$ 的长度
- 空间复杂度： $O(1)$，忽略排序产生的栈开销

---

### 合并

分组进行排序还是有点麻烦，那为什么不放在一起排序呢？

还是贪心的思路，尝试取前 $cnt$ 个数，假如它们的累加和是偶数，那不就达到理论最大了。

当然，一般情况下累加和是奇数。目前的和 $sum$ 是奇数，如果想要保持 $cnt$ 个数，只有两种方法：

- 去掉一个奇数，补充一个偶数
- 去掉一个偶数，补充一个奇数

已知 $sum$ 是理论最大，还想要让答案最大，那就只能从前 $cnt$ 个数中去掉一个最小的奇数或者最小的偶数，然后从后 $n-cnt$ 个数中取出一个最大的偶数或者奇数。

因为数组已经排序，所以找到前半段（前 $cnt$）的最值与后半段（后 $n-cnt$）的最值很容易，直接去遍历一次就能得到。

代码中已附详细注释。

```Python
# python
class Solution:
    def maxmiumScore(self, cards: List[int], cnt: int) -> int:
        # 从大到小排序
        cards.sort(reverse=True)
        # 计算最大的 cnt 个数之和
        total_sum = sum(cards[:cnt])
        # 如果总和是偶数，直接返回
        if total_sum % 2 == 0:
            return total_sum
        
        def find_replacement_sum(last_card: int) -> int:
            # 找到一个与 last_card 奇偶性不同的最大数
            for card in cards[cnt:]:
                if card % 2 != last_card % 2:
                    return total_sum - last_card + card
            return 0
        
        last_card = cards[cnt - 1]
        # 先替换一次得到答案
        max_score = find_replacement_sum(last_card)
        
        # 再去尝试替换另一个奇偶性不同的最小数
        for card in reversed(cards[:cnt]):
            if card % 2 != last_card % 2:  # 找到了
                max_score = max(max_score, find_replacement_sum(card))
                break
        
        return max_score
```

```java
// java
class Solution {
    public int maxmiumScore(int[] cards, int cnt) {
        // 排序
        Arrays.sort(cards);
        // 倒序，变成从大到小
        for (int i = 0; i < cards.length / 2; i++) {
            int temp = cards[i];
            cards[i] = cards[cards.length - 1 - i];
            cards[cards.length - 1 - i] = temp;
        }
        
        // 计算最大的 cnt 个数之和
        int totalSum = 0;
        for (int i = 0; i < cnt; i++) {
            totalSum += cards[i];
        }
        
        // 如果总和是偶数，直接返回
        if (totalSum % 2 == 0) {
            return totalSum;
        }
        
        // 先替换一次得到答案
        int lastCard = cards[cnt - 1];
        int maxScore = findReplacementSum(cards, cnt, totalSum, lastCard);
        
        // 再去尝试替换另一个奇偶性不同的最小数
        for (int i = cnt - 1; i >= 0; i--) {
            if (cards[i] % 2 != lastCard % 2) {
                maxScore = Math.max(maxScore, findReplacementSum(cards, cnt, totalSum, cards[i]));
                break;
            }
        }
        
        return maxScore;
    }

    // 替换掉lastCard
    private int findReplacementSum(int[] cards, int cnt, int totalSum, int lastCard) {
        for (int i = cnt; i < cards.length; i++) {
            if (cards[i] % 2 != lastCard % 2) {  // 找到一个最大的奇偶性不同的数
                return totalSum - lastCard + cards[i];
            }
        }
        return 0;
    }
}
```

- 时间复杂度： $O(nlogn)$，其中 $n$ 为数组 $nums$ 的长度
- 空间复杂度： $O(1)$，忽略排序产生的栈开销

> 题解已发布力扣平台 [我的题解](https://leetcode.cn/problems/uOAnQW/solutions/2864885/tan-xin-shuang-jie-fen-zu-pai-xu-he-bing-x3h9/)
