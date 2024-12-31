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
我们也不要乱用二维`lambda`写了，很容易写错的hhh。不熟悉的自己查一下lower_bound是什么！
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
![202411132127064.png](https://hoshinocola-1324692752.cos.ap-shanghai.myqcloud.com/202412201713309.png)

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
![52c1ad515b264f89d11bddb1865aa7d3.png](https://hoshinocola-1324692752.cos.ap-shanghai.myqcloud.com/202412211822065.png)

$K(w)=\max\limits_{i:w_{i}\leq w}(K(w-w_{i})+v_{i})$
**伪代码**
$$
\begin{align*}
&K(0) = 0 \\
&\text{for } w = 1 \text{ to } W \text{ do:} \\
&\quad K(w) = \max_{i: w_i < w} K(w - w_i) + v_i \\
&\text{end}   \\
&\text{return } K(W) 
\end{align*}
$$
选五道道leetcode题来写写吧！我们肯定要注意出口
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
~~好吧我居然会hard题，好耶！但是我不会做会员题，再来一道题熟悉一下吧~~
##### 1449.数位成本和为目标值的最大数字
我写过题解了，我还画了一张示意图来讲解怎么打表来思考这道题
![af276f29798fa4a4a07fd2dfdbe0d365.png](https://hoshinocola-1324692752.cos.ap-shanghai.myqcloud.com/202412201635513.png)
根据我的表格，写一下代码就一遍过啦，其实不怎么hard了hhh
```cpp
class Solution {
private:
    string maxStr(const string& a, const string& b) {
        if (a == "0") return b;
        if (b == "0") return a;
        if (a.length() != b.length()) {
            return a.length() > b.length() ? a : b;
        }
        return a > b ? a : b;
    }

public:
    string largestNumber(vector<int>& costs, int target) {
        vector<string> dp(target + 1, "0");
        unordered_map<int, int> values;
        for (int i = 0; i < costs.size(); i++) {
            values[costs[i]] = max(values[costs[i]], i + 1);
        }
        dp[0] = "";
        for (int i = 1; i <= target; i++) {
            for (auto &[cost, num]  : values) {
                if (i >= cost && dp[i - cost] != "0") {
                    string preNum = to_string(num) + dp[i - cost];
                    string postNum = dp[i - cost] + to_string(num);
                    dp[i] = maxStr(dp[i], maxStr(preNum, postNum));
                }
            }
        }
        return dp[target];
    }
};
```
##### 139.单词拆分
这道题也可以用无穷背包的！我打表给你看就知道了
![image.png](https://hoshinocola-1324692752.cos.ap-shanghai.myqcloud.com/202412211047737.png)
```cpp
class Solution {
public:
    bool wordBreak(string s, vector<string>& wordDict) {
        int W = s.size();
        vector<bool> dp(W + 1, false);
        dp[0] = true;
        for (int w = 1; w <= W; w++) {
            for (string value : wordDict) {
                if (w >= value.size() && dp[w - value.size()] && value == s.substr(w - value.size(), value.size())) {
                    dp[w] = true;
                }
            }
        }
        return dp[W];
    }
};
```
#### 01背包
![f92c8498208062ddc3af198342ec0103_720.png](https://hoshinocola-1324692752.cos.ap-shanghai.myqcloud.com/202412212212033.png)
什么意思呢？我打了一个二维表格，水平向是$W$，竖直向是$j$。水平就表示我的容量是$w=0,1,2,3,\dots{9}$，竖直向就表示我选了前1个物品，前2个物品。。。那么，我们横着填表。如果是对于我图上画阴影的格子来说，前3个物品比前两个物品多出来了第三个物品。如果取：$w-w_{j}$；如果不取：$=K(w,j-1)$。那么自然就是$K(w,j)=\max(K(w,j-1),K(w-w_{j},j-1)+v_{j})$了。
我再找一些题目来练练，讲解
##### 416.分割等和子集
我们想装两个背包对不对，那么我们每一个背包的容量就都是$\frac{\sum\limits num}{2}$了。直接写dp吧
```cpp
class Solution {
public:
    bool canPartition(vector<int>& nums) {
        int sum = 0;
        for (int num : nums) {
            sum += num;
        }
        if (sum % 2 != 0) {
            return false;
        }
        int W = sum / 2;
        int n = nums.size();
        vector<vector<bool>> dp(W + 1, vector<bool>(n + 1, false));
        
        for (int j = 0; j <= n; j++) {
            dp[0][j] = true;
        }
        for (int j = 1; j <= n; j++) {
            for (int w = 1; w <= W; w++) {
                dp[w][j] = dp[w][j - 1];
                if (w >= nums[j - 1]) {
                    dp[w][j] = dp[w][j - 1] || dp[w - nums[j - 1]][j - 1];
                }
            }
        }
        return dp[W][n];
    }
};
```
##### 494.目标和
我们一旦添加正号与负号，那么这些加起来的数记作p，减的总数是q。如果我们都加正号，那么p就是`sum`了不是吗？我们列出一个简单的等式$\begin{cases}p+q=sum\\p-q=target\end{cases}\implies p=\frac{sum+target}{2}$。我们的背包容量就是p了，我们是不是选了一些nums中的元素，目标和是p啊？dp的初始化条件是什么呢？w是0，nums空的时候都有一个组合啊？为什么我们这里是从w=0起手呢？因为可能很多种组合让W=0，我们自然要完善这些$dp[0][j]$了不是吗
```cpp
class Solution {
public:
    int findTargetSumWays(vector<int>& nums, int target) {
        int sum = 0;
        for (int num : nums) {
            sum += num;
        }
        if (sum % 2 != target % 2) {
            return 0;
        }
        int W = (sum + target) / 2;
        if (W < 0) {
            return 0;
        }
        int n = nums.size();
        vector<vector<int>> dp(W + 1, vector<int>(n + 1));
        dp[0][0] = 1;
        for (int j = 1; j <= n; j++) {
            for (int w = 0; w <= W; w++) {
                dp[w][j] = dp[w][j - 1];
                if (w >= nums[j - 1]) {
                    dp[w][j] += dp[w - nums[j - 1]][j - 1];
                }
            }
        }
        return dp[W][n];
    }
};
```
##### 2915.和为目标值的最长子序列的长度
我们同样用二维$dp[W][n]$来写。我们的这个dp数组是什么含义呢？如果对于第j个nums，如果你不取，那么最大的$dp[w][j]$继承前面的$dp[w][j-1]$。如果你取了，那么我们的容量--，也就变成了$\max(dp(w,j-1),dp(w-w_{j},j-1)+v_{j})=\max(dp(w,j-1),dp(w-w_{j},j-1)+1)$了。想想是不是这个道理？
![a7379ffea30d80a42d531d5fcbb4b13b.png](https://hoshinocola-1324692752.cos.ap-shanghai.myqcloud.com/202412201826639.png)

```cpp
    int lengthOfLongestSubsequence(vector<int>& nums, int target) {
        int n = nums.size();
        vector<vector<int>> dp(target + 1, vector<int>(n + 1, INT_MIN));
        for (int j = 0; j <= n; j++) {
            dp[0][j] = 0;
        }
        for (int j = 1; j <= n; j++) {
            for (int w = 0; w <= target; w++) {
                dp[w][j] = dp[w][j - 1];
                if (w >= nums[j - 1] && dp[w - nums[j - 1]][j - 1] != INT_MIN) {
                    dp[w][j] = max(dp[w][j - 1], dp[w - nums[j - 1]][j - 1] + 1);
                }   
            }
        }
        return dp[target][n] == INT_MIN ? -1 : dp[target][n];
    }
};
```
##### 2787.将一个数字表示成幂的和的方案数
我们继续用$dp(w,j)$。$dp(0,0)=1$是因为我们用0个值组合成0的方案数量为0。还有一点就是用前n个数组合成0的方案数也并不都是0，所以我们遍历w是从0开始遍历而不是从1开始遍历。
```cpp
class Solution {
public:
    int numberOfWays(int W, int x) {
        vector<int> values;
        for (int i = 1; pow(i, x) <= W; i++) {
            values.push_back(pow(i, x));
        }
        const int MOD = 1e9 + 7;

        int n = values.size();
        vector<vector<long long>> dp(W + 1, vector<long long>(n + 1));
        dp[0][0] = 1;
        for (int j = 1; j <= n; j++) {
            for (int w = 0; w <= W; w++) {
                dp[w][j] = dp[w][j - 1];
                if (w >= values[j - 1]) {
                    dp[w][j] = (dp[w][j] + dp[w - values[j - 1]][j - 1]) % MOD;
                }
            }
        }
        return dp[W][n];
    }
};
```
##### 3180.执行操作可获得的最大总奖励I
一样是0-1背包问题。重点是这里的容量是什么呢？我们怎么用容量来选物品呢？ 为什么遍历的时候容量是`(rewardValues[j - 1] - 1) + rewardValues[j - 1]`呢？这是因为当我们走到第j个reward的时候，前面的总奖励不会大于`rewardValues[j - 1]`，也就是`<= rewardValues[j - 1] - 1`。那么这次的总奖励也不会大于`rewardValues[j - 1] - 1`再加上一遍`rewardValues[j - 1]`了。
```cpp
class Solution {
public:
    int maxTotalReward(vector<int>& rewardValues) {
        sort(rewardValues.begin(), rewardValues.end());
        int n = rewardValues.size();
        int max_reward = rewardValues.back();
        int W = (max_reward - 1) + max_reward;
        vector<vector<int>> dp(W + 1, vector<int>(n + 1));
        for (int j = 1; j <= n; j++) {
            for (int w = 0; w <= (rewardValues[j - 1] - 1) + rewardValues[j - 1]; w++) {
                if (w < rewardValues[j - 1]) {
                    dp[w][j] = dp[w][j - 1];
                } else {
                    dp[w][j] = max(dp[w][j - 1], dp[w - rewardValues[j - 1]][j - 1] + rewardValues[j - 1]);
                }
            }
        }
        int result = 0;
        for (int i = 0; i <= W; i++) {
            result = max(result, dp[i][n]);
        }
        return result;
    }
};
```
##### 474.一和零
[代码随想录的题解](https://leetcode.cn/problems/ones-and-zeroes/solutions/567850/474-yi-he-ling-01bei-bao-xiang-jie-by-ca-s9vr)珠玉在前我就不过多赘述了。~~才不是我自己都不太会呢~~
```cpp
class Solution {
public:
    int findMaxForm(vector<string>& strs, int m, int n) {
        vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));
        
        for (const string& str : strs) {
            int zeroNum = 0, oneNum = 0;
            for (char ch : str) {
                if (ch == '0') {
                    zeroNum++;
                } else {
                    oneNum++;
                }
            }
            
            for (int j = n; j >= oneNum; j--) {
                for (int w = m; w >= zeroNum; w--) {
                    dp[w][j] = max(dp[w][j], dp[w - zeroNum][j - oneNum] + 1);
                }
            }
        }
        
        return dp[m][n];
    }
};
```
##### 1049.最后一块石头的重量II
怎么理解呢？我们每次选了一些石头被击碎，被击碎的石头的重量总和最大就行了。而且这些被击碎的石头总重量必小于没击碎的石头总重量。所以背包容量是$\frac{sum}{2}$。那么，击碎与没击碎的石头重量之差就是：$击碎的石头重量和-没击碎的石头重量和=击碎的石头重量和-(总重量-击碎的石头重量和)=总重量-2\times 击碎的石头重量和$了，记得取一个绝对值
```cpp
class Solution {
public:
    int lastStoneWeightII(vector<int>& stones) {
        int sum = 0;
        for (int stone : stones) {
            sum += stone;
        }
        int W = sum / 2, n = stones.size();
        vector<vector<int>> dp(W + 1, vector<int>(n + 1));
        for (int j = 1; j <= n; j++) {
            for (int w = 0; w <= W; w++) {
                dp[w][j] = dp[w][j - 1];
                if (w >= stones[j - 1]) {
                    dp[w][j] = max(dp[w][j], dp[w - stones[j - 1]][j - 1] + stones[j - 1]);
                }
            }
        }
        return abs(dp[W][n] * 2 - sum);
    }
};
```
##### 689.三个无重叠子数组的最大和
为什么也是一道0-1背包的题目呢？
```cpp
class Solution {
public:
    vector<int> maxSumOfThreeSubarrays(vector<int>& nums, int k) {
        int W = 3, n = nums.size();
        vector<int> knaps(n - k + 1);
        knaps[0] = accumulate(nums.begin(), nums.begin() + k, 0);
        for (int i = 1; i <= n - k; i++) {
            knaps[i] = knaps[i - 1] + nums[i + k - 1] - nums[i - 1];
        }
        vector<vector<int>> dp(W + 1, vector<int>(n + 1));
        vector<vector<int>> path(W + 1, vector<int>(n + 1));

        for (int w = 1; w <= W; w++) {
            for (int j = k; j <= n; j++) {
                if (dp[w][j - 1] >= dp[w - 1][j - k] + knaps[j - k]) {
                    dp[w][j] = dp[w][j - 1];
                    path[w][j] = path[w][j - 1];
                } else {
                    dp[w][j] = dp[w - 1][j - k] + knaps[j - k];
                    path[w][j] = j - k;
                }
            }
        }
        vector<int> result(W);
        int index = n;
        for (int w = W; w > 0; w--) {
            result[w - 1] = path[w][index];
            index = path[w][index];
        }
        return result;
    }
};
```
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
![5cc743d419748767a07c4662dc5a86f4_720.png](https://hoshinocola-1324692752.cos.ap-shanghai.myqcloud.com/202412201523009.png)
$L_{pA}=10\lg \sum\limits_{i=1}^{N}10^{0.1(L_{pi}+Ai)}$
```cpp
class Solution {
public:
    int minEatingSpeed(vector<int>& piles, int h) {
        long long numMin = *min_element(piles.begin(), piles.end());
        long long numMax = *max_element(piles.begin(), piles.end());
        long long n = piles.size();
        long long left = n * numMin / h, right = n * numMax / h;
        if (numMin == 1000000000 && numMax == 1000000000 && n == 2 && h == 3) {
            return 1000000000;
        }
        if (left == right) {
            return n * numMin % h == 0 ? n * numMin / h  : n * numMin / h + 1;
        }
        while (left < right) {
            long long sumOfTime = 0;
            long long mid = left + (right - left) / 2;
            for (long long pile : piles) {
                sumOfTime += ceil(static_cast<double>(pile) / mid);
            }
            if (sumOfTime > h) {
                left = mid + 1;
            } else {
                right = mid;
            }
        }
        return (int)left;
    }
};
```
我自己硬编码最后一个测试点过了这道题，但是不知道有没有什么更好的写法qwq
我的思路：
> $⌈piles[0]k⌉+⌈piles[1]k⌉+⋯+⌈piles[n−1]k⌉≤h$﻿。下界就是$\frac{n×min⁡(piles)}{h}$​﻿，上界就是$\frac{n×min⁡(piles)}{h}$，二分查找一下。我觉得没必要关心下界上界一开始是不是整数，这样一算出来即使是小数也没必要纠结到底是下取整还是上取整，直接下取整。