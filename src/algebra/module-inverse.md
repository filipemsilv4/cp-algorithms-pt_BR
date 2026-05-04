---
tags:
  - Translated
e_maxx_link: reverse_element
---

# Inverso Multiplicativo Modular

## Definição

Um [inverso multiplicativo modular](http://en.wikipedia.org/wiki/Modular_multiplicative_inverse) de um inteiro $a$ é um inteiro $x$ tal que $a \cdot x$ é congruente a $1$ módulo $m$.
Para escrever formalmente: queremos encontrar um inteiro $x$ tal que

$$a \cdot x \equiv 1 \mod m.$$

Também denotaremos $x$ simplesmente como $a^{-1}$.

Devemos notar que o inverso modular nem sempre existe. Por exemplo, seja $m = 4$, $a = 2$.
Ao testar todos os valores possíveis módulo $m$, deve ficar claro que não podemos encontrar um $a^{-1}$ que satisfaça a equação acima.
Pode ser provado que o inverso modular existe se e somente se $a$ e $m$ forem primos entre si (ou seja, coprimos, $\gcd(a, m) = 1$).

Neste artigo, apresentamos dois métodos para encontrar o inverso modular caso ele exista, e um método para encontrar o inverso modular para todos os números em tempo linear.

## Encontrando o Inverso Modular usando o Algoritmo de Euclides Estendido

Considere a seguinte equação (com incógnitas $x$ e $y$):

$$a \cdot x + m \cdot y = 1$$

Essa é uma [Equação Diofantina Linear de duas variáveis](linear-diophantine-equation.md).
Como mostrado no artigo mencionado, quando $\gcd(a, m) = 1$, a equação tem uma solução que pode ser encontrada usando o [algoritmo de Euclides estendido](extended-euclid-algorithm.md).
Note que $\gcd(a, m) = 1$ também é a condição para que o inverso modular exista.

Agora, se tirarmos o módulo $m$ de ambos os lados, podemos nos livrar de $m \cdot y$, e a equação se torna:

$$a \cdot x \equiv 1 \mod m$$

Portanto, o inverso modular de $a$ é $x$.

A implementação é a seguinte:

```cpp
int x, y;
int g = extended_euclidean(a, m, x, y);
if (g != 1) {
    cout << "No solution!";
}
else {
    x = (x % m + m) % m;
    cout << x << endl;
}
```

Observe a forma como modificamos `x`.
O `x` resultante do algoritmo de Euclides estendido pode ser negativo, logo, `x % m` também pode ser negativo, e primeiro temos que somar `m` para torná-lo positivo.

<div id="fermat-euler"></div>
## Encontrando o Inverso Modular usando Exponenciação Binária

Outro método para encontrar o inverso modular é utilizar o teorema de Euler, que afirma que a seguinte congruência é verdadeira se $a$ e $m$ forem primos entre si:

$$a^{\phi (m)} \equiv 1 \mod m$$

$\phi$ é a [Função Totiente de Euler](phi-function.md).
Mais uma vez, note que $a$ e $m$ serem primos entre si também era a condição para a existência do inverso modular.

Se $m$ for um número primo, isso se simplifica para o [Pequeno teorema de Fermat](http://en.wikipedia.org/wiki/Fermat's_little_theorem):

$$a^{m - 1} \equiv 1 \mod m$$

Multiplique ambos os lados das equações acima por $a^{-1}$ e obtemos:

* Para um módulo $m$ arbitrário (porém coprimo): $a ^ {\phi (m) - 1} \equiv a ^{-1} \mod m$
* Para um módulo $m$ primo: $a ^ {m - 2} \equiv a ^ {-1} \mod m$

A partir desses resultados, podemos encontrar facilmente o inverso modular usando o [algoritmo de exponenciação binária](binary-exp.md), que roda em tempo $O(\log m)$.

Apesar desse método ser mais fácil de entender do que o método descrito no parágrafo anterior, no caso de $m$ não ser um número primo, precisamos calcular a função totiente de Euler, o que envolve a fatoração de $m$, que pode ser bastante custosa. Se a fatoração prima de $m$ for conhecida, a complexidade desse método é $O(\log m)$.

<div id="finding-the-modular-inverse-using-euclidean-division"></div>
## Encontrando o inverso modular para módulos primos usando Divisão Euclidiana

Dado um módulo primo $m > a$ (ou podemos aplicar o módulo para torná-lo menor em 1 passo), de acordo com a [Divisão Euclidiana](https://en.wikipedia.org/wiki/Euclidean_division)

$$m = k \cdot a + r$$

onde $k = \left\lfloor \frac{m}{a} \right\rfloor$ e $r = m \bmod a$, então

$$
\begin{align*}
& \implies & 0          & \equiv k \cdot a + r   & \mod m \\
& \iff & r              & \equiv -k \cdot a      & \mod m \\
& \iff & r \cdot a^{-1} & \equiv -k              & \mod m \\
& \iff & a^{-1}         & \equiv -k \cdot r^{-1} & \mod m
\end{align*}
$$

Note que esse raciocínio não se mantém se $m$ não for primo, já que a existência de $a^{-1}$ não implica a existência de $r^{-1}$
no caso geral. Para notar isso, vamos tentar calcular $5^{-1}$ módulo $12$ com a fórmula acima. Gostaríamos de chegar a $5$,
já que $5 \cdot 5 \equiv 1 \bmod 12$. Porém, $12 = 2 \cdot 5 + 2$, logo temos $k=2$ e $r=2$, em que $2$ não é invertível módulo $12$.

Contudo, se o módulo for primo, todos os $a$ tal que $0 < a < m$ são invertíveis módulo $m$, e podemos ter a seguinte função recursiva (em C++) para calcular o inverso modular para o número $a$ em relação a $m$:

```{.cpp file=modular_inverse_euclidean_division}
int inv(int a) {
  return a <= 1 ? a : m - (long long)(m/a) * inv(m % a) % m;
}
```

A complexidade de tempo exata dessa recursão não é conhecida. Estima-se que seja algo entre $O(\frac{\log m}{\log\log m})$ e $O(m^{\frac{1}{3} - \frac{2}{177} + \epsilon})$.
Veja [Sobre o comprimento das expansões de Pierce](https://arxiv.org/abs/2211.08374).
Na prática, essa implementação é rápida; por exemplo, para o módulo $10^9 + 7$, ela sempre terminará em menos de 50 iterações.

<div id="mod-inv-all-num"></div>
Aplicando essa fórmula, também podemos pré-calcular o inverso modular para todos os números no intervalo $[1, m-1]$ em $O(m)$.

```{.cpp file=modular_inverse_euclidean_division_all}
inv[1] = 1;
for(int a = 2; a < m; ++a)
    inv[a] = m - (long long)(m/a) * inv[m%a] % m;
```

## Encontrando o inverso modular para um array de números módulo $m$

Suponha que tenhamos um array e queremos encontrar o inverso modular para todos os números dele (sendo todos invertíveis).
Em vez de calcular o inverso para cada número, podemos expandir a fração pelo produto de prefixo (excluindo a si próprio) e pelo produto de sufixo (excluindo a si próprio), terminando com o cálculo de apenas um único inverso.

$$
\begin{align}
x_i^{-1} &= \frac{1}{x_i} = \frac{\overbrace{x_1 \cdot x_2 \cdots x_{i-1}}^{\text{prefix}_{i-1}} \cdot ~1~ \cdot \overbrace{x_{i+1} \cdot x_{i+2} \cdots x_n}^{\text{suffix}_{i+1}}}{x_1 \cdot x_2 \cdots x_{i-1} \cdot x_i \cdot x_{i+1} \cdot x_{i+2} \cdots x_n} \\
&= \text{prefix}_{i-1} \cdot \text{suffix}_{i+1} \cdot \left(x_1 \cdot x_2 \cdots x_n\right)^{-1}
\end{align}
$$

No código, podemos apenas construir um array com produtos de prefixo (excluindo a si próprio e iniciando do elemento de identidade), calcular o inverso modular do produto de todos os números e então multiplicá-lo pelo produto de prefixo e o produto de sufixo (excluindo a si próprio).
O produto de sufixo é computado iterando de trás para frente.

```cpp
std::vector<int> invs(const std::vector<int> &a, int m) {
    int n = a.size();
    if (n == 0) return {};
    std::vector<int> b(n);
    int v = 1;
    for (int i = 0; i != n; ++i) {
        b[i] = v;
        v = static_cast<long long>(v) * a[i] % m;
    }
    int x, y;
    extended_euclidean(v, m, x, y);
    x = (x % m + m) % m;
    for (int i = n - 1; i >= 0; --i) {
        b[i] = static_cast<long long>(x) * b[i] % m;
        x = static_cast<long long>(x) * a[i] % m;
    }
    return b;
}
```

## Problemas Práticos

* [UVa 11904 - One Unit Machine](https://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&problem=3055)
* [Hackerrank - Longest Increasing Subsequence Arrays](https://www.hackerrank.com/contests/world-codesprint-5/challenges/longest-increasing-subsequence-arrays)
* [Codeforces 300C - Beautiful Numbers](http://codeforces.com/problemset/problem/300/C)
* [Codeforces 622F - The Sum of the k-th Powers](http://codeforces.com/problemset/problem/622/F)
* [Codeforces 717A - Festival Organization](http://codeforces.com/problemset/problem/717/A)
* [Codeforces 896D - Nephren Runs a Cinema](http://codeforces.com/problemset/problem/896/D)
