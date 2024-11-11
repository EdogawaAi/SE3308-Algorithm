#### 最小生成树
![image.png](https://hoshinocola-1324692752.cos.ap-shanghai.myqcloud.com/202410311833853.png)![image.png](https://hoshinocola-1324692752.cos.ap-shanghai.myqcloud.com/202410311834254.png)

我们怎么把它变成最小生成树呢？
* Lemma1：给定任何一个环，去掉一个边，并不能让这个图不连通
> Input：$G=(V,E)$，edge weight $w_{e}$
> Output: $T=(V,E'),E'\subseteq E$，我们希望$\min \text{}\sum\limits_{v \in E'}w_{e}$

* Lemma2显然，一个有$n$个node的tree有$n-1$个edges。
* Lemma3：如果一个无向图$G=(V,E)$其中$|E|=|V|-1$那么这个图必然是树
> 如果不是树，必然有环。那么我们根据Lemma1：去掉一个边，还是连通。直到$G'=(V,E')，E'\subseteq E$，直到不是环。但是我们同时$G'$是树，$E'=E$

* Lemma4：一个图是树$\impliedby\implies$任意两个node必有一个unique path
##### Kruskal's Algorithm





###### 伪代码
> KRUSKAL(G, w)
> input: undirected graph $G=(V, E)$，with edge weight $w_{e}$
> output:A minimun spanning tree defined by the edges X

```c
for u : V :
	makeset (u)
end
// 我们把每一个u都变成一个集合

X = {}
sort edges E by weight
// [edge](const edge1, const edge2) -> bool {
//      return edge1 < edge2
// }
for all (u, v) in E :
	if find(u) != find(v) :
		add (u, v) to X
		union(u, v)
	end
end
```
##### Prim Algorithm
###### 广义Kruskal Algo
```c
X = {}
repeat until |X| = |V| - 1:
	pick a set S in V: X has no edges between S and V - S
	let e in E: S and V - S ; minimum-weight edge
	X = X + {e}
end
```

