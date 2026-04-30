---
tags:
    - Original
---

# Testes de primalidade

Este artigo descreve vários algoritmos para determinar se um número é primo ou não.

## Divisão por tentativas

Por definição, um número primo não tem divisores além do $1$ e ele mesmo.
Um número composto tem pelo menos um divisor adicional, vamos chamá-lo de $d$.
Naturalmente $\frac{n}{d}$ também é um divisor de $n$.
É fácil perceber que, ou $d \le \sqrt{n}$ ou $\frac{n}{d} \le \sqrt{n}$, portanto, um dos divisores $d$ e $\frac{n}{d}$ é $\le \sqrt{n}$.
Podemos usar esta informação para verificar a primalidade.

Tentamos encontrar um divisor não trivial, verificando se algum dos números entre $2$ e $\sqrt{n}$ é um divisor de $n$.
Se for um divisor, então $n$ definitivamente não é primo; caso contrário, é primo.

```cpp
bool isPrime(int x) {
    for (int d = 2; d * d <= x; d++) {
        if (x % d == 0)
            return false;
    }
    return x >= 2;
}
```

Esta é a forma mais simples de verificação de primos.
Você pode otimizar esta função consideravelmente, por exemplo, verificando apenas números ímpares no laço de repetição, já que o único número primo par é o 2.
Múltiplas otimizações como esta são descritas no artigo sobre [fatoração de inteiros](factorization.md).

## Teste de primalidade de Fermat

Este é um teste probabilístico.

O Pequeno Teorema de Fermat (veja também a [Função Totiente de Euler](phi-function.md)) afirma que, para um número primo $p$ e um inteiro coprimo $a$, a seguinte equação é verdadeira:

$$a^{p-1} \equiv 1 \bmod p$$

Em geral, este teorema não é verdadeiro para números compostos.

Isso pode ser usado para criar um teste de primalidade.
Escolhemos um inteiro $2 \le a \le p - 2$, e verificamos se a equação é verdadeira ou não.
Se não for verdadeira, ou seja, $a^{p-1} \not\equiv 1 \bmod p$, sabemos que $p$ não pode ser um número primo.
Neste caso, chamamos a base $a$ de uma *testemunha de Fermat* (Fermat witness) para a compositividade de $p$.

No entanto, também é possível que a equação seja verdadeira para um número composto.
Então, se a equação for verdadeira, não temos uma prova de primalidade.
Só podemos dizer que $p$ é *provavelmente primo*.
Se acontecer do número ser, de fato, composto, chamamos a base $a$ de um *mentiroso de Fermat* (Fermat liar).

Ao executar o teste para todas as possíveis bases $a$, podemos, de fato, provar que um número é primo.
Entretanto, isso não é feito na prática, já que dá muito mais trabalho do que simplesmente fazer uma *divisão por tentativas*.
Em vez disso, o teste será repetido várias vezes com escolhas aleatórias para $a$.
Se não encontrarmos testemunhas para a compositividade, é muito provável que o número seja realmente primo.

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

Usamos a [Exponenciação Binária](binary-exp.md) para calcular a potência $a^{p-1}$ de maneira eficiente.

Há uma má notícia, entretanto:
existem alguns números compostos nos quais $a^{n-1} \equiv 1 \bmod n$ é verdadeiro para todo $a$ coprimo com $n$, por exemplo, para o número $561 = 3 \cdot 11 \cdot 17$.
Tais números são chamados de *números de Carmichael*.
O teste de primalidade de Fermat só pode identificar esses números se tivermos muita sorte e escolhermos uma base $a$ com $\gcd(a, n) \ne 1$.

O teste de Fermat ainda é usado na prática, por ser muito rápido e porque os números de Carmichael são muito raros.
Por exemplo, existem apenas 646 desses números abaixo de $10^9$.

## Teste de primalidade de Miller-Rabin

O teste de Miller-Rabin estende as ideias do teste de Fermat.

Para um número ímpar $n$, $n-1$ é par e podemos fatorar todas as potências de 2.
Podemos escrever:

$$n - 1 = 2^s \cdot d,~\text{com}~d~\text{ímpar}.$$

Isso nos permite fatorar a equação do Pequeno Teorema de Fermat:

$$\begin{array}{rl}
a^{n-1} \equiv 1 \bmod n &\Longleftrightarrow a^{2^s d} - 1 \equiv 0 \bmod n \\\\
&\Longleftrightarrow (a^{2^{s-1} d} + 1) (a^{2^{s-1} d} - 1) \equiv 0 \bmod n \\\\
&\Longleftrightarrow (a^{2^{s-1} d} + 1) (a^{2^{s-2} d} + 1) (a^{2^{s-2} d} - 1) \equiv 0 \bmod n \\\\
&\quad\vdots \\\\
&\Longleftrightarrow (a^{2^{s-1} d} + 1) (a^{2^{s-2} d} + 1) \cdots (a^{d} + 1) (a^{d} - 1) \equiv 0 \bmod n \\\\
\end{array}$$

Se $n$ é primo, então $n$ tem que dividir um desses fatores.
E no teste de primalidade de Miller-Rabin nós verificamos exatamente essa afirmação, que é uma versão mais restrita da afirmação do teste de Fermat.
Para uma base $2 \le a \le n-2$, verificamos se a seguinte equação

$$a^d \equiv 1 \bmod n$$

é verdadeira ou se

$$a^{2^r d} \equiv -1 \bmod n$$

é verdadeira para algum $0 \le r \le s - 1$.

Se encontrarmos uma base $a$ que não satisfaz nenhuma das igualdades acima, então encontramos uma *testemunha* da compositividade de $n$.
Neste caso, provamos que $n$ não é um número primo.

Similar ao teste de Fermat, também é possível que o conjunto de equações seja satisfeito para um número composto.
Nesse caso, a base $a$ é chamada de *mentiroso forte* (strong liar).
Se uma base $a$ satisfaz as equações (uma delas), $n$ é apenas um *forte provável primo*.
No entanto, não há números como os números de Carmichael, onde todas as bases não triviais mentem.
Na verdade, é possível provar que, no máximo $\frac{1}{4}$ das bases podem ser fortes mentirosos.
Se $n$ é composto, temos uma probabilidade de $\ge 75\%$ que uma base aleatória nos diga que ele é composto.
Ao fazer várias iterações, escolhendo diferentes bases aleatórias, podemos dizer com uma probabilidade muito alta se o número é realmente primo ou se é composto.

Aqui está uma implementação para um inteiro de 64 bits.

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

bool MillerRabin(u64 n, int iter=5) { // retorna true se n é provavelmente primo, caso contrário retorna false.
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
Isso pode acelerar muito o teste, pois a maioria dos números compostos tem divisores primos muito pequenos.
Por exemplo, $88\%$ de todos os números têm um fator primo menor que $100$.

### Versão determinística

Miller mostrou que é possível tornar o algoritmo determinístico testando apenas todas as bases $\le O((\ln n)^2)$.
Bach mais tarde deu um limite concreto, só é necessário testar todas as bases $a \le 2 \ln(n)^2$.

Ainda assim, é um número bem grande de bases.
Portanto, muita capacidade computacional foi investida na busca por limites inferiores.
Acontece que, para testar um inteiro de 32 bits, só é necessário verificar os 4 primeiros números primos: 2, 3, 5 e 7.
O menor número composto que falha neste teste é $3.215.031.751 = 151 \cdot 751 \cdot 28351$.
E para testar um inteiro de 64 bits, basta testar os 12 primeiros primos: 2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31 e 37.

Isso resulta na seguinte implementação determinística:

```cpp
bool MillerRabin(u64 n) { // retorna true se n é primo, caso contrário retorna false.
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
Entretanto, como esses números (exceto 2) não são primos, você precisa verificar adicionalmente se o número que está verificando é igual a qualquer divisor primo dessas bases: 2, 3, 5, 13, 19, 73, 193, 407521, 299210837.

## Problemas Práticos

- [SPOJ - Prime or Not](https://www.spoj.com/problems/PON/)
- [Project euler - Investigating a Prime Pattern](https://projecteuler.net/problem=146)