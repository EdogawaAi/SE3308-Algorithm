###### 前言
树状数组与线段树为什么会有这个东西？如果我们有一个数组{1, 9, 1, 9, 8, 1, 0}，我们有一系列查询{{0, 3}, {1, 5}, {2, 4}}，如果我们查询区间内总和，那么很简单用前缀和：prefixSum={0, 1, 10, 11, 20, 28, 29, 29}即可，复杂度是$\mathcal{O}(n)$。但是如果我们频繁更新单点呢？那么复杂度一样是$\mathcal{O}(n^{2})$，怎么打下来复杂度呢？
### Lazy线段树
理论以后再补，先用模板训练一下
#### 树状数组
这里我给出我的线段树模板。这玩意就是不直观理解
```cpp
class lazySegmentTree {
private:
    vector<int> tree, lazy;
    int n;

    void pushDown(int node, int start, int end) {
        if (lazy[node] != 0) {
            tree[node] = max(tree[node], lazy[node]);
            int leftChild = 2 * node + 1;
            int rightChild = 2 * node + 2;

            if (start != end) {
                lazy[leftChild] = max(lazy[node], lazy[leftChild]);
                lazy[rightChild] = max(lazy[node], lazy[rightChild]);
            }

            lazy[node] = 0;
        }
    }

    void updateRange(int node, int start, int end, int left, int right, int val) {
        pushDown(node, start, end);

        if (start > right || end < left) {
            return;
        }

        int mid = start + (end - start) / 2;
        int leftChild = 2 * node + 1;
        int rightChild = 2 * node + 2;

        if (left <= start && right >= end) {
            tree[node] = max(tree[node], val);
            if (start != end) {
                lazy[leftChild] = max(val, lazy[leftChild]);
                lazy[rightChild] = max(val, lazy[rightChild]);
            }
            return;
        }

        updateRange(leftChild, start, mid, left, right, val);
        updateRange(rightChild, mid + 1, end, left, right, val);
        tree[node] = max(tree[leftChild], tree[rightChild]);
    }    

    int queryRange(int node, int start, int end, int left, int right) {
        pushDown(node, start, end);

        if (start > right || end < left) {
            return 0;
        }

        if (left <= start && right >= end) {
            return tree[node];
        }

        int mid = start + (end - start) / 2;
        int leftChild = 2 * node + 1;
        int rightChild = 2 * node + 2;

        int leftMax = queryRange(leftChild, start, mid, left, right);
        int rightMax = queryRange(rightChild, mid + 1, end, left, right);

        return max(leftMax, rightMax);
    }


public:
    lazySegmentTree(int size) : n(size) {
        tree.resize(4 * n);
        lazy.resize(4 * n);
    }

    void update(int left, int right, int val) {
        updateRange(0, 0, n - 1, left, right, val);
    }

    int query(int left, int right) {
        return queryRange(0, 0, n - 1, left, right);
    }
};
```
为什么我不讲一般的单点更新线段树呢？因为我们用lazy线段树，那么查询单点就是`query(i, i)`了。我拿两道题目练练手
##### 2381.字母移位II
这里我们就是频繁更改一个区间的值，用差分数组也能做，但是上来更容易想到lazy线段树。
```cpp
class lazySegmentTree {
private:
    vector<long long> tree, lazy;
    int n;

    void pushDown(int node, int start, int end) {
        if (lazy[node] != 0) {
            int mid = start + (end - start) / 2;
            int leftChild = 2 * node + 1;
            int rightChild = 2 * node + 2;

            tree[leftChild] += lazy[node] * (mid - start + 1);
            lazy[leftChild] += lazy[node];

            tree[rightChild] += lazy[node] * (end - mid);
            lazy[rightChild] += lazy[node];

            lazy[node] = 0;
        }
    }

    void updateRange(int node, int start, int end, int left, int right, int val) {
        if (left > end || right < start) {
            return;
        }

        if (left <= start && right >= end) {
            tree[node] += val * (end - start + 1);
            lazy[node] += val;
            return;
        }

        pushDown(node, start, end);

        int mid = start + (end - start) / 2;
        int leftChild = 2 * node + 1;
        int rightChild = 2 * node + 2;

        updateRange(leftChild, start, mid, left, right, val);
        updateRange(rightChild, mid + 1, end, left, right, val);

        tree[node] = tree[leftChild] + tree[rightChild];
    }

    int queryRange(int node, int start, int end, int left, int right) {
        if (left > end || right < start) {
            return 0;
        }
        if (left <= start && end <= right) {
            return tree[node];
        }

        pushDown(node, start, end);

        int mid = start + (end - start) / 2;
        int leftChild = 2 * node + 1;
        int rightChild = 2 * node + 2;

        int leftSum =  queryRange(leftChild, start, mid, left, right);
        int rightSum = queryRange(rightChild, mid + 1, end, left, right);

        return leftSum + rightSum;
    }

public:
    lazySegmentTree(int size) : n(size) {
        tree.resize(4 * n);
        lazy.resize(4 * n);
    }    

    void update(int left, int right, int val) {
        updateRange(0, 0, n - 1, left, right, val);
    }

    int query(int index) {
        return queryRange(0, 0, n - 1, index, index);
    }
};

class Solution {
public:
    string shiftingLetters(string s, vector<vector<int>>& shifts) {
        int n = s.size();
        lazySegmentTree lazySt(n);
        for (auto shift : shifts) {
            int dir = shift[2];
            int start = shift[0], end = shift[1];
            int val = dir == 1 ? 1 : -1;
            lazySt.update(start, end, val);
        }

        string result = s;
        for (int i = 0; i < n; i++) {
            int shift = lazySt.query(i);
            shift %= 26;
            if (shift < 0) {
                shift += 26;
            }
            result[i] = 'a' + (result[i] - 'a' + shift) % 26;
        }
        return result;
    }
};
```
我们反复更新值即可，想想是不是很自然的思路？
##### 218.都市天际线
上一道题目我们只是不断更新区间值查询某一个点的值。这一次我们不断更新查询最大值。我们用树状数组先做一下。
###### 树状数组的做法
```cpp
class lazySegmentTree {
private:
    vector<int> tree, lazy;
    int n;

    void pushDown(int node, int start, int end) {
        if (lazy[node] != 0) {
            tree[node] = max(tree[node], lazy[node]);
            int leftChild = 2 * node + 1;
            int rightChild = 2 * node + 2;

            if (start != end) {
                lazy[leftChild] = max(lazy[node], lazy[leftChild]);
                lazy[rightChild] = max(lazy[node], lazy[rightChild]);
            }

            lazy[node] = 0;
        }
    }

    void updateRange(int node, int start, int end, int left, int right, int val) {
        pushDown(node, start, end);

        if (start > right || end < left) {
            return;
        }

        int mid = start + (end - start) / 2;
        int leftChild = 2 * node + 1;
        int rightChild = 2 * node + 2;

        if (left <= start && right >= end) {
            tree[node] = max(tree[node], val);
            if (start != end) {
                lazy[leftChild] = max(val, lazy[leftChild]);
                lazy[rightChild] = max(val, lazy[rightChild]);
            }
            return;
        }

        updateRange(leftChild, start, mid, left, right, val);
        updateRange(rightChild, mid + 1, end, left, right, val);
        tree[node] = max(tree[leftChild], tree[rightChild]);
    }    

    int queryRange(int node, int start, int end, int left, int right) {
        pushDown(node, start, end);

        if (start > right || end < left) {
            return 0;
        }

        if (left <= start && right >= end) {
            return tree[node];
        }

        int mid = start + (end - start) / 2;
        int leftChild = 2 * node + 1;
        int rightChild = 2 * node + 2;

        int leftMax = queryRange(leftChild, start, mid, left, right);
        int rightMax = queryRange(rightChild, mid + 1, end, left, right);

        return max(leftMax, rightMax);
    }


public:
    lazySegmentTree(int size) : n(size) {
        tree.resize(4 * n);
        lazy.resize(4 * n);
    }

    void update(int left, int right, int val) {
        updateRange(0, 0, n - 1, left, right, val);
    }

    int query(int left, int right) {
        return queryRange(0, 0, n - 1, left, right);
    }
};

class Solution {
public:
    vector<vector<int>> getSkyline(vector<vector<int>>& buildings) {
        if (buildings.empty()) {
            return {};
        }

        long long maxIndex = 0;
        for (auto building : buildings) {
            maxIndex = max(maxIndex, static_cast<long long>(building[1]));
        }

        if (4LL * (maxIndex + 1) > INT_MAX) {
            maxIndex = INT_MAX / 4 - 1;
        }

        lazySegmentTree st(maxIndex + 1);

        for (auto building : buildings) {
            int left = building[0], right = building[1] - 1;
            int height = building[2];
            st.update(left, right, height);
        }

        vector<vector<int>> skyline;
        int prevHeight = 0;
        for (int i = 0; i <= maxIndex; i++) {
            int currHeight = st.query(i, i);
            if (currHeight != prevHeight) {
                skyline.push_back({i, currHeight});
                prevHeight = currHeight;
            }
        }

        return skyline;
    }
};
```
如果真进行测试，会发现很容易爆内存。就是因为我们太离散化这些building的index了，如果我们让它更连续一些呢？把{2, 3, 5, 7, 10}。。。全转化为一个index，{0, 1, 2, 3, 4, 5。。。}就ok了
```cpp
class lazySegmentTree {
private:
    vector<int> tree, lazy;
    int n;

    void pushDown(int node, int start, int end) {
        if (lazy[node] != 0) {
            tree[node] = max(tree[node], lazy[node]);
            int leftChild = 2 * node + 1;
            int rightChild = 2 * node + 2;

            if (start != end) {
                lazy[leftChild] = max(lazy[node], lazy[leftChild]);
                lazy[rightChild] = max(lazy[node], lazy[rightChild]);
            }

            lazy[node] = 0;
        }
    }

    void updateRange(int node, int start, int end, int left, int right, int val) {
        pushDown(node, start, end);

        if (start > right || end < left) {
            return;
        }

        int mid = start + (end - start) / 2;
        int leftChild = 2 * node + 1;
        int rightChild = 2 * node + 2;

        if (left <= start && right >= end) {
            tree[node] = max(tree[node], val);
            if (start != end) {
                lazy[leftChild] = max(val, lazy[leftChild]);
                lazy[rightChild] = max(val, lazy[rightChild]);
            }
            return;
        }

        updateRange(leftChild, start, mid, left, right, val);
        updateRange(rightChild, mid + 1, end, left, right, val);
        tree[node] = max(tree[leftChild], tree[rightChild]);
    }    

    int queryRange(int node, int start, int end, int left, int right) {
        pushDown(node, start, end);

        if (start > right || end < left) {
            return 0;
        }

        if (left <= start && right >= end) {
            return tree[node];
        }

        int mid = start + (end - start) / 2;
        int leftChild = 2 * node + 1;
        int rightChild = 2 * node + 2;

        int leftMax = queryRange(leftChild, start, mid, left, right);
        int rightMax = queryRange(rightChild, mid + 1, end, left, right);

        return max(leftMax, rightMax);
    }


public:
    lazySegmentTree(int size) : n(size) {
        tree.resize(4 * n);
        lazy.resize(4 * n);
    }

    void update(int left, int right, int val) {
        updateRange(0, 0, n - 1, left, right, val);
    }

    int query(int left, int right) {
        return queryRange(0, 0, n - 1, left, right);
    }
};

class Solution {
public:
    vector<vector<int>> getSkyline(vector<vector<int>>& buildings) {
        if (buildings.empty()) {
            return {};
        }

        set<int> uniquePoints;
        for (auto building : buildings) {
            uniquePoints.insert(building[0]);
            uniquePoints.insert(building[1]);
        }

        vector<int> sortedPoints(uniquePoints.begin(), uniquePoints.end());
        unordered_map<int, int> pointToIndex;
        for (int i = 0; i < sortedPoints.size(); i++) {
            pointToIndex[sortedPoints[i]] = i;
        }

        lazySegmentTree st(sortedPoints.size());

        for (auto building : buildings) {
            int left = pointToIndex[building[0]];
            int right = pointToIndex[building[1]] - 1;
            int height = building[2];
            st.update(left, right, height);
        }

        vector<vector<int>> skyline;
        int prevHeight = 0;
        for (int i = 0; i < sortedPoints.size(); i++) {
            int currHeight = st.query(i, i);
            if (currHeight != prevHeight) {
                skyline.push_back({sortedPoints[i], currHeight});
                prevHeight = currHeight;
            }
        }
        return skyline;
    }
};
```
#### 动态开点线段树
我们可以用一个TreeNode来建立线段树，这样就再也不用担心爆内存辣！
动态开点线段树强大之处是，我们既可以用update进行更新单点，也可以用query查询区间最大值。不用单点更新update和查询区间和，更加灵活
附上我的模板
```cpp
class DynamicSegmentTree {
private:
    struct TreeNode {
        int left = 0, right = 0;  // 区间范围
        int val = 0;              // 当前节点值（最大值）
        int lazy = 0;             // 懒标记
        TreeNode *leftChild = nullptr;
        TreeNode *rightChild = nullptr;

        TreeNode(int l, int r) : left(l), right(r) {}
    };

    TreeNode *root;

    void pushDown(TreeNode *node) {
        if (!node || node->lazy == 0) {
            return;
        }

        int mid = node->left + (node->right - node->left) / 2;

        if (!node->leftChild) {
            node->leftChild = new TreeNode(node->left, mid);
        }

        if (!node->rightChild) {
            node->rightChild = new TreeNode(mid + 1, node->right);
        }

        node->leftChild->val = max(node->leftChild->val, node->lazy);
        node->leftChild->lazy = max(node->leftChild->lazy, node->lazy);
        node->rightChild->val = max(node->rightChild->val, node->lazy);
        node->rightChild->lazy = max(node->rightChild->lazy, node->lazy);

        node->lazy = 0;
    }

    void updateRange(TreeNode *node, int l, int r, int val) {
        if (!node || node->left > r || node->right < l) {
            return;
        }

        if (l <= node->left && node->right <= r) {
            node->val = max(node->val, val);
            node->lazy = max(node->lazy, val);
            return;
        }

        pushDown(node);  // 下传懒标记

        int mid = node->left + (node->right - node->left) / 2;

        // 动态创建子节点
        if (!node->leftChild) {
            node->leftChild = new TreeNode(node->left, mid);
        }
        if (!node->rightChild) {
            node->rightChild = new TreeNode(mid + 1, node->right);
        }

        // 递归更新左右子节点
        updateRange(node->leftChild, l, r, val);
        updateRange(node->rightChild, l, r, val);

        // 合并子节点的值
        node->val = max(node->leftChild->val, node->rightChild->val);
    }

    // 区间查询
    int queryRange(TreeNode *node, int l, int r) {
        if (!node || node->left > r || node->right < l) {
            return 0;  // 当前节点区间与目标区间无交集
        }

        if (l <= node->left && node->right <= r) {
            return node->val;  // 当前节点区间完全包含在目标区间内
        }

        pushDown(node);  // 下传懒标记

        int mid = node->left + (node->right - node->left) / 2;

        // 动态创建子节点
        if (!node->leftChild) {
            node->leftChild = new TreeNode(node->left, mid);
        }
        if (!node->rightChild) {
            node->rightChild = new TreeNode(mid + 1, node->right);
        }

        // 递归查询左右子节点
        return max(queryRange(node->leftChild, l, r),
                   queryRange(node->rightChild, l, r));
    }

public:
    DynamicSegmentTree(int l, int r) {
        root = new TreeNode(l, r);
    }

    void update(int l, int r, int val) {
        updateRange(root, l, r, val);
    }

    int query(int l, int r) {
        return queryRange(root, l, r);
    }
};
```
同样，我们做一下这几道题目。
##### 218.都市天际线
```cpp
class DynamicSegmentTree {
private:
    struct TreeNode {
        int left = 0, right = 0;  // 区间范围
        int val = 0;              // 当前节点值（最大值）
        int lazy = 0;             // 懒标记
        TreeNode *leftChild = nullptr;
        TreeNode *rightChild = nullptr;

        TreeNode(int l, int r) : left(l), right(r) {}
    };

    TreeNode *ro
    void pushDown(TreeNode *node) {
        if (!node || node->lazy == 0) {
            return;
        }

        int mid = node->left + (node->right - node->left) / 2;

        if (!node->leftChild) {
            node->leftChild = new TreeNode(node->left, mid);
        }

        if (!node->rightChild) {
            node->rightChild = new TreeNode(mid + 1, node->right);
        }

        node->leftChild->val = max(node->leftChild->val, node->lazy);
        node->leftChild->lazy = max(node->leftChild->lazy, node->lazy);
        node->rightChild->val = max(node->rightChild->val, node->lazy);
        node->rightChild->lazy = max(node->rightChild->lazy, node->lazy);

        node->lazy = 0;
    }

    void updateRange(TreeNode *node, int l, int r, int val) {
        if (!node || node->left > r || node->right < l) {
            return;
        }

        if (l <= node->left && node->right <= r) {
            node->val = max(node->val, val);
            node->lazy = max(node->lazy, val);
            return;
        }

        pushDown(node);

        int mid = node->left + (node->right - node->left) / 2;

        if (!node->leftChild) {
            node->leftChild = new TreeNode(node->left, mid);
        }
        if (!node->rightChild) {
            node->rightChild = new TreeNode(mid + 1, node->right);
        }

        updateRange(node->leftChild, l, r, val);
        updateRange(node->rightChild, l, r, val);

        node->val = max(node->leftChild->val, node->rightChild->val);
    }

    int queryRange(TreeNode *node, int l, int r) {
        if (!node || node->left > r || node->right < l) {
            return 0;
        }

        if (l <= node->left && node->right <= r) {
            return node->val;
        }

        pushDown(node);

        int mid = node->left + (node->right - node->left) / 2;

        if (!node->leftChild) {
            node->leftChild = new TreeNode(node->left, mid);
        }
        if (!node->rightChild) {
            node->rightChild = new TreeNode(mid + 1, node->right);
        }

        return max(queryRange(node->leftChild, l, r),
                   queryRange(node->rightChild, l, r));
    }

public:
    DynamicSegmentTree(int l, int r) {
        root = new TreeNode(l, r);
    }

    void update(int l, int r, int val) {
        updateRange(root, l, r, val);
    }

    int query(int l, int r) {
        return queryRange(root, l, r);
    }
};

class Solution {
public:
    vector<vector<int>> getSkyline(vector<vector<int>>& buildings) {
        set<int> uniquePoints;
        for (auto& building : buildings) {
            uniquePoints.insert(building[0]);
            uniquePoints.insert(building[1]);
        }

        vector<int> sortedPoints(uniquePoints.begin(), uniquePoints.end());
        unordered_map<int, int> pointToIndex;
        for (int i = 0; i < sortedPoints.size(); i++) {
            pointToIndex[sortedPoints[i]] = i;
        }

        DynamicSegmentTree segTree(0, sortedPoints.size() - 1);

        for (auto& building : buildings) {
            int l = pointToIndex[building[0]];
            int r = pointToIndex[building[1]] - 1;
            int h = building[2];
            segTree.update(l, r, h);
        }

        vector<vector<int>> skyline;
        int prevHeight = 0;
        for (int i = 0; i < sortedPoints.size(); i++) {
            int currHeight = segTree.query(i, i);
            if (currHeight != prevHeight) {
                skyline.push_back({sortedPoints[i], currHeight});
                prevHeight = currHeight;
            }
        }

        return skyline;
    }
};
```
##### 731.我的日程安排表II
这里就是更新单点val，查询区间最大值了
```cpp
class DynamicSegmentTree {
private:
    struct TreeNode {
        int left = 0, right = 0;
        int val = 0;
        int lazy = 0;
        TreeNode *leftChild = nullptr;
        TreeNode *rightChild = nullptr;

        TreeNode(int l, int r) : left(l), right(r) {}
    };

    TreeNode *root;

    void pushDown(TreeNode *node) {
        if (!node || node->lazy == 0) {
            return;
        }

        int mid = node->left + (node->right - node->left) / 2;

        if (node->leftChild == nullptr) {
            node->leftChild = new TreeNode(node->left, mid);
        }
        if (node->rightChild == nullptr) {
            node->rightChild = new TreeNode(mid + 1, node->right);
        }

        node->leftChild->val += node->lazy;
        node->leftChild->lazy += node->lazy;
        node->rightChild->val += node->lazy;
        node->rightChild->lazy += node->lazy;

        node->lazy = 0;
    }

    // 区间更新
    void updateRange(TreeNode *node, int l, int r, int val) {
        if (!node || node->left > r || node->right < l) {
            return;
        }

        if (l <= node->left && node->right <= r) {
            node->val += val;
            node->lazy += val;
            return;
        }

        pushDown(node);

        int mid = node->left + (node->right - node->left) / 2;

        if (!node->leftChild) {
            node->leftChild = new TreeNode(node->left, mid);
        }
        if (!node->rightChild) {
            node->rightChild = new TreeNode(mid + 1, node->right);
        }

        updateRange(node->leftChild, l, r, val);
        updateRange(node->rightChild, l, r, val);

        node->val = max(node->leftChild->val, node->rightChild->val);
    }

    // 区间查询
    int queryRange(TreeNode *node, int l, int r) {
        if (!node || node->left > r || node->right < l) {
            return 0;  // 当前节点区间与目标区间无交集
        }

        if (l <= node->left && node->right <= r) {
            return node->val;  // 当前节点区间完全包含在目标区间内
        }

        pushDown(node);  // 下传懒标记

        int mid = node->left + (node->right - node->left) / 2;

        // 动态创建子节点
        if (!node->leftChild) {
            node->leftChild = new TreeNode(node->left, mid);
        }
        if (!node->rightChild) {
            node->rightChild = new TreeNode(mid + 1, node->right);
        }

        // 递归查询左右子节点
        return max(queryRange(node->leftChild, l, r),
                   queryRange(node->rightChild, l, r));
    }

public:
    DynamicSegmentTree(int l, int r) {
        root = new TreeNode(l, r);
    }

    void update(int l, int r, int val) {
        updateRange(root, l, r, val);
    }

    int query(int l, int r) {
        return queryRange(root, l, r);
    }
};

class MyCalendarTwo {
private:
    DynamicSegmentTree *st;

public:
    MyCalendarTwo() {
        st = new DynamicSegmentTree(0, 1e9);
    }

    bool book(int start, int end) {
        // 检查是否有双重预订
        if (st->query(start, end - 1) >= 2) {
            return false;
        }

        st->update(start, end - 1, 1);
        return true;
    }
};
```
##### 732.我的日程安排表III
与731题一样的题目，不同之处在于我们查询是整个区间1~1e9，而不是每一次的start与end
```cpp
class dynamicSegmentTree {
private:
    struct TreeNode {
        int left = 0, right = 0;
        int val = 0;
        int lazy = 0;
        TreeNode *leftChild = nullptr;
        TreeNode *rightChild = nullptr;

        TreeNode(int l, int r) : left(l), right(r) {}
    };

    TreeNode *root;

    void pushDown(TreeNode *node) {
        if (!node || node->lazy == 0) {
            return;
        }

        int mid = node->left + (node->right - node->left) / 2;

        if (node->leftChild == nullptr) {
            node->leftChild = new TreeNode(node->left, mid);
        }
        if (node->rightChild == nullptr) {
            node->rightChild = new TreeNode(mid + 1, node->right);
        }

        node->leftChild->val += node->lazy;
        node->leftChild->lazy += node->lazy;
        node->rightChild->val += node->lazy;
        node->rightChild->lazy += node->lazy;

        node->lazy = 0;
    }

    void updateRange(TreeNode *node, int l, int r, int val) {
        if (!node || node->left > r || node->right < l) {
            return;
        }

        if (l <= node->left && r >= node->right) {
            node->val += val;
            node->lazy += val;
            return;
        }

        pushDown(node);

        int mid = node->left + (node->right - node->left) / 2;

        if (node->leftChild == nullptr) {
            node->leftChild = new TreeNode(node->left, mid);
        }
        if (node->rightChild == nullptr) {
            node->rightChild = new TreeNode(mid + 1, node->right);
        }

        updateRange(node->leftChild, l, r, val);
        updateRange(node->rightChild, l, r, val);

        node->val = max(node->leftChild->val, node->rightChild->val);
    }

    int queryRange(TreeNode *node, int l, int r) {
        if (!node || node->left > r || node->right < l) {
            return 0;
        }

        if (l <= node->left && r >= node->right) {
            return node->val;
        }

        pushDown(node);

        int mid = node->left + (node->right - node->left) / 2;

        if (node->leftChild == nullptr) {
            node->leftChild = new TreeNode(node->left, mid);
        }
        if (node->rightChild == nullptr) {
            node->rightChild = new TreeNode(mid + 1, node->right);
        }

        return max(queryRange(node->leftChild, l, r), queryRange(node->rightChild, l, r));
    }


public:
    dynamicSegmentTree(int l, int r) {
        root = new TreeNode(l, r);
    }

    void update(int l, int r, int val) {
        updateRange(root, l, r, val);
    }

    int query(int l, int r) {
        return queryRange(root, l, r);
    }
};

class MyCalendarThree {
private:
    dynamicSegmentTree *st;

public:
    MyCalendarThree() {
        st = new dynamicSegmentTree(0, 1e9);
    }
    
    int book(int start, int end) {
        st->update(start, end - 1, 1);
        return st->query(0, 1e9);
    }
};
```