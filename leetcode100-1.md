[TOC]

# 1. 两数之和

https://leetcode.cn/problems/two-sum/

给定一个整数数组 nums 和一个整数目标值 target，请你在该数组中找出 和为目标值 target  的那 两个 整数，并返回它们的数组下标。

你可以假设每种输入只会对应一个答案，并且你不能使用两次相同的元素。

你可以按任意顺序返回答案。

 

示例 1：

输入：nums = [2,7,11,15], target = 9
输出：[0,1]
解释：因为 nums[0] + nums[1] == 9 ，返回 [0, 1] 。
示例 2：

输入：nums = [3,2,4], target = 6
输出：[1,2]
示例 3：

输入：nums = [3,3], target = 6
输出：[0,1]


提示：

2 <= nums.length <= 104
-109 <= nums[i] <= 109
-109 <= target <= 109
只会存在一个有效答案

---

## (1) 双指针解法

```c++
class Solution {
    public:
        vector<int> twoSum(vector<int>& nums, int target) {
            vector<pair<int, int>> kv;
            for (int i = 0; i < nums.size(); i ++) {
                kv.push_back({nums[i], i});
            }
            sort(kv.begin(), kv.end());

            int i = 0, j = kv.size() - 1;

            while (i < j) {
                if (kv[i].first + kv[j].first == target) {
                    int _i = min(kv[i].second, kv[j].second);
                    int _j = max(kv[i].second, kv[j].second);
                    return {_i, _j};
                }

                if (kv[i].first + kv[j].first > target)
                    j--;
                else
                    i ++;
            }

            return {};
        }
    };
```

`sort()`在头文件`algorithm`。默认非降序排列，不保证相等元素相邻。可以传入函数对象自定义比较函数

```c++
#include <vector>
#include <fmt/ranges.h>
#include <algorithm>

using namespace std;

bool comp1(int a, int b) { return a >= b; }

class
{
    public:
    bool operator()(int a, int b) const { return a >= b; }
}
comp2;

struct
{
    bool operator()(int a, int b) const { return a >= b; }
}
comp3;

int main() {
    std::vector<int> vec = {1, -2, 33, 42, 5};
    sort(vec.begin(), vec.end(), comp3);

    fmt::print("Vector contents: {:*^10}\n", fmt::join(vec, ", "));
    return 0;
}
```



## (2) 哈希解法

```c++
class Solution {
    public:
        vector<int> twoSum(vector<int>& nums, int target) {
            map<int, int> hs;
            for (int i = 0; i < nums.size(); i ++) {
                if (hs.count(target - nums[i]) == 1) {
                    return {hs[target - nums[i]], i};
                }
                hs[nums[i]] = i;
            }

            return {};
        }
    };
```

`map` 类 STL 查找有没有某个键的 API:

```c++
count() # 不是 multimap 都只返回 0 or 1
find() # 返回对应元素的迭代器， multimap 返回任意一个，没有则返回 end()
contains() # c++ 20 返回 bool 
```

# 2. 字母异位词

https://leetcode.cn/problems/group-anagrams/description/?envType=study-plan-v2&envId=top-100-liked

给你一个字符串数组，请你将 字母异位词 组合在一起。可以按任意顺序返回结果列表。

字母异位词 是由重新排列源单词的所有字母得到的一个新单词。

 

示例 1:

输入: strs = ["eat", "tea", "tan", "ate", "nat", "bat"]
输出: [["bat"],["nat","tan"],["ate","eat","tea"]]
示例 2:

输入: strs = [""]
输出: [[""]]
示例 3:

输入: strs = ["a"]
输出: [["a"]]


提示：

1 <= strs.length <= 104
0 <= strs[i].length <= 100
strs[i] 仅包含小写字母

## (1) 暴力表示哈希解法

单词由哪些字母组成最暴力的想法就是用一个 26 维的向量表示：

```c++
class jhash {
public:
    size_t operator()(const vector<int>& vec) const {
        size_t seed = 0;
        for (size_t i = 0; i < vec.size(); i++) {
            seed ^= hash<int>()(vec[i]) + 0x9e3779b9 + (seed << 6) + (seed >> 2);
        }
        return seed;
    }
};

class jequal {
public:
    bool operator()(const vector<int>& vec1, const vector<int>& vec2) const {
        return vec1 == vec2;
    }
};

class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        unordered_map<vector<int>, vector<string>, jhash, jequal> hs;

        for (const auto& item : strs) {
            vector<int> key(26, 0);
            for (const char& c : item) {
                key[c - 'a']++;
            }
            hs[key].push_back(item);
        }

        vector<vector<string>> res;
        for (const auto& [k, v] : hs) {
            res.push_back(v);
        }

        return res;
    }
};
```

自定义 map 类数据结构需要提供 hash 和 equal 的函数对象。

## (2) 优化表示哈希解法

hash 时自定义的 hash 和 equal 通过stl内部保证冲突的时候可以区分不同的 key，不同的异位词映射到同一 hash 时有 equal 保证不出错。或者换个角度说 map 的 key 就是 26 维的向量，不可能出错。

这个题目的O(n)复杂度在判别属于哪个异位词的，使用暴力表示+自定义哈希可以无脑设计一个hash获得不错的复杂度。如果能设计一个不会冲突的hash那就可以优化到最佳的复杂度。

那么就把问题转化成一个26维的向量怎么映射成一个不会冲突的数。算数基本定理表明了一个数可以由它的素因子和对应的指数表示，这就给我们提供了表示方案

```c++
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        int primes[26] = {2, 3, 5, 7, 11, 13, 17, 19, 23, 29,
            31, 37, 41, 43, 47, 53, 59, 61, 67, 71,
            73, 79, 83, 89, 97, 101};

        unordered_map<uint64_t, vector<string>> hs; // 防止溢出

        int mod = 1e9 + 7;

        for (auto& item : strs) {
            uint64_t key = 1;
            for (const auto& c : item) {
                key = key * primes[c - 'a'] % mod;
            }
            hs[key].emplace_back(item);
        }

        vector<vector<string>> res;
        for (auto& [k, v] : hs) {
            res.push_back(v);
        }
        return res;
    }
};
```



## (3) 排序写法

```c++
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        unordered_map<string, vector<string>> hs;

        for (auto& item : strs) {
            vector<int> key;
            for (const auto& c: item) {
                key.push_back(c - 'a');
            }
            sort(key.begin(), key.end());
            hs[string(key.begin(), key.end())].push_back(item);
        }

        vector<vector<string>> res;
        for (auto& [k, v] : hs) {
            res.push_back(v);
        }

        return res;
    }
};
```

高级版

```c++
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        unordered_map<string, vector<string>> hs;

        for (auto& item : strs) {
            auto key = item;
            sort(key.begin(), key.end());
            hs[key].emplace_back(item);
        }

        vector<vector<string>> res;
        for (auto& [k, v] : hs) {
            res.push_back(v);
        }
        return res;
    }
};
```

1. string 可以直接用迭代器 sort。
2. emplace_back 可以代替键存在插入，不存在就用参数作为模板参数类型的构造函数的参数构造。

# 3. 最长连续序列

https://leetcode.cn/problems/longest-consecutive-sequence/description/?envType=study-plan-v2&envId=top-100-liked

给定一个未排序的整数数组 nums ，找出数字连续的最长序列（不要求序列元素在原数组中连续）的长度。

请你设计并实现时间复杂度为 O(n) 的算法解决此问题。

 

示例 1：

输入：nums = [100,4,200,1,3,2]
输出：4
解释：最长数字连续序列是 [1, 2, 3, 4]。它的长度为 4。
示例 2：

输入：nums = [0,3,7,2,5,8,4,6,0,1]
输出：9
示例 3：

输入：nums = [1,0,1,2]
输出：3


提示：

0 <= nums.length <= 105
-109 <= nums[i] <= 109

## (1) hash解法

```c++
class Solution {
public:
    int longestConsecutive(vector<int>& nums) {
        set<int> hs(nums.begin(), nums.end());

        int mn = 1e9, mx = -1e9;
        for (const auto& item: nums) {
            if (item < mn)  {
                mn = item;
            }

            if (item > mx) {
                mx = item;
            }
        }

        int res = 0, cnt = 0;
        for (int i = mn; i <= mx; i ++ ){
            if (hs.count(i) == 1) {
                cnt ++;
            } else {
                res = max(res, cnt);
                cnt = 0;
                i = *hs.upper_bound(i) - 1;
            }
        }

        res = max(res, cnt);

        return res;
    }
};
```

更快版本，应该利用好 O(1) 的查找效率，上面的写法 upper_bound() 并不好，因为既需要 unordered_set，又要用二分去查。

```c++
class Solution {
public:
    int longestConsecutive(vector<int>& nums) {
        unordered_set<int> st(nums.begin(), nums.end());
        int res = 0;

        for(const auto& item: st) {
            if (st.count(item - 1) == 1) {
                continue;
            }

            int next = item + 1;
            while (st.count(next)) {
                next ++;
            }

            res = max(res, next - item);
        }

        return res;
    }
};
```

# 4. 移动零

https://leetcode.cn/problems/move-zeroes/description/?envType=study-plan-v2&envId=top-100-liked

给定一个数组 nums，编写一个函数将所有 0 移动到数组的末尾，同时保持非零元素的相对顺序。

请注意 ，必须在不复制数组的情况下原地对数组进行操作。

 

示例 1:

输入: nums = [0,1,0,3,12]
输出: [1,3,12,0,0]
示例 2:

输入: nums = [0]
输出: [0]


提示:

1 <= nums.length <= 104
-231 <= nums[i] <= 231 - 1

## (1) 双指针

```c++
class Solution {
public:
    void moveZeroes(vector<int>& nums) {
        int j = 0;
        for (int i = 0; i < nums.size(); i ++) {
            if (nums[i] != 0) {
                nums[j ++] = nums[i];
            }
        }

        for (; j < nums.size(); j ++) {
            nums[j] = 0;
        }
    }
};
```

双指针我常常想不到，原因是缺乏对这个技巧的思考。

用暴力做法，每次遇到 0，后面所有元素前移，一直处理完所有的 0。时间复杂度 O(n^2)，扫描所有的 0 是必须的，这是一层有复杂度 O(n)，移动元素暴力做法也是 O(n)，原因是因为每个元素都在每次遇到 0 时都移动一次。要想达到 O(n) 就要实现 O(1) 的复杂度移动元素，也就是每个元素只移动大概一次。这就要求我们在每次移动元素时就知道这个元素最终应该在的位置，那我们可以知道吗?

可以的！在外层循环扫描的时候，记录遇到的非 0 数是第几个数，那它的位置也就清楚了，这时候将这个非 0 数移动。

0 虽然也可以知道他们所处的位置，但是为了不覆盖非 0 数需要将各个 0 的位置记录下来，最后移动 0 到相应的位置。

将上述思路对应成代码：

```c++
class Solution {
public:
    void moveZeroes(vector<int>& nums) {
        int non_zero_cnt = 0;
        vector<int> idx_rec;
        for (int i = 0; i < nums.size(); i ++) {
            if (nums[i] != 0) {
                non_zero_cnt += 1;
                int cur_non_zero_idx = non_zero_cnt - 1; 
                nums[cur_non_zero_idx] = nums[i];
            } else {
                idx_rec.push_back(nums[i]);
            }
        }
        for (int i = non_zero_cnt; i < nums.size(); i ++) {
            nums[i] = idx_rec[i - non_zero_cnt];
        }
    }
};
```

因为 nums[idx_rec[i - non_zero_cnt]] 都是 0，所以 idx_rec 无需记录，优化一下就变成最上面的样子了。

# 5. 盛最多水的容器

https://leetcode.cn/problems/container-with-most-water/description/?envType=study-plan-v2&envId=top-100-liked

给定一个长度为 n 的整数数组 height 。有 n 条垂线，第 i 条线的两个端点是 (i, 0) 和 (i, height[i]) 。

找出其中的两条线，使得它们与 x 轴共同构成的容器可以容纳最多的水。

返回容器可以储存的最大水量。

说明：你不能倾斜容器。

 

示例 1：



输入：[1,8,6,2,5,4,8,3,7]
输出：49 
解释：图中垂直线代表输入数组 [1,8,6,2,5,4,8,3,7]。在此情况下，容器能够容纳水（表示为蓝色部分）的最大值为 49。
示例 2：

输入：height = [1,1]
输出：1


提示：

n == height.length
2 <= n <= 105
0 <= height[i] <= 104

## (1)  双指针

```c++
class Solution {
 public:
  int maxArea(vector<int>& height) {
    int ans = 0;
    int i = 0;
    int j = height.size() - 1;

    while (i < j) {
      if (height[i] < height[j]) {
        ans = max(ans, min(height[j], height[i]) * (j - i));
        i ++;
      } else {
        ans = max(ans, min(height[i], height[j]) * (j - i));
        j --;
      }
    }

    return ans;
  }
};
```

# 6. 三数之和

https://leetcode.cn/problems/3sum/description/?envType=study-plan-v2&envId=top-100-liked

给你一个整数数组 nums ，判断是否存在三元组 [nums[i], nums[j], nums[k]] 满足 i != j、i != k 且 j != k ，同时还满足 nums[i] + nums[j] + nums[k] == 0 。请你返回所有和为 0 且不重复的三元组。

注意：答案中不可以包含重复的三元组。

 

 

示例 1：

输入：nums = [-1,0,1,2,-1,-4]
输出：[[-1,-1,2],[-1,0,1]]
解释：
nums[0] + nums[1] + nums[2] = (-1) + 0 + 1 = 0 。
nums[1] + nums[2] + nums[4] = 0 + 1 + (-1) = 0 。
nums[0] + nums[3] + nums[4] = (-1) + 2 + (-1) = 0 。
不同的三元组是 [-1,0,1] 和 [-1,-1,2] 。
注意，输出的顺序和三元组的顺序并不重要。
示例 2：

输入：nums = [0,1,1]
输出：[]
解释：唯一可能的三元组和不为 0 。
示例 3：

输入：nums = [0,0,0]
输出：[[0,0,0]]
解释：唯一可能的三元组和为 0 。


提示：

3 <= nums.length <= 3000
-105 <= nums[i] <= 105

## (1) 排序 + 循环 + 双指针

```c++
class Solution {
  public:
      vector<vector<int>> threeSum(vector<int>& nums) {
          int size = nums.size();
          sort(nums.begin(), nums.end());
          vector<vector<int>> res;
          for (int i = 0; i < size - 2; i ++) {
              int j = i + 1;
              int k = size - 1;
              
              // 因为排序过了, 大于 0 必定没有
              if (nums[i] > 0) {
                  break;
              }
              
              // 因为枚举的是第一个元素, 后一个可选择的数比前一个少, 可以配对成功的概率更小.
              // 如果后一个和前一个还一样就pass, 因为如果前面的没成, 后面的必不成
              // 前面的成了, 后面的就有可能重复
              if (i > 0 && nums[i] == nums[i - 1]) {
                continue;
              }

              while(j < k) {
                  if (nums[i] + nums[j] + nums[k] > 0) {
                      k --;
                  } else if (nums[i] + nums[j] + nums[k] < 0) {
                      j ++;
                  } else{
                      res.push_back({nums[i], nums[j], nums[k]});


                      // 1. 不考虑重复的待选择数
                      // 2. j < k: 为了防止 j < k 的条件被破坏要加上
                      while (j < k && nums[j] == nums[j + 1]) j ++;
                      while (j < k && nums[k] == nums[k - 1]) k --;

                      j ++;
                      k --;
                  }
              }
          }
  
          return res;
      }
  };
```

总体思路就是消除一个变量，再用双指针。

# 7. 接雨水

https://leetcode.cn/problems/trapping-rain-water/description/?envType=study-plan-v2&envId=top-100-liked

给定 n 个非负整数表示每个宽度为 1 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。

 

示例 1：



输入：height = [0,1,0,2,1,0,1,3,2,1,2,1]
输出：6
解释：上面是由数组 [0,1,0,2,1,0,1,3,2,1,2,1] 表示的高度图，在这种情况下，可以接 6 个单位的雨水（蓝色部分表示雨水）。 
示例 2：

输入：height = [4,2,0,3,2,5]
输出：9


提示：

n == height.length
1 <= n <= 2 * 104
0 <= height[i] <= 105

## (1) 动态规划

```python
class Solution:
    def trap(self, height: List[int]) -> int:
        l = len(height)
        lm = [0] * l
        rm = [0] * l

        for i in range(1, l):
            lm[i] = max(lm[i - 1], height[i - 1])
            rm[l - 1 - i] = max(rm[l - i], height[l - i])
        
        ans = 0
        for i in range(l):
            h = min(lm[i], rm[i])
            ans += max(h - height[i], 0)

        return ans
```

预处理出 lm 和 rm，每个height可以积累的雨水就等于 max(min(lm[i], rm[i]) - height, 0)。

## (2) 单调栈

可以用单调栈做，但是我不懂。。。

## (3) 双指针

```python
class Solution:
    def trap(self, height: List[int]) -> int:
        l = len(height)
        ans = 0
        lm = 0
        rm = 0
        i = 0
        j = l - 1
        while (i <= j):
            if (lm < rm):
                ans += max(lm - height[i], 0)
                lm = max(lm, height[i])
                i += 1
            else:
                ans += max(rm - height[j], 0)
                rm = max(rm, height[j])
                j -= 1

        return ans
```

这个思路来自于对预处理方法（上面的动态规划，虽然我并不知道为什么这个叫）的优化。

第一层优化来自因为从左到右遍历时每个 i 只需要用 lm[i]，而且只用 lm[i] 一次，所以用一个变量保存下当时的 lm[i] 即可。

为什么不能优化 rm 呢？

因为要使用 rm[i]，而 rm[i] 需要从右向左更新到 i 才能获得 rm[i]。

```python
class Solution:
    def trap(self, height: List[int]) -> int:
        l = len(height)
        lm = 0
        rm = [0] * l

        for i in range(1, l):
            rm[l - 1 - i] = max(rm[l - i], height[l - i])
        
        ans = 0
        for i in range(l):
            if i > 0:
                lm = max(lm, height[i - 1])
            h = min(lm, rm[i])
            ans += max(h - height[i], 0)

        return ans
```

左侧和右侧是对称的，左边的 i 可以有非 list 的 lm，从右开始循环的 j 也可以有它的 rm。

这就进入了双指针优化。

虽然 i 和 j 可能不知道它们的 rm[i]、lm[j]，但是接雨水问题只要知道 lm 和 rm 中的小的哪个就行了。

只要有 lm < rm，那么 lm 一定小于 rm[i]，因为 rm[i] 来自于当前的 rm 和 max(height[i ~ j])。

rm 同理。

# 8. 无重复字符的最长子串

https://leetcode.cn/problems/longest-substring-without-repeating-characters/

给定一个字符串 `s` ，请你找出其中不含有重复字符的 **最长 子串** 的长度。

 

**示例 1:**

```
输入: s = "abcabcbb"
输出: 3 
解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。
```

**示例 2:**

```
输入: s = "bbbbb"
输出: 1
解释: 因为无重复字符的最长子串是 "b"，所以其长度为 1。
```

**示例 3:**

```
输入: s = "pwwkew"
输出: 3
解释: 因为无重复字符的最长子串是 "wke"，所以其长度为 3。
     请注意，你的答案必须是 子串 的长度，"pwke" 是一个子序列，不是子串。
```

 

**提示：**

- `0 <= s.length <= 5 * 104`
- `s` 由英文字母、数字、符号和空格组成

## (1) 滑动窗口

```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        if len(s) == 0:
            return 0

        occ = {s[0]}
        cnt = 1
        ans = 1
        st = 0

        for i in range(1, len(s)):
            if s[i] not in occ:
                cnt += 1
                occ.add(s[i])
            else:
                ans = max(ans, cnt)
                while s[st] != s[i]:
                    occ.remove(s[st])
                    st += 1
                # occ.remove(s[st]) 因为s[st]重复了不用删除
                st += 1
                cnt = i - st + 1

        ans = max(ans, cnt)

        return ans
```

用一个set记录有哪些字母，一个 st 下标记录当前串的起点，循环变量 i 表示当前串的终点，当遇到重复的时候从 st 开始在 set 中删除。

# 9. [找到字符串中所有字母异位词](https://leetcode.cn/problems/find-all-anagrams-in-a-string/)
给定两个字符串 `s` 和 `p`，找到 `s` 中所有 `p` 的 **异位词** 的子串，返回这些子串的起始索引。不考虑答案输出的顺序。

 

**示例 1:**

```
输入: s = "cbaebabacd", p = "abc"
输出: [0,6]
解释:
起始索引等于 0 的子串是 "cba", 它是 "abc" 的异位词。
起始索引等于 6 的子串是 "bac", 它是 "abc" 的异位词。
```

 **示例 2:**

```
输入: s = "abab", p = "ab"
输出: [0,1,2]
解释:
起始索引等于 0 的子串是 "ab", 它是 "ab" 的异位词。
起始索引等于 1 的子串是 "ba", 它是 "ab" 的异位词。
起始索引等于 2 的子串是 "ab", 它是 "ab" 的异位词。
```

 

**提示:**

- `1 <= s.length, p.length <= 3 * 104`
- `s` 和 `p` 仅包含小写字母

## (1)  不定长滑窗

```python
class Solution:
    def findAnagrams(self, s: str, p: str) -> List[int]:
        from collections import Counter, defaultdict

        # 记录每个字母需要多少个
        target = Counter(p)
        # 记录每个字母已经有多少个
        window = defaultdict(int)
        # 记录异位词的起点用于出队列
        st = None
        # 记录正在检查的串的长度
        cnt = 0
        ans = list()

        for i in range(len(s)):
            c = s[i]

            # c 不在 p 中
            if c not in target:
                st = None
                cnt = 0
                window.clear()

            # c 在 p 中
            else:
                # window 如果已经有足够的 c
                # 则为了匹配要删除第一个 c 之前的内容
                while window[c] == target[c]:
                    window[s[st]] -= 1
                    st += 1
                    cnt -= 1
                
                # 现在虽然 window 就算有 c 但都不够
                # 可以顺利加入
                window[c] += 1
                cnt += 1

                # 有可能是当前序列的第一个字母
                if st is None:
                    st = i

            # 有了足够的长度说明凑齐了
            if cnt == len(p):
                ans.append(i - cnt + 1)

                # 这里注释的逻辑就是有重复多个 s[i + 1] 了可以在循环到 i + 1 时处理
                # # 为匹配下一组最少删除第一个字母
                # window[st] -= 1
                # st += 1
                # cnt -= 1

        return ans
```

思路是用 dict 记录滑窗中每个字母的数目，每次加入新的字母更新记录。如加入字母时该字母以达到最大数目则更新窗口到最大数目减一。

在这样的条件下如果 cnt 等于 p 的长度就说明找到一个 ans。

## (2) 不定长滑窗

```python
class Solution:
    def findAnagrams(self, s: str, p: str) -> List[int]:
        from collections import Counter, defaultdict

        target = Counter(p)
        window = defaultdict(int)
        ans = list()

        lp = len(p)
        ls = len(s)
        

        for r in range(ls):
            # 出窗口的位置
            if r - lp >= 0:
                window[s[r - lp]] -= 1

                if window[s[r - lp]] == 0:
                    del window[s[r - lp]]

            window[s[r]] += 1

            if window == target:
                ans.append(r - lp + 1)

        return ans
```

因为 p 串长度固定，只要维护大小为 lp 的窗口就行。

优化版

```python
class Solution:
    def findAnagrams(self, s: str, p: str) -> List[int]:
        from collections import Counter, defaultdict

        target = [0] * 26
        window = [0] * 26
        ans = []

        lp = len(p)
        ls = len(s)

        for i in range(lp):
            target[ord(p[i]) - 97] += 1
        
        if lp > ls:
            return ans

        for r in range(ls):
            # 出窗口的位置
            if r - lp >= 0:
                window[ord(s[r - lp]) - 97] -= 1

            window[ord(s[r]) - 97] += 1

            if window == target:
                ans.append(r - lp + 1)

        return ans
```



# 10. [和为 K 的子数组](https://leetcode.cn/problems/subarray-sum-equals-k/)

给你一个整数数组 `nums` 和一个整数 `k` ，请你统计并返回 *该数组中和为 `k` 的子数组的个数* 。

子数组是数组中元素的连续非空序列。

 

**示例 1：**

```
输入：nums = [1,1,1], k = 2
输出：2
```

**示例 2：**

```
输入：nums = [1,2,3], k = 3
输出：2
```

 

**提示：**

- `1 <= nums.length <= 2 * 104`
- `-1000 <= nums[i] <= 1000`
- `-107 <= k <= 107`

## (1) 前缀和 + 哈希

```python
class Solution:
    def subarraySum(self, nums: List[int], k: int) -> int:
        from collections import defaultdict
        l = len(nums)
        p = [0] * (l + 1)

        # p[x] = sum(num[0 ~ x-1])
        # sum(num[x ~ y]) = p[y + 1] - p[x]
        for i, x in enumerate(nums):
            p[i + 1] = p[i] + x

        ans = 0
        mp = defaultdict(int)
        mp[p[0]] = 1
        # 枚举右边界
        for j in range(l):
            # sum[num[i~j]] = p[j + 1] - p[i] = k
            # p[i] = p[j + 1] - k
            ans += mp[p[j + 1] - k]
            # 右边界以前的 mp 被定义保证不会找到 i，i > j
            mp[p[j + 1]] += 1

        return ans
```

前缀和 + 枚举左右边界超时，把枚举右边界，把查找符合条件的右边界优化成哈希。
