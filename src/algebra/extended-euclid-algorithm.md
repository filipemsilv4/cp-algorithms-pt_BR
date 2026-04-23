---
tags:
  - Translated
e_maxx_link: extended_euclid_algorithm
---

# Algoritmo de Euclides Estendido

Enquanto o [Algoritmo de Euclides](euclid-algorithm.md) calcula apenas o máximo divisor comum (GCD, ou em português, MDC) de dois inteiros $a$ e $b$, a versão estendida também encontra uma maneira de representar o GCD em termos de $a$ e $b$, ou seja, coeficientes $x$ e $y$ para os quais:

$$a \cdot x + b \cdot y = \gcd(a, b)$$

É importante notar que, pela [Identidade de Bézout](https://pt.wikipedia.org/wiki/Identidade_de_B%C3%A9zout) sempre podemos encontrar tal representação. Por exemplo, $\gcd(55, 80) = 5$, portanto podemos representar $5$ como uma combinação linear com os termos $55$ e $80$: $55 \cdot 3 + 80 \cdot (-2) = 5$

Uma forma mais geral desse problema é discutida no artigo sobre [Equações Diofantinas Lineares](linear-diophantine-equation.md).
Isso será construído sobre este algoritmo.

## Algoritmo

Denotaremos o GCD de $a$ e $b$ por $g$ nesta seção.

As alterações no algoritmo original são muito simples.
Se nos lembrarmos do algoritmo, podemos ver que o algoritmo termina com $b = 0$ e $a = g$.
Para esses parâmetros podemos facilmente encontrar os coeficientes, a saber $g \cdot 1 + 0 \cdot 0 = g$.

Partindo desses coeficientes $(x, y) = (1, 0)$, podemos retroceder pelas chamadas recursivas.
Tudo o que precisamos fazer é descobrir como os coeficientes $x$ e $y$ mudam durante a transição de $(a, b)$ para $(b, a \bmod b)$.

Vamos supor que encontramos os coeficientes $(x_1, y_1)$ para $(b, a \bmod b)$:

$$b \cdot x_1 + (a \bmod b) \cdot y_1 = g$$

e queremos encontrar o par $(x, y)$ para $(a, b)$:

$$ a \cdot x + b \cdot y = g$$

Podemos representar $a \bmod b$ como:

$$ a \bmod b = a - \left\lfloor \frac{a}{b} \right\rfloor \cdot b$$

Substituindo esta expressão na equação de coeficientes de $(x_1, y_1)$ nos dá:

$$ g = b \cdot x_1 + (a \bmod b) \cdot y_1 = b \cdot x_1 + \left(a - \left\lfloor \frac{a}{b} \right\rfloor \cdot b \right) \cdot y_1$$

e depois de rearranjar os termos:

$$g = a \cdot y_1 + b \cdot \left( x_1 - y_1 \cdot \left\lfloor \frac{a}{b} \right\rfloor \right)$$

Encontramos os valores de $x$ e $y$:

$$\begin{cases}
x = y_1 \\
y = x_1 - y_1 \cdot \left\lfloor \frac{a}{b} \right\rfloor
\end{cases} $$

## Implementação

```{.cpp file=extended_gcd}
int gcd(int a, int b, int& x, int& y) {
    if (b == 0) {
        x = 1;
        y = 0;
        return a;
    }
    int x1, y1;
    int d = gcd(b, a % b, x1, y1);
    x = y1;
    y = x1 - y1 * (a / b);
    return d;
}
```

A função recursiva acima retorna o GCD e os valores dos coeficientes para `x` e `y` (que são passados por referência para a função).

Esta implementação do algoritmo de Euclides estendido produz resultados corretos também para inteiros negativos.

## Versão iterativa

Também é possível escrever o algoritmo de Euclides Estendido de forma iterativa.
Como evita recursão, o código rodará um pouco mais rápido que o recursivo.

```{.cpp file=extended_gcd_iter}
int gcd(int a, int b, int& x, int& y) {
    x = 1, y = 0;
    int x1 = 0, y1 = 1, a1 = a, b1 = b;
    while (b1) {
        int q = a1 / b1;
        tie(x, x1) = make_tuple(x1, x - q * x1);
        tie(y, y1) = make_tuple(y1, y - q * y1);
        tie(a1, b1) = make_tuple(b1, a1 - q * b1);
    }
    return a1;
}
```

Se você observar atentamente as variáveis `a1` e `b1`, perceberá que elas assumem exatamente os mesmos valores que na versão iterativa do [Algoritmo de Euclides](euclid-algorithm.md#implementation) normal. Então o algoritmo pelo menos calculará o GCD corretamente.

Para ver por que o algoritmo calcula os coeficientes corretos, considere que as seguintes invariantes se mantêm a qualquer momento (antes do início do loop `while` e no final de cada iteração):

$$x \cdot a + y \cdot b = a_1$$

$$x_1 \cdot a + y_1 \cdot b = b_1$$

Deixe os valores no final de uma iteração serem indicados por uma linha ($'$), e suponha $q = \frac{a_1}{b_1}$. Do [Algoritmo de Euclides](euclid-algorithm.md), nós temos:

$$a_1' = b_1$$

$$b_1' = a_1 - q \cdot b_1$$

Para a primeira invariante se manter, o seguinte deve ser verdadeiro:

$$x' \cdot a + y' \cdot b = a_1' = b_1$$

$$x' \cdot a + y' \cdot b = x_1 \cdot a + y_1 \cdot b$$

Da mesma forma para a segunda invariante, o seguinte deve ser válido:

$$x_1' \cdot a + y_1' \cdot b = a_1 - q \cdot b_1$$

$$x_1' \cdot a + y_1' \cdot b = (x - q \cdot x_1) \cdot a + (y - q \cdot y_1) \cdot b$$

Ao comparar os coeficientes de $a$ e $b$, as equações de atualização para cada variável podem ser derivadas, garantindo que as invariantes sejam mantidas durante todo o algoritmo.


No final, sabemos que $a_1$ contém o GCD, então $x \cdot a + y \cdot b = g$.
O que significa que encontramos os coeficientes necessários.

Você pode até otimizar mais o código e remover a variável $a_1$ e $b_1$ do código e apenas reutilizar $a$ e $b$.
No entanto, se o fizer, perderá a capacidade de argumentar sobre as invariantes.

## Problemas Práticos

* [UVA - 10104 - Euclid Problem](https://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&problem=1045)
* [GYM - (J) Once Upon A Time](http://codeforces.com/gym/100963)
* [UVA - 12775 - Gift Dilemma](https://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&problem=4628)
