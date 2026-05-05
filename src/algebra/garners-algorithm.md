# Algoritmo de Garner

Uma consequência do [Teorema Chinês do Resto](chinese-remainder-theorem.md) é que podemos representar números grandes usando um array de inteiros pequenos.
Por exemplo, seja $p$ o produto dos primeiros $1000$ números primos. $p$ tem cerca de $3000$ dígitos.

Qualquer número $a$ menor que $p$ pode ser representado como um array $a_1, \ldots, a_k$, onde $a_i \equiv a \pmod{p_i}$.
Mas para fazer isso nós obviamente precisamos saber como obter o número original $a$ a partir da sua representação.
Uma maneira é discutida no artigo sobre o Teorema Chinês do Resto.

Neste artigo discutiremos uma alternativa, o Algoritmo de Garner, que também pode ser usado para este propósito.

## Representação de Raiz Mista

Nós podemos representar o número $a$ na representação de **raiz mista** (*mixed radix*):

$$a = x_1 + x_2 p_1 + x_3 p_1 p_2 + \ldots + x_k p_1 \cdots p_{k-1} \text{ com }x_i \in [0, p_i)$$

Uma representação de raiz mista é um sistema de numeração posicional, sendo uma generalização dos sistemas numéricos típicos, como o sistema de numeração binário ou o sistema de numeração decimal.
Por exemplo, o sistema decimal é um sistema de numeração posicional com a raiz (ou base) 10.
Todo número é representado como uma string de dígitos $d_1 d_2 d_3 \dots d_n$ entre $0$ e $9$. Por exemplo, a string $415$ representa o número $4 \cdot 10^2 + 1 \cdot 10^1 + 5 \cdot 10^0$.
Em geral a string de dígitos $d_1 d_2 d_3 \dots d_n$ representa o número $d_1 b^{n-1} + d_2 b^{n-2} + \cdots + d_n b^0$ no sistema de numeração posicional com raiz $b$.

Em um sistema de raiz mista, não temos mais apenas uma raiz. A base varia de posição para posição.

## Algoritmo de Garner

O algoritmo de Garner computa os dígitos $x_1, \ldots, x_k$.
Note que os dígitos são relativamente pequenos.
O dígito $x_i$ é um inteiro entre $0$ e $p_i - 1$.

Seja $r_{ij}$ o inverso de $p_i$ módulo $p_j$:

$$r_{ij} = (p_i)^{-1} \pmod{p_j}$$

o qual pode ser encontrado usando o algoritmo descrito em [Inverso Multiplicativo Modular](module-inverse.md).

Substituindo $a$ da representação de raiz mista na primeira equação de congruência, obtemos:

$$a_1 \equiv x_1 \pmod{p_1}.$$

Substituindo na segunda equação, resulta em:

$$a_2 \equiv x_1 + x_2 p_1 \pmod{p_2},$$

que pode ser reescrita subtraindo $x_1$ e dividindo por $p_1$ para obter:

$$\begin{array}{rclr}
    a_2 - x_1 &\equiv& x_2 p_1 &\pmod{p_2} \\
    (a_2 - x_1) r_{12} &\equiv& x_2 &\pmod{p_2} \\
    x_2 &\equiv& (a_2 - x_1) r_{12} &\pmod{p_2}
\end{array}$$

Similarmente, obtemos que:

$$x_3 \equiv ((a_3 - x_1) r_{13} - x_2) r_{23} \pmod{p_3}.$$

Agora, podemos ver claramente um padrão surgindo, que pode ser expresso pelo seguinte código:

```cpp
for (int i = 0; i < k; ++i) {
    x[i] = a[i];
    for (int j = 0; j < i; ++j) {
        x[i] = r[j][i] * (x[i] - x[j]);

        x[i] = x[i] % p[i];
        if (x[i] < 0)
            x[i] += p[i];
    }
}
```

Então, aprendemos a calcular os dígitos $x_i$ em tempo $O(k^2)$. O número $a$ agora pode ser calculado usando a fórmula mencionada anteriormente:

$$a = x_1 + x_2 \cdot p_1 + x_3 \cdot p_1 \cdot p_2 + \ldots + x_k \cdot p_1 \cdots p_{k-1}$$

Vale ressaltar que, na prática, quase certamente precisaremos calcular a resposta $a$ usando [Aritmética de Precisão Arbitrária](big-integer.md), mas os dígitos $x_i$ (por serem pequenos) geralmente podem ser calculados utilizando os tipos nativos das linguagens (built-in types), e portanto, o algoritmo de Garner é muito eficiente.

## Implementação do Algoritmo de Garner

É conveniente implementar este algoritmo usando Java, pois ele possui suporte nativo para números grandes com a classe `BigInteger`.

Aqui mostramos uma implementação que consegue armazenar números grandes na forma de um conjunto de equações de congruência.
Ele suporta adição, subtração e multiplicação.
E com o algoritmo de Garner, nós podemos converter o conjunto de equações em um único número inteiro.
Neste código, consideramos 100 números primos maiores que $10^9$, o que permite representar números de até $10^{900}$.

```java
final int SZ = 100;
int pr[] = new int[SZ];
int r[][] = new int[SZ][SZ];

void init() {
    for (int x = 1000 * 1000 * 1000, i = 0; i < SZ; ++x)
        if (BigInteger.valueOf(x).isProbablePrime(100))
            pr[i++] = x;

    for (int i = 0; i < SZ; ++i)
        for (int j = i + 1; j < SZ; ++j)
            r[i][j] =
                BigInteger.valueOf(pr[i]).modInverse(BigInteger.valueOf(pr[j])).intValue();
}

class Number {
    int a[] = new int[SZ];

    public Number() {
    }

    public Number(int n) {
        for (int i = 0; i < SZ; ++i)
            a[i] = n % pr[i];
    }

    public Number(BigInteger n) {
        for (int i = 0; i < SZ; ++i)
            a[i] = n.mod(BigInteger.valueOf(pr[i])).intValue();
    }

    public Number add(Number n) {
        Number result = new Number();
        for (int i = 0; i < SZ; ++i)
            result.a[i] = (a[i] + n.a[i]) % pr[i];
        return result;
    }

    public Number subtract(Number n) {
        Number result = new Number();
        for (int i = 0; i < SZ; ++i)
            result.a[i] = (a[i] - n.a[i] + pr[i]) % pr[i];
        return result;
    }

    public Number multiply(Number n) {
        Number result = new Number();
        for (int i = 0; i < SZ; ++i)
            result.a[i] = (int)((a[i] * 1l * n.a[i]) % pr[i]);
        return result;
    }

    public BigInteger bigIntegerValue(boolean can_be_negative) {
        BigInteger result = BigInteger.ZERO, mult = BigInteger.ONE;
        int x[] = new int[SZ];
        for (int i = 0; i < SZ; ++i) {
            x[i] = a[i];
            for (int j = 0; j < i; ++j) {
                long cur = (x[i] - x[j]) * 1l * r[j][i];
                x[i] = (int)((cur % pr[i] + pr[i]) % pr[i]);
            }
            result = result.add(mult.multiply(BigInteger.valueOf(x[i])));
            mult = mult.multiply(BigInteger.valueOf(pr[i]));
        }

        if (can_be_negative)
            if (result.compareTo(mult.shiftRight(1)) >= 0)
                result = result.subtract(mult);

        return result;
    }
}
```
