---
tags:
  - Translated
e_maxx_link: chinese_theorem
---

# Teorema Chinês do Resto

O Teorema Chinês do Resto (que será referido como CRT, ou em português, Teorema Chinês do Resto, no resto deste artigo) foi descoberto pelo matemático chinês Sun Zi.

## Formulação

Seja $m = m_1 \cdot m_2 \cdots m_k$, onde $m_i$ são coprimos dois a dois. Além de $m_i$, também nos é dado um sistema de congruências

$$\left\{\begin{array}{rcl}
    a & \equiv & a_1 \pmod{m_1} \\
    a & \equiv & a_2 \pmod{m_2} \\
      & \vdots & \\
    a & \equiv & a_k \pmod{m_k}
\end{array}\right.$$

onde $a_i$ são algumas constantes dadas. A forma original do CRT então afirma que o sistema de congruências dado sempre tem *uma e exatamente uma* solução módulo $m$.

Por exemplo, o sistema de congruências

$$\left\{\begin{array}{rcl}
    a & \equiv & 2 \pmod{3} \\
    a & \equiv & 3 \pmod{5} \\
    a & \equiv & 2 \pmod{7}
\end{array}\right.$$

tem a solução $23$ módulo $105$, porque $23 \bmod{3} = 2$, $23 \bmod{5} = 3$, e $23 \bmod{7} = 2$.
Podemos escrever cada solução como $23 + 105\cdot k$ para $k \in \mathbb{Z}$.

### Corolário

Uma consequência do CRT é que a equação

$$x \equiv a \pmod{m}$$

é equivalente ao sistema de equações

$$\left\{\begin{array}{rcl}
    x & \equiv & a_1 \pmod{m_1} \\
      & \vdots & \\
    x & \equiv & a_k \pmod{m_k}
\end{array}\right.$$

(Como acima, assuma que $m = m_1 m_2 \cdots m_k$ e $m_i$ são coprimos dois a dois).

## Solução para Dois Módulos

Considere um sistema de duas equações para $m_1, m_2$ coprimos:

$$
\left\{\begin{align}
    a &\equiv a_1 \pmod{m_1} \\
    a &\equiv a_2 \pmod{m_2} \\
\end{align}\right.
$$

Queremos encontrar uma solução para $a \pmod{m_1 m_2}$. Usando o [Algoritmo de Euclides Estendido](extended-euclid-algorithm.md) podemos encontrar os coeficientes de Bézout $n_1, n_2$ tais que

$$n_1 m_1 + n_2 m_2 = 1.$$

De fato, $n_1$ e $n_2$ são apenas os [inversos modulares](module-inverse.md) de $m_1$ e $m_2$ módulo $m_2$ e $m_1$.
Temos $n_1 m_1 \equiv 1 \pmod{m_2}$ então $n_1 \equiv m_1^{-1} \pmod{m_2}$, e vice-versa $n_2 \equiv m_2^{-1} \pmod{m_1}$.

Com esses dois coeficientes podemos definir uma solução:

$$a = a_1 n_2 m_2 + a_2 n_1 m_1 \bmod{m_1 m_2}$$

É fácil verificar que esta é de fato uma solução calculando $a \bmod{m_1}$ e $a \bmod{m_2}$.

$$
\begin{array}{rcll}
a & \equiv & a_1 n_2 m_2 + a_2 n_1 m_1 & \pmod{m_1}\\
  & \equiv & a_1 (1 - n_1 m_1) + a_2 n_1 m_1 & \pmod{m_1}\\
  & \equiv & a_1 - a_1 n_1 m_1 + a_2 n_1 m_1 & \pmod{m_1}\\
  & \equiv & a_1 & \pmod{m_1}
\end{array}
$$

Note que o Teorema Chinês do Resto também garante que existe apenas 1 solução módulo $m_1 m_2$.
Isso também é fácil de provar.

Vamos assumir que você tem duas soluções diferentes $x$ e $y$.
Como $x \equiv a_i \pmod{m_i}$ e $y \equiv a_i \pmod{m_i}$, segue-se que $x − y \equiv 0 \pmod{m_i}$ e portanto $x − y \equiv 0 \pmod{m_1 m_2}$ ou equivalentemente $x \equiv y \pmod{m_1 m_2}$.
Então $x$ e $y$ são na verdade a mesma solução.

## Solução para o Caso Geral

### Solução Indutiva

Como $m_1 m_2$ é coprimo a $m_3$, podemos aplicar repetidamente por indução a solução para dois módulos para qualquer número de módulos.
Primeiro você calcula $b_2 := a \pmod{m_1 m_2}$ usando as primeiras duas congruências,
depois você pode calcular $b_3 := a \pmod{m_1 m_2 m_3}$ usando as congruências $a \equiv b_2 \pmod{m_1 m_2}$ e $a \equiv a_3 \pmod {m_3}$, etc.

### Construção Direta

Uma construção direta semelhante à interpolação de Lagrange é possível.

Seja $M_i := \prod_{i \neq j} m_j$, o produto de todos os módulos exceto $m_i$, e $N_i$ os inversos modulares $N_i := M_i^{-1} \bmod{m_i}$.
Então uma solução para o sistema de congruências é:

$$a \equiv \sum_{i=1}^k a_i M_i N_i \pmod{m_1 m_2 \cdots m_k}$$

Podemos checar que esta é de fato uma solução, calculando $a \bmod{m_i}$ para todo $i$.
Como $M_j$ é um múltiplo de $m_i$ para $i \neq j$ nós temos

$$\begin{array}{rcll}
a & \equiv & \sum_{j=1}^k a_j M_j N_j & \pmod{m_i} \\
  & \equiv & a_i M_i N_i              & \pmod{m_i} \\
  & \equiv & a_i M_i M_i^{-1}         & \pmod{m_i} \\
  & \equiv & a_i                      & \pmod{m_i}
\end{array}$$

### Implementação

```{.cpp file=chinese_remainder_theorem}
struct Congruence {
    long long a, m;
};

long long chinese_remainder_theorem(vector<Congruence> const& congruences) {
    long long M = 1;
    for (auto const& congruence : congruences) {
        M *= congruence.m;
    }

    long long solution = 0;
    for (auto const& congruence : congruences) {
        long long a_i = congruence.a;
        long long M_i = M / congruence.m;
        long long N_i = mod_inv(M_i, congruence.m);
        solution = (solution + a_i * M_i % M * N_i) % M;
    }
    return solution;
}
```

## Solução para módulos não coprimos

Como mencionado, o algoritmo acima funciona apenas para módulos coprimos $m_1, m_2, \dots m_k$.

No caso não coprimo, um sistema de congruências tem exatamente uma solução módulo $\text{lcm}(m_1, m_2, \dots, m_k)$, ou não tem solução alguma.

Por exemplo, no sistema a seguir, a primeira congruência implica que a solução é ímpar, e a segunda congruência implica que a solução é par.
Não é possível que um número seja par e ímpar simultaneamente, portanto claramente não há solução.

$$\left\{\begin{align}
    a & \equiv 1 \pmod{4} \\
    a & \equiv 2 \pmod{6}
\end{align}\right.$$

É bastante simples determinar se um sistema tem uma solução.
E se tiver uma, podemos usar o algoritmo original para resolver um sistema de congruências ligeiramente modificado.

Uma única congruência $a \equiv a_i \pmod{m_i}$ é equivalente ao sistema de congruências $a \equiv a_i \pmod{p_j^{n_j}}$ onde $p_1^{n_1} p_2^{n_2}\cdots p_k^{n_k}$ é a fatoração em primos de $m_i$.

Com este fato, podemos modificar o sistema de congruências para um sistema que tem apenas potências de primos como módulos.
Por exemplo, o sistema de congruências acima é equivalente a:

$$\left\{\begin{array}{ll}
    a \equiv 1          & \pmod{4} \\
    a \equiv 2 \equiv 0 & \pmod{2} \\
    a \equiv 2          & \pmod{3}
\end{array}\right.$$

Como originalmente alguns módulos tinham fatores comuns, obteremos alguns módulos de congruências baseados no mesmo primo, no entanto possivelmente com potências de primos diferentes.

Você pode observar que a congruência com o maior módulo de potência de primo será a congruência mais forte de todas as congruências baseadas no mesmo número primo.
Ou ela dará uma contradição com alguma outra congruência, ou ela já implicará em todas as outras congruências.

No nosso caso, a primeira congruência $a \equiv 1 \pmod{4}$ implica em $a \equiv 1 \pmod{2}$, e portanto contradiz a segunda congruência $a \equiv 0 \pmod{2}$.
Portanto este sistema de congruências não tem solução.

Se não houver contradições, então o sistema de equações tem uma solução.
Podemos ignorar todas as congruências exceto as com os maiores módulos de potências de primos.
Esses módulos agora são coprimos, e portanto podemos resolver este com o algoritmo discutido nas seções acima.

Por exemplo, o sistema a seguir tem uma solução módulo $\text{lcm}(10, 12) = 60$.

$$\left\{\begin{align}
    a & \equiv 3 \pmod{10} \\
    a & \equiv 5 \pmod{12}
\end{align}\right.$$

O sistema de congruência é equivalente ao sistema de congruências:

$$\left\{\begin{align}
    a & \equiv 3 \equiv 1 \pmod{2} \\
    a & \equiv 3 \equiv 3 \pmod{5} \\
    a & \equiv 5 \equiv 1 \pmod{4} \\
    a & \equiv 5 \equiv 2 \pmod{3}
\end{align}\right.$$

As únicas congruências com o mesmo primo módulo são $a \equiv 1 \pmod{4}$ e $a \equiv 1 \pmod{2}$.
A primeira já implica a segunda, então podemos ignorar a segunda, e em vez disso resolver o seguinte sistema com módulos coprimos:

$$\left\{\begin{align}
    a & \equiv 3 \equiv 3 \pmod{5} \\
    a & \equiv 5 \equiv 1 \pmod{4} \\
    a & \equiv 5 \equiv 2 \pmod{3}
\end{align}\right.$$

Tem a solução $53 \pmod{60}$, e de fato $53 \bmod{10} = 3$ e $53 \bmod{12} = 5$.

## Algoritmo de Garner

Outra consequência do CRT é que podemos representar números grandes usando um array de inteiros pequenos.

Em vez de fazer muitas computações com números muito grandes, que podem ser caras (pense em fazer divisões com números de 1000 dígitos), você pode escolher um par de módulos coprimos e representar o número grande como um sistema de congruências, e realizar todas as operações no sistema de equações.
Qualquer número $a$ menor que $m_1 m_2 \cdots m_k$ pode ser representado como um array $a_1, \ldots, a_k$, onde $a \equiv a_i \pmod{m_i}$.

Usando o algoritmo acima, você pode reconstruir o número grande novamente sempre que precisar dele.

Alternativamente você pode representar o número na representação de **radix misto** (base mista):

$$a = x_1 + x_2 m_1 + x_3 m_1 m_2 + \ldots + x_k m_1 \cdots m_{k-1} \text{ com }x_i \in [0, m_i)$$

O algoritmo de Garner, que é discutido no artigo dedicado [Algoritmo de Garner](garners-algorithm.md), calcula os coeficientes $x_i$.
E com esses coeficientes você pode restaurar o número completo.

## Problemas Práticos:

* [Google Code Jam - Golf Gophers](https://github.com/google/coding-competitions-archive/blob/main/codejam/2019/round_1a/golf_gophers/statement.pdf)
* [Hackerrank - Number of sequences](https://www.hackerrank.com/contests/w22/challenges/number-of-sequences)
* [Codeforces - Remainders Game](http://codeforces.com/problemset/problem/687/B)
