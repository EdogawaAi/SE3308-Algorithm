##### 距离
![image.png](https://hoshinocola-1324692752.cos.ap-shanghai.myqcloud.com/202410241901113.png)
没什么好说的(
#### BFS算法伪代码
借助数据结构**Queue**
```c
BFS(G, s)
# input : Graph G = (V, E), s in V
# output : dist[u]
for all u in V:
	dist(u) = inf
end

dist[s] = 0
Q.push(s)
while Q is not empty:
	u = Q.pop()
	for all (u, v) in E:
		if dist[v] = inf:
			Q.push(v)
			dist[v] = dist[u] + 1
		end
	end
end
```
##### 一个Lemma
对于任意的$d=0, 1,2,\dots$一定存在这个时刻：
1. all nodes at distance ≤ d from s have their distances correctly set;  
2. all other nodes have their distances set to ∞; and  
3. the queue contains exactly the nodes at distance d.  

什么时刻呢？
当Queue含有d-1,d,d,...d然后pop的时候
<img src="https://hoshinocola-1324692752.cos.ap-shanghai.myqcloud.com/202410241922606.png" alt="image.png" style="zoom: 33%;" />

**当每一个edge有权值的时候：**
#### Dijkstra算法
首先，$\mathscr{l}_{e}$是一个映射：$\mathcal{E}\to \mathbb{N}^{+} / \mathbb{R}^{+}$。~~这就是权值怎么来的~~
对于**BFS**，$\mathscr{l}_{e}=1$，所以替代时间复杂度公式
$$
\mathcal{O}\left( |V|+\sum\limits_{e \in E}\mathscr{l}_{e} \right)=\mathcal{O}(|V|+|E|)
$$
运用==闹钟==法，我们确认一下最短路径：
<img src="https://hoshinocola-1324692752.cos.ap-shanghai.myqcloud.com/202410241936233.png" alt="image.png" style="zoom:40%;" />
从S->B，我们成功从200调整到150
下面是手动求解步骤：
##### 手动求解
我们在运筹学里也学过
![image.png](https://hoshinocola-1324692752.cos.ap-shanghai.myqcloud.com/202410241937878.png)
![image.png](https://hoshinocola-1324692752.cos.ap-shanghai.myqcloud.com/202410241938582.png)
![image.png](https://hoshinocola-1324692752.cos.ap-shanghai.myqcloud.com/202410241938443.png)
![image.png](https://hoshinocola-1324692752.cos.ap-shanghai.myqcloud.com/202410241938265.png)
![image.png](https://hoshinocola-1324692752.cos.ap-shanghai.myqcloud.com/202410241939386.png)
##### 伪代码
```C
DIJKSTRA(G, l, s)
# input : G = (V, E), le, v in V
# output : dist(u)
for all u in V:
	dist(u) = inf, prev(u) = nil
end

dist(s) = 0
H = priority_queue(s)
while H is not empty:
	u = deletemin(H)
	for all edge(u, v) in E:
		if dist(v) > dist(u) + l(u, v):
			dist(v) = dist(u) + l(u, v)
			decreasekey(v) // 这里我们一般push进新的node
		end
	end
end
```
###### 一道leetcode题

```c++
class Solution {
public:
    int minimumEffortPath(vector<vector<int>>& heights) {
        int m = heights.size(), n = heights[0].size();

        vector<vector<int>> dirs = {{1, 0}, {0, 1}, {-1, 0}, {0, -1}};
        vector<vector<int>> dist(m, vector<int>(n, INT_MAX));

        priority_queue<pair<int, pair<int, int>>, vector<pair<int, pair<int, int>>>, greater<pair<int, pair<int, int>>>> H;
        dist[0][0] = 0;
        H.push({0, {0, 0}});

        while (!H.empty()) {
            auto [effort, pos] = H.top();
            H.pop();
            int x = pos.first, y = pos.second;
  
            if (x == m - 1 && y == n - 1) {
                return effort;
            }

            for (auto &dir : dirs) {
                int nx = x + dir[0], ny = y + dir[1];
                if (nx >= 0 && nx < m && ny >= 0 && ny < n) {
                    int new_effort = max(effort, abs(heights[x][y] - heights[nx][ny]));
                    if (dist[nx][ny] > new_effort) {
                        dist[nx][ny] = new_effort;
                        H.push({new_effort, {nx, ny}});
                    }
                }
            }
        }
        return -1;
    }
};
```

为什么堆是最好的？

| implementation | deletemin                                            | insert/decreasekey                                | \|V\| $\times$ deletemin+(\|V\| + \|E\|) $\times$ insert |
| -------------- | ---------------------------------------------------- | ------------------------------------------------- | -------------------------------------------------------- |
| Array          | $\mathcal{O}(\|V\|)$                                 | $\mathcal{O}(1)$                                  | $\mathcal{O}(V^{2})$                                     |
| Binary Heap    | $\mathcal{O}(\log V)$                                | $\mathcal{O}(\log V)$                             | $\mathcal{O}((V + E)\log V)$                             |
| d-ary Heap     | $\mathcal{O}\left( \frac{d \log V}{d\log d} \right)$ | $\mathcal{O}\left( \frac{\log V}{\log d} \right)$ | $\frac{\mathcal{O}(d(V+E)\log V)}{\log D}$               |
| Fibonacci heap | $\mathcal{O}(\log V)$                                | $\mathcal{O}(1)$                                  | $\mathcal{O}(V\log V+E)$                                 |
##### 证明Dijkstra的正确性？
