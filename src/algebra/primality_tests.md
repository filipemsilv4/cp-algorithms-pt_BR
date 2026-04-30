---
tags:
    - Original
---

# Testes de primalidade

Este artigo descreve vários algoritmos para determinar se um número é primo ou não.

## Divisão por tentativa

Por definição, um número primo não tem divisores além de $1$ e dele mesmo.
Um número composto tem pelo menos um divisor adicional, vamos chamá-lo de $d$.
Naturalmente, $\frac{n}{d}$ também é um divisor de $n$.
É fácil ver que ou $d \le \sqrt{n}$ ou $\frac{n}{d} \le \sqrt{n}$, portanto, um dos divisores $d$ e $\frac{n}{d}$ é $\le \sqrt{n}$.
Podemos usar essa informação para verificar a primalidade.

Tentamos encontrar um divisor não trivial, verificando se algum dos números entre $2$ e $\sqrt{n}$ é um divisor de $n$.
Se for um divisor, então $n$ definitivamente não é primo, caso contrário, ele é.

```cpp
bool isPrime(int x) {
    for (int d = 2; d * d <= x; d++) {
        if (x % d == 0)
            return false;
    }
    return x >= 2;
}
```

Esta é a forma mais simples de verificar primalidade.
Você pode otimizar bastante essa função, por exemplo, verificando apenas todos os números ímpares no loop, já que o único número primo par é 2.
Várias dessas otimizações são descritas no artigo sobre [fatoração de inteiros](factorization.md).

## Teste de primalidade de Fermat

Este é um teste probabilístico.

O pequeno teorema de Fermat (veja também [Função totiente de Euler](phi-function.md)) afirma que, para um número primo $p$ e um inteiro coprimo $a$, a seguinte equação é válida:

$$a^{p-1} \equiv 1 \bmod p$$

Em geral, este teorema não se aplica a números compostos.

Isso pode ser usado para criar um teste de primalidade.
Escolhemos um inteiro $2 \le a \le p - 2$ e verificamos se a equação é válida ou não.
Se não for, ou seja, $a^{p-1} \not\equiv 1 \bmod p$, sabemos que $p$ não pode ser um número primo.
Neste caso, chamamos a base $a$ de *testemunha de Fermat* (Fermat witness) para a composição de $p$.

No entanto, também é possível que a equação seja válida para um número composto.
Portanto, se a equação for válida, não temos uma prova de primalidade.
Podemos apenas dizer que $p$ é *provavelmente primo*.
Se descobrirmos que o número é realmente composto, chamamos a base $a$ de *mentiroso de Fermat* (Fermat liar).

Ao executar o teste para todas as bases possíveis $a$, podemos realmente provar que um número é primo.
No entanto, isso não é feito na prática, já que exige muito mais esforço do que apenas fazer a *divisão por tentativa*.
Em vez disso, o teste será repetido várias vezes com escolhas aleatórias para $a$.
Se não encontrarmos nenhuma testemunha para a composição, é muito provável que o número seja de fato primo.

```cpp
bool probablyPrimeFermat(int n, int iter=5) {
    if (n < 4)
        return n == 2 || n == 3;

    for (int i = 0; i < iter; i++) {
        int a = 2 + rand() % (n - 3);
        if (binpower(a, n - 1, n) != 1)
            return false;
    }
    return true;
}
```

Usamos a [Exponenciação Binária](binary-exp.md) para calcular eficientemente a potência $a^{p-1}$.

Mas há uma má notícia:
existem alguns números compostos em que $a^{n-1} \equiv 1 \bmod n$ é válido para todos os $a$ coprimos a $n$, por exemplo, para o número $561 = 3 \cdot 11 \cdot 17$.
Tais números são chamados de *números de Carmichael*.
O teste de primalidade de Fermat só pode identificar esses números se tivermos muita sorte e escolhermos uma base $a$ com $\gcd(a, n) \ne 1$.

O teste de Fermat ainda é usado na prática, pois é muito rápido e os números de Carmichael são muito raros.
Por exemplo, existem apenas 646 desses números abaixo de $10^9$.

## Teste de primalidade de Miller-Rabin

O teste de Miller-Rabin estende as ideias do teste de Fermat.

Para um número ímpar $n$, $n-1$ é par e podemos fatorar todas as potências de 2.
Podemos escrever:

$$n - 1 = 2^s \cdot d,~\text{com}~d~\text{ímpar}.$$

Isso nos permite fatorar a equação do pequeno teorema de Fermat:

$$\begin{array}{rl}
a^{n-1} \equiv 1 \bmod n &\Longleftrightarrow a^{2^s d} - 1 \equiv 0 \bmod n \\\\
&\Longleftrightarrow (a^{2^{s-1} d} + 1) (a^{2^{s-1} d} - 1) \equiv 0 \bmod n \\\\
&\Longleftrightarrow (a^{2^{s-1} d} + 1) (a^{2^{s-2} d} + 1) (a^{2^{s-2} d} - 1) \equiv 0 \bmod n \\\\
&\quad\vdots \\\\
&\Longleftrightarrow (a^{2^{s-1} d} + 1) (a^{2^{s-2} d} + 1) \cdots (a^{d} + 1) (a^{d} - 1) \equiv 0 \bmod n \\\\
\end{array}$$

Se $n$ é primo, então $n$ tem que dividir um desses fatores.
E no teste de primalidade de Miller-Rabin verificamos exatamente essa afirmação, que é uma versão mais rigorosa da afirmação do teste de Fermat.
Para uma base $2 \le a \le n-2$, verificamos se

$$a^d \equiv 1 \bmod n$$

ou se

$$a^{2^r d} \equiv -1 \bmod n$$

é válido para algum $0 \le r \le s - 1$.

Se encontrarmos uma base $a$ que não satisfaz nenhuma das igualdades acima, então encontramos uma *testemunha* da composição de $n$.
Neste caso, provamos que $n$ não é um número primo.

Semelhante ao teste de Fermat, também é possível que o conjunto de equações seja satisfeito para um número composto.
Nesse caso, a base $a$ é chamada de *mentiroso forte* (strong liar).
Se uma base $a$ satisfizer as equações (uma delas), $n$ é apenas um *primo provável forte*.
No entanto, não existem números como os números de Carmichael, onde todas as bases não triviais mentem.
Na verdade, é possível mostrar que no máximo $\frac{1}{4}$ das bases podem ser mentirosas fortes.
Se $n$ for composto, temos uma probabilidade de $\ge 75\%$ de que uma base aleatória nos dirá que ele é composto.
Fazendo várias iterações, escolhendo diferentes bases aleatórias, podemos dizer com uma probabilidade muito alta se o número é realmente primo ou se é composto.

Aqui está uma implementação para inteiros de 64 bits.

```cpp
using u64 = uint64_t;
using u128 = __uint128_t;

u64 binpower(u64 base, u64 e, u64 mod) {
    u64 result = 1;
    base %= mod;
    while (e) {
        if (e & 1)
            result = (u128)result * base % mod;
        base = (u128)base * base % mod;
        e >>= 1;
    }
    return result;
}

bool check_composite(u64 n, u64 a, u64 d, int s) {
    u64 x = binpower(a, d, n);
    if (x == 1 || x == n - 1)
        return false;
    for (int r = 1; r < s; r++) {
        x = (u128)x * x % n;
        if (x == n - 1)
            return false;
    }
    return true;
};

bool MillerRabin(u64 n, int iter=5) { // retorna verdadeiro se n é provavelmente primo, senão retorna falso.
    if (n < 4)
        return n == 2 || n == 3;

    int s = 0;
    u64 d = n - 1;
    while ((d & 1) == 0) {
        d >>= 1;
        s++;
    }

    for (int i = 0; i < iter; i++) {
        int a = 2 + rand() % (n - 3);
        if (check_composite(n, a, d, s))
            return false;
    }
    return true;
}
```

Antes do teste de Miller-Rabin, você pode testar adicionalmente se um dos primeiros números primos é um divisor.
Isso pode acelerar muito o teste, já que a maioria dos números compostos tem divisores primos muito pequenos.
Por exemplo, $88\%$ de todos os números têm um fator primo menor que $100$.

### Versão determinística

Miller mostrou que é possível tornar o algoritmo determinístico verificando apenas todas as bases $\le O((\ln n)^2)$.
Bach mais tarde forneceu um limite concreto, é necessário apenas testar todas as bases $a \le 2 \ln(n)^2$.

Ainda é um número muito grande de bases.
Portanto, as pessoas investiram muito poder de computação para encontrar limites inferiores.
Acontece que, para testar um inteiro de 32 bits, só é necessário verificar as primeiras 4 bases primas: 2, 3, 5 e 7.
O menor número composto que falha neste teste é $3.215.031.751 = 151 \cdot 751 \cdot 28351$.
E para testar inteiros de 64 bits é suficiente verificar as primeiras 12 bases primas: 2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31 e 37.

Isso resulta na seguinte implementação determinística:

```cpp
bool MillerRabin(u64 n) { // retorna verdadeiro se n for primo, senão retorna falso.
    if (n < 2)
        return false;

    int r = 0;
    u64 d = n - 1;
    while ((d & 1) == 0) {
        d >>= 1;
        r++;
    }

    for (int a : {2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37}) {
        if (n == a)
            return true;
        if (check_composite(n, a, d, r))
            return false;
    }
    return true;
}
```

Também é possível fazer a verificação com apenas 7 bases: 2, 325, 9375, 28178, 450775, 9780504 e 1795265022.
No entanto, como esses números (exceto 2) não são primos, você precisa verificar adicionalmente se o número que você está testando é igual a qualquer divisor primo dessas bases: 2, 3, 5, 13, 19, 73, 193, 407521, 299210837.

## Problemas Práticos

- [SPOJ - Prime or Not](https://www.spoj.com/problems/PON/)
- [Project euler - Investigating a Prime Pattern](https://projecteuler.net/problem=146)
