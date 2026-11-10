# Transformações Geométricas

Transformações geométricas alteram a posição, o tamanho ou a orientação dos
objetos de uma cena. Em computação gráfica, é comum representar cada ponto por
um vetor e aplicar uma matriz para obter sua nova posição.

Esta página usa **vetores-coluna**: a matriz aparece à esquerda do ponto
(`P' = M P`). Assim, a translação fica na última coluna e, em uma composição,
a matriz mais à direita é aplicada primeiro.

::: info Ponto e vetor
Um ponto em 2D pode ser escrito em coordenadas homogêneas como
$P = (x, y, 1)^T$. O último componente permite representar translações como
multiplicações de matrizes. Em 3D, usamos $P = (x, y, z, 1)^T$.
:::

## Identidade: não transformar

A matriz identidade preserva o ponto. Ela é o elemento neutro da multiplicação
de matrizes: multiplicar uma transformação pela identidade não altera essa
transformação.

```math
I =
\begin{bmatrix}
1 & 0 & 0 \\
0 & 1 & 0 \\
0 & 0 & 1
\end{bmatrix}
\qquad
I P = P
```

## Translação: deslocar

Transladar significa somar um deslocamento às coordenadas do ponto. Para
`P = (x, y)`, com deslocamento `(dx, dy)`, temos:

```math
x' = x + dx
\qquad
y' = y + dy
```

Em coordenadas homogêneas, a mesma operação é uma multiplicação:

```math
\begin{bmatrix}
x' \\
y' \\
1
\end{bmatrix}
=
\begin{bmatrix}
1 & 0 & dx \\
0 & 1 & dy \\
0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
x \\
y \\
1
\end{bmatrix}
```

Por exemplo, transladar `P = (1, 2)` por `(3, -1)` produz `P' = (4, 1)`.
O deslocamento neutro é `(0, 0)`.

::: tip Em 3D
Para deslocar um ponto por `(dx, dy, dz)`, usa-se a matriz homogênea:

```math
T =
\begin{bmatrix}
1 & 0 & 0 & dx \\
0 & 1 & 0 & dy \\
0 & 0 & 1 & dz \\
0 & 0 & 0 & 1
\end{bmatrix}
```
:::

## Escala: ampliar ou reduzir

A escala multiplica cada coordenada por um fator. Em 2D, com fatores `sx` e
`sy`:

```math
x' = sx \cdot x
\qquad
y' = sy \cdot y
```

```math
\begin{bmatrix}
x' \\
y' \\
1
\end{bmatrix}
=
\begin{bmatrix}
sx & 0 & 0 \\
0 & sy & 0 \\
0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
x \\
y \\
1
\end{bmatrix}
```

- Fatores maiores que `1` ampliam naquela direção.
- Fatores entre `0` e `1` reduzem.
- Fator `1` mantém a coordenada.
- Fator `0` colapsa a coordenada correspondente.
- Fator negativo também inverte a orientação naquela direção.

Quando `sx = sy`, a escala é uniforme e preserva as proporções. Quando os
fatores são diferentes, o objeto pode ficar mais largo ou mais alto.

::: warning Escala em relação à origem
A matriz acima escala em relação à origem `(0, 0)`. Um objeto afastado da
origem também se aproxima ou se afasta dela. Para escalar em torno de outro
ponto, é necessário combinar translações com a escala.
:::

Em 3D, a matriz é:

```math
S =
\begin{bmatrix}
sx & 0 & 0 & 0 \\
0 & sy & 0 & 0 \\
0 & 0 & sz & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
```

## Rotação: girar

No plano, uma rotação de ângulo `θ` em torno da origem transforma
`P = (x, y)` em:

```math
x' = x \cos(\theta) - y \sin(\theta)
\qquad
y' = x \sin(\theta) + y \cos(\theta)
```

```math
\begin{bmatrix}
x' \\
y' \\
1
\end{bmatrix}
=
\begin{bmatrix}
\cos(\theta) & -\sin(\theta) & 0 \\
\sin(\theta) & \cos(\theta) & 0 \\
0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
x \\
y \\
1
\end{bmatrix}
```

Com ângulos positivos no sentido anti-horário, girar `(1, 0)` em `90°` resulta
em `(0, 1)`. A rotação preserva distâncias e ângulos; ela muda a orientação,
mas não o tamanho do objeto.

::: tip Identidades do seno e do cosseno
As coordenadas de um ponto girado dependem da projeção de seus componentes nos
novos eixos, descrita por seno e cosseno. Como
$\cos(-\theta) = \cos(\theta)$ e $\sin(-\theta) = -\sin(\theta)$, trocar o
sinal do ângulo inverte o sentido da rotação.
:::

Em 3D, a rotação é definida em torno de um eixo. Para vetores-coluna, as
matrizes são:

```math
R_x(\theta) =
\begin{bmatrix}
1 & 0 & 0 & 0 \\
0 & \cos(\theta) & -\sin(\theta) & 0 \\
0 & \sin(\theta) & \cos(\theta) & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
\qquad
R_y(\theta) =
\begin{bmatrix}
\cos(\theta) & 0 & \sin(\theta) & 0 \\
0 & 1 & 0 & 0 \\
-\sin(\theta) & 0 & \cos(\theta) & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
```

```math
R_z(\theta) =
\begin{bmatrix}
\cos(\theta) & -\sin(\theta) & 0 & 0 \\
\sin(\theta) & \cos(\theta) & 0 & 0 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
```

O ângulo neutro é `0°` (ou uma volta completa, `360°`). As imagens a seguir
ilustram a rotação:

![Demonstração geométrica da rotação — etapa 1](imgs/Transformacao_Rotacao_1.png)
![Demonstração geométrica da rotação — etapa 2](imgs/Transformacao_Rotacao_2.png)

Para ver a dedução geométrica da rotação, consulte também
[Como chegar na matriz de rotação](./rotacao.md).

## Transformar em torno de um ponto fixo

As matrizes de escala e rotação atuam em torno da origem. Para usar um ponto
`C = (cx, cy)` como pivô:

1. Translade o pivô para a origem: `T(-cx, -cy)`.
2. Aplique a escala ou rotação desejada: `A`.
3. Translade de volta: `T(cx, cy)`.

Com vetores-coluna, a transformação completa é:

```math
P' = T(cx, cy) \, A \, T(-cx, -cy) \, P
```

O fator à direita atua primeiro. Por exemplo, para girar `(2, 0)` em `90°`
ao redor do pivô `(1, 0)`, primeiro o ponto passa a `(1, 0)` em relação à
origem; depois da rotação, fica em `(0, 1)`; a translação de volta produz
`(1, 1)`.

## Composição: combinar transformações

Uma sequência de transformações pode ser combinada em uma única matriz:

```math
P' = M_n \cdots M_2 M_1 P
```

A multiplicação de matrizes é associativa, mas não comutativa. Portanto, mudar
a ordem geralmente muda o resultado. Por exemplo, para `P = (1, 0)`, uma
translação de `(1, 0)` seguida de uma rotação de `90°` resulta em `(0, 2)`;
primeiro girar e depois transladar resulta em `(1, 1)`.

::: warning Leia a composição da direita para a esquerda
Como usamos vetores-coluna, em `P' = R T P` a translação `T` acontece primeiro
e a rotação `R` depois. Outras bibliotecas podem usar vetores-linha e uma
convenção diferente. O importante é manter consistente a ordem dos vetores,
das matrizes e das multiplicações.
:::

![Rotação seguida de translação](imgs/Transformacao_Composicao_1.png)
![Translação seguida de rotação](imgs/Transformacao_Composicao_2.png)

## Resumo

| Transformação | Efeito | Parâmetro neutro |
| --- | --- | --- |
| Translação | Soma um deslocamento às coordenadas | Deslocamento zero |
| Escala | Multiplica as coordenadas por fatores | Fator `1` |
| Rotação | Gira o ponto em torno de um eixo | Ângulo `0°` |

Em coordenadas homogêneas, essas operações podem ser representadas por
matrizes `3 × 3` em 2D e `4 × 4` em 3D. A matriz identidade não altera os
pontos, e a composição permite aplicar várias transformações por meio de uma
única multiplicação por ponto.
