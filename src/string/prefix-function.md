---
tags:
  - Translated
e_maxx_link: prefix_function
---

# Função de Prefixo. Algoritmo de Knuth–Morris–Pratt

## Definição da função de prefixo

Dada uma string $s$ de comprimento $n$.
A **função de prefixo** para esta string é definida como um array $\pi$ de comprimento $n$, onde $\pi[i]$ é o comprimento do maior prefixo próprio da substring $s[0 \dots i]$ que também é um sufixo desta substring.
Um prefixo próprio de uma string é um prefixo que não é igual à própria string.
Por definição, $\pi[0] = 0$.

Matematicamente, a definição da função de prefixo pode ser escrita da seguinte forma:

$$\pi[i] = \max_ {k = 0 \dots i} \{k : s[0 \dots k-1] = s[i-(k-1) \dots i] \}$$

Por exemplo, a função de prefixo da string "abcabcd" é $[0, 0, 0, 1, 2, 3, 0]$ e a função de prefixo da string "aabaaab" é $[0, 1, 0, 1, 2, 2, 3]$.

## Algoritmo Trivial

Um algoritmo que segue a definição da função de prefixo exatamente é o seguinte:

```{.cpp file=prefix_slow}
vector<int> prefix_function(string s) {
    int n = (int)s.length();
    vector<int> pi(n);
    for (int i = 0; i < n; i++)
        for (int k = 0; k <= i; k++)
            if (s.substr(0, k) == s.substr(i-k+1, k))
                pi[i] = k;
    return pi;
}
```

É fácil ver que sua complexidade é $O(n^3)$, que pode ser melhorada.

## Algoritmo Eficiente

Este algoritmo foi proposto por Knuth e Pratt e, independentemente deles, por Morris em 1977.
Ele foi usado como a função principal de um algoritmo de busca de substring.

### Primeira otimização

A primeira observação importante é que os valores da função de prefixo podem aumentar no máximo em um.

De fato, caso contrário, se $\pi[i + 1] \gt \pi[i] + 1$, então podemos pegar este sufixo terminando na posição $i + 1$ com o comprimento $\pi[i + 1]$ e remover o último caractere dele.
Terminamos com um sufixo terminando na posição $i$ com o comprimento $\pi[i + 1] - 1$, que é melhor que $\pi[i]$, ou seja, chegamos a uma contradição.

A seguinte ilustração mostra essa contradição.
O maior sufixo próprio na posição $i$ que também é um prefixo tem comprimento $2$ e, na posição $i+1$, tem comprimento $4$.
Portanto, a string $s_0 ~ s_1 ~ s_2 ~ s_3$ é igual à string $s_{i-2} ~ s_{i-1} ~ s_i ~ s_{i+1}$, o que significa que também as strings $s_0 ~ s_1 ~ s_2$ e $s_{i-2} ~ s_{i-1} ~ s_i$ são iguais, portanto $\pi[i]$ deve ser $3$.

$$\underbrace{\overbrace{s_0 ~ s_1}^{\pi[i] = 2} ~ s_2 ~ s_3}_{\pi[i+1] = 4} ~ \dots ~ \underbrace{s_{i-2} ~ \overbrace{s_{i-1} ~ s_{i}}^{\pi[i] = 2} ~ s_{i+1}}_{\pi[i+1] = 4}$$

Assim, ao mover para a próxima posição, o valor da função de prefixo pode aumentar em um, permanecer o mesmo ou diminuir em alguma quantidade.
Esse fato já nos permite reduzir a complexidade do algoritmo para $O(n^2)$, porque em uma etapa a função de prefixo pode crescer no máximo em um.
No total, a função pode crescer no máximo $n$ etapas e, portanto, também pode diminuir apenas um total de $n$ etapas.
Isso significa que precisamos realizar apenas $O(n)$ comparações de strings e atingir a complexidade $O(n^2)$.

### Segunda otimização

Vamos além, queremos nos livrar das comparações de strings.
Para conseguir isso, temos que usar todas as informações computadas nas etapas anteriores.

Então, vamos calcular o valor da função de prefixo $\pi$ para $i + 1$.
Se $s[i+1] = s[\pi[i]]$, então podemos dizer com certeza que $\pi[i+1] = \pi[i] + 1$, já que sabemos que o sufixo na posição $i$ de comprimento $\pi[i]$ é igual ao prefixo de comprimento $\pi[i]$.
Isso é ilustrado novamente com um exemplo.

$$\underbrace{\overbrace{s_0 ~ s_1 ~ s_2}^{\pi[i]} ~ \overbrace{s_3}^{s_3 = s_{i+1}}}_{\pi[i+1] = \pi[i] + 1} ~ \dots ~ \underbrace{\overbrace{s_{i-2} ~ s_{i-1} ~ s_{i}}^{\pi[i]} ~ \overbrace{s_{i+1}}^{s_3 = s_{i + 1}}}_{\pi[i+1] = \pi[i] + 1}$$

Se este não for o caso, $s[i+1] \neq s[\pi[i]]$, então precisamos tentar uma string mais curta.
Para acelerar as coisas, gostaríamos de ir imediatamente para o maior comprimento $j \lt \pi[i]$, de modo que a propriedade do prefixo na posição $i$ seja válida, ou seja, $s[0 \dots j-1] = s[i-j+1 \dots i]$:

$$\overbrace{\underbrace{s_0 ~ s_1}_j ~ s_2 ~ s_3}^{\pi[i]} ~ \dots ~ \overbrace{s_{i-3} ~ s_{i-2} ~ \underbrace{s_{i-1} ~ s_{i}}_j}^{\pi[i]} ~ s_{i+1}$$

De fato, se encontrarmos esse comprimento $j$, novamente só precisamos comparar os caracteres $s[i+1]$ e $s[j]$.
Se eles forem iguais, podemos atribuir $\pi[i+1] = j + 1$.
Caso contrário, precisaremos encontrar o maior valor menor que $j$, para o qual a propriedade do prefixo é válida, e assim por diante.
Pode acontecer que isso vá até $j = 0$.
Se então $s[i+1] = s[0]$, atribuímos $\pi[i+1] = 1$ e $\pi[i+1] = 0$ caso contrário.

Então, já temos um esquema geral do algoritmo.
A única questão que resta é como encontrar efetivamente os comprimentos para $j$.
Vamos recapitular: para o comprimento atual $j$ na posição $i$ para o qual a propriedade do prefixo é válida, ou seja, $s[0 \dots j-1] = s[i-j+1 \dots i]$, queremos encontrar o maior $k \lt j$, para o qual a propriedade do prefixo é válida.


$$\overbrace{\underbrace{s_0 ~ s_1}_k ~ s_2 ~ s_3}^j ~ \dots ~ \overbrace{s_{i-3} ~ s_{i-2} ~ \underbrace{s_{i-1} ~ s_{i}}_k}^j ~s_{i+1}$$

A ilustração mostra que este deve ser o valor de $\pi[j-1]$, que já calculamos anteriormente.

### Algoritmo final

Então, finalmente podemos construir um algoritmo que não realiza nenhuma comparação de strings e realiza apenas $O(n)$ ações.

Aqui está o procedimento final:

- Calculamos os valores de prefixo $\pi[i]$ em um loop iterando de $i = 1$ a $i = n-1$ ($\pi[0]$ apenas recebe $0$).
- Para calcular o valor atual $\pi[i]$, definimos a variável $j$ denotando o comprimento do melhor sufixo para $i-1$. Inicialmente $j = \pi[i-1]$.
- Teste se o sufixo de comprimento $j+1$ também é um prefixo comparando $s[j]$ e $s[i]$.
Se eles forem iguais, atribuímos $\pi[i] = j + 1$; caso contrário, reduzimos $j$ para $\pi[j-1]$ e repetimos esta etapa.
- Se atingirmos o comprimento $j = 0$ e ainda não tivermos uma correspondência, atribuímos $\pi[i] = 0$ e vamos para o próximo índice $i + 1$.

### Implementação

A implementação acaba sendo surpreendentemente curta e expressiva.

```{.cpp file=prefix_fast}
vector<int> prefix_function(string s) {
    int n = (int)s.length();
    vector<int> pi(n);
    for (int i = 1; i < n; i++) {
        int j = pi[i-1];
        while (j > 0 && s[i] != s[j])
            j = pi[j-1];
        if (s[i] == s[j])
            j++;
        pi[i] = j;
    }
    return pi;
}
```

Este é um algoritmo **online**, ou seja, ele processa os dados à medida que chegam - por exemplo, você pode ler os caracteres da string um por um e processá-los imediatamente, encontrando o valor da função de prefixo para cada próximo caractere.
O algoritmo ainda requer o armazenamento da própria string e dos valores calculados anteriormente da função de prefixo, mas se soubermos de antemão o valor máximo $M$ que a função de prefixo pode assumir na string, podemos armazenar apenas $M+1$ primeiros caracteres da string e o mesmo número de valores da função de prefixo.


## Aplicações

### Busca por uma substring em uma string. O algoritmo de Knuth-Morris-Pratt

A tarefa é a aplicação clássica da função de prefixo.

Dado um texto $t$ e uma string $s$, queremos encontrar e exibir as posições de todas as ocorrências da string $s$ no texto $t$.

Para conveniência, denotamos com $n$ o comprimento da string $s$ e com $m$ o comprimento do texto $t$.

Geramos a string $s + \# + t$, onde $\#$ é um separador que não aparece nem em $s$ nem em $t$.
Vamos calcular a função de prefixo para esta string.
Agora pense sobre o significado dos valores da função de prefixo, exceto para as primeiras $n + 1$ entradas (que pertencem à string $s$ e ao separador).
Por definição, o valor $\pi[i]$ mostra o maior comprimento de uma substring terminando na posição $i$ que coincide com o prefixo.
Mas, no nosso caso, isso nada mais é do que o maior bloco que coincide com $s$ e termina na posição $i$.
Este comprimento não pode ser maior que $n$ devido ao separador.
Mas se a igualdade $\pi[i] = n$ for alcançada, isso significa que a string $s$ aparece completamente nesta posição, ou seja, termina na posição $i$.
Apenas não se esqueça de que as posições são indexadas na string $s + \# + t$.

Assim, se em alguma posição $i$ tivermos $\pi[i] = n$, então na posição $i - (n + 1) - n + 1 = i - 2n$ na string $t$ a string $s$ aparece.

Como já mencionado na descrição do cálculo da função de prefixo, se sabemos que os valores do prefixo nunca excedem um determinado valor, não precisamos armazenar a string inteira e toda a função, mas apenas seu início.
No nosso caso, isso significa que precisamos apenas armazenar a string $s + \#$ e os valores da função de prefixo para ela.
Podemos ler um caractere de cada vez da string $t$ e calcular o valor atual da função de prefixo.

Assim, o algoritmo de Knuth-Morris-Pratt resolve o problema em tempo $O(n + m)$ e memória $O(n)$.

### Contando o número de ocorrências de cada prefixo

Aqui discutimos dois problemas de uma só vez.
Dada uma string $s$ de comprimento $n$.
Na primeira variação do problema, queremos contar o número de aparições de cada prefixo $s[0 \dots i]$ na mesma string.
Na segunda variação do problema, outra string $t$ é fornecida e queremos contar o número de aparições de cada prefixo $s[0 \dots i]$ em $t$.

Primeiro, resolvemos o primeiro problema.
Considere o valor da função de prefixo $\pi[i]$ em uma posição $i$.
Por definição, isso significa que o prefixo de comprimento $\pi[i]$ da string $s$ ocorre e termina na posição $i$, e não há prefixo mais longo que siga essa definição.
Ao mesmo tempo, prefixos mais curtos podem terminar nesta posição.
Não é difícil ver que temos a mesma pergunta que já respondemos quando calculamos a própria função de prefixo:
Dado um prefixo de comprimento $j$ que é um sufixo terminando na posição $i$, qual é o próximo prefixo menor $\lt j$ que também é um sufixo terminando na posição $i$?
Assim, na posição $i$ termina o prefixo de comprimento $\pi[i]$, o prefixo de comprimento $\pi[\pi[i] - 1]$, o prefixo $\pi[\pi[\pi[i] - 1] - 1]$ e assim por diante, até que o índice se torne zero.
Assim, podemos calcular a resposta da seguinte maneira.

```{.cpp file=prefix_count_each_prefix}
vector<int> ans(n + 1);
for (int i = 0; i < n; i++)
    ans[pi[i]]++;
for (int i = n-1; i > 0; i--)
    ans[pi[i-1]] += ans[i];
for (int i = 0; i <= n; i++)
    ans[i]++;
```

Aqui, para cada valor da função de prefixo, primeiro contamos quantas vezes ela ocorre no array $\pi$ e, em seguida, calculamos as respostas finais:
se sabemos que o prefixo de comprimento $i$ aparece exatamente $\text{ans}[i]$ vezes, esse número deve ser adicionado ao número de ocorrências de seu maior sufixo que também é um prefixo.
No final, precisamos adicionar $1$ a cada resultado, pois também precisamos contar os prefixos originais.

Agora vamos considerar o segundo problema.
Aplicamos o truque de Knuth-Morris-Pratt:
criamos a string $s + \# + t$ e calculamos sua função de prefixo.
As únicas diferenças em relação à primeira tarefa são que estamos interessados apenas nos valores do prefixo que se relacionam à string $t$, ou seja, $\pi[i]$ para $i \ge n + 1$.
Com esses valores, podemos realizar exatamente os mesmos cálculos da primeira tarefa.

### O número de substrings diferentes em uma string

Dada uma string $s$ de comprimento $n$.
Queremos calcular o número de substrings diferentes que aparecem nela.

Resolveremos este problema iterativamente.
Ou seja, aprenderemos, sabendo o número atual de substrings diferentes, como recalcular essa contagem adicionando um caractere ao final.

Então, seja $k$ o número atual de substrings diferentes em $s$ e adicionamos o caractere $c$ ao final de $s$.
Obviamente, algumas novas substrings terminando em $c$ aparecerão.
Queremos contar essas novas substrings que não apareceram antes.

Pegamos a string $t = s + c$ e a invertemos.
Agora, a tarefa é transformada em calcular quantos prefixos existem que não aparecem em nenhum outro lugar.
Se calcularmos o valor máximo da função de prefixo $\pi_{\text{max}}$ da string invertida $t$, o prefixo mais longo que aparece em $s$ terá comprimento $\pi_{\text{max}}$.
Claramente, todos os prefixos de menor comprimento também aparecem nele.

Portanto, o número de novas substrings que aparecem quando adicionamos um novo caractere $c$ é $|s| + 1 - \pi_{\text{max}}$.

Assim, para cada caractere anexado, podemos calcular o número de novas substrings em $O(n)$, o que dá uma complexidade de tempo de $O(n^2)$ no total.

Vale ressaltar que também podemos calcular o número de substrings diferentes acrescentando os caracteres no início ou excluindo caracteres do início ou do fim.

### Comprimindo uma string

Dada uma string $s$ de comprimento $n$.
Queremos encontrar a representação "compactada" mais curta da string, ou seja, queremos encontrar uma string $t$ de menor comprimento, de modo que $s$ possa ser representada como uma concatenação de uma ou mais cópias de $t$.

É claro que precisamos apenas encontrar o comprimento de $t$. Conhecendo o comprimento, a resposta para o problema será o prefixo de $s$ com este comprimento.

Vamos calcular a função de prefixo para $s$.
Usando seu último valor, definimos o valor $k = n - \pi[n - 1]$.
Mostraremos que, se $k$ dividir $n$, então $k$ será a resposta; caso contrário, não há compressão efetiva e a resposta é $n$.

Seja $n$ divisível por $k$.
Então a string pode ser particionada em blocos de comprimento $k$.
Por definição da função de prefixo, o prefixo de comprimento $n - k$ será igual ao seu sufixo.
Mas isso significa que o último bloco é igual ao bloco anterior.
E o bloco anterior deve ser igual ao bloco anterior a ele.
E assim por diante.
Como resultado, verifica-se que todos os blocos são iguais, portanto podemos compactar a string $s$ para o comprimento $k$.

Claro que ainda precisamos mostrar que este é realmente o ótimo.
De fato, se houvesse uma compressão menor que $k$, o valor da função de prefixo no final seria maior que $n - k$.
Portanto, $k$ é realmente a resposta.

Agora vamos supor que $n$ não é divisível por $k$.
Mostramos que isso implica que o comprimento da resposta é $n$.
Provamos por contradição.
Supondo que exista uma resposta e que a compressão tenha comprimento $p$ ($p$ divide $n$).
Então o último valor da função de prefixo deve ser maior que $n - p$, ou seja, o sufixo cobrirá parcialmente o primeiro bloco.
Agora considere o segundo bloco da string.
Como o prefixo é igual ao sufixo e o prefixo e o sufixo cobrem esse bloco e seu deslocamento relativo um ao outro $k$ não divide o comprimento do bloco $p$ (caso contrário $k$ divide $n$), então todos os caracteres do bloco devem ser idênticos.
Mas então a string consiste em apenas um caractere repetido continuamente, portanto podemos compactá-la para uma string de tamanho $1$, o que dá $k = 1$, e $k$ divide $n$.
Contradição.

$$\overbrace{s_0 ~ s_1 ~ s_2 ~ s_3}^p ~ \overbrace{s_4 ~ s_5 ~ s_6 ~ s_7}^p$$

$$s_0 ~ s_1 ~ s_2 ~ \underbrace{\overbrace{s_3 ~ s_4 ~ s_5 ~ s_6}^p ~ s_7}_{\pi[7] = 5}$$

$$s_4 = s_3, ~ s_5 = s_4, ~ s_6 = s_5, ~ s_7 = s_6 ~ \Rightarrow ~ s_0 = s_1 = s_2 = s_3$$


### Construindo um autômato de acordo com a função de prefixo

Voltemos à concatenação das duas strings por meio de um separador, ou seja, para as strings $s$ e $t$, calculamos a função de prefixo para a string $s + \# + t$.
Obviamente, como $\#$ é um separador, o valor da função de prefixo nunca excederá $|s|$.
Segue-se que é suficiente armazenar apenas a string $s + \#$ e os valores da função de prefixo para ela, e podemos calcular a função de prefixo para todos os caracteres subsequentes dinamicamente:

$$\underbrace{s_0 ~ s_1 ~ \dots ~ s_{n-1} ~ \#}_{\text{precisa armazenar}} ~ \underbrace{t_0 ~ t_1 ~ \dots ~ t_{m-1}}_{\text{não precisa armazenar}}$$

De fato, em tal situação, conhecer o próximo caractere $c \in t$ e o valor da função de prefixo da posição anterior é informação suficiente para calcular o próximo valor da função de prefixo, sem usar nenhum caractere anterior da string $t$ e o valor da função de prefixo neles.

Em outras palavras, podemos construir um **autômato** (uma máquina de estados finitos): o estado nele é o valor atual da função de prefixo e a transição de um estado para outro será realizada por meio do próximo caractere.

Assim, mesmo sem ter a string $t$, podemos construir tal tabela de transição $(\text{old}_\pi, c) \rightarrow \text{new}_\pi$ usando o mesmo algoritmo para calcular a tabela de transição:

```{.cpp file=prefix_automaton_slow}
void compute_automaton(string s, vector<vector<int>>& aut) {
    s += '#';
    int n = s.size();
    vector<int> pi = prefix_function(s);
    aut.assign(n, vector<int>(26));
    for (int i = 0; i < n; i++) {
        for (int c = 0; c < 26; c++) {
            int j = i;
            while (j > 0 && 'a' + c != s[j])
                j = pi[j-1];
            if ('a' + c == s[j])
                j++;
            aut[i][c] = j;
        }
    }
}
```

No entanto, nesta forma, o algoritmo é executado em tempo $O(n^2 26)$ para as letras minúsculas do alfabeto.
Observe que podemos aplicar programação dinâmica e usar as partes já calculadas da tabela.
Sempre que vamos do valor $j$ para o valor $\pi[j-1]$, queremos dizer que a transição $(j, c)$ leva ao mesmo estado que a transição $(\pi[j-1], c)$, e esta resposta já está calculada com precisão.


```{.cpp file=prefix_automaton_fast}
void compute_automaton(string s, vector<vector<int>>& aut) {
    s += '#';
    int n = s.size();
    vector<int> pi = prefix_function(s);
    aut.assign(n, vector<int>(26));
    for (int i = 0; i < n; i++) {
        for (int c = 0; c < 26; c++) {
            if (i > 0 && 'a' + c != s[i])
                aut[i][c] = aut[pi[i-1]][c];
            else
                aut[i][c] = i + ('a' + c == s[i]);
        }
    }
}
```

Como resultado, construímos o autômato em tempo $O(26 n)$.

Quando esse autômato é útil?
Para começar, lembre-se de que usamos a função de prefixo para a string $s + \# + t$ e seus valores principalmente para um único propósito: encontrar todas as ocorrências da string $s$ na string $t$.

Portanto, o benefício mais óbvio deste autômato é a **aceleração do cálculo da função de prefixo** para a string $s + \# + t$.
Construindo o autômato para $s + \#$, não precisamos mais armazenar a string $s$ ou os valores da função de prefixo nela.
Todas as transições já estão computadas na tabela.

Mas há uma segunda aplicação menos óbvia.
Podemos usar o autômato quando a string $t$ é uma **string gigantesca construída usando algumas regras**.
Isso pode ser, por exemplo, as strings de Gray ou uma string formada por uma combinação recursiva de várias strings curtas da entrada.


Para completar, resolveremos tal problema:
dado um número $k \le 10^5$ e uma string $s$ de comprimento $\le 10^5$.
Temos que calcular o número de ocorrências de $s$ na $k$-ésima string de Gray.
Lembre-se de que as strings de Gray são definidas da seguinte forma:

$$\begin{align}
g_1 &= \text{"a"}\\
g_2 &= \text{"aba"}\\
g_3 &= \text{"abacaba"}\\
g_4 &= \text{"abacabadabacaba"}
\end{align}$$

Em tais casos, mesmo construir a string $t$ será impossível, devido ao seu comprimento astronômico.
A $k$-ésima string de Gray tem $2^k-1$ caracteres de comprimento.
No entanto, podemos calcular o valor da função de prefixo no final da string de forma eficaz, apenas conhecendo o valor da função de prefixo no início.

Além do próprio autômato, também calculamos os valores $G[i][j]$ - o valor do autômato após processar a string $g_i$ começando com o estado $j$.
E, adicionalmente, calculamos os valores $K[i][j]$ - o número de ocorrências de $s$ em $g_i$ antes e durante o processamento de $g_i$, começando com o estado $j$.
Na verdade, $K[i][j]$ é o número de vezes que a função de prefixo assumiu o valor $|s|$ durante a execução das operações.
A resposta para o problema será então $K[k][0]$.

Como podemos calcular esses valores?
Primeiro, os valores básicos são $G[0][j] = j$ e $K[0][j] = 0$.
E todos os valores subsequentes podem ser calculados a partir dos valores anteriores e usando o autômato.
Para calcular o valor para algum $i$, lembramos que a string $g_i$ consiste em $g_{i-1}$, o $i$-ésimo caractere do alfabeto e $g_{i-1}$.
Assim, o autômato irá para o estado:

$$\text{mid} = \text{aut}[G[i-1][j]][i]$$

$$G[i][j] = G[i-1][\text{mid}]$$

Os valores para $K[i][j]$ também podem ser facilmente contados.

$$K[i][j] = K[i-1][j] + (\text{mid} == |s|) + K[i-1][\text{mid}]$$

Assim, podemos resolver o problema para strings de Gray e, da mesma forma, também um grande número de outros problemas semelhantes.
Por exemplo, o mesmo método também resolve o seguinte problema:
recebemos uma string $s$ e alguns padrões $t_i$, cada um dos quais é especificado da seguinte forma:
é uma string de caracteres comuns e pode haver algumas inserções recursivas das strings anteriores da forma $t_k^{\text{cnt}}$, o que significa que neste local temos que inserir a string $t_k$ $\text{cnt}$ vezes.
Um exemplo de tais padrões:

$$\begin{align}
t_1 &= \text{"abdeca"}\\
t_2 &= \text{"abc"} + t_1^{30} + \text{"abd"}\\
t_3 &= t_2^{50} + t_1^{100}\\
t_4 &= t_2^{10} + t_3^{100}
\end{align}$$

As substituições recursivas explodem a string, de modo que seus comprimentos podem atingir a ordem de $100^{100}$.

Temos que encontrar o número de vezes que a string $s$ aparece em cada uma das strings.

O problema pode ser resolvido da mesma maneira construindo o autômato da função de prefixo e, em seguida, calculamos as transições para cada padrão usando os resultados anteriores.

## Problemas Práticos

* [UVA # 455 "Periodic Strings"](http://uva.onlinejudge.org/index.php?option=onlinejudge&page=show_problem&problem=396)
* [UVA # 11022 "String Factoring"](http://uva.onlinejudge.org/index.php?option=onlinejudge&page=show_problem&problem=1963)
* [UVA # 11452 "Dancing the Cheeky-Cheeky"](http://uva.onlinejudge.org/index.php?option=onlinejudge&page=show_problem&problem=2447)
* [UVA 12604 - Caesar Cipher](https://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&problem=4282)
* [UVA 12467 - Secret Word](https://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&problem=3911)
* [UVA 11019 - Matrix Matcher](https://uva.onlinejudge.org/index.php?option=onlinejudge&page=show_problem&problem=1960)
* [SPOJ - Pattern Find](http://www.spoj.com/problems/NAJPF/)
* [SPOJ - A Needle in the Haystack](https://www.spoj.com/problems/NHAY/)
* [Codeforces - Anthem of Berland](http://codeforces.com/contest/808/problem/G)
* [Codeforces - MUH and Cube Walls](http://codeforces.com/problemset/problem/471/D)
* [Codeforces - Prefixes and Suffixes](https://codeforces.com/contest/432/problem/D)
