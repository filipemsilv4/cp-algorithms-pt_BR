---
tags:
  - Translated
e_maxx_link: binary_pow
---

# Exponenciação Binária

A exponenciação binária (também conhecida como exponenciação por quadratura) é um truque que permite calcular $a^n$ usando apenas $O(\log n)$ multiplicações (em vez das $O(n)$ multiplicações exigidas pela abordagem ingênua).

Ela também tem aplicações importantes em muitas tarefas não relacionadas à aritmética, pois pode ser usada com quaisquer operações que tenham a propriedade da **associatividade**:

$$(X \cdot Y) \cdot Z = X \cdot (Y \cdot Z)$$

Mais obviamente, isso se aplica à multiplicação modular, à multiplicação de matrizes e a outros problemas que discutiremos abaixo.

## Algoritmo

Elevar $a$ à potência de $n$ é expresso de forma ingênua como a multiplicação por $a$ feita $n - 1$ vezes:
$a^{n} = a \cdot a \cdot \ldots \cdot a$. No entanto, essa abordagem não é prática para grandes valores de $a$ ou $n$.

$a^{b+c} = a^b \cdot a^c$ e $a^{2b} = a^b \cdot a^b = (a^b)^2$.

A ideia da exponenciação binária é dividirmos o trabalho usando a representação binária do expoente.

Vamos escrever $n$ na base 2, por exemplo:

$$3^{13} = 3^{1101_2} = 3^8 \cdot 3^4 \cdot 3^1$$

Como o número $n$ tem exatamente $\lfloor \log_2 n \rfloor + 1$ dígitos na base 2, só precisamos realizar $O(\log n)$ multiplicações, se conhecermos as potências $a^1, a^2, a^4, a^8, \dots, a^{2^{\lfloor \log_2 n \rfloor}}$.

Portanto, só precisamos saber uma maneira rápida de calculá-las.
Felizmente, isso é muito fácil, já que um elemento na sequência é apenas o quadrado do elemento anterior.

$$\begin{align}
3^1 &= 3 \\
3^2 &= \left(3^1\right)^2 = 3^2 = 9 \\
3^4 &= \left(3^2\right)^2 = 9^2 = 81 \\
3^8 &= \left(3^4\right)^2 = 81^2 = 6561
\end{align}$$

Assim, para obter a resposta final de $3^{13}$, só precisamos multiplicar três delas (pulando $3^2$ porque o bit correspondente em $n$ não está ativado):
$3^{13} = 6561 \cdot 81 \cdot 3 = 1594323$

A complexidade final deste algoritmo é $O(\log n)$: temos que calcular $\log n$ potências de $a$, e então temos que fazer no máximo $\log n$ multiplicações para obter a resposta final a partir delas.

A seguinte abordagem recursiva expressa a mesma ideia:

$$a^n = \begin{cases}
1 &\text{se } n == 0 \\
\left(a^{\frac{n}{2}}\right)^2 &\text{se } n > 0 \text{ e } n \text{ par}\\
\left(a^{\frac{n - 1}{2}}\right)^2 \cdot a &\text{se } n > 0 \text{ e } n \text{ ímpar}\\
\end{cases}$$

## Implementação

Primeiro, a abordagem recursiva, que é uma tradução direta da fórmula recursiva:

```cpp
long long binpow(long long a, long long b) {
    if (b == 0)
        return 1;
    long long res = binpow(a, b / 2);
    if (b % 2)
        return res * res * a;
    else
        return res * res;
}
```

A segunda abordagem realiza a mesma tarefa sem recursão.
Ela calcula todas as potências em um loop e multiplica aquelas com o bit correspondente ativado em $n$.
Embora a complexidade de ambas as abordagens seja idêntica, esta abordagem será mais rápida na prática, pois não temos a sobrecarga das chamadas recursivas.

```cpp
long long binpow(long long a, long long b) {
    long long res = 1;
    while (b > 0) {
        if (b & 1)
            res = res * a;
        a = a * a;
        b >>= 1;
    }
    return res;
}
```

## Aplicações

### Cálculo eficiente de grandes expoentes módulo um número

**Problema:**
Calcule $x^n \bmod m$.
Esta é uma operação muito comum. Por exemplo, é usada no cálculo do [inverso multiplicativo modular](module-inverse.md).

**Solução:**
Como sabemos que o operador de módulo não interfere nas multiplicações ($a \cdot b \equiv (a \bmod m) \cdot (b \bmod m) \pmod m$), podemos usar diretamente o mesmo código e apenas substituir cada multiplicação por uma multiplicação modular:

```cpp
long long binpow(long long a, long long b, long long m) {
    a %= m;
    long long res = 1;
    while (b > 0) {
        if (b & 1)
            res = res * a % m;
        a = a * a % m;
        b >>= 1;
    }
    return res;
}
```

**Nota:**
É possível acelerar este algoritmo para um $b >> m$ grande.
Se $m$ for um número positivo e $\gcd(x, m) = 1$, então $x^n \equiv x^{n \bmod (m-1)} \pmod{m}$ para $m$ primo, e $x^n \equiv x^{n \bmod{\phi(m)}} \pmod{m}$ para $m$ composto.
Isso segue diretamente do pequeno teorema de Fermat e do teorema de Euler, veja o artigo sobre [Inversos Modulares](module-inverse.md#fermat-euler) para mais detalhes.

### Cálculo eficiente de números de Fibonacci

**Problema:** Calcule o $n$-ésimo número de Fibonacci $F_n$.

**Solução:** Para mais detalhes, veja o artigo sobre o [Número de Fibonacci](fibonacci-numbers.md).
Faremos apenas uma visão geral do algoritmo.
Para calcular o próximo número de Fibonacci, apenas os dois anteriores são necessários, pois $F_n = F_{n-1} + F_{n-2}$.
Podemos construir uma matriz $2 \times 2$ que descreve esta transformação:
a transição de $F_i$ e $F_{i+1}$ para $F_{i+1}$ e $F_{i+2}$.
Por exemplo, aplicar essa transformação ao par $F_0$ e $F_1$ os transformaria em $F_1$ e $F_2$.
Portanto, podemos elevar esta matriz de transformação à $n$-ésima potência para encontrar $F_n$ na complexidade de tempo $O(\log n)$.

### Aplicando uma permutação $k$ vezes { data-toc-label='Aplicando uma permutação <script type="math/tex">k</script> vezes' }

**Problema:** É dada uma sequência de comprimento $n$. Aplique nela uma permutação dada $k$ vezes.

**Solução:** Simplesmente eleve a permutação à $k$-ésima potência usando a exponenciação binária e, em seguida, aplique-a à sequência. Isso lhe dará uma complexidade de tempo de $O(n \log k)$.

```cpp
vector<int> applyPermutation(vector<int> sequence, vector<int> permutation) {
    vector<int> newSequence(sequence.size());
    for(int i = 0; i < sequence.size(); i++) {
        newSequence[i] = sequence[permutation[i]];
    }
    return newSequence;
}

vector<int> permute(vector<int> sequence, vector<int> permutation, long long k) {
    while (k > 0) {
        if (k & 1) {
            sequence = applyPermutation(sequence, permutation);
        }
        permutation = applyPermutation(permutation, permutation);
        k >>= 1;
    }
    return sequence;
}
```

**Nota:** Esta tarefa pode ser resolvida de forma mais eficiente em tempo linear construindo o grafo de permutação e considerando cada ciclo independentemente. Você poderia então calcular $k$ módulo o tamanho do ciclo e encontrar a posição final para cada número que faz parte desse ciclo.

### Aplicação rápida de um conjunto de operações geométricas a um conjunto de pontos

**Problema:** Dados $n$ pontos $p_i$, aplique $m$ transformações a cada um desses pontos. Cada transformação pode ser um deslocamento, um escalonamento ou uma rotação em torno de um eixo dado por um ângulo dado. Existe também uma operação de "loop" que aplica uma lista de transformações dadas $k$ vezes (operações de "loop" podem ser aninhadas). Você deve aplicar todas as transformações mais rapidamente que $O(n \cdot length)$, onde $length$ é o número total de transformações a serem aplicadas (após desenrolar as operações de "loop").

**Solução:** Vamos ver como os diferentes tipos de transformações alteram as coordenadas:

* Operação de deslocamento (shift): adiciona uma constante diferente a cada uma das coordenadas.
* Operação de escalonamento (scaling): multiplica cada uma das coordenadas por uma constante diferente.
* Operação de rotação (rotation): a transformação é mais complicada (não entraremos em detalhes aqui), mas cada uma das novas coordenadas ainda pode ser representada como uma combinação linear das antigas.

Como você pode ver, cada uma das transformações pode ser representada como uma operação linear nas coordenadas. Portanto, uma transformação pode ser escrita como uma matriz $4 \times 4$ da forma:

$$\begin{pmatrix}
a_{11} & a_ {12} & a_ {13} & a_ {14} \\
a_{21} & a_ {22} & a_ {23} & a_ {24} \\
a_{31} & a_ {32} & a_ {33} & a_ {34} \\
a_{41} & a_ {42} & a_ {43} & a_ {44}
\end{pmatrix}$$

que, quando multiplicada por um vetor com as coordenadas antigas e uma unidade, fornece um novo vetor com as novas coordenadas e uma unidade:

$$\begin{pmatrix} x & y & z & 1 \end{pmatrix} \cdot
\begin{pmatrix}
a_{11} & a_ {12} & a_ {13} & a_ {14} \\
a_{21} & a_ {22} & a_ {23} & a_ {24} \\
a_{31} & a_ {32} & a_ {33} & a_ {34} \\
a_{41} & a_ {42} & a_ {43} & a_ {44}
\end{pmatrix}
 = \begin{pmatrix} x' & y' & z' & 1 \end{pmatrix}$$

(Por que introduzir uma quarta coordenada fictícia, você pergunta? Essa é a beleza das [coordenadas homogêneas](https://en.wikipedia.org/wiki/Homogeneous_coordinates), que encontram grande aplicação na computação gráfica. Sem isso, não seria possível implementar operações afins como a operação de deslocamento como uma única multiplicação de matriz, pois exige que _adicionemos_ uma constante às coordenadas. A transformação afim se torna uma transformação linear na dimensão superior!)

Aqui estão alguns exemplos de como as transformações são representadas na forma de matriz:

* Operação de deslocamento: deslocar a coordenada $x$ em $5$, a coordenada $y$ em $7$ e a coordenada $z$ em $9$.

$$\begin{pmatrix}
1 & 0 & 0 & 0 \\
0 & 1 & 0 & 0 \\
0 & 0 & 1 & 0 \\
5 & 7 & 9 & 1
\end{pmatrix}$$

* Operação de escalonamento: escalar a coordenada $x$ em $10$ e as outras duas em $5$.

$$\begin{pmatrix}
10 & 0 & 0 & 0 \\
0 & 5 & 0 & 0 \\
0 & 0 & 5 & 0 \\
0 & 0 & 0 & 1
\end{pmatrix}$$

* Operação de rotação: rotacionar $\theta$ graus em torno do eixo $x$ seguindo a regra da mão direita (direção anti-horária).

$$\begin{pmatrix}
1 & 0 & 0 & 0 \\
0 & \cos \theta & -\sin \theta & 0 \\
0 & \sin \theta & \cos \theta & 0 \\
0 & 0 & 0 & 1
\end{pmatrix}$$

Agora, uma vez que cada transformação é descrita como uma matriz, a sequência de transformações pode ser descrita como um produto dessas matrizes, e um "loop" de $k$ repetições pode ser descrito como a matriz elevada à potência de $k$ (o que pode ser calculado usando a exponenciação binária em $O(\log{k})$). Dessa forma, a matriz que representa todas as transformações pode ser calculada primeiro em $O(m \log{k})$ e, em seguida, pode ser aplicada a cada um dos $n$ pontos em $O(n)$ para uma complexidade total de $O(n + m \log{k})$.


### Número de caminhos de comprimento $k$ em um grafo { data-toc-label='Número de caminhos de comprimento <script type="math/tex">k</script> em um grafo' }

**Problema:** Dado um grafo direcionado e não ponderado de $n$ vértices, encontre o número de caminhos de comprimento $k$ de qualquer vértice $u$ para qualquer outro vértice $v$.

**Solução:** Este problema é considerado em mais detalhes em [um artigo separado](../graph/fixed_length_paths.md). O algoritmo consiste em elevar a matriz de adjacência $M$ do grafo (uma matriz onde $m_{ij} = 1$ se houver uma aresta de $i$ para $j$, ou $0$ caso contrário) à $k$-ésima potência. Agora $m_{ij}$ será o número de caminhos de comprimento $k$ de $i$ para $j$. A complexidade de tempo desta solução é $O(n^3 \log k)$.

**Nota:** Nesse mesmo artigo, é considerada outra variação deste problema: quando as arestas são ponderadas e é necessário encontrar o caminho de peso mínimo contendo exatamente $k$ arestas. Como mostrado no artigo, esse problema também é resolvido pela exponenciação da matriz de adjacência. A matriz teria o peso da aresta de $i$ para $j$, ou $\infty$ se não houver tal aresta.
Em vez da operação usual de multiplicar duas matrizes, deve-se usar uma operação modificada:
em vez de multiplicação, ambos os valores são adicionados, e em vez de uma soma, obtém-se o mínimo.
Ou seja: $result_{ij} = \min\limits_{1\ \leq\ k\ \leq\ n}(a_{ik} + b_{kj})$.

### Variação da exponenciação binária: multiplicando dois números módulo $m$ { data-toc-label='Variação da exponenciação binária: multiplicando dois números módulo <script type="math/tex">m</script>' }

**Problema:** Multiplique dois números $a$ e $b$ módulo $m$. $a$ e $b$ cabem nos tipos de dados nativos, mas seu produto é grande demais para caber em um inteiro de 64 bits. A ideia é calcular $a \cdot b \pmod m$ sem usar aritmética para números grandes (bignum).

**Solução:** Nós simplesmente aplicamos o algoritmo de construção binária descrito acima, apenas realizando adições em vez de multiplicações. Em outras palavras, "expandimos" a multiplicação de dois números para $O (\log m)$ operações de adição e multiplicação por dois (que, na essência, é uma adição).

$$a \cdot b = \begin{cases}
0 &\text{se }a = 0 \\
2 \cdot \frac{a}{2} \cdot b &\text{se }a > 0 \text{ e }a \text{ par} \\
2 \cdot \frac{a-1}{2} \cdot b + b &\text{se }a > 0 \text{ e }a \text{ ímpar}
\end{cases}$$

**Nota:** Você pode resolver esta tarefa de uma forma diferente usando operações de ponto flutuante. Primeiro calcule a expressão $\frac{a \cdot b}{m}$ usando números de ponto flutuante e converta para um inteiro sem sinal (unsigned) $q$. Subtraia $q \cdot m$ de $a \cdot b$ usando aritmética de inteiros sem sinal e tire o módulo $m$ para encontrar a resposta. Esta solução parece não ser muito confiável, mas é muito rápida e fácil de implementar. Veja [aqui](https://cs.stackexchange.com/questions/77016/modular-multiplication) para obter mais informações.

## Problemas Práticos

* [UVa 1230 - MODEX](http://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&category=24&page=show_problem&problem=3671)
* [UVa 374 - Big Mod](http://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&category=24&page=show_problem&problem=310)
* [UVa 11029 - Leading and Trailing](https://uva.onlinejudge.org/index.php?option=onlinejudge&page=show_problem&problem=1970)
* [Codeforces - Parking Lot](http://codeforces.com/problemset/problem/630/I)
* [leetcode - Count good numbers](https://leetcode.com/problems/count-good-numbers/)
* [Codechef - Chef and Riffles](https://www.codechef.com/JAN221B/problems/RIFFLES)
* [Codeforces - Decoding Genome](https://codeforces.com/contest/222/problem/E)
* [Codeforces - Neural Network Country](https://codeforces.com/contest/852/problem/B)
* [Codeforces - Magic Gems](https://codeforces.com/problemset/problem/1117/D)
* [SPOJ - The last digit](http://www.spoj.com/problems/LASTDIG/)
* [SPOJ - Locker](http://www.spoj.com/problems/LOCKER/)
* [LA - 3722 Jewel-eating Monsters](https://vjudge.net/problem/UVALive-3722)
* [SPOJ - Just add it](http://www.spoj.com/problems/ZSUM/)
* [Codeforces - Stairs and Lines](https://codeforces.com/contest/498/problem/E)
