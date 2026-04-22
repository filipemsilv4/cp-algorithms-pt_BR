---
tags:
  - Translated
e_maxx_link: euclid_algorithm
---

# Algoritmo de Euclides para calcular o máximo divisor comum

Dados dois inteiros não-negativos $a$ e $b$, temos que encontrar seu **MDC** (máximo divisor comum), ou seja, o maior número que é divisor tanto de $a$ quanto de $b$.
É comumente denotado por $\gcd(a, b)$ ou em português, $\text{mdc}(a, b)$. Matematicamente, ele é definido como:

$$\gcd(a, b) = \max \{k > 0 : (k \mid a) \text{ e } (k \mid b) \}$$

(aqui o símbolo "$\mid$" denota divisibilidade, ou seja, "$k \mid a$" significa que "$k$ divide $a$")

Quando um dos números é zero, enquanto o outro é não-zero, o máximo divisor comum deles, por definição, é o segundo número. Quando ambos os números são zero, seu máximo divisor comum é indefinido (pode ser qualquer número arbitrariamente grande), mas é conveniente defini-lo como zero também para preservar a associatividade do $\gcd$. O que nos dá uma regra simples: se um dos números for zero, o máximo divisor comum é o outro número.

O algoritmo de Euclides, discutido abaixo, permite encontrar o máximo divisor comum de dois números $a$ e $b$ em $O(\log \min(a, b))$. Como a função é **associativa**, para encontrar o MDC de **mais de dois números**, podemos fazer $\gcd(a, b, c) = \gcd(a, \gcd(b, c))$ e assim por diante.

O algoritmo foi descrito pela primeira vez em "Os Elementos" de Euclides (cerca de 300 a.C.), mas é possível que o algoritmo tenha origens ainda mais antigas.

## Algoritmo

Originalmente, o algoritmo de Euclides foi formulado da seguinte maneira: subtraia o número menor do número maior até que um dos números seja zero. De fato, se $g$ divide $a$ e $b$, ele também divide $a-b$. Por outro lado, se $g$ divide $a-b$ e $b$, então ele também divide $a = b + (a-b)$, o que significa que os conjuntos dos divisores comuns de $\{a, b\}$ e $\{b,a-b\}$ coincidem.

Note que $a$ permanece como o número maior até que $b$ seja subtraído dele pelo menos $\left\lfloor\frac{a}{b}\right\rfloor$ vezes. Portanto, para acelerar o processo, $a-b$ é substituído por $a-\left\lfloor\frac{a}{b}\right\rfloor b = a \bmod b$. Então, o algoritmo é formulado de maneira extremamente simples:

$$\gcd(a, b) = \begin{cases}a,&\text{se }b = 0 \\ \gcd(b, a \bmod b),&\text{caso contrário.}\end{cases}$$

## Implementation

```cpp
int gcd (int a, int b) {
    if (b == 0)
        return a;
    else
        return gcd (b, a % b);
}
```

Usando o operador ternário em C++, podemos escrevê-lo em uma única linha.

```cpp
int gcd (int a, int b) {
    return b ? gcd (b, a % b) : a;
}
```

E finalmente, aqui está uma implementação não-recursiva:

```cpp
int gcd (int a, int b) {
    while (b) {
        a %= b;
        swap(a, b);
    }
    return a;
}
```

Note que, desde o C++17, o `gcd` é implementado como uma [função padrão](https://en.cppreference.com/w/cpp/numeric/gcd) em C++.

## Complexidade de Tempo

O tempo de execução do algoritmo é estimado pelo teorema de Lamé, que estabelece uma conexão surpreendente entre o algoritmo de Euclides e a sequência de Fibonacci:

Se $a > b \geq 1$ e $b < F_n$ para algum $n$, o algoritmo de Euclides realiza no máximo $n-2$ chamadas recursivas.

Além disso, é possível mostrar que o limite superior deste teorema é ótimo. Quando $a = F_n$ e $b = F_{n-1}$, $\gcd(a, b)$ realizará exatamente $n-2$ chamadas recursivas. Em outras palavras, números consecutivos de Fibonacci são a entrada de pior caso para o algoritmo de Euclides.

Dado que os números de Fibonacci crescem exponencialmente, obtemos que o algoritmo de Euclides funciona em $O(\log \min(a, b))$.

Outra maneira de estimar a complexidade é notar que $a \bmod b$ para o caso $a \geq b$ é pelo menos $2$ vezes menor que $a$, portanto, o número maior é reduzido pela metade em cada iteração do algoritmo. Aplicando este raciocínio para o caso em que calculamos o MDC do conjunto de números $a_1,\dots,a_n \leq C$, isto também nos permite estimar o tempo de execução total como $O(n + \log C)$, ao invés de $O(n \log C)$, pois cada iteração não-trivial do algoritmo reduz o candidato atual a MDC por um fator de pelo menos $2$.

## Mínimo múltiplo comum

O cálculo do mínimo múltiplo comum (comumente denotado como **MMC**) pode ser reduzido ao cálculo do MDC com a seguinte fórmula simples:

$$\text{lcm}(a, b) = \frac{a \cdot b}{\gcd(a, b)}$$

Assim, o MMC pode ser calculado usando o algoritmo de Euclides com a mesma complexidade de tempo:

Uma possível implementação, que evita inteligentemente overflow de inteiros dividindo primeiro $a$ com o MDC, é dada aqui:

```cpp
int lcm (int a, int b) {
    return a / gcd(a, b) * b;
}
```

## MDC Binário

O algoritmo de MDC Binário é uma otimização do algoritmo de Euclides normal.

A parte lenta do algoritmo normal são as operações de módulo. Operações de módulo, embora as vejamos como $O(1)$, são muito mais lentas que operações mais simples como adição, subtração ou operações bitwise.
Então seria melhor evitá-las.

Acontece que você pode projetar um algoritmo rápido de MDC que evita operações de módulo.
É baseado em algumas propriedades:

  - Se ambos os números forem pares, podemos fatorar um dois de ambos e calcular o MDC dos números restantes: $\gcd(2a, 2b) = 2 \gcd(a, b)$.
  - Se um dos números for par e o outro for ímpar, então podemos remover o fator 2 do número par: $\gcd(2a, b) = \gcd(a, b)$ se $b$ for ímpar.
  - Se ambos os números forem ímpares, então a subtração de um número do outro não alterará o MDC: $\gcd(a, b) = \gcd(b, a-b)$

Usando apenas essas propriedades, e algumas funções bitwise rápidas do GCC, podemos implementar uma versão rápida:

```cpp
int gcd(int a, int b) {
    if (!a || !b)
        return a | b;
    unsigned shift = __builtin_ctz(a | b);
    a >>= __builtin_ctz(a);
    do {
        b >>= __builtin_ctz(b);
        if (a > b)
            swap(a, b);
        b -= a;
    } while (b);
    return a << shift;
}
```

Note que essa otimização geralmente não é necessária, e a maioria das linguagens de programação já possui uma função de MDC em suas bibliotecas padrão.
Por exemplo, C++17 possui essa função `std::gcd` no cabeçalho `numeric`.

## Problemas Práticos

- [CSAcademy - Greatest Common Divisor](https://csacademy.com/contest/archive/task/gcd/)
- [Codeforces 1916B - Two Divisors](https://codeforces.com/contest/1916/problem/B)