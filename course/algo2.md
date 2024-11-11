我们研究
* 给定$\mathbb{N}$，把它分解因子
* 判断素数
我们表示一个数，不同进制复杂性一般一样。因为$b$进制下有$\log_{b}(N+1)$bits，而$a$进制下，$\log _a(N+1)=\log_{b}(N+1)\times \frac{\log a}{\log b}$。==1进制下可能考虑多项式问题==
##### MULTIPLY
对于$n\text{ }bits$的$x,y$来说，肯定是复杂度为$\mathcal{O}(n^{2})$。伪代码如下：
```c
MULTIPLY(x, y):
if y = 0:
	ret 0
z = MULTIPLY(x, ⌊y / 2⌋)
	if y is even:
		ret 2 * z
	else: 
		ret x + 2 * z
end
```
$$
x·y=\begin{cases}
2\left( x·\left[ \frac{y}{2} \right]\right) , \text{if y is even} \\
x + 2 \left( x·\left[ \frac{y}{2} \right] \right),\text{if y is odd}
\end{cases}
$$
##### DIVIDE
伪代码：
```c
DIVIDE(x, y):
if x = 0:
	ret (0, 0)
(q, r) = DIVIDE(x / 2, y)
q = 2 * q, r = 2 * r
	if x is odd:
		r = r + 1
	if r >= y:
		r = r - y, q = q + 1
	return (q, r)
end
```
显然：递归n次，每次时间复杂度为$\mathcal{O}(n)$，~~由于比大小~~，时间复杂度$\mathcal{O}(n^{2})$
##### MODULAR
模运算几个不成文的规则：
举个栗子：$2^{345}=(2^{5})^{69}=32^{69}=1^{69}=1 \text{ mod }31$
如果$x,y$是从$0\sim N-1$，运行时间是$\mathcal{O}(n),\text{此时}n=\log N$
###### 计算一下Modular Exponentiation
```c
MODEXP(x, y, N)
	if y = 0:
		return 1
	z = MODEXP(x, y / 2, N)
	if y is even:
		return z² mod N
	else:
		return x * z² mod N
end
```
$$
x^{y}\text{ mod }N=\begin{cases}
x^{y/2}\text{ mod }N, \text{if y is even} \\
x·(x^{y/2})^{2}\text{  mod }N,\text{if y is odd}
\end{cases}
$$
显然，n次，每次复杂度为$\mathcal{O}(n^{2})$，总时间复杂度$\mathcal{O}(n^{3})$
#### Euclid Algorithm
怎么计算==gcd(x, y)==呢？
> 定律：$x,y$ is positive, $x\geq y$：then $gcd(x,y)=gcd((x\text{ mod }y),y)$

证明：
$$
\begin{align}
&gcd(x,y)=t\implies gcd(x-y,y)=t。 \\
& \text{let }gcd(x-y,y)=t\implies重复上述操作
\end{align}
$$
伪代码：
```C
EUCLID(x,y)
	if y = 0
		return x
	else:
		return EUCLID(y, x mod y)
```
我们还有一个定理：
> if $a\geq b \geq 0,\text{then }a \text{ mod }b< \frac{a}{2}$

证明：
$$
\begin{align}
if\text{ }&b\leq \frac{a}{2}:a\text{ mod }b<b\leq \frac{a}{2} \\
if \text{ }&b> \frac{a}{2}:a \text{ mod }b=(a-b)\text{ mod }b < \frac{a}{2}
\end{align}
$$
再来一个定理：
> if $d$ 能被$x,y$整除，$d=ax+by$，其中$a,b\in \mathbb{N}$。那么，$d=gcd(x,y)$

为什么？
$$
\begin{align}
&gcd(x,y)=t，x,y|t\implies d=ax+by|t \\
&\implies d\geq t 。又有x,y|d\implies d\leq gcd=t。 \\
&\implies d = t = gcd(x,y)
\end{align}
$$
##### Euchlid's Algorithm extension
```c
EXTENDED-EUCLID(x, y)
# x >=y >= 0
	if y = 0
		return (1, 0, x)
	(x', y', d') = EXTENDED-EUCLID(y, x mod y)
	return (y', x' - (x / y)y', d)
```
我们定义模倒数 **MODULAR INVERSE**
> $x$ is the multiplicative inverse of $a \text{ mod }N$ if $ax \equiv 1\text{ mod }N$

再来一个定理：
> if $gcd(a,N)>1$ then $ax \not\equiv 1\text{ mod }N$

$$
\begin{align}
&ax\text{ mod }N=(ax + kN)\text{ mod }N \\
&if\text{ }gcd(a,N)=t\implies ax|t，kN|t，ax+kN|t
\end{align}
$$
自然$\not\equiv1$。