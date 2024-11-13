###### 费马小定理
我们首先回顾`费马小定理`：
> if $p$ is a prime,then for every $1\leq a < p$，有$a^{p-1} \equiv 1(\mod p)$

证明：if $i \in S= \{1,2,\dots,p-1\}$。这里所有的i就构成了一个完全剩余系对于p来说，因为这$p-1$个数$\mod p$互不相同。
显然${a·i}=S$。毕竟对于$i\neq j$，$p·i\neq p·j$，如果$p$ is prime
从而有$a^{p-1}(p-1)! = (p-1)! = 1(\mod p)$。
伪代码
```c
PRIMALITY(N)
a < N, random a
	if a^(N-1) = 1 (mod N)
		return yes
	else
		return no
	end
```
这里的时间复杂度为$\mathcal{O}(N^{3})$。
##### N is composite
**但是这只是一个测试。** 如果$N$ is not a prime(is a composite), 还是可以$a^{N-1}=1 (\mod N)$。比如说$341=11 \times 31,2^{340}\equiv1(\mod 341)$。
* 我们定义$N$ is Carmichael Number：composite number N every a < N relatively prime to N:$a^{N-1}\equiv 1(\mod N)$。比如说$561=3\times 11 \times 17$

定义一个Lemma(费马测试)
> 对于一个composite number $N$ ,$a^{N-1}\equiv1(\mod N)$，概率不高于$\frac{1}{2}$

如果我们将1~N-1的数分成两波：$a^{N-1}\not\equiv 1(\mod N)$和$b^{N-1}\not\equiv 1(\mod N)$。那么$(ab)^{N-1}=a^{N-1}b^{N-1}\not\equiv 1(\mod N)$。我画个图：
![76efcd6f1e4f2f685ea729474e750c6b.png](https://hoshinocola-1324692752.cos.ap-shanghai.myqcloud.com/202411132051002.png)

所以通不过的一定比通过的多！
```c
PRIMALITY2(N):
a1, a2, ... ak < N
	if a_i ^ (N - 1) = 1 for all 1 <= i <= k:
		return yes
	else:
		return no
```
这个概率出错的时候小于$\frac{1}{2}\times \frac{1}{2} \times \dots \times \left( \frac{1}{2} \right)=\left( \frac{1}{2} \right)^{k}$。
#### 产生随机的素数
$\pi(x)$表示num of primes $\leq$ x，那么$\pi(x)\approx \frac{x}{\ln(x)}$。也就是说$\lim_{ x \to \infty } \frac{\pi(x)}{(x/\ln(x))}=1$。显然n-bits，$\mathcal{O}(n)$。
#### Cryptography
![image.png](https://hoshinocola-1324692752.cos.ap-shanghai.myqcloud.com/202411132057257.png)
* 特定消息(x)
* e(x)加密发送
* d(e(x))解密 = x
我们希望e(x)非常巧妙。
##### RSA Cryptography
选择两个prime $p$和$q$，让$N=pq$。从而得到了任何一个与$(p-1)(q-1)$互素的$e$，再得到$d$是e对$(p-1)(q-1)$的模倒数。从而有
$$
(x^{e})^{d} \equiv x(\mod N)
$$
$x\to x^{e}$的映射就是一个，Bob拿到了$(N, e)$。就可以解码啦！
证明：$ed\equiv 1 \mod (p-1)(q-1)$，那么$ed=1+k(p-1)(q-1)$对于某个$k$。那么为了证明$x^{ed}-x=x^{1+k(p-1)(q-1)}-x\equiv0 \mod N$，从`费马小定理`我们有$x^{p-1}\equiv 1 \mod p$，$x^{q-1}\equiv{1} \mod q$。相乘：$x^{(p-1)(q-1)}\equiv1 \mod N$。

**public key 与 secret key**
* public：(N, e)
* secret key：d
#### 数字签名：
NSPK Protocol
![](https://hoshinocola-1324692752.cos.ap-shanghai.myqcloud.com/202411132111044.png)
An Attack
![image.png](https://hoshinocola-1324692752.cos.ap-shanghai.myqcloud.com/202411132112835.png)
The Fixed NSPK Protocol
![image.png](https://hoshinocola-1324692752.cos.ap-shanghai.myqcloud.com/202411132112138.png)
为什么这么写呢？~~因为我也没听懂这个协议🤡~~