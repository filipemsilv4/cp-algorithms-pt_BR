---
tags:
  - Original
---

# Fatoração de Inteiros

Neste artigo, listamos vários algoritmos para a fatoração de inteiros, cada um podendo ser rápido ou lento em diferentes níveis, dependendo de sua entrada.

Note que, se o número que você deseja fatorar for na verdade um número primo, a maioria dos algoritmos executará muito lentamente. Isso é especialmente verdadeiro para os algoritmos de fatoração de Fermat, p-1 de Pollard e rho de Pollard.
Portanto, faz mais sentido realizar um [teste de primalidade](primality_tests.md) probabilístico (ou um determinístico rápido) antes de tentar fatorar o número.

## Divisão por tentativas

Este é o algoritmo mais básico para encontrar uma fatoração em primos.

Nós dividimos por cada divisor possível $d$.
Pode-se observar que é impossível que todos os fatores primos de um número composto $n$ sejam maiores que $\sqrt{n}$.
Portanto, precisamos apenas testar os divisores $2 \le d \le \sqrt{n}$, o que nos dá a fatoração em primos em $O(\sqrt{n})$.
(Isto é [tempo pseudo-polinomial](https://en.wikipedia.org/wiki/Pseudo-polynomial_time), ou seja, polinomial no valor da entrada, mas exponencial no número de bits da entrada.)

O menor divisor deve ser um número primo.
Nós removemos o número fatorado e continuamos o processo.
Se não conseguirmos encontrar nenhum divisor no intervalo $[2; \sqrt{n}]$, então o próprio número tem que ser primo.

```{.cpp file=factorization_trial_division1}
vector<long long> trial_division1(long long n) {
    vector<long long> factorization;
    for (long long d = 2; d * d <= n; d++) {
        while (n % d == 0) {
            factorization.push_back(d);
            n /= d;
        }
    }
    if (n > 1)
        factorization.push_back(n);
    return factorization;
}
```

### Roda de fatoração (Wheel factorization)

Esta é uma otimização da divisão por tentativas.
Uma vez que sabemos que o número não é divisível por 2, não precisamos verificar outros números pares.
Isso nos deixa com apenas $50\%$ dos números para verificar.
Depois de fatorar o 2 e obter um número ímpar, podemos simplesmente começar com 3 e contar apenas outros números ímpares.

```{.cpp file=factorization_trial_division2}
vector<long long> trial_division2(long long n) {
    vector<long long> factorization;
    while (n % 2 == 0) {
        factorization.push_back(2);
        n /= 2;
    }
    for (long long d = 3; d * d <= n; d += 2) {
        while (n % d == 0) {
            factorization.push_back(d);
            n /= d;
        }
    }
    if (n > 1)
        factorization.push_back(n);
    return factorization;
}
```

Este método pode ser estendido ainda mais.
Se o número não for divisível por 3, também podemos ignorar todos os outros múltiplos de 3 nos cálculos futuros.
Portanto, precisamos apenas verificar os números $5, 7, 11, 13, 17, 19, 23, \dots$.
Podemos observar um padrão desses números restantes.
Precisamos verificar todos os números com $d \bmod 6 = 1$ e $d \bmod 6 = 5$.
Isso nos deixa com apenas 33,3% dos números para verificar.
Podemos implementar isso fatorando primeiro os primos 2 e 3, após o qual começamos com 5 e contamos apenas os restos $1$ e $5$ módulo $6$.

Aqui está uma implementação para os números primos 2, 3 e 5.
É conveniente armazenar os saltos em um array.

```{.cpp file=factorization_trial_division3}
vector<long long> trial_division3(long long n) {
    vector<long long> factorization;
    for (int d : {2, 3, 5}) {
        while (n % d == 0) {
            factorization.push_back(d);
            n /= d;
        }
    }
    static array<int, 8> increments = {4, 2, 4, 2, 4, 6, 2, 6};
    int i = 0;
    for (long long d = 7; d * d <= n; d += increments[i++]) {
        while (n % d == 0) {
            factorization.push_back(d);
            n /= d;
        }
        if (i == 8)
            i = 0;
    }
    if (n > 1)
        factorization.push_back(n);
    return factorization;
}
```

Se continuarmos estendendo este método para incluir ainda mais primos, porcentagens melhores podem ser alcançadas, mas as listas de saltos se tornarão maiores.

### Primos pré-computados

Estendendo o método da roda de fatoração indefinidamente, restaremos apenas com números primos para verificar.
Uma boa maneira de verificar isso é pré-computar todos os números primos com o [Crivo de Eratóstenes](sieve-of-eratosthenes.md) até $\sqrt{n}$, e testá-los individualmente.

```{.cpp file=factorization_trial_division4}
vector<long long> primes;

vector<long long> trial_division4(long long n) {
    vector<long long> factorization;
    for (long long d : primes) {
        if (d * d > n)
            break;
        while (n % d == 0) {
            factorization.push_back(d);
            n /= d;
        }
    }
    if (n > 1)
        factorization.push_back(n);
    return factorization;
}
```

## Método de fatoração de Fermat

Podemos escrever um número composto ímpar $n = p \cdot q$ como a diferença de dois quadrados $n = a^2 - b^2$:

$$n = \left(\frac{p + q}{2}\right)^2 - \left(\frac{p - q}{2}\right)^2$$

O método de fatoração de Fermat tenta explorar esse fato adivinhando o primeiro quadrado $a^2$, e verificando se a parte restante, $b^2 = a^2 - n$, também é um número quadrado perfeito.
Se for, então encontramos os fatores $a - b$ e $a + b$ de $n$.

```cpp
int fermat(int n) {
    int a = ceil(sqrt(n));
    int b2 = a*a - n;
    int b = round(sqrt(b2));
    while (b * b != b2) {
        a = a + 1;
        b2 = a*a - n;
        b = round(sqrt(b2));
    }
    return a - b;
}
```

Este método de fatoração pode ser muito rápido se a diferença entre os dois fatores $p$ e $q$ for pequena.
O algoritmo roda em tempo $O(|p - q|)$.
Na prática, porém, este método raramente é usado. Uma vez que os fatores se distanciam, ele se torna extremamente lento.

No entanto, ainda há um grande número de opções de otimização em relação a essa abordagem.
Olhando para os quadrados $a^2$ módulo um número fixo pequeno, pode-se observar que certos valores de $a$ não precisam ser avaliados, já que não podem produzir um quadrado perfeito $a^2 - n$.


## Método $p - 1$ de Pollard { data-toc-label="Método <script type='math/tex'>p - 1</script> de Pollard" }

É muito provável que um número $n$ tenha pelo menos um fator primo $p$ tal que $p - 1$ seja $\mathrm{B}$**-powersmooth** para um $\mathrm{B}$ pequeno. Um inteiro $m$ é dito ser $\mathrm{B}$-powersmooth se toda potência de primo que divide $m$ for no máximo $\mathrm{B}$. Formalmente, seja $\mathrm{B} \geqslant 1$ e seja $m$ um inteiro positivo qualquer. Suponha que a fatoração em primos de $m$ seja $m = \prod {q_i}^{e_i}$, onde cada $q_i$ é um primo e $e_i \geqslant 1$. Então $m$ é $\mathrm{B}$-powersmooth se, para todo $i$, ${q_i}^{e_i} \leqslant \mathrm{B}$.
Ex.: a fatoração em primos de $4817191$ é $1303 \cdot 3697$.
E os valores, $1303 - 1$ e $3697 - 1$, são $31$-powersmooth e $16$-powersmooth respectivamente, porque $1303 - 1 = 2 \cdot 3 \cdot 7 \cdot 31$ e $3697 - 1 = 2^4 \cdot 3 \cdot 7 \cdot 11$.
Em 1974, John Pollard inventou um método para extrair fatores $p$, tal que $p-1$ é $\mathrm{B}$-powersmooth, de um número composto.

A ideia vem do [Pequeno Teorema de Fermat](phi-function.md#application).
Seja uma fatoração de $n$ dada por $n = p \cdot q$.
O teorema diz que se $a$ é coprimo de $p$, a seguinte afirmação é verdadeira:

$$a^{p - 1} \equiv 1 \pmod{p}$$

Isso também significa que

$${\left(a^{(p - 1)}\right)}^k \equiv a^{k \cdot (p - 1)} \equiv 1 \pmod{p}.$$

Portanto, para qualquer $M$ onde $p - 1 ~|~ M$, sabemos que $a^M \equiv 1$.
Isso significa que $a^M - 1 = p \cdot r$, e por causa disso também temos $p ~|~ \gcd(a^M - 1, n)$.

Assim, se $p - 1$ para um fator $p$ de $n$ dividir $M$, podemos extrair um fator usando o [Algoritmo de Euclides](euclid-algorithm.md).

Fica claro que o menor $M$ que é múltiplo de todo número $\mathrm{B}$-powersmooth é $\text{lcm}(1,~2~,3~,4~,~\dots,~B)$.
Ou alternativamente:

$$M = \prod_{\text{primo } q \le B} q^{\lfloor \log_q B \rfloor}$$

Note que, se $p-1$ dividir $M$ para todos os fatores primos $p$ de $n$, então $\gcd(a^M - 1, n)$ será apenas $n$.
Neste caso, não obtemos um fator.
Portanto, tentaremos realizar o $\gcd$ várias vezes, enquanto calculamos $M$.

Alguns números compostos não têm fatores $p$ tais que $p-1$ é $\mathrm{B}$-powersmooth para um pequeno $\mathrm{B}$.
Por exemplo, para o número composto $100~000~000~000~000~493 = 763~013 \cdot 131~059~365~961$, os valores de $p-1$ são $190~753$-powersmooth e $1~092~161~383$-powersmooth, respectivamente.
Teremos que escolher $B \geq 190~753$ para fatorar o número.

Na implementação a seguir, começamos com $\mathrm{B} = 10$ e aumentamos $\mathrm{B}$ após cada iteração.

```{.cpp file=factorization_p_minus_1}
long long pollards_p_minus_1(long long n) {
    int B = 10;
    long long g = 1;
    while (B <= 1000000 && g < n) {
        long long a = 2 + rand() %  (n - 3);
        g = gcd(a, n);
        if (g > 1)
            return g;

        // calcula a^M
        for (int p : primes) {
            if (p >= B)
                continue;
            long long p_power = 1;
            while (p_power * p <= B)
                p_power *= p;
            a = power(a, p_power, n);

            g = gcd(a - 1, n);
            if (g > 1 && g < n)
                return g;
        }
        B *= 2;
    }
    return 1;
}

```

Observe que este é um algoritmo probabilístico.
Uma consequência disso é que há uma possibilidade de o algoritmo ser incapaz de encontrar qualquer fator.

A complexidade é $O(B \log B \log^2 n)$ por iteração.

## Algoritmo rho de Pollard

O Algoritmo Rho de Pollard é mais um algoritmo de fatoração de John Pollard.

Seja a fatoração em primos de um número $n = p q$.
O algoritmo analisa uma sequência pseudoaleatória $\{x_i\} = \{x_0,~f(x_0),~f(f(x_0)),~\dots\}$ onde $f$ é uma função polinomial, geralmente escolhe-se $f(x) = (x^2 + c) \bmod n$ com $c = 1$.

Neste caso, não estamos interessados na sequência $\{x_i\}$.
Estamos mais interessados na sequência $\{x_i \bmod p\}$.
Como $f$ é uma função polinomial e todos os valores estão no intervalo $[0;~p)$, essa sequência eventualmente convergirá para um loop (ciclo).
O **paradoxo do aniversário** sugere na verdade que o número esperado de elementos é $O(\sqrt{p})$ até que a repetição comece.
Se $p$ for menor que $\sqrt{n}$, a repetição provavelmente começará em $O(\sqrt[4]{n})$.

Aqui está uma visualização de tal sequência $\{x_i \bmod p\}$ com $n = 2206637$, $p = 317$, $x_0 = 2$ e $f(x) = x^2 + 1$.
Pela forma da sequência, você pode ver muito claramente por que o algoritmo é chamado de algoritmo $\rho$ (rho) de Pollard.

<div style="text-align: center;">
  <img src="pollard_rho.png" alt="Visualização rho de Pollard">
</div>

Ainda assim, existe uma questão em aberto.
Como podemos explorar as propriedades da sequência $\{x_i \bmod p\}$ a nosso favor sem nem mesmo saber o número $p$ em si?

É na verdade bastante fácil.
Existe um ciclo na sequência $\{x_i \bmod p\}_{i \le j}$ se e somente se houver dois índices $s, t \le j$ tais que $x_s \equiv x_t \bmod p$.
Esta equação pode ser reescrita como $x_s - x_t \equiv 0 \bmod p$ o que é o mesmo que $p ~|~ \gcd(x_s - x_t, n)$.

Portanto, se encontrarmos dois índices $s$ e $t$ com $g = \gcd(x_s - x_t, n) > 1$, encontramos um ciclo e também um fator $g$ de $n$.
É possível que $g = n$.
Nesse caso, não encontramos um fator adequado, então devemos repetir o algoritmo com parâmetros diferentes (um valor inicial $x_0$ diferente, ou uma constante $c$ diferente na função polinomial $f$).

Para encontrar o ciclo, podemos usar qualquer algoritmo comum de detecção de ciclo.

### Algoritmo de Floyd para detecção de ciclo

Este algoritmo encontra um ciclo usando dois ponteiros movendo-se pela sequência a velocidades diferentes.
Durante cada iteração, o primeiro ponteiro avança um elemento, enquanto o segundo ponteiro avança para cada segundo elemento (dois a dois).
Usando essa ideia, é fácil observar que se houver um ciclo, em algum momento o segundo ponteiro dará a volta e encontrará o primeiro durante os loops.
Se o comprimento do ciclo for $\lambda$ e $\mu$ for o primeiro índice em que o ciclo começa, então o algoritmo rodará em tempo $O(\lambda + \mu)$.

Este algoritmo também é conhecido como o [Algoritmo da Tartaruga e da Lebre](../others/tortoise_and_hare.md), baseado na fábula em que uma tartaruga (o ponteiro lento) e uma lebre (o ponteiro mais rápido) disputam uma corrida.

É na verdade possível determinar os parâmetros $\lambda$ e $\mu$ usando este algoritmo (também em tempo $O(\lambda + \mu)$ e espaço $O(1)$).
Quando um ciclo é detectado, o algoritmo retornará 'True' (Verdadeiro).
Se a sequência não tiver um ciclo, a função entrará em um loop infinito.
No entanto, usando o Algoritmo Rho de Pollard, isso pode ser prevenido.

```text
function floyd(f, x0):
    tortoise = x0
    hare = f(x0)
    while tortoise != hare:
        tortoise = f(tortoise)
        hare = f(f(hare))
    return true
```

### Implementação

Primeiro, aqui está uma implementação usando o **algoritmo de Floyd para detecção de ciclo**.
O algoritmo geralmente roda em tempo $O(\sqrt[4]{n} \log(n))$.

```{.cpp file=pollard_rho}
long long mult(long long a, long long b, long long mod) {
    return (__int128)a * b % mod;
}

long long f(long long x, long long c, long long mod) {
    return (mult(x, x, mod) + c) % mod;
}

long long rho(long long n, long long x0=2, long long c=1) {
    long long x = x0;
    long long y = x0;
    long long g = 1;
    while (g == 1) {
        x = f(x, c, n);
        y = f(y, c, n);
        y = f(y, c, n);
        g = gcd(abs(x - y), n);
    }
    return g;
}
```

A tabela a seguir mostra os valores de $x$ e $y$ durante o algoritmo para $n = 2206637$, $x_0 = 2$ e $c = 1$.

$$
\newcommand\T{\Rule{0pt}{1em}{.3em}}
\begin{array}{|l|l|l|l|l|l|}
\hline
i & x_i \bmod n & x_{2i} \bmod n & x_i \bmod 317 & x_{2i} \bmod 317 & \gcd(x_i - x_{2i}, n) \\
\hline
0   & 2       & 2       & 2       & 2       & -   \\
1   & 5       & 26      & 5       & 26      & 1   \\
2   & 26      & 458330  & 26      & 265     & 1   \\
3   & 677     & 1671573 & 43      & 32      & 1   \\
4   & 458330  & 641379  & 265     & 88      & 1   \\
5   & 1166412 & 351937  & 169     & 67      & 1   \\
6   & 1671573 & 1264682 & 32      & 169     & 1   \\
7   & 2193080 & 2088470 & 74      & 74      & 317 \\
\hline
\end{array}$$

A implementação usa uma função `mult`, que multiplica dois inteiros $\le 10^{18}$ sem overflow usando o tipo `__int128` do GCC para inteiros de 128 bits.
Se o GCC não estiver disponível, você pode usar uma ideia semelhante à da [exponenciação binária](binary-exp.md).

```{.cpp file=pollard_rho_mult2}
long long mult(long long a, long long b, long long mod) {
    long long result = 0;
    while (b) {
        if (b & 1)
            result = (result + a) % mod;
        a = (a + a) % mod;
        b >>= 1;
    }
    return result;
}
```

Alternativamente, você também pode implementar a [multiplicação de Montgomery](montgomery_multiplication.md).

Conforme dito anteriormente, se $n$ é composto e o algoritmo retorna $n$ como fator, você tem que repetir o procedimento com parâmetros diferentes $x_0$ e $c$.
Ex.: a escolha $x_0 = c = 1$ não fatorará $25 = 5 \cdot 5$.
O algoritmo retornará $25$.
No entanto, a escolha $x_0 = 1$, $c = 2$ o fatorará.

### Algoritmo de Brent

Brent implementa um método semelhante ao de Floyd, usando dois ponteiros.
A diferença é que, em vez de avançar os ponteiros em uma e duas posições respectivamente, eles são avançados por potências de dois.
Assim que $2^i$ for maior que $\lambda$ e $\mu$, encontraremos o ciclo.

```text
function floyd(f, x0):
    tortoise = x0
    hare = f(x0)
    l = 1
    while tortoise != hare:
        tortoise = hare
        repeat l times:
            hare = f(hare)
            if tortoise == hare:
                return true
        l *= 2
    return true
```

O algoritmo de Brent também roda em tempo linear, mas geralmente é mais rápido que o de Floyd, pois usa menos avaliações da função $f$.

### Implementação

A implementação direta do algoritmo de Brent pode ser acelerada omitindo os termos $x_l - x_k$ se $k < \frac{3 \cdot l}{2}$.
Além disso, em vez de realizar o cálculo do $\gcd$ a cada passo, nós multiplicamos os termos e só checamos o $\gcd$ de fato a cada poucos passos, retrocedendo caso passe do limite (overshot).

```{.cpp file=pollard_rho_brent}
long long brent(long long n, long long x0=2, long long c=1) {
    long long x = x0;
    long long g = 1;
    long long q = 1;
    long long xs, y;

    int m = 128;
    int l = 1;
    while (g == 1) {
        y = x;
        for (int i = 1; i < l; i++)
            x = f(x, c, n);
        int k = 0;
        while (k < l && g == 1) {
            xs = x;
            for (int i = 0; i < m && i < l - k; i++) {
                x = f(x, c, n);
                q = mult(q, abs(y - x), n);
            }
            g = gcd(q, n);
            k += m;
        }
        l *= 2;
    }
    if (g == n) {
        do {
            xs = f(xs, c, n);
            g = gcd(abs(xs - y), n);
        } while (g == 1);
    }
    return g;
}
```

A combinação de uma divisão por tentativas para primos pequenos junto com a versão de Brent do algoritmo rho de Pollard forma um algoritmo de fatoração muito poderoso.

## Problemas Práticos

- [SPOJ - FACT0](https://www.spoj.com/problems/FACT0/)
- [SPOJ - FACT1](https://www.spoj.com/problems/FACT1/)
- [SPOJ - FACT2](https://www.spoj.com/problems/FACT2/)
- [GCPC 15 - Divisions](https://codeforces.com/gym/100753)
