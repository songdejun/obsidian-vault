# 11. [滑动窗口最大值]([239. 滑动窗口最大值 - 力扣（LeetCode）](https://leetcode.cn/problems/sliding-window-maximum/description/?envType=study-plan-v2&envId=top-100-liked))

给你一个整数数组 `nums`，有一个大小为 `k` 的滑动窗口从数组的最左侧移动到数组的最右侧。你只可以看到在滑动窗口内的 `k` 个数字。滑动窗口每次只向右移动一位。

返回 *滑动窗口中的最大值* 。

 

**示例 1：**

```
输入：nums = [1,3,-1,-3,5,3,6,7], k = 3
输出：[3,3,5,5,6,7]
解释：
滑动窗口的位置                最大值
---------------               -----
[1  3  -1] -3  5  3  6  7       3
 1 [3  -1  -3] 5  3  6  7       3
 1  3 [-1  -3  5] 3  6  7       5
 1  3  -1 [-3  5  3] 6  7       5
 1  3  -1  -3 [5  3  6] 7       6
 1  3  -1  -3  5 [3  6  7]      7
```

**示例 2：**

```
输入：nums = [1], k = 1
输出：[1]
```

 

**提示：**

- `1 <= nums.length <= 105`
- `-104 <= nums[i] <= 104`
- `1 <= k <= nums.length`

## (1) 单调队列

```python
class Solution:
    def maxSlidingWindow(self, nums: List[int], k: int) -> List[int]:
        from collections import deque

        stk = deque()
        ans = []
        for i in range(len(nums)):
            if len(stk) > 0 and i - stk[0] + 1 > k:
                stk.popleft()

            while len(stk) > 0 and nums[i] > nums[stk[-1]]:
                stk.pop()
            
            stk.append(i)

            if i >= k - 1:
                ans.append(nums[stk[0]])

        return ans
```

ACWing原题。

# 12. [最小覆盖子串](https://leetcode.cn/problems/minimum-window-substring/)

给你一个字符串 `s` 、一个字符串 `t` 。返回 `s` 中涵盖 `t` 所有字符的最小子串。如果 `s` 中不存在涵盖 `t` 所有字符的子串，则返回空字符串 `""` 。

 

**注意：**

- 对于 `t` 中重复字符，我们寻找的子字符串中该字符数量必须不少于 `t` 中该字符数量。
- 如果 `s` 中存在这样的子串，我们保证它是唯一的答案。

 

**示例 1：**

```
输入：s = "ADOBECODEBANC", t = "ABC"
输出："BANC"
解释：最小覆盖子串 "BANC" 包含来自字符串 t 的 'A'、'B' 和 'C'。
```

**示例 2：**

```
输入：s = "a", t = "a"
输出："a"
解释：整个字符串 s 是最小覆盖子串。
```

**示例 3:**

```
输入: s = "a", t = "aa"
输出: ""
解释: t 中两个字符 'a' 均应包含在 s 的子串中，
因此没有符合条件的子字符串，返回空字符串。
```

 

**提示：**

- `m == s.length`
- `n == t.length`
- `1 <= m, n <= 105`
- `s` 和 `t` 由英文字母组成

## (1)  滑动窗口 + 哈希

找字串问题常常用到滑动窗口。

异位词常用的处理方法就是哈希成词频，优化一点的做法就是表示成list[26]即可。

最小覆盖的核心思想就是大胆扩大窗口，一旦全部字母频数达到就一直缩小窗口到破环完全匹配的条件。那时候就找到了当下最小的情况。

```python
class Solution:
    def minWindow(self, s: str, t: str) -> str:
        from collections import Counter, defaultdict
        
        lt = len(t)
        ls = len(s)
        if (lt == 0 or ls == 0):
            return ""

        target = Counter(t)
        ltar = len(target)

        have = defaultdict(int)
        ok = 0
        st = 0
        ans = ""
        la = float('inf')

        for i, c in enumerate(s):
            have[c] += 1
            if have[c] == target[c]:
                ok += 1
            
            # 只要一全部匹配到就尝试缩小窗口
            if (ok == ltar):
                # 缩小的条件就是直到破坏全部匹配的情况
                while (True):
                    a = s[st]
                    have[a] -= 1
                    st += 1

                    if have[a] < target[a]:
                        ok -= 1
                        break
                
                # 到这里全部匹配被破坏
                # 本次循环实现的全部匹配的最短情况就是 s[st-1 ~ i]
                # la > i - (st - 1) + 1
                if (la > i - st + 2):
                    la = i - st + 2
                    ans = s[st - 1: i + 1]

        return ans
```

# 13. [最大子数组和](https://leetcode.cn/problems/maximum-subarray/?envType=study-plan-v2&envId=top-100-liked)

给你一个整数数组 `nums` ，请你找出一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

**子数组**是数组中的一个连续部分。

 

**示例 1：**

```
输入：nums = [-2,1,-3,4,-1,2,1,-5,4]
输出：6
解释：连续子数组 [4,-1,2,1] 的和最大，为 6 。
```

**示例 2：**

```
输入：nums = [1]
输出：1
```

**示例 3：**

```
输入：nums = [5,4,-1,7,8]
输出：23
```

 

**提示：**

- `1 <= nums.length <= 105`
- `-104 <= nums[i] <= 104`

 

**进阶：**如果你已经实现复杂度为 `O(n)` 的解法，尝试使用更为精妙的 **分治法** 求解。



## (1)  动态规划

```python
class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        """
        计算数组中的最大子数组和。

        采用贪心算法，遍历数组，实时计算当前子数组的和，并与已知最大和比较。
        如果当前和变为负数，则重置当前和，因为负数会降低后续子数组的和。

        :param nums: 整数数组
        :return: 最大子数组和
        """
        # 初始化最大和为负无穷，确保任何数组的最大子数组和都不会小于此值
        ans = float('-inf')
        # 初始化当前和为0
        s = 0
        for x in nums:
            # 将当前元素加到当前和中
            s += x
            # 更新最大和，取当前和与已知最大和中的较大值
            ans = max(ans, s)
            
            # 如果当前和小于0，则重置当前和为0
            # 因为负数和不会对寻找最大子数组和有所帮助
            # 相当于减小问题规模
            if s < 0:
                s = 0

        return ans
```

对于任何后缀来说，一旦前缀大于0，对总和总是有益的。一旦小于0，就没用了，该舍弃。

## (2) 分治

```python
class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        """
        计算数组中的最大子数组和。

        :param nums: 整数数组
        :return: 最大子数组和
        """
        return self.f(nums, 0, len(nums) - 1)

    def f(self, nums, l, r):
        """
        使用分治法计算数组中从索引l到r的最大子数组和。

        :param nums: 整数数组
        :param l: 左边界索引
        :param r: 右边界索引
        :return: 最大子数组和
        """
        # 当只有一个元素时，直接返回该元素
        if l == r:
            return nums[l]

        # 计算中间索引
        mid = (l + r) // 2

        # 初始化左边和右边的最大连续和为负无穷
        ls = float('-inf')
        rs = float('-inf')

        # 计算左边连续区间的最大和
        s = 0
        for i in reversed(range(l, mid + 1)):
            s += nums[i]
            ls = max(ls, s)

        # 重置当前和，计算右边连续区间的最大和
        s = 0
        for i in range(mid + 1, r + 1):
            s += nums[i]
            rs = max(rs, s)

        # 返回左边、右边以及跨越中间的最大子数组和中的最大值
        return max(self.f(nums, l, mid), self.f(nums, mid + 1, r), ls + rs)

```

就是最大和等于max(最半边最大，右半边最大，连续最大)

这里分治并不快时间复杂度是 O(nlogn)

# 14. [合并区间 ](https://leetcode.cn/problems/merge-intervals/description/?envType=study-plan-v2&envId=top-100-liked)

以数组 `intervals` 表示若干个区间的集合，其中单个区间为 `intervals[i] = [starti, endi]` 。请你合并所有重叠的区间，并返回 *一个不重叠的区间数组，该数组需恰好覆盖输入中的所有区间* 。

 

**示例 1：**

```
输入：intervals = [[1,3],[2,6],[8,10],[15,18]]
输出：[[1,6],[8,10],[15,18]]
解释：区间 [1,3] 和 [2,6] 重叠, 将它们合并为 [1,6].
```

**示例 2：**

```
输入：intervals = [[1,4],[4,5]]
输出：[[1,5]]
解释：区间 [1,4] 和 [4,5] 可被视为重叠区间。
```

 

**提示：**

- `1 <= intervals.length <= 104`
- `intervals[i].length == 2`
- `0 <= starti <= endi <= 104`

## (1) 贪心

```python
class Solution:
    def merge(self, intervals: List[List[int]]) -> List[List[int]]:
        # 按区间的起始位置排序
        intervals.sort()

        # 初始化当前区间的起始和结束
        st, ed = intervals[0]

        # 用于存储合并后的区间
        ans = []

        # 遍历排序后的区间列表
        for item in intervals[1:]:
            l, r = item

            # 如果当前区间的起始位置大于当前合并区间的结束位置
            # 说明当前区间与合并区间无重叠，将合并区间加入结果
            if l > ed:
                ans.append([st, ed])
                # 更新当前合并区间为当前区间
                st, ed = l, r
            # 否则，当前区间与合并区间有重叠，尝试更新合并区间的结束位置
            else:
                ed = max(ed, r)

        # 将最后一个合并区间加入结果
        ans.append([st, ed])

        return ans
```

# 15. [轮转数组](https://leetcode.cn/problems/rotate-array/description/?envType=study-plan-v2&envId=top-100-liked)

给定一个整数数组 `nums`，将数组中的元素向右轮转 `k` 个位置，其中 `k` 是非负数。

 

**示例 1:**

```
输入: nums = [1,2,3,4,5,6,7], k = 3
输出: [5,6,7,1,2,3,4]
解释:
向右轮转 1 步: [7,1,2,3,4,5,6]
向右轮转 2 步: [6,7,1,2,3,4,5]
向右轮转 3 步: [5,6,7,1,2,3,4]
```

**示例 2:**

```
输入：nums = [-1,-100,3,99], k = 2
输出：[3,99,-1,-100]
解释: 
向右轮转 1 步: [99,-1,-100,3]
向右轮转 2 步: [3,99,-1,-100]
```

 

**提示：**

- `1 <= nums.length <= 105`
- `-231 <= nums[i] <= 231 - 1`
- `0 <= k <= 105`

 

**进阶：**

- 尽可能想出更多的解决方案，至少有 **三种** 不同的方法可以解决这个问题。
- 你可以使用空间复杂度为 `O(1)` 的 **原地** 算法解决这个问题吗？

## (1) 额外数组

太简单了不写了

## (2) 反转再反转

```python
class Solution:
    def rotate(self, nums: List[int], k: int) -> None:
        """
        Do not return anything, modify nums in-place instead.
        """
        l = len(nums)
        k = k % l
        self.reverse(nums, 0, l - 1)
        self.reverse(nums, 0, k - 1)
        self.reverse(nums, k, l - 1)

        
    def reverse(self, nums, l, r):
        i = l
        j = r
        while i < j:
            nums[i], nums[j] = nums[j], nums[i]
            i += 1
            j -= 1
```

## (3) 环状依赖

数学推导比代码重要，仔细看也不难理解：https://leetcode.cn/problems/rotate-array/solutions/551039/xuan-zhuan-shu-zu-by-leetcode-solution-nipk

# 16. [除自身以外数组的乘积](https://leetcode.cn/problems/product-of-array-except-self/description/?envType=study-plan-v2&envId=top-100-liked)

给你一个整数数组 `nums`，返回 数组 `answer` ，其中 `answer[i]` 等于 `nums` 中除 `nums[i]` 之外其余各元素的乘积 。

题目数据 **保证** 数组 `nums`之中任意元素的全部前缀元素和后缀的乘积都在 **32 位** 整数范围内。

请 **不要使用除法，**且在 `O(n)` 时间复杂度内完成此题。

 

**示例 1:**

```
输入: nums = [1,2,3,4]
输出: [24,12,8,6]
```

**示例 2:**

```
输入: nums = [-1,1,0,-3,3]
输出: [0,0,9,0,0]
```

 

**提示：**

- `2 <= nums.length <= 105`
- `-30 <= nums[i] <= 30`
- 输入 **保证** 数组 `answer[i]` 在 **32 位** 整数范围内

 

**进阶：**你可以在 `O(1)` 的额外空间复杂度内完成这个题目吗？（ 出于对空间复杂度分析的目的，输出数组 **不被视为** 额外空间。）

## (1) 预处理+除法

题目说了不能用除法，出发点是如果数组中有0，那 0 的那一项就在“预处理+除法”方法的“定义域”之外了。

但其实仔细一项如果有不止一个 0，那就都是 0。

如果只有一个 0，那只为 0 这一项找乘积即可。

```python
class Solution:
    def productExceptSelf(self, nums: List[int]) -> List[int]:
        # 初始化乘积为 1
        p = 1
        # 记录数组中 0 的位置
        zero_idx = None

        # 遍历数组
        for i, x in enumerate(nums):
            if x == 0:
                # 如果已经有一个 0 了，直接返回全 0 数组
                if zero_idx is not None:
                    return [0] * len(nums)
                # 否则记录当前 0 的位置
                else:
                    zero_idx = i
            else:
                # 计算非零元素的乘积
                p *= x

        # 如果数组中只有一个 0
        if zero_idx is not None:
            # 返回一个数组，只有 0 的位置为 p，其他位置为 0
            return [0 if i != zero_idx else p for i in range(len(nums))]
        # 如果数组中没有 0
        else:
            # 返回每个位置的总乘积除以当前元素的值
            return [p // nums[i] for i in range(len(nums))]
```

## (2) 预处理前缀后缀乘积数组





# 17. [缺失的第一个正数](https://leetcode.cn/problems/first-missing-positive/?envType=study-plan-v2&envId=top-100-liked)

给你一个未排序的整数数组 `nums` ，请你找出其中没有出现的最小的正整数。

请你实现时间复杂度为 `O(n)` 并且只使用常数级别额外空间的解决方案。

 

**示例 1：**

```
输入：nums = [1,2,0]
输出：3
解释：范围 [1,2] 中的数字都在数组中。
```

**示例 2：**

```
输入：nums = [3,4,-1,1]
输出：2
解释：1 在数组中，但 2 没有。
```

**示例 3：**

```
输入：nums = [7,8,9,11,12]
输出：1
解释：最小的正数 1 没有出现。
```

 

**提示：**

- `1 <= nums.length <= 105`
- `-231 <= nums[i] <= 231 - 1`



## (1) 原地哈希

```python
class Solution:
    def firstMissingPositive(self, nums: List[int]) -> int:
        l = len(nums)
        
        # 遍历数组中的每个元素
        for x in nums:
            # 如果当前元素 x 在有效范围内（1 <= x <= l），则进行哈希标记
            while 1 <= x <= l:
                # 记录 nums[x - 1] 的值，因为 nums[x - 1] 将被标记为 float('inf')
                y = nums[x - 1]
                # 将 nums[x - 1] 标记为 float('inf')，表示 x 已经出现过
                nums[x - 1] = float('inf')
                # 如果被覆盖的值 y 也在有效范围内，则继续处理 y
                x = y

        # 检查 1 到 l 之间的最小未出现正整数
        for i in range(1, l + 1):
            # 哈希位置为 i - 1
            hash_value = i - 1
            # 如果 nums[hash_value] 未被标记为 float('inf')，说明 i 未出现
            if nums[hash_value] != float('inf'):
                return i
        
        # 如果 1 到 l 都出现了，则返回 l + 1
        return l + 1
```





# 18. [矩阵置零](https://leetcode.cn/problems/set-matrix-zeroes/?envType=study-plan-v2&envId=top-100-liked)

给定一个 `*m* x *n*` 的矩阵，如果一个元素为 **0** ，则将其所在行和列的所有元素都设为 **0** 。请使用 **[原地](http://baike.baidu.com/item/原地算法)** 算法**。**



 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/08/17/mat1.jpg)

```
输入：matrix = [[1,1,1],[1,0,1],[1,1,1]]
输出：[[1,0,1],[0,0,0],[1,0,1]]
```

**示例 2：**

![img](https://assets.leetcode.com/uploads/2020/08/17/mat2.jpg)

```
输入：matrix = [[0,1,2,0],[3,4,5,2],[1,3,1,5]]
输出：[[0,0,0,0],[0,4,5,0],[0,3,1,0]]
```

 

**提示：**

- `m == matrix.length`
- `n == matrix[0].length`
- `1 <= m, n <= 200`
- `-231 <= matrix[i][j] <= 231 - 1`

 

**进阶：**

- 一个直观的解决方案是使用  `O(*m**n*)` 的额外空间，但这并不是一个好的解决方案。
- 一个简单的改进方案是使用 `O(*m* + *n*)` 的额外空间，但这仍然不是最好的解决方案。
- 你能想出一个仅使用常量空间的解决方案吗？



## (1) 用列表存储要置 0 的行和列

## (2) 原地操作

```python
class Solution:
    def setZeroes(self, matrix: List[List[int]]) -> None:
        """
        Do not return anything, modify matrix in-place instead.
        """
        m = len(matrix)
        n = len(matrix[0])
        r0 = False
        c0 = False
        
        for i in range(m):
            if matrix[i][0] == 0:
                c0 = True
        for j in range(n):
            if matrix[0][j] == 0:
                r0 = True
        
        for i in range(m):
            for j in range(n):
                if matrix[i][j] == 0:
                    # 用 0 来标记该行要置为 0
                    matrix[i][0] = 0
                    # 用 0 来标记该列要置为 0
                    matrix[0][j] = 0

        for i in range(1, m):
            if matrix[i][0] == 0:
                for j in range(n):
                     matrix[i][j] = 0
        for j in range(1, n):
            if matrix[0][j] == 0:
                for i in range(m):
                    matrix[i][j] = 0

        if r0:
            for j in range(n):
                matrix[0][j] = 0
        if c0:
            for i in range(m):
                matrix[i][0] = 0
```

当 (i, j) 是 0 时就代表 i 行，j 列的元素没用了可以覆盖。

我们取 i 行、j 列的第一个元素作为标记置为 0，也就是将第 0 行，第 0 列作为做法 (1) 的列表。

也就是把问题规模缩小到 `matrix[1:][1:]`。

再用两个布尔变量指示 0 行，0列要不要被置零。

> 1. **第 0 行，第 0 列本身的数据会被错误覆盖吗？**
>    不会。
>    因为当 (i, 0) 产生数据覆盖时，说明 i 行有数据为 0。
>    说明这一行本来就要被覆盖，在这里覆盖没有影响。





# 19. [螺旋矩阵](https://leetcode.cn/problems/spiral-matrix/?envType=study-plan-v2&envId=top-100-liked)

给你一个 `m` 行 `n` 列的矩阵 `matrix` ，请按照 **顺时针螺旋顺序** ，返回矩阵中的所有元素。

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/11/13/spiral1.jpg)

```
输入：matrix = [[1,2,3],[4,5,6],[7,8,9]]
输出：[1,2,3,6,9,8,7,4,5]
```

**示例 2：**

![img](https://assets.leetcode.com/uploads/2020/11/13/spiral.jpg)

```
输入：matrix = [[1,2,3,4],[5,6,7,8],[9,10,11,12]]
输出：[1,2,3,4,8,12,11,10,9,5,6,7]
```

 

**提示：**

- `m == matrix.length`
- `n == matrix[i].length`
- `1 <= m, n <= 10`
- `-100 <= matrix[i][j] <= 100`



## (1) 用布尔数组标记已经走过的位置

```python
class Solution:
    def spiralOrder(self, matrix: List[List[int]]) -> List[int]:
        m = len(matrix)
        n = len(matrix[0])

        # 代表走过 (i, j) 没有
        go = [[False] * n for i in range(m)]

        ans = []

        # 四个移动方向
        ds = [[0, 1], [1, 0], [0, -1], [-1, 0]]
        # 代表当前是第几个方向
        i = 0

        def check(x, y):
            if 0 <= x < m and 0 <= y < n and not go[x][y]:
                return True
            return False

        def arrive(x, y):
            ans.append(matrix[x][y])
            go[x][y] = True

        # 代表当前位置
        # 假设从 (0, -1) 开始，顺利进入循环
        x, y = 0, -1
        # 每次进入 while 代表准备从当前位置离开
        while True:
            try_x = x + ds[i][0]
            try_y = y + ds[i][1]

            # 如果不能走就换方向
            if not check(try_x, try_y):
                i = (i + 1) % 4
                continue
			
            # 可以走则更新位置
            x = try_x
            y = try_y

            arrive(x, y)

            if len(ans) == m * n:
                break

        return ans

```



## (2) 分层输出

```python
class Solution:
    def spiralOrder(self, matrix: List[List[int]]) -> List[int]:
        m = len(matrix)
        n = len(matrix[0])

        ans = []
        l, t = (0, 0)
        r, b = (n - 1, m - 1)

        while l < r and t < b:
            for j in range(l, r):
                ans.append(matrix[t][j])
            for i in range(t, b):
                ans.append(matrix[i][r])
            for j in range(r, l, -1):
                ans.append(matrix[b][j])
            for i in range(b, t, -1):
                ans.append(matrix[i][l])
            l += 1
            r -= 1
            t += 1
            b -= 1

        if l == r:
            for i in range(t, b + 1):
                ans.append(matrix[i][l])
        elif t == b:
            for j in range(l, r + 1):
                ans.append(matrix[t][j])
        return ans
```

# 20. [旋转图像](https://leetcode.cn/problems/rotate-image/description/?envType=study-plan-v2&envId=top-100-liked)

给定一个 *n* × *n* 的二维矩阵 `matrix` 表示一个图像。请你将图像顺时针旋转 90 度。

你必须在**[ 原地](https://baike.baidu.com/item/原地算法)** 旋转图像，这意味着你需要直接修改输入的二维矩阵。**请不要** 使用另一个矩阵来旋转图像。

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/08/28/mat1.jpg)

```
输入：matrix = [[1,2,3],[4,5,6],[7,8,9]]
输出：[[7,4,1],[8,5,2],[9,6,3]]
```

**示例 2：**

![img](https://assets.leetcode.com/uploads/2020/08/28/mat2.jpg)

```
输入：matrix = [[5,1,9,11],[2,4,8,10],[13,3,6,7],[15,14,12,16]]
输出：[[15,13,2,5],[14,3,4,1],[12,6,8,9],[16,7,10,11]]
```

 

**提示：**

- `n == matrix.length == matrix[i].length`
- `1 <= n <= 20`
- `-1000 <= matrix[i][j] <= 1000`

## (1) 用轴对称实现旋转

```python
class Solution:
    def rotate(self, matrix: List[List[int]]) -> None:
        """
        Do not return anything, modify matrix in-place instead.
        """
        n = len(matrix)
        for i in range(n):
            for j in range(n - i - 1):
                matrix[i][j], matrix[n - 1 - j][n - 1 - i] = matrix[n - 1 - j][n - 1 - i], matrix[i][j]

        for i in range(n // 2):
            for j in range(n):
                matrix[i][j], matrix[n - 1 - i][j] = matrix[n - 1 - i][j], matrix[i][j]
```
