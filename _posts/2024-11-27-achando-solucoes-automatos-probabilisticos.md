---
title: "Achando soluções para autômatos probabilísticos"
date: 2024-11-27 00:00:00 -0300
categories: [sistemas lineares, linguagens formais]
tags: [sistemas lineares, linguagens formais]
math: true
mermaid: true
# image: //
#   path: /commons/devices-mockup.png
#   width: 800
#   height: 500
#   alt: Responsive rendering of Chirpy theme on multiple devices.
---

Em Linguagens Formais, vemos o conceito de autômatos. Eles basicamente descrevem o que é aceito por uma linguagem regular. Existem os autômatos finitos determinísticos (AFD) e os não-determinísticos (AFND). Um problema comum na criação de autômatos é a minimização. Olhando para um DFA, como podemos obter sua menor representação possível, isto é, com o menor número de estados possíveis?

## Autômatos Finitos Determinísticos

Eles são definidos como uma 5-upla ($\Sigma$, $Q$, ${q_0}$, $F$, $\delta$) onde:

$\Sigma$ é o alfabeto que descreve a linguagem\
$Q$ é o conjunto de estados do autômato\
${q_i}$ é o estado inicial do autômato\
$F$ é o conjunto de estados finais\
$\delta$ é a função de transição, que basicamente vai descrever o que cada estado vai ler e que vai levar para um outro estado (ou ele mesmo). As funções de transição vão ser bem importantes aqui neste trabalho.

### Exemplo de AFD

Vamos ver um exemplo de autômato que é descrito da seguinte forma:

$\Sigma$ = {${0, 1}$}\
$Q$ = {${q_0}, {q_1}, {q_2}, {q_3}, {q_4}, {q_5}$}\
${q_i}$ = ${q_0}$\
$F$ = {${q_1}, {q_2}, {q_4}$}\
$\delta$, a função de transição, vai ser definida da seguinte forma:

| $\delta$ | 0  | 1  |
|----------|----|----|
| q0       | q3 | q1 |
| q1       | q2 | q5 |
| q2       | q2 | q5 |
| q3       | q0 | q4 |
| q4       | q2 | q5 |
| q5       | q5 | q5 |

Isso significa que, por exemplo, a partir do ${q_0}$ podemos ler um `0` e ir para o estado ${q_3}$, ou a partir do ${q_0}$ podemos ler um `1` e ir para o estado ${q_1}$

Podemos construir um autômato a partir disso: 

![Automato_1](/assets/img/automata1.png)

Para verificar se o autômato construído aceita uma palavra, temos que ler essa palavra (percorrendo o autômato) e terminar em um estado final.

No autômato da imagem, podemos ver que ele aceita a palavra `'010'`, já que termina no estado final ${q_2}$

![Automato1_010](/assets/img/automata1_painted.png)

## Minimização de AFDs

A partir de um autômato, é possível gerar um novo autômato menor que reconhece a mesma linguagem?

Vamos olhar para o autômato do exemplo anterior: 

![Automato_1](/assets/img/automata1.png)

Esse autômato define que linguagem vai ser reconhecida. O que queremos é criar um autômato menor que este e que reconheça a exata mesma linguagem. Em outras palavras, queremos que:

$L(A) = L(B)$

Em que $L(A)$ é a linguagem reconhecida pelo AFD $A$ e $L(B)$ é a linguagem reconhecida pelo AFD $B$

### Corretude

Além de preservar a linguagem (isto é, que $L(A) = L(B)$), queremos garantir que apenas estados equivalentes sejam "unidos" em um estado só. Basicamente queremos que apenas estados indistinguíveis com respeito a linguagem aceita sejam "unidos".

Precisamos então que $L(A) \subseteq L(M)$,, sendo $L(M)$ as palavras aceitas pelo autômato $M$ minimizado.

Assim, qualquer palavra aceita pelo autômato $A$ deve ser aceita também pelo autômato minimizado $M$.

Também queremos no fim das contas que $L(M) \subseteq L(A)$. Isto é, que qualquer palavra $w$ aceita por $M$ também seja aceita por $A$. Lembrando que $M$ é criado a partir do $A$, preservando as transições e unindo apenas estados equivalentes. Dessa forma, $M$ não pode aceitar nenhuma palavra que $A$ não aceite.

Juntando tudo:

Se $L(A) \subseteq L(M)$ e $L(M) \subseteq L(A)$, ent'ao $L(A) = L(M)$

### Exemplos de minimização

Ao procurar sobre minimização de AFDs, podemos encontrar alguns algoritmos por aí. Olhando [esse site](https://www.geeksforgeeks.org/minimization-of-dfa/), podemos ver uma demonstração de minimização usando o autômato mostrado anteriormente:

![Automato_1](/assets/img/automata1.png)

Usando o algoritmo mencionado lá, podemos encontrar o seguinte autômato minimizado correspondente:

![automato_1_min](/assets/img/minimized_automata1.png)

A grande vantagem é que eliminamos estados redundantes, o que resulta em melhor eficiência no que diz respeito a complexidade de tempo e utilização de espaço.

## Representação da matriz de transição

Para objetivos futuros, é interessante pegar a função de transição definida anteriormente e transformar isso em algo que podemos trabalhar e fazer contas.

Por isso, iremos pegar a função de transição do autômato e transformar isso em uma matriz!

Pegue o seguinte dicionário para o autômato que estamos usando como exemplo:

```python
transitions = {
    0: {'a': 3, 'b': 1},
    1: {'a': 2, 'b': 5},
    2: {'a': 2, 'b': 5},
    3: {'a': 0, 'b': 4},
    4: {'a': 2, 'b': 5},
    5: {'a': 5, 'b': 5},
}
```

Ele diz que, a partir de um estado `0`, se lermos um `'a'` iremos para o estado `3` e se lermos um `'b'` iremos para o estado `1`. Se, a partir do estado `1` lermos um `'a'`, iremos para o estado `2` e se lermos um `'b'` iremos para o estado `5`. Por aí vai.

Para efetuar as contas, vamos pegar essa função de transição (em código definida como um dicionário) e transformar isso em uma matriz de matrizes. Cada matriz dentro dessa matriz maior irá representar as transições de um estado.

Vamos ver, por exemplo, como representamos as transições do estado `0`:

$$\begin{bmatrix}
0 & 0\\\
0 & 1\\\
0 & 0\\\
1 & 0\\\
0 & 0\\\
0 & 0\\
\end{bmatrix}$$

Essa vai ser a primeira matriz dentro da matriz maior e indica o estado `0`. A primeira linha indica as transições para o estado `0`, a segunda indica as transições para o estado `1`, e assim por diante. A primeira coluna indica uma leitura do `'a'` (ou da primeira letra do alfabeto que foi definido) e a segunda uma leitura do `'b'`. Isso quer dizer que, ao olhar a primeira linha e primeira coluna da primeira matriz, estamos vendo se o estado `0` faz transições para o estado `0` a partir da leitura de um `'a'`. Como o valor encontrado foi `0`, ele não faz uma transição nessas condições.

Vamos para a segunda linha da primeira matriz:

$$\begin{pmatrix}
    0 & 1
\end{pmatrix}$$

Isso indica que, do estado `0`, iremos para o estado `1` se lermos um `'b'`

Olhando para a matriz de transição inteira do autômato que estamos usando temos:

$$\begin{bmatrix}
    \begin{bmatrix}
        0 & 0\\\
        0 & 1\\\
        0 & 0\\\
        1 & 0\\\
        0 & 0\\\
        0 & 0\\
    \end{bmatrix}

    \begin{bmatrix}
        0 & 0\\\
        0 & 0\\\
        1 & 0\\\
        0 & 0\\\
        0 & 0\\\
        0 & 1\\
    \end{bmatrix}

    \begin{bmatrix}
        0 & 0\\\
        0 & 0\\\
        1 & 0\\\
        0 & 0\\\
        0 & 0\\\
        0 & 1\\
    \end{bmatrix}

    \begin{bmatrix}
        1 & 0\\\
        0 & 0\\\
        0 & 0\\\
        0 & 0\\\
        0 & 1\\\
        0 & 0\\
    \end{bmatrix}

    \begin{bmatrix}
        0 & 0\\\
        0 & 0\\\
        1 & 0\\\
        0 & 0\\\
        0 & 0\\\
        0 & 1\\
    \end{bmatrix}

    \begin{bmatrix}
        0 & 0\\\
        0 & 0\\\
        0 & 0\\\
        0 & 0\\\
        0 & 0\\\
        1 & 1\\
    \end{bmatrix}

\end{bmatrix}$$

## Decomposição QR

A minha dúvida era se seria possível minimizar um AFD utilizando decomposição QR. Isso é parte do motivo pelo qual fizemos uma matriz de transição daquele jeito na seção anterior.

Realizando decomposição QR da matriz de transição anterior, obtemos as seguintes matrizes:

Matriz $Q$ = 

$$\begin{bmatrix}
    \begin{bmatrix}
        0 & 0\\\
        0 & 1\\\
        0 & 0\\\
        -1 & 0\\\
        0 & 0\\\
        0 & 0\\
    \end{bmatrix}

    \begin{bmatrix}
        0 & 0\\\
        0 & 0\\\
        -1 & 0\\\
        0 & 0\\\
        0 & 0\\\
        0 & -1\\
    \end{bmatrix}

    \begin{bmatrix}
        0 & 0\\\
        0 & 0\\\
        -1 & 0\\\
        0 & 0\\\
        0 & 0\\\
        0 & -1\\
    \end{bmatrix}

    \begin{bmatrix}
        1 & 0\\\
        0 & 0\\\
        0 & 0\\\
        0 & 0\\\
        0 & -1\\\
        0 & 0\\
    \end{bmatrix}

    \begin{bmatrix}
        0 & 0\\\
        0 & 0\\\
        -1 & 0\\\
        0 & 0\\\
        0 & 0\\\
        0 & -1\\
    \end{bmatrix}

    \begin{bmatrix}
        0 & 0\\\
        0 & 1\\\
        0 & 0\\\
        0 & 0\\\
        0 & 0\\\
        -1 & 0\\
    \end{bmatrix}
\end{bmatrix}$$

Matriz $R$ = 

$$\begin{bmatrix}
    \begin{bmatrix}
        -1 & 0\\\
        0 & 1\\
    \end{bmatrix}

    \begin{bmatrix}
        -1 & 0\\\
        0 & -1\\
    \end{bmatrix}

    \begin{bmatrix}
        -1 & 0\\\
        0 & -1\\
    \end{bmatrix}

    \begin{bmatrix}
        1 & 0\\\
        0 & -1\\
    \end{bmatrix}

    \begin{bmatrix}
        -1 & 0\\\
        0 & -1\\
    \end{bmatrix}

    \begin{bmatrix}
        -1 & -1\\\
        0 & 0\\
    \end{bmatrix}
\end{bmatrix}$$

A intuição a partir daqui é olhar os estados equivalentes na matriz ortogonal Q, já que os estados vão estar linearmente independentes e possivelmente o processo vai eliminar algumas redundâncias entre os estados (ocasionando uma minimização)

A partir daí, podemos ver que na matriz Q os estados 1, 2 e 4 são equivalentes, enquanto os outros continuam completamente diferentes. Assim, juntamos eles e formamos o novo autômato:

![automato_1_qr](/assets/img/automata1_minimized_qr.png)

Nisso reduzimos 6 estados para 4 estados.

No entanto, se olharmos para a matriz de transição por si só, também já vemos as equivalências entre os estados. Na verdade, a decomposição QR não está fazendo nada muito útil para nós, já que continuamos apenas com uma minimização em 1 passo. Decidi então mudar o problema.

## Mudando o problema

A partir de agora iremos olhar para os autômatos probabilísticos.

A diferença dele pro anterior é que, agora, teremos uma _probabilidade_ de chegar em qualquer outro estado a partir de um estado que estamos olhando.

![prob_automata1](/assets/img/prob_automata1.png)

Olhando para esse autômato, temos uma probabilidade de 1/2 de ir para o estado $q_1$ a partir do $q_0$ ao ler um `a` e probabilidade de 1/2 de permanecer em $q_0$ ao ler um `b`.

### Exemplo maior

![prob_automata2](/assets/img/prob_automata2.png)

A partir deste exemplo, criaremos uma matriz de transição da seguinte forma: 

$$\begin{pmatrix}
    0 & 1/2 & 1/2\\\
    2/3 & 1/3 & 0\\\
    0 & 1/3 & 2/3\\
\end{pmatrix}$$

Olhando bem, isso parece muito um sistema linear dinâmico (e de fato é um!). Lembra muito o problema de PageRank.

Então, como achamos uma solução para esse sistema?

Sabemos que temos que começar a leitura do autômato a partir do estado inicial (que nesse caso é o $q_0$). Então um chute inicial seria:

$$\begin{pmatrix}
    1\\\
    0\\\
    0\\
\end{pmatrix}$$

A partir daí podemos aplicar um dos vários métodos que conhecemos para resolver sistemas desse tipo.

Você pode usar essa função em Julia:

```julia
function sist_dinamico(A, X0, n)
    Xn = A^n * X0
    return Xn
end
```

Encontramos que esse sistema converge para a matriz:

$$\begin{pmatrix}
    1/4\\\
    3/8\\\
    3/8\\
\end{pmatrix}$$

Isso quer dizer que depois de um certo tempo a probabilidade de estar em cada um desses estados é definida por esta matriz.

No entanto, nem sempre encontraremos um autômato que vai convergir para uma solução.

No autômato a seguir, todas as transições vão ter probabilidade de `1/2`:

![nao_converge](/assets/img/automato_n_converge.png)

Sua matriz de transição é:

$P$ = $$\begin{pmatrix}
    0 & 1/2 & 0 & 1/2\\\
    1/2 & 0 & 1/2 & 0\\\
    0 & 1/2 & 0 & 1/2\\\
    1/2 & 0 & 1/2 & 0\\
\end{pmatrix}$$

Ao tentar ver se achamos alguma convergência, não dá certo. Partindo do chute inicial igual ao do exemplo anterior, na iteração 1000 temos:

$$\begin{pmatrix}
    1/2\\\
    0\\\
    1/2\\\
    0\\
\end{pmatrix}$$

Enquanto que na iteração 1001 temos:
$$\begin{pmatrix}
    0\\\
    1/2\\\
    0\\\
    1/2\\
\end{pmatrix}$$

_(O comportamento continua para mais iterações.)_

## Método da potência e teorema de Perron-Frobenius

Um autovalor $\lambda$ e seu autovetor correspondente $v$ de $P$ satisfazem a equação:

$P$ $v$ = $\lambda$ $v$

Quando $\lambda = 1$, teremos:

$P$ $v$ = $v$

Que indica que o autovetor $v$ não muda sob a ação da matriz $P$. Daí podemos dizer que esse é o **estado estacionário do sistema**

### Teorema de Perron-Frobenius

O teorema nos diz que se uma matriz é primitiva, então ela tem uma distribuição **estacionária** e vai convergir para essa distribuição.

Só que também acabamos de ver que o estado estacionário do sistema acontece quando $\lambda = 1$. Dessa forma, se o autovalor dominante = 1,podemos dizer que a matriz converge e, consequentemente, tem solução.

Vamos testar isso para os exemplos anteriores aplicando o método da potência:

```julia
function potencia(C)
    n,m=size(C)
    v=randn(n,1)
    for i=1:100
        v=C*v
        v=v/norm(v)
    end
    return v, only(v'*C*v)
end
```
Para a matriz
$P$ = $$\begin{pmatrix}
    0 & 1/2 & 0 & 1/2\\\
    1/2 & 0 & 1/2 & 0\\\
    0 & 1/2 & 0 & 1/2\\\
    1/2 & 0 & 1/2 & 0\\
\end{pmatrix}$$, que sabemos que não converge:

```shell
julia> potencia(B)
([0.677848522815677; 0.2012992302931254; 0.677848522815677; 0.2012992302931254;;], 0.5458015435925113)
```

O autovalor dominante é `0.5458015435925113`, o que nos diz que o sistema não converge para o seu estado estacionário.

Vamos testar agora para o seguinte autômato:

$P$ = $$\begin{pmatrix}
    0 & 1/2 & 1/2\\\
    2/3 & 1/3 & 0\\\
    0 & 1/3 & 2/3\\
\end{pmatrix}$$

```shell
julia> potencia(A)
([-0.5773502691896257; -0.5773502691896257; -0.5773502691896257;;], 0.9999999999999998)
```

O autovalor dominante é justamente `1`, o que nos dá uma confirmação de que o sistema converge para uma solução estacionária, como vimos anteriormente.

# Referências
[1] S. C. Coutinho, L. M. Schechter. Autômatos, Linguagens Formais e Computabilidade\
[2] [Geeks for Geeks: Minimization of DFA](https://www.geeksforgeeks.org/minimization-of-dfa/)\
[3] [Lecture 8: Probabilistic Finite Automata](https://santoshv.github.io/2019CS4510/L923_scribed.pdf)\
[4] [Perron-Frobenius Theorem](https://people.math.harvard.edu/~knill/teaching/math19b_2011/handouts/lecture34.pdf)\
[5] [Probabilistic automata and Markov chains](https://www.pouly.fr/data/mpri_prob_automata.pdf)