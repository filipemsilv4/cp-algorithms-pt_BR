---
tags:
  - Original
---

# Número de divisores / soma dos divisores

Neste artigo, discutimos como calcular o número de divisores $d(n)$ e a soma dos divisores $\sigma(n)$ de um dado número $n$.

## Número de divisores

Deve ser óbvio que a fatoração em primos de um divisor $d$ precisa ser um subconjunto da fatoração em primos de $n$, por exemplo, $6 = 2 \cdot 3$ é um divisor de $60 = 2^2 \cdot 3 \cdot 5$.
Portanto, só precisamos encontrar todos os diferentes subconjuntos da fatoração em primos de $n$.

Geralmente, o número de subconjuntos é $2^x$ para um conjunto com $x$ elementos.
No entanto, isso não é mais verdade se houver elementos repetidos no conjunto. Em nosso caso, alguns fatores primos podem aparecer várias vezes na fatoração em primos de $n$.

Se um fator primo $p$ aparece $e$ vezes na fatoração em primos de $n$, então podemos usar o fator $p$ até $e$ vezes no subconjunto.
O que significa que temos $e+1$ escolhas.

Portanto, se a fatoração em primos de $n$ for $p_1^{e_1} \cdot p_2^{e_2} \cdots p_k^{e_k}$, onde os $p_i$ são números primos distintos, então o número de divisores é:

$$d(n) = (e_1 + 1) \cdot (e_2 + 1) \cdots (e_k + 1)$$

Uma forma de pensar sobre isso é a seguinte:

* Se houver apenas um divisor primo distinto $n = p_1^{e_1}$, então há obviamente $e_1 + 1$ divisores ($1, p_1, p_1^2, \dots, p_1^{e_1}$).

* Se houver dois divisores primos distintos $n = p_1^{e_1} \cdot p_2^{e_2}$, então você pode organizar todos os divisores em forma de tabela.

$$\begin{array}{c|ccccc}
& 1 & p_2 & p_2^2 & \dots & p_2^{e_2} \\\\\hline
1 & 1 & p_2 & p_2^2 & \dots & p_2^{e_2} \\\\
p_1 & p_1 & p_1 \cdot p_2 & p_1 \cdot p_2^2 & \dots & p_1 \cdot p_2^{e_2} \\\\
p_1^2 & p_1^2 & p_1^2 \cdot p_2 & p_1^2 \cdot p_2^2 & \dots & p_1^2 \cdot p_2^{e_2} \\\\
\vdots & \vdots & \vdots & \vdots & \ddots & \vdots \\\\
p_1^{e_1} & p_1^{e_1} & p_1^{e_1} \cdot p_2 & p_1^{e_1} \cdot p_2^2 & \dots & p_1^{e_1} \cdot p_2^{e_2} \\\\
\end{array}$$

Portanto, o número de divisores é trivialmente $(e_1 + 1) \cdot (e_2 + 1)$.

* Um argumento semelhante pode ser feito se houver mais de dois fatores primos distintos.


```cpp
long long numberOfDivisors(long long num) {
    long long total = 1;
    for (int i = 2; (long long)i * i <= num; i++) {
        if (num % i == 0) {
            int e = 0;
            do {
                e++;
                num /= i;
            } while (num % i == 0);
            total *= e + 1;
        }
    }
    if (num > 1) {
        total *= 2;
    }
    return total;
}
```

## Soma dos divisores

Podemos usar o mesmo argumento da seção anterior.

* Se houver apenas um divisor primo distinto $n = p_1^{e_1}$, então a soma é:

$$1 + p_1 + p_1^2 + \dots + p_1^{e_1} = \frac{p_1^{e_1 + 1} - 1}{p_1 - 1}$$

* Se houver dois divisores primos distintos $n = p_1^{e_1} \cdot p_2^{e_2}$, então podemos fazer a mesma tabela de antes.
  A única diferença é que agora queremos calcular a soma em vez de contar os elementos.
  É fácil ver que a soma de cada combinação pode ser expressa como:

$$\left(1 + p_1 + p_1^2 + \dots + p_1^{e_1}\right) \cdot \left(1 + p_2 + p_2^2 + \dots + p_2^{e_2}\right)$$

$$ = \frac{p_1^{e_1 + 1} - 1}{p_1 - 1} \cdot \frac{p_2^{e_2 + 1} - 1}{p_2 - 1}$$

* Em geral, para $n = p_1^{e_1} \cdot p_2^{e_2} \cdots p_k^{e_k}$ recebemos a fórmula:

$$\sigma(n) = \frac{p_1^{e_1 + 1} - 1}{p_1 - 1} \cdot \frac{p_2^{e_2 + 1} - 1}{p_2 - 1} \cdots \frac{p_k^{e_k + 1} - 1}{p_k - 1}$$

```cpp
long long SumOfDivisors(long long num) {
    long long total = 1;

    for (int i = 2; (long long)i * i <= num; i++) {
        if (num % i == 0) {
            int e = 0;
            do {
                e++;
                num /= i;
            } while (num % i == 0);

            long long sum = 0, pow = 1;
            do {
                sum += pow;
                pow *= i;
            } while (e-- > 0);
            total *= sum;
        }
    }
    if (num > 1) {
        total *= (1 + num);
    }
    return total;
}
```

## Funções multiplicativas

Uma função multiplicativa é uma função $f(x)$ que satisfaz

$$f(a \cdot b) = f(a) \cdot f(b)$$

se $a$ e $b$ são coprimos.

Tanto $d(n)$ quanto $\sigma(n)$ são funções multiplicativas.

Funções multiplicativas têm uma enorme variedade de propriedades interessantes, que podem ser muito úteis em problemas de teoria dos números.
Por exemplo, a convolução de Dirichlet de duas funções multiplicativas também é multiplicativa.

## Problemas Práticos

  - [SPOJ - COMDIV](https://www.spoj.com/problems/COMDIV/)
  - [SPOJ - DIVSUM](https://www.spoj.com/problems/DIVSUM/)
  - [SPOJ - DIVSUM2](https://www.spoj.com/problems/DIVSUM2/)
