---
tags:
  - Translated
e_maxx_link: diofant_1_equation
---

# Equações de Congruência Linear

Esta equação é da forma:

$$a \cdot x \equiv b \pmod n,$$

onde $a$, $b$ e $n$ são inteiros dados e $x$ é um inteiro desconhecido.

É necessário encontrar o valor $x$ do intervalo $[0, n-1]$ (claramente, em toda a reta numérica pode haver infinitas soluções que diferirão umas das outras em $n \cdot k$ , onde $k$ é qualquer inteiro). Se a solução não for única, consideraremos como obter todas as soluções.

## Solução encontrando o elemento inverso

Vamos primeiro considerar um caso mais simples onde $a$ e $n$ são **coprimos** ($\gcd(a, n) = 1$).
Então podemos encontrar o [inverso](module-inverse.md) de $a$, e multiplicando ambos os lados da equação pelo inverso, podemos obter uma solução **única**.

$$x \equiv b \cdot a ^ {- 1} \pmod n$$

Agora considere o caso em que $a$ e $n$ **não são coprimos** ($\gcd(a, n) \ne 1$).
Então a solução nem sempre existirá (por exemplo, $2 \cdot x \equiv 1 \pmod 4$ não tem solução).

Seja $g = \gcd(a, n)$, ou seja, o [máximo divisor comum](euclid-algorithm.md) de $a$ e $n$ (que neste caso é maior que um).

Então, se $b$ não for divisível por $g$, não há solução. De fato, para qualquer $x$ o lado esquerdo da equação $a \cdot x \pmod n$ , é sempre divisível por $g$, enquanto o lado direito não é divisível por ele, logo não há soluções.

Se $g$ divide $b$, então dividindo ambos os lados da equação por $g$ (ou seja, dividindo $a$, $b$ e $n$ por $g$), recebemos uma nova equação:

$$a^\prime \cdot x \equiv b^\prime \pmod{n^\prime}$$

na qual $a^\prime$ e $n^\prime$ já são relativamente primos, e já aprendemos a lidar com uma equação desse tipo.
Obtemos $x^\prime$ como solução para $x$.

É claro que esse $x^\prime$ também será uma solução da equação original.
No entanto, **não será a única solução**.
Pode-se mostrar que a equação original tem exatamente $g$ soluções, e elas ficarão assim:

$$x_i \equiv (x^\prime + i\cdot n^\prime) \pmod n \quad \text{para } i = 0 \ldots g-1$$

Resumindo, podemos dizer que o **número de soluções** da equação de congruência linear é igual a $g = \gcd(a, n)$ ou a zero.

## Solução com o Algoritmo de Euclides Estendido

Podemos reescrever a congruência linear para a seguinte equação Diofantina:

$$a \cdot x + n \cdot k = b,$$

onde $x$ e $k$ são inteiros desconhecidos.

O método de resolução dessa equação é descrito no artigo correspondente [Equações Diofantinas Lineares](linear-diophantine-equation.md) e consiste na aplicação do [Algoritmo de Euclides Estendido](extended-euclid-algorithm.md).

Ele também descreve o método de obter todas as soluções dessa equação a partir de uma solução encontrada, e incidentalmente esse método, quando considerado cuidadosamente, é absolutamente equivalente ao método descrito na seção anterior.
