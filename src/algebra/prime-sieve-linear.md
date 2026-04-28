---
tags:
  - Translated
e_maxx_link: prime_sieve_linear
---

# Crivo Linear

Dado um número $n$, encontre todos os números primos em um intervalo $[2;n]$.

A maneira padrão de resolver este problema é usar [o crivo de Eratóstenes](sieve-of-eratosthenes.md). Este algoritmo é muito simples, mas tem tempo de execução $O(n \log \log n)$.

Embora existam muitos algoritmos conhecidos com tempo de execução sublinear (ou seja, $o(n)$), o algoritmo descrito abaixo é interessante pela sua simplicidade: não é mais complexo que o crivo clássico de Eratóstenes.

Além disso, o algoritmo fornecido aqui calcula **fatorações de todos os números** no intervalo $[2; n]$ como um efeito colateral, e isso pode ser útil em muitas aplicações práticas.

A fraqueza do algoritmo fornecido está em usar mais memória do que o crivo clássico de Eratóstenes: ele requer um array de $n$ números, enquanto que para o crivo clássico de Eratóstenes é suficiente ter $n$ bits de memória (o que é 32 vezes menos).

Assim, faz sentido usar o algoritmo descrito apenas para números da ordem de $10^7$ e não maiores.

O algoritmo é de autoria de Paul Pritchard. É uma variante do Algoritmo 3.3 em (Pritchard, 1987: veja as referências no final do artigo).

## Algoritmo

Nosso objetivo é calcular o **menor fator primo** $lp [i]$ para cada número $i$ no intervalo $[2; n]$.

Além disso, precisamos armazenar a lista de todos os números primos encontrados - vamos chamá-la de $pr []$.

Inicializaremos os valores de $lp [i]$ com zeros, o que significa que assumimos que todos os números são primos. Durante a execução do algoritmo, este array será preenchido gradualmente.

Agora vamos percorrer os números de 2 até $n$. Temos dois casos para o número atual $i$:

- $lp[i] = 0$ - isso significa que $i$ é primo, ou seja, não encontramos fatores menores para ele.
  Portanto, atribuímos $lp [i] = i$ e adicionamos $i$ ao final da lista $pr[]$.

- $lp[i] \neq 0$ - isso significa que $i$ é composto, e o seu menor fator primo é $lp [i]$.

Em ambos os casos, atualizamos os valores de $lp []$ para os números que são divisíveis por $i$. No entanto, nosso objetivo é aprender a fazer isso de forma a definir um valor $lp []$ no máximo uma vez para cada número. Podemos fazer isso da seguinte maneira:

Vamos considerar os números $x_j = i \cdot p_j$, onde $p_j$ são todos os números primos menores ou iguais a $lp [i]$ (é por isso que precisamos armazenar a lista de todos os números primos).

Definiremos um novo valor $lp [x_j] = p_j$ para todos os números dessa forma.

A prova de corretude deste algoritmo e o seu tempo de execução podem ser encontrados após a implementação.

## Implementação

```cpp
const int N = 10000000;
vector<int> lp(N+1);
vector<int> pr;
 
for (int i=2; i <= N; ++i) {
	if (lp[i] == 0) {
		lp[i] = i;
		pr.push_back(i);
	}
	for (int j = 0; i * pr[j] <= N; ++j) {
		lp[i * pr[j]] = pr[j];
		if (pr[j] == lp[i]) {
			break;
		}
	}
}
```

## Prova de Corretude

Precisamos provar que o algoritmo define todos os valores de $lp []$ corretamente e que cada valor será definido exatamente uma vez. Portanto, o algoritmo terá tempo de execução linear, já que todas as ações restantes do algoritmo, obviamente, funcionam em $O(n)$.

Note que cada número $i$ tem exatamente uma representação na forma:

$$i = lp [i] \cdot x,$$

onde $lp [i]$ é o menor fator primo de $i$, e o número $x$ não tem fatores primos menores que $lp [i]$, ou seja,

$$lp [i] \le lp [x].$$

Agora, vamos comparar isso com as ações de nosso algoritmo: de fato, para cada $x$ ele passa por todos os números primos pelos quais ele poderia ser multiplicado, ou seja, todos os números primos até $lp [x]$ inclusive, a fim de obter os números na forma dada acima.

Portanto, o algoritmo passará por cada número composto exatamente uma vez, definindo os valores corretos de $lp []$ ali. C.Q.D.

## Tempo de Execução e Memória

Embora o tempo de execução de $O(n)$ seja melhor que $O(n \log \log n)$ do crivo clássico de Eratóstenes, a diferença entre eles não é tão grande.
Na prática, o crivo linear roda quase tão rápido quanto uma implementação típica do crivo de Eratóstenes.

Em comparação com as versões otimizadas do crivo de Eratóstenes, por exemplo, o crivo segmentado, ele é muito mais lento.

Considerando os requisitos de memória deste algoritmo - um array $lp []$ de comprimento $n$, e um array $pr []$ de comprimento $\frac n {\ln n}$, este algoritmo parece ser pior do que o crivo clássico de todas as maneiras.

No entanto, a sua grande vantagem é que este algoritmo calcula um array $lp []$, o que nos permite encontrar a fatoração de qualquer número no intervalo $[2; n]$ em tempo proporcional ao tamanho dessa fatoração. Além disso, usar apenas um array extra nos permitirá evitar divisões ao procurar a fatoração.

Conhecer as fatorações de todos os números é muito útil para algumas tarefas, e este algoritmo é um dos poucos que permite encontrá-las em tempo linear.

## Referências

- Paul Pritchard, **Linear Prime-Number Sieves: a Family Tree**, Science of Computer Programming, vol. 9 (1987), pp.17-35.
