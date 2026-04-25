#report

### Random variables
###### 1. Given a fair dice with its face value when the dice is tossed as a discrete random variable X
a. $E(X)$
$$
\begin{align}
E(X) &= \sum_{x \in \{1,2,3,4,5,6\}} P(X = x)x \\
&= \frac{21}{6} = 3.5
\end{align}
$$
b. $V(X)$
$$
\begin{align}
V(X) &= E(X^2) - E^2(X)\\
&= \sum_{x \in \{1,2,3,4,5,6\}} P(X = x)x^2 - \frac{49}{4}\\
&= \frac{1+4+9+16+25+36}{6} - \frac{49}{4}\\
&= \frac{91}{6} - \frac{49}{4} = \frac{182 - 147}{12}\\
& = \frac{35}{12}
\end{align}
$$

###### 2. Given two fair dices with their face values when the dices are tossed as two discrete random variables $X$ and $Y$, what are $E(Z)$ and $V(Z)$ if $Z=X+2Y$ ?
$$
\begin{align}
E(Z) &= E(X + 2Y) = E(X) +2E(Y)\\
&= 3.5 + 7 = 10.5
\end{align}
$$

$$
\begin{align}
V(Z) &= V(X+2Y) = V(X)+2^2V(Y)-2\text{Cov}(X,2Y)   \\
&= \frac{35 + 4*35}{12}\\
&= \frac{175}{12}
\end{align}
$$



###### 3. Given two random variables $X$ and $Y$, with their sample values of $[1,3,2,1,3]$ and $[2,4,5,4,5]$, respectively, compute the covariance matrix of these two random variables.
$$
\begin{align}
Cov(X,Y) &=\frac{\sum_{i = 0}^{5} (X_i - E(X)) (Y_i - E(Y))}{5-1}\\
&= \frac{(-1\times -2) + (1 \times 0) + (0 \times 1) + (-1 \times 0) + (1 \times 1)}{4}\\
&= \frac{3}{4}
\end{align}
$$

###### 6. The Poisson distribution is described by $p(x,\lambda)=\frac{e^{−\lambda}\lambda^k}{k!}$. Prove that $\sum^\infty_{x=0} p(x,λ)=1$
$$
\int^{\infty}_0 \frac{e^{−\lambda}\lambda^k}{k!} = e^{-\lambda}\int^{\infty}_0\frac{\lambda^k}{k!} = e^{-\lambda}e^\lambda  = 1
$$

$\int^{\infty}_0\frac{\lambda^k}{k!} = e^\lambda$ 是指數函數的泰勒展開。

