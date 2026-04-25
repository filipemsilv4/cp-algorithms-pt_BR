---
tags:
  - Translated
e_maxx_link: fibonacci_numbers
---

# Números de Fibonacci

A sequência de Fibonacci é definida da seguinte forma:

$$F_0 = 0, F_1 = 1, F_n = F_{n-1} + F_{n-2}$$

Os primeiros elementos da sequência ([OEIS A000045](http://oeis.org/A000045)) são:

$$0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, ...$$

## Propriedades

Os números de Fibonacci possuem muitas propriedades interessantes. Aqui estão algumas delas:

* Identidade de Cassini:
  
$$F_{n-1} F_{n+1} - F_n^2 = (-1)^n$$

>Isso pode ser provado por indução. Uma prova de uma linha de Knuth vem de tomar o determinante da forma de matriz 2x2 abaixo.

* A regra da "adição":
  
$$F_{n+k} = F_k F_{n+1} + F_{k-1} F_n$$

* Aplicando a identidade anterior para o caso $k = n$, obtemos:
  
$$F_{2n} = F_n (F_{n+1} + F_{n-1})$$

* A partir disso, podemos provar por indução que para qualquer inteiro positivo $k$, $F_{nk}$ é múltiplo de $F_n$.

* O inverso também é verdadeiro: se $F_m$ é múltiplo de $F_n$, então $m$ é múltiplo de $n$.

* Identidade do MDC (GCD, ou em português, MDC):
  
$$GCD(F_m, F_n) = F_{GCD(m, n)}$$

* Os números de Fibonacci são os piores casos de entrada possíveis para o Algoritmo de Euclides (veja o Teorema de Lamé no [Algoritmo de Euclides](euclid-algorithm.md))

## Codificação de Fibonacci

Podemos usar a sequência para codificar números inteiros positivos em palavras de código binário. De acordo com o teorema de Zeckendorf, qualquer número natural $n$ pode ser representado de forma única como uma soma de números de Fibonacci:

$$N = F_{k_1} + F_{k_2} + \ldots + F_{k_r}$$

tal que $k_1 \ge k_2 + 2,\ k_2 \ge k_3 + 2,\  \ldots,\  k_r \ge 2$ (ou seja: a representação não pode usar dois números de Fibonacci consecutivos).

Segue-se que qualquer número pode ser codificado de forma única na codificação de Fibonacci.
E podemos descrever essa representação com códigos binários $d_0 d_1 d_2 \dots d_s 1$, onde $d_i$ é $1$ se $F_{i+2}$ é usado na representação.
O código será acrescentado de um $1$ para indicar o fim da palavra de código.
Note que esta é a única ocorrência em que dois bits 1 consecutivos aparecem.

$$\begin{eqnarray}
1 &=& 1 &=& F_2 &=& (11)_F \\
2 &=& 2 &=& F_3 &=& (011)_F \\
6 &=& 5 + 1 &=& F_5 + F_2 &=& (10011)_F \\
8 &=& 8 &=& F_6 &=& (000011)_F \\
9 &=& 8 + 1 &=& F_6 + F_2 &=& (100011)_F \\
19 &=& 13 + 5 + 1 &=& F_7 + F_5 + F_2 &=& (1001011)_F
\end{eqnarray}$$

A codificação de um inteiro $n$ pode ser feita com um simples algoritmo guloso (greedy algorithm):

1. Itere pelos números de Fibonacci do maior para o menor até encontrar um que seja menor ou igual a $n$.

2. Suponha que esse número seja $F_i$. Subtraia $F_i$ de $n$ e coloque um $1$ na posição $i-2$ da palavra de código (indexando a partir do 0 do bit mais à esquerda para o mais à direita).

3. Repita até que não haja mais resto.

4. Adicione um $1$ final à palavra de código para indicar o seu fim.

Para decodificar uma palavra de código, primeiro remova o $1$ final. Então, se o $i$-ésimo bit estiver definido (indexando a partir de 0 do bit mais à esquerda para o mais à direita), some $F_{i+2}$ ao número.


## Fórmulas para o $n^{\text{ésimo}}$ número de Fibonacci { data-toc-label="Fórmulas para o <script type='math/tex'>n</script>-ésimo número de Fibonacci" }

### Expressão de forma fechada

Existe uma fórmula conhecida como "Fórmula de Binet", embora já fosse conhecida por Moivre:

$$F_n = \frac{\left(\frac{1 + \sqrt{5}}{2}\right)^n - \left(\frac{1 - \sqrt{5}}{2}\right)^n}{\sqrt{5}}$$

Essa fórmula é fácil de provar por indução, mas pode ser deduzida com a ajuda do conceito de funções geradoras ou resolvendo uma equação funcional.

Você pode notar imediatamente que o valor absoluto do segundo termo é sempre menor que $1$, e também decresce muito rapidamente (exponencialmente). Portanto, o valor do primeiro termo sozinho é "quase" $F_n$. Isso pode ser escrito estritamente como:

$$F_n = \left[\frac{\left(\frac{1 + \sqrt{5}}{2}\right)^n}{\sqrt{5}}\right]$$

onde os colchetes denotam o arredondamento para o inteiro mais próximo.

Como essas duas fórmulas exigiriam uma precisão muito alta ao trabalhar com números fracionários, elas são de pouco uso em cálculos práticos.

### Fibonacci em tempo linear

O $n$-ésimo número de Fibonacci pode ser facilmente encontrado em $O(n)$ computando os números um por um até $n$. No entanto, existem formas mais rápidas, como veremos.

Podemos começar a partir de uma abordagem iterativa, para aproveitar o uso da fórmula $F_n = F_{n-1} + F_{n-2}$, assim, simplesmente precalcularemos esses valores em um array. Levando em conta os casos base para $F_0$ e $F_1$.

```{.cpp file=fibonacci_linear}
int fib(int n) {
    int a = 0;
    int b = 1;
    for (int i = 0; i < n; i++) {
        int tmp = a + b;
        a = b;
        b = tmp;
    }
    return a;
}
```

Dessa forma, obtemos uma solução linear, em tempo $O(n)$, salvando todos os valores anteriores a $n$ na sequência.

### Forma de Matriz

Para ir de $(F_n, F_{n-1})$ para $(F_{n+1}, F_n)$, podemos expressar a recorrência linear como uma multiplicação de matriz 2x2:

$$
\begin{pmatrix}
1 & 1 \\
1 & 0
\end{pmatrix}
\begin{pmatrix}
F_n \\
F_{n-1}
\end{pmatrix}
=
\begin{pmatrix}
F_n + F_{n-1}  \\
F_{n}
\end{pmatrix}
=
\begin{pmatrix}
F_{n+1}  \\
F_{n}
\end{pmatrix}
$$

Isso nos permite tratar a iteração da recorrência como uma multiplicação repetida de matrizes, o que possui propriedades interessantes. Em particular,

$$
\begin{pmatrix}
1 & 1 \\
1 & 0
\end{pmatrix}^n
\begin{pmatrix}
F_1 \\
F_0
\end{pmatrix}
=
\begin{pmatrix}
F_{n+1}  \\
F_{n}
\end{pmatrix}
$$

onde $F_1 = 1, F_0 = 0$.
Na verdade, uma vez que

$$
\begin{pmatrix} 1 & 1 \\ 1 & 0 \end{pmatrix}
= \begin{pmatrix} F_2 & F_1 \\ F_1 & F_0 \end{pmatrix}
$$

podemos usar a matriz diretamente:

$$
\begin{pmatrix} 1 & 1 \\ 1 & 0 \end{pmatrix}^n
= \begin{pmatrix} F_{n+1} & F_n \\ F_n & F_{n-1} \end{pmatrix}
$$

Portanto, para encontrar $F_n$ em tempo $O(\log n)$, precisamos elevar a matriz a $n$. (Veja [Exponenciação Binária](binary-exp.md))

```{.cpp file=fibonacci_matrix}
struct matrix {
    long long mat[2][2];
    matrix friend operator *(const matrix &a, const matrix &b){
        matrix c;
        for (int i = 0; i < 2; i++) {
          for (int j = 0; j < 2; j++) {
              c.mat[i][j] = 0;
              for (int k = 0; k < 2; k++) {
                  c.mat[i][j] += a.mat[i][k] * b.mat[k][j];
              }
          }
        }
        return c;
    }
};

matrix matpow(matrix base, long long n) {
    matrix ans{ {
      {1, 0},
      {0, 1}
    } };
    while (n) {
        if(n&1)
            ans = ans*base;
        base = base*base;
        n >>= 1;
    }
    return ans;
}

long long fib(int n) {
    matrix base{ {
      {1, 1},
      {1, 0}
    } };
    return matpow(base, n).mat[0][1];
}
```

### Método de Duplicação Rápida (Fast Doubling)

Expandindo a expressão de matriz acima para $n = 2\cdot k$

$$
\begin{pmatrix}
F_{2k+1} & F_{2k}\\
F_{2k} & F_{2k-1}
\end{pmatrix}
=
\begin{pmatrix}
1 & 1\\
1 & 0
\end{pmatrix}^{2k}
=
\begin{pmatrix}
F_{k+1} & F_{k}\\
F_{k} & F_{k-1}
\end{pmatrix}
^2
$$

podemos encontrar estas equações mais simples:

$$ \begin{align}
F_{2k+1} &= F_{k+1}^2 + F_{k}^2 \\
F_{2k} &= F_k(F_{k+1}+F_{k-1}) = F_k (2F_{k+1} - F_{k})\\
\end{align}.$$

Assim, usando as duas equações acima, os números de Fibonacci podem ser calculados facilmente pelo seguinte código:

```{.cpp file=fibonacci_doubling}
pair<int, int> fib (int n) {
    if (n == 0)
        return {0, 1};

    auto p = fib(n >> 1);
    int c = p.first * (2 * p.second - p.first);
    int d = p.first * p.first + p.second * p.second;
    if (n & 1)
        return {d, c + d};
    else
        return {c, d};
}
```
O código acima retorna $F_n$ e $F_{n+1}$ como um pair.

## Periodicidade módulo p

Considere a sequência de Fibonacci módulo $p$. Vamos provar que a sequência é periódica.

Provaremos isso por contradição. Considere os primeiros $p^2 + 1$ pares de números de Fibonacci tomados módulo $p$:

$$(F_0,\ F_1),\ (F_1,\ F_2),\ \ldots,\ (F_{p^2},\ F_{p^2 + 1})$$

Podem haver apenas $p$ diferentes restos módulo $p$, e no máximo $p^2$ diferentes pares de restos, portanto há pelo menos dois pares idênticos entre eles. Isso é suficiente para provar que a sequência é periódica, já que um número de Fibonacci é determinado unicamente por seus dois predecessores. Por isso, se dois pares de números consecutivos se repetem, isso também significaria que os números depois do par se repetirão da mesma maneira.

Vamos agora escolher dois pares de restos idênticos com os menores índices na sequência. Sejam os pares $(F_a,\ F_{a + 1})$ e $(F_b,\ F_{b + 1})$. Vamos provar que $a = 0$. Se isso fosse falso, haveria dois pares anteriores $(F_{a-1},\ F_a)$ e $(F_{b-1},\ F_b)$, que, pela propriedade dos números de Fibonacci, também seriam iguais. No entanto, isso contradiz o fato de que havíamos escolhido pares com os menores índices, concluindo a nossa prova de que não há pré-período (ou seja, os números são periódicos a partir de $F_0$).

## Problemas Práticos

* [SPOJ - Euclid Algorithm Revisited](http://www.spoj.com/problems/MAIN74/)
* [SPOJ - Fibonacci Sum](http://www.spoj.com/problems/FIBOSUM/)
* [HackerRank - Is Fibo](https://www.hackerrank.com/challenges/is-fibo/problem)
* [Project Euler - Even Fibonacci numbers](https://www.hackerrank.com/contests/projecteuler/challenges/euler002/problem)
* [DMOJ - Fibonacci Sequence](https://dmoj.ca/problem/fibonacci)
* [DMOJ - Fibonacci Sequence (Harder)](https://dmoj.ca/problem/fibonacci2)
* [DMOJ UCLV - Numbered sequence of pencils](https://dmoj.uclv.edu.cu/problem/secnum)
* [DMOJ UCLV - Fibonacci 2D](https://dmoj.uclv.edu.cu/problem/fibonacci)
* [DMOJ UCLV - fibonacci calculation](https://dmoj.uclv.edu.cu/problem/fibonaccicalculatio)
* [LightOJ -  Number Sequence](https://lightoj.com/problem/number-sequence)
* [Codeforces - C. Fibonacci](https://codeforces.com/problemset/gymProblem/102644/C)
* [Codeforces - A. Hexadecimal's theorem](https://codeforces.com/problemset/problem/199/A)
* [Codeforces - B. Blackboard Fibonacci](https://codeforces.com/problemset/problem/217/B)
* [Codeforces - E. Fibonacci Number](https://codeforces.com/problemset/problem/193/E)
