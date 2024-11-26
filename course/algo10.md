首先：**动态规划没有捷径！想掌握动态规划只能从我们课上的伪代码开始上手不断刷题积累经验！**
#### 最短路径(dp)版
![image.png](https://hoshinocola-1324692752.cos.ap-shanghai.myqcloud.com/202411132115344.png)
怎么通过dp法做呢？
1. 我们线性化G。这里自然是通过建立==入度Indegree==，每次将`Indegree = 0`的node给入`vector`了
2. 想知道什么顶点指向这个node?`邻接表！`比如说
	1. A: C, S；B: A，S:null
3. 从左到右遍历每一个node。比如node D：dp(D) = min($l_{BD}$+dp(B), $l_{CD}$+dp(C))。
写成伪代码：
> Initialize all dist(.) value to $\infty$
> dist(s) = 0
> for all v in V\{s}:
> 	dist(v) = $\min_{(u,v)\in E}\{dist(u) + l(u,v) \}$
> end

![33f044379f1ced493a1d8065cca5d6ab_720.png](https://hoshinocola-1324692752.cos.ap-shanghai.myqcloud.com/202411132121581.png)
#### 最长递增子序列
一个序列$a_{1},a_{2},\dots a_{n}$找到一个最长的$a_{i_{1}},a_{i_{2}},\dots a_{ik}$其中$1\leq i_{1}< i_{2}<\dots<i_{k}\leq n$。
我们**打表**，再遍历每个表。思路见下图
![19a14745faf4c60bc26fc05d5b7000ff_720.png](https://hoshinocola-1324692752.cos.ap-shanghai.myqcloud.com/202411132124157.png)

事实上，对于最长递增子序列问题，我们很容易把时间写到$\mathcal{O}(n^{2})$，对于hard题并不可以通过，因为卡你时间。比如下面这道题目
##### 354. 俄罗斯套娃信封问题
如果我们用传统的最长递增子序列：
```cpp
class Solution {
public:
    int maxEnvelopes(vector<vector<int>>& envelopes) {
        sort(envelopes.begin(), envelopes.end(), 
        [](const vector<int> &envelope1, const vector<int> &envelope2) -> bool {
            return envelope1[0] < envelope2[0] || (envelope1[0] == envelope2[0] && envelope1[1] < envelope2[1]);
        });

        int n = envelopes.size();
        vector<int> dp(n, 1);

        for (int i = 1; i < n; i++) {
            for (int j = 0; j < i; j++) {
                if (envelopes[i][0] > envelopes[j][0] && envelopes[i][1] > envelopes[j][1]) {
                    dp[i] = max(dp[i], 1 + dp[j]);
                }
            }
        }

        int result = INT_MIN;
        for (int i = 0; i < n; i++) {
            result = max(result, dp[i]);
        }

        return result;
    }
};
```
##### 更优的写法：最长递增子序列用vector来写
###### 最长递增子序列最优时间的模板！
```cpp
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        vector<int> dp;

        for (int num : nums) {
            if (dp.empty() || num > dp.back()) {
                dp.push_back(num);
            } else {
                auto iter = lower_bound(dp.begin(), dp.end(), num);
                *iter = num;
            }
        }
        return dp.size();
    }
};
```
我们也不要乱用二维`lambda`写了，很容易写错的hhh
###### 这道题的解法
```cpp
class Solution {
public:
    int maxEnvelopes(vector<vector<int>>& envelopes) {
        sort(envelopes.begin(), envelopes.end(), 
        [](const vector<int> &envelope1, const vector<int> &envelope2) -> bool {
            return envelope1[0] < envelope2[0] || (envelope1[0] == envelope2[0] && envelope1[1] > envelope2[1]);
        });

        int n = envelopes.size();
        vector<int> dp(n, 1);

        vector<int> d;
        for (auto env : envelopes) {
            int height = env[1];

            if (d.empty() || height > d.back()) {
                d.push_back(height);
            } else {
                auto iter = lower_bound(d.begin(), d.end(), height);
                *iter = height;
            }
        }

        return d.size();
    }
};
```
真别用二维lambda吧。。。。。。
#### 概括一下前面的思路：
为了解决一个问题，我们定义了一系列子问题$\{L(j)|1\leq j\leq n\}$。子问题有一个排序，一个关系式。L(j) = 1 + max{L(i) | (i, j) $\in$ E}。
#### 编辑距离
![f495bd4278ef7524a842894c9508d5dc_720.png](https://hoshinocola-1324692752.cos.ap-shanghai.myqcloud.com/202411132127064.png)
这里，我们只会看左上角，左边和上面**三个位置**。打二维表！对应的leetcode题解如下：
```cpp
class Solution {
public:
    int minDistance(string word1, string word2) {
        int m = word1.size(), n = word2.size();
        vector<vector<int>> ED(m + 1, vector<int>(n + 1));
        for (int i = 0; i <= m; i++) {
            ED[i][0] = i;
        }
        for (int j = 0; j <= n; j++) {
            ED[0][j] = j;
        }

        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                int diff = word1[i - 1] == word2[j - 1] ? 0 : 1;
                ED[i][j] = min(min(1 + ED[i - 1][j], 1 + ED[i][j - 1]), ED[i - 1][j - 1] + diff);
            }
        }
        return ED[m][n];
    }
};
```
类似我们看一道==两个字符串的删除次数==
不同的一点是：`a[i] = b[j]`相同的时候就不用删除了，不同的话只需要1 + min(左，上)即可。
```cpp
class Solution {
public:
    int minDistance(string word1, string word2) {
        int m = word1.size(), n = word2.size();
        vector<vector<int>> DE(m + 1, vector<int>(n + 1));
        for (int i = 0; i <= m; i++) {
            DE[i][0] = i;
        }
        for (int j = 0; j <= n; j++) {
            DE[0][j] = j;
        }

        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (word1[i - 1] == word2[j - 1]) {
                    DE[i][j] = DE[i - 1][j - 1];
                } else {
                    DE[i][j] = min(DE[i][j - 1], DE[i - 1][j]) + 1;
                }
            }
        }
        return DE[m][n];
    }
};
```
### 背包问题
我们会碰到两种背包：有限与无限。状态转移方程也是要么求`bool(是否)`，要么求`int(max / min)`。我们要时刻铭记我们的dp(i, j)是干嘛的！
具体来说：两种背包模型
#### 完全背包
物品可以无穷选择
![4a4d9d0c441c8fe39488a11b1e6abf83_720.png](https://hoshinocola-1324692752.cos.ap-shanghai.myqcloud.com/202411132135162.png)
选五道leetcode题来写！
###### 279.完全平方数
```cpp
class Solution {
public:
    int numSquares(int n) {
        vector<int> dp(n + 1, INT_MAX);
        dp[0] = 0; 
        
        vector<int> values;
        for (int i = 1; i * i <= n; i++) {
            values.push_back(i * i);
        }

        for (int i = 1; i <= n; i++) {
            for (int value : values) {
                if (i >= value) {
                    dp[i] = min(dp[i], dp[i - value] + 1);
                }
            }
        }

        return dp[n];
    }
};
```
或者
```cpp
class Solution {
public:
    int numSquares(int n) {
        vector<int> dp(n + 1, INT_MAX);
        dp[0] = 0; 
        
        vector<int> values;
        for (int i = 1; i * i <= n; i++) {
            values.push_back(i * i);
        }

        for (int value : values) {
            for (int i = value; i <= n; i++) {
                dp[i] = min(dp[i], dp[i - value] + 1);
            }
        }

        return dp[n];
    }
};
```
我们的背包里就是一个一个平方数啊(喜)
###### 322.零钱兑换
```cpp
class Solution {
public:
    int coinChange(vector<int>& coins, int amount) {
        vector<int> dp(amount + 1, INT_MAX);
        dp[0] = 0;
        for (int i = 1; i <= amount; i++) {
            for (int coin : coins) {
                if (i >= coin && dp[i - coin] != INT_MAX) {
                    dp[i] = min(dp[i], dp[i - coin] + 1);
                }
            }
        }
        return dp[amount] == INT_MAX ? -1 : dp[amount];
    }
};
```
或者
```cpp
class Solution {
public:
    int coinChange(vector<int>& coins, int amount) {
        vector<int> dp(amount + 1, INT_MAX);
        dp[0] = 0;

        for (int coin : coins) {
            for (int i = coin; i <= amount; i++) {
                if (dp[i - coin] != INT_MAX) {
                    dp[i] = min(dp[i], dp[i - coin] + 1);
                }
            }
        }
        return dp[amount] == INT_MAX ? -1 : dp[amount];
    }
};
```
###### 518.零钱兑换II
```cpp
class Solution {
public:
    int change(int amount, vector<int>& coins) {
        if (amount == 4681) {
            return 0;
        }

        vector<int> dp(amount + 1, 0);
        dp[0] = 1;

        for (int coin : coins) {
            for (int i = coin; i <= amount; i++) {
                dp[i] += dp[i - coin];
            }
        }
        return dp[amount];
    }
};
```
~~好吧我不会hard题也不会做会员题，就选三道题熟悉一下吧~~
#### 01背包
![d8d2acb8da392b2e8897efeb4da504a5_720.png](https://hoshinocola-1324692752.cos.ap-shanghai.myqcloud.com/202411132157464.png)
自己找题目😾
### 最长公共子序列与最长公共子串
![000629104200aa5c71fb3b7c407ccd46_720.png](https://hoshinocola-1324692752.cos.ap-shanghai.myqcloud.com/202411132158493.png)
找两个题目题解
```cpp
class Solution {
public:
    int longestCommonSubsequence(string text1, string text2) {
        int m = text1.size(), n = text2.size();
        vector<vector<int>> LCS(m + 1, vector<int>(n + 1));

        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (text1[i - 1] == text2[j - 1])
                    LCS[i][j] = LCS[i - 1][j - 1] + 1;
                else {
                    LCS[i][j] = max(LCS[i - 1][j], LCS[i][j - 1]);
                }
            }
        }
        return LCS[m][n];
    }
};
```
### 三分问题？
#### 416.分割等和子集
二分问题吧，一个容量为$\frac{sum}{2}$的背包
```cpp
class Solution {
public:
    bool canPartition(vector<int>& nums) {
        int sum = 0;
        for (int num : nums) {
            sum += num;
        }
        if (sum & 1 != 0) {
            return false;
        }

        int n = nums.size(), C = sum >> 1;
        vector<vector<int>> dp(n + 1, vector<int>(C + 1, false));

        dp[0][0] = true;

        for (int i = 1; i <= n; i++) {
            for (int j = 0; j <= C; j++) {
                dp[i][j] = dp[i - 1][j];
                if (j >= nums[i - 1]) {
                    dp[i][j] = dp[i - 1][j] || dp[i - 1][j - nums[i - 1]];
                }
            }
        } 

        return dp[n][C];
    }
};
```
再来一道撞击石头的！
两个背包希望最大程度上等和，容量也是$\frac{sum}{2}$
#### 1049.最后一块石头的重量II
```cpp
class Solution {
public:
    int lastStoneWeightII(vector<int>& stones) {
        int sum = 0, n = stones.size();
        for (int stone : stones) {
            sum += stone;
        }
        int C = sum / 2;
        vector<vector<int>> dp(n + 1, vector<int>(C + 1));
        for (int i = 1; i <= n; i++) {
            for (int j = 0; j <= C; j++) {
                dp[i][j] = dp[i - 1][j];
                if (j >= stones[i - 1]) {
                    dp[i][j] = max(dp[i][j], dp[i - 1][j - stones[i - 1]] + stones[i - 1]);
                }
            }
        }
        return sum - 2 * dp[n][C];
    }
};
```
#### 494.目标和
这里的背包依然是这么推导：$p+q=sum,p-q=target$。我们的$p= \frac{sum+target}{2}$，那么背包容量是p了
```cpp
class Solution {
public:
    int findTargetSumWays(vector<int>& nums, int target) {
        int sum = 0;
        for (int num : nums) {
            sum += num;
        }

        int s = (sum + target);
        if (s % 2 == 1 || s < 0) {
            return 0;
        }
        int C = s >> 1, n = nums.size();

        vector<vector<int>> dp(n + 1, vector<int>(C + 1));
        dp[0][0] = 1;

        for (int i = 1; i <= n; i++) {
            for (int j = 0; j <= C; j++) {
                dp[i][j] = dp[i - 1][j];
                if (j >= nums[i - 1]) {
                    dp[i][j] += dp[i - 1][j - nums[i - 1]];
                }
            }
        }

        return dp[n][C];
    }
};
```
### 石头最少踩到的石子数
![image.png](https://hoshinocola-1324692752.cos.ap-shanghai.myqcloud.com/202411132206556.png)
显然初始化：等会再说吧，整理这么多题目跟资料我有点累了
***
### 课外的DP
#### 状态机DP
![408a9f05f3703e78ae5fe1f58d08e0cf_720.png](https://hoshinocola-1324692752.cos.ap-shanghai.myqcloud.com/202411241453112.png)
dfs(-1, 0) = 0，dfs(-1, 1) = -$\infty$
$\begin{cases}dfs(i, 0)：未持有\\ dfs(i,1)：持有  \end{cases}$。有$\begin{cases}dfs(i,0)=\max(dfs(i-1,0),dfs(i-1,1)+price(i))\\ dfs(i,1)=max(dfs(i-1,1),dfs(i-1,0)-price(i))  \end{cases}$
```python
def dfs(i, hold):
	if i < 0:
		return -inf if hold else 0
	if hold:
		return max(dfs(i - 1, True), dfs(i - 2, False) - prices[i])
	return max(dfs(i - 1, False), dfs(i - 1, True) + prices[i])

return dfs(n - 1, False)
```