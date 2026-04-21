---
tags:
  - Translated
e_maxx_link: string_hashes
---

# Hashing de Strings

Algoritmos de hashing são úteis na resolução de diversos problemas.

Queremos resolver o problema de comparar strings eficientemente.
A forma bruta de fazer isso é comparar as letras de ambas as strings, o que tem uma complexidade de tempo de $O(\min(n_1, n_2))$ se $n_1$ e $n_2$ são os tamanhos das duas strings.
Queremos fazer melhor.
A ideia por trás do hashing de strings é a seguinte: mapeamos cada string para um inteiro e comparamos esses inteiros em vez das strings.
Fazer isso nos permite reduzir o tempo de execução da comparação de strings para $O(1)$.

Para a conversão, precisamos de uma chamada **função de hash**.
O objetivo dela é converter uma string em um inteiro, o chamado **hash** da string.
A seguinte condição deve ser válida: se duas strings $s$ e $t$ são iguais ($s = t$), então seus hashes também devem ser iguais ($\text{hash}(s) = \text{hash}(t)$).
Caso contrário, não seremos capazes de comparar strings.

Observe que a direção oposta não precisa ser válida.
Se os hashes são iguais ($\text{hash}(s) = \text{hash}(t)$), as strings não precisam necessariamente ser iguais.
Por exemplo, uma função de hash válida seria simplesmente $\text{hash}(s) = 0$ para cada $s$.
Agora, este é apenas um exemplo trivial, porque esta função será completamente inútil, mas é uma função de hash válida.
A razão pela qual a direção oposta não precisa ser válida é porque existem exponencialmente muitas strings.
Se quisermos apenas que essa função de hash distinga entre todas as strings que consistem em caracteres minúsculos de comprimento menor que 15, o hash já não caberia em um inteiro de 64 bits (por exemplo, unsigned long long), porque existem muitas delas.
E, claro, não queremos comparar inteiros arbitrariamente longos, porque isso também terá a complexidade $O(n)$.

Então, geralmente queremos que a função de hash mapeie strings para números de um intervalo fixo $[0, m)$, então comparar strings é apenas uma comparação de dois inteiros com um comprimento fixo.
E, claro, queremos que $\text{hash}(s) \neq \text{hash}(t)$ seja muito provável se $s \neq t$.

Essa é a parte importante que você deve ter em mente.
Usar hashing não será 100% deterministicamente correto, porque duas strings completamente diferentes podem ter o mesmo hash (os hashes colidem).
No entanto, na grande maioria das tarefas, isso pode ser ignorado com segurança, pois a probabilidade de colisão dos hashes de duas strings diferentes ainda é muito pequena.
E discutiremos algumas técnicas neste artigo sobre como manter a probabilidade de colisões muito baixa.

## Cálculo do hash de uma string

Uma maneira boa e amplamente utilizada para definir o hash de uma string $s$ de comprimento $n$ é

$$\begin{align}
\text{hash}(s) &= s[0] + s[1] \cdot p + s[2] \cdot p^2 + ... + s[n-1] \cdot p^{n-1} \mod m \\
&= \sum_{i=0}^{n-1} s[i] \cdot p^i \mod m,
\end{align}$$

onde $p$ e $m$ são alguns números positivos escolhidos.
É chamada de **função de hash polinomial rolante**.

É razoável fazer $p$ um número primo aproximadamente igual ao número de caracteres no alfabeto de entrada.
Por exemplo, se a entrada for composta apenas por letras minúsculas do alfabeto inglês, $p = 31$ é uma boa escolha.
Se a entrada puder conter letras maiúsculas e minúsculas, $p = 53$ é uma escolha possível.
O código neste artigo usará $p = 31$.

Obviamente, $m$ deve ser um número grande, pois a probabilidade de duas strings aleatórias colidirem é de cerca de $\approx \frac{1}{m}$.
Às vezes, $m = 2^{64}$ é escolhido, pois os overflows de inteiros de 64 bits funcionam exatamente como a operação de módulo.
No entanto, existe um método que gera strings conflitantes (que funcionam independentemente da escolha de $p$).
Portanto, na prática, $m = 2^{64}$ não é recomendado.
Uma boa escolha para $m$ é algum número primo grande.
O código neste artigo usará apenas $m = 10^9+9$.
Este é um número grande, mas ainda pequeno o suficiente para que possamos realizar a multiplicação de dois valores usando inteiros de 64 bits.

Aqui está um exemplo de cálculo do hash de uma string $s$, que contém apenas letras minúsculas.
Convertemos cada caractere de $s$ em um inteiro.
Aqui usamos a conversão $a \rightarrow 1$, $b \rightarrow 2$, $\dots$, $z \rightarrow 26$.
Converter $a \rightarrow 0$ não é uma boa ideia, porque então os hashes das strings $a$, $aa$, $aaa$, $\dots$ são todos avaliados como $0$.

```{.cpp file=hashing_function}
long long compute_hash(string const& s) {
    const int p = 31;
    const int m = 1e9 + 9;
    long long hash_value = 0;
    long long p_pow = 1;
    for (char c : s) {
        hash_value = (hash_value + (c - 'a' + 1) * p_pow) % m;
        p_pow = (p_pow * p) % m;
    }
    return hash_value;
}
```

Pré-calcular as potências de $p$ pode aumentar o desempenho.

## Exemplos de Tarefas

### Procurar strings duplicadas em um array de strings

Problema: Dada uma lista de $n$ strings $s_i$, cada uma com no máximo $m$ caracteres, encontre todas as strings duplicadas e divida-as em grupos.

A partir do algoritmo óbvio que envolve a ordenação das strings, obteríamos uma complexidade de tempo de $O(n m \log n)$, onde a ordenação requer $O(n \log n)$ comparações e cada comparação leva $O(m)$ tempo.
No entanto, usando hashes, reduzimos o tempo de comparação para $O(1)$, fornecendo um algoritmo que é executado em $O(n m + n \log n)$ tempo.

Calculamos o hash para cada string, ordenamos os hashes juntamente com os índices e, em seguida, agrupamos os índices por hashes idênticos.

```{.cpp file=hashing_group_identical_strings}
vector<vector<int>> group_identical_strings(vector<string> const& s) {
    int n = s.size();
    vector<pair<long long, int>> hashes(n);
    for (int i = 0; i < n; i++)
        hashes[i] = {compute_hash(s[i]), i};

    sort(hashes.begin(), hashes.end());

    vector<vector<int>> groups;
    for (int i = 0; i < n; i++) {
        if (i == 0 || hashes[i].first != hashes[i-1].first)
            groups.emplace_back();
        groups.back().push_back(hashes[i].second);
    }
    return groups;
}
```

### Cálculo rápido de hash de substrings de uma determinada string

Problema: Dada uma string $s$ e os índices $i$ e $j$, encontre o hash da substring $s [i \dots j]$.

Por definição, temos:

$$\text{hash}(s[i \dots j]) = \sum_{k = i}^j s[k] \cdot p^{k-i} \mod m$$

Multiplicando por $p^i$ dá:

$$\begin{align}
\text{hash}(s[i \dots j]) \cdot p^i &= \sum_{k = i}^j s[k] \cdot p^k \mod m \\
&= \text{hash}(s[0 \dots j]) - \text{hash}(s[0 \dots i-1]) \mod m
\end{align}$$

Então, conhecendo o valor do hash de cada prefixo da string $s$, podemos calcular o hash de qualquer substring diretamente usando esta fórmula.
O único problema que enfrentamos ao calculá-lo é que devemos ser capazes de dividir $\text{hash}(s[0 \dots j]) - \text{hash}(s[0 \dots i-1])$ por $p^i$.
Portanto, precisamos encontrar o [inverso multiplicativo modular](../algebra/module-inverse.md) de $p^i$ e então realizar a multiplicação com esse inverso.
Podemos pré-calcular o inverso de cada $p^i$, o que permite calcular o hash de qualquer substring de $s$ em tempo $O(1)$.

No entanto, existe uma maneira mais fácil.
Na maioria dos casos, em vez de calcular os hashes da substring exatamente, é suficiente calcular o hash multiplicado por alguma potência de $p$.
Suponha que tenhamos dois hashes de duas substrings, um multiplicado por $p^i$ e o outro por $p^j$.
Se $i < j$, multiplicamos o primeiro hash por $p^{j-i}$, caso contrário, multiplicamos o segundo hash por $p^{i-j}$.
Fazendo isso, obtemos ambos os hashes multiplicados pela mesma potência de $p$ (que é o máximo de $i$ e $j$) e agora esses hashes podem ser comparados facilmente sem a necessidade de qualquer divisão.

## Aplicações de Hashing

Aqui estão algumas aplicações típicas de Hashing:

* [Algoritmo de Rabin-Karp](rabin-karp.md) para encontrar padrões em uma string em tempo $O(n)$
* Calcular o número de substrings diferentes de uma string em $O(n^2)$ (veja abaixo)
* Calcular o número de substrings palíndromas em uma string.

### Determinar o número de substrings diferentes em uma string

Problema: Dada uma string $s$ de comprimento $n$, consistindo apenas de letras minúsculas do inglês, encontre o número de substrings diferentes nesta string.

Para resolver este problema, iteramos sobre todos os comprimentos de substring $l = 1 \dots n$.
Para cada comprimento de substring $l$, construímos um array de hashes de todas as substrings de comprimento $l$ multiplicadas pela mesma potência de $p$.
O número de elementos diferentes no array é igual ao número de substrings distintas de comprimento $l$ na string.
Este número é adicionado à resposta final.

Para conveniência, usaremos $h[i]$ como o hash do prefixo com $i$ caracteres e definiremos $h[0] = 0$.

```{.cpp file=hashing_count_unique_substrings}
int count_unique_substrings(string const& s) {
    int n = s.size();
    
    const int p = 31;
    const int m = 1e9 + 9;
    vector<long long> p_pow(n);
    p_pow[0] = 1;
    for (int i = 1; i < n; i++)
        p_pow[i] = (p_pow[i-1] * p) % m;

    vector<long long> h(n + 1, 0);
    for (int i = 0; i < n; i++)
        h[i+1] = (h[i] + (s[i] - 'a' + 1) * p_pow[i]) % m;

    int cnt = 0;
    for (int l = 1; l <= n; l++) {
        unordered_set<long long> hs;
        for (int i = 0; i <= n - l; i++) {
            long long cur_h = (h[i + l] + m - h[i]) % m;
            cur_h = (cur_h * p_pow[n-i-1]) % m;
            hs.insert(cur_h);
        }
        cnt += hs.size();
    }
    return cnt;
}
```

Observe que $O(n^2)$ não é a melhor complexidade de tempo possível para este problema.
Uma solução com $O(n \log n)$ é descrita no artigo sobre [Suffix Arrays](suffix-array.md), e é até possível calculá-la em $O(n)$ usando uma [Suffix Tree](./suffix-tree-ukkonen.md) ou um [Suffix Automaton](./suffix-automaton.md).

## Melhorar a probabilidade de não colisão

Muitas vezes, o hash polinomial mencionado acima é bom o suficiente e nenhuma colisão ocorrerá durante os testes.
Lembre-se, a probabilidade de que uma colisão aconteça é apenas $\approx \frac{1}{m}$.
Para $m = 10^9 + 9$, a probabilidade é $\approx 10^{-9}$, o que é bastante baixo.
Mas observe que fizemos apenas uma comparação.
E se compararmos uma string $s$ com $10^6$ strings diferentes?
A probabilidade de que pelo menos uma colisão aconteça agora é $\approx 10^{-3}$.
E se quisermos comparar $10^6$ strings diferentes umas com as outras (por exemplo, contando quantas strings únicas existem), a probabilidade de pelo menos uma colisão acontecer já é $\approx 1$.
É praticamente garantido que esta tarefa terminará com uma colisão e retornará o resultado errado.

Existe um truque muito fácil para obter melhores probabilidades.
Podemos simplesmente calcular dois hashes diferentes para cada string (usando dois $p$ diferentes e/ou $m$ diferentes e comparar esses pares.
Se $m$ for cerca de $10^9$ para cada uma das duas funções de hash, isso é mais ou menos equivalente a ter uma função de hash com $m \approx 10^{18}$.
Ao comparar $10^6$ strings entre si, a probabilidade de que pelo menos uma colisão aconteça agora é reduzida para $\approx 10^{-6}$.

## Problemas para praticar
* [Good Substrings - Codeforces](https://codeforces.com/contest/271/problem/D)
* [A Needle in the Haystack - SPOJ](http://www.spoj.com/problems/NHAY/)
* [String Hashing - Kattis](https://open.kattis.com/problems/hashing)
* [Double Profiles - Codeforces](http://codeforces.com/problemset/problem/154/C)
* [Password - Codeforces](http://codeforces.com/problemset/problem/126/B)
* [SUB_PROB - SPOJ](http://www.spoj.com/problems/SUB_PROB/)
* [INSQ15_A](https://www.codechef.com/problems/INSQ15_A)
* [SPOJ - Ada and Spring Cleaning](http://www.spoj.com/problems/ADACLEAN/)
* [GYM - Text Editor](http://codeforces.com/gym/101466/problem/E)
* [12012 - Detection of Extraterrestrial](https://uva.onlinejudge.org/index.php?option=onlinejudge&page=show_problem&problem=3163)
* [Codeforces - Games on a CD](http://codeforces.com/contest/727/problem/E)
* [UVA 11855 - Buzzwords](https://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&problem=2955)
* [Codeforces - Santa Claus and a Palindrome](http://codeforces.com/contest/752/problem/D)
* [Codeforces - String Compression](http://codeforces.com/contest/825/problem/F)
* [Codeforces - Palindromic Characteristics](http://codeforces.com/contest/835/problem/D)
* [SPOJ - Test](http://www.spoj.com/problems/CF25E/)
* [Codeforces - Palindrome Degree](http://codeforces.com/contest/7/problem/D)
* [Codeforces - Deletion of Repeats](http://codeforces.com/contest/19/problem/C)
* [HackerRank - Gift Boxes](https://www.hackerrank.com/contests/womens-codesprint-5/challenges/gift-boxes)
