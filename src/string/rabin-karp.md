---
tags:
  - Translated
e_maxx_link: rabin_karp
---

# Algoritmo de Rabin-Karp para busca de padrões em strings

Este algoritmo é baseado no conceito de hashing, então se você não estiver familiarizado com hashing de strings, consulte o artigo sobre [hashing de strings](string-hashing.md).

Este algoritmo foi criado por Rabin e Karp em 1987.

Problema: Dadas duas strings - um padrão $s$ e um texto $t$, determine se o padrão aparece no texto e, se aparecer, enumere todas as suas ocorrências em tempo $O(|s| + |t|)$.

Algoritmo: Calcule o hash para o padrão $s$.
Calcule os valores de hash para todos os prefixos do texto $t$.
Agora, podemos comparar uma substring de comprimento $|s|$ com $s$ em tempo constante usando os hashes calculados.
Então, compare cada substring de comprimento $|s|$ com o padrão. Isso levará um tempo total de $O(|t|)$.
Portanto, a complexidade final do algoritmo é $O(|t| + |s|)$: $O(|s|)$ é necessário para calcular o hash do padrão e $O(|t|)$ para comparar cada substring de comprimento $|s|$ com o padrão.

## Implementação
```{.cpp file=rabin_karp}
vector<int> rabin_karp(string const& s, string const& t) {
    const int p = 31; 
    const int m = 1e9 + 9;
    int S = s.size(), T = t.size();

    vector<long long> p_pow(max(S, T)); 
    p_pow[0] = 1; 
    for (int i = 1; i < (int)p_pow.size(); i++) 
        p_pow[i] = (p_pow[i-1] * p) % m;

    vector<long long> h(T + 1, 0); 
    for (int i = 0; i < T; i++)
        h[i+1] = (h[i] + (t[i] - 'a' + 1) * p_pow[i]) % m; 
    long long h_s = 0; 
    for (int i = 0; i < S; i++) 
        h_s = (h_s + (s[i] - 'a' + 1) * p_pow[i]) % m; 

    vector<int> occurrences;
    for (int i = 0; i + S - 1 < T; i++) {
        long long cur_h = (h[i+S] + m - h[i]) % m;
        if (cur_h == h_s * p_pow[i] % m)
            occurrences.push_back(i);
    }
    return occurrences;
}
```

## Problemas para praticar

* [SPOJ - Pattern Find](http://www.spoj.com/problems/NAJPF/)
* [Codeforces - Good Substrings](http://codeforces.com/problemset/problem/271/D)
* [Codeforces - Palindromic characteristics](https://codeforces.com/problemset/problem/835/D)
* [Leetcode - Longest Duplicate Substring](https://leetcode.com/problems/longest-duplicate-substring/)

