# Computação Gráfica - Unidade 2  

Conceitos básicos de Computação Gráfica: estruturas de dados para geometria, sistemas de coordenadas na biblioteca gráfica (OpenGL/OpenTK), primitivas básicas (vértices, linhas, polígonos, círculos e curvas cúbicas – splines).  
Objetivo: aplicar os conceitos básicos de sistemas de referências e modelagem geométrica em Computação Gráfica.  

[Anotações do quadro](aulaAnotacoesQuadro)  

## [Atividades - Aula](./Atividade2/README.md "Atividades - Aula")  

## Ambiente de Desenvolvimento

Para iniciar as atividades precisamos configurar o [Ambiente de Desenvolvimento](AmbienteDesenvolvimento.md "Ambiente de Desenvolvimento")  
Agora que já temos o Ambiente de Desenvolvimento instalado vamos testá-lo usando alguns projetos de exemplo: [OpenTK_TestarAmbiente](./OpenTK/README.md).  

## Conteúdo

### Software de interface para o hardware gráfico

![Software de interface para o hardware gráfico](cg-slides_u2_imagens/slide-03-interface-hardware.png)

### OpenGL - Open Graphics Library

- **Interface:** aplicações de “renderização” gráfica
  - imagens coloridas de alta qualidade
    - primitivas geométricas (2D e 3D) e
    - por imagens
  - independência de sistemas de janelas
  - independência de sistemas operacionais
  - compatível com quase todas as arquiteturas
  - interface gráfica dominante

#### Renderização

- primitivas geométricas (2D e 3D) e
- por imagens

![Canais de imagem e geometria no pipeline OpenGL](cg-slides_u2_imagens/slide-05-pipeline-opengl.png)  

### OpenGL - Pipeline Gráfico: Visão geral

```mermaid
  flowchart LR
    A[Aplicação C#/CPU<br/>Cria vértices e estado] --> B[VBO/VAO<br/>Upload de dados para GPU]
    B --> C[Vertex Shader<br/>Transforma cada vértice]
    C --> D[Primitive Assembly<br/>Monta triângulos/linhas]
    D --> E[Rasterization<br/>Converte primitivas em fragmentos]
    E --> F[Fragment Shader<br/>Calcula cor de cada fragmento]
    F --> G[Testes por fragmento<br/>Depth/Stencil/Alpha]
    G --> H[Blending<br/>Combina com cor já existente]
    H --> I[Framebuffer]
    I --> J[Swap Buffers<br/>Imagem exibida na tela]

    K[Uniforms/Texturas] --> C
    K --> F
```

#### Visão detalhada por etapas

```mermaid
  flowchart LR
    %% ---------------------------------
    %% OpenGL Graphics Pipeline (staged)
    %% ---------------------------------

    subgraph IA[Input Assembler]
        IA1[Vertex Buffers VBO]
        IA2[Index Buffer EBO opcional]
        IA3[Vertex Array Object VAO<br/>layout dos atributos]
        IA4[Montagem de primitivas<br/>pontos linhas triangulos]
        IA1 --> IA3
        IA2 --> IA4
        IA3 --> IA4
    end

    subgraph VS[Vertex Processing / Shaders]
        VS1[Vertex Shader<br/>transformacao por vertice]
        VS2[Uniforms<br/>M V P materiais parametros]
        VS3[Saidas por vertice<br/>posicao clip-space e varyings]
        VS2 --> VS1
        VS1 --> VS3
    end

    subgraph PR[Primitive Processing]
        PR1[Clipping no volume de visao]
        PR2[Perspective Divide]
        PR3[Viewport Transform<br/>NDC para coordenadas de tela]
        PR4[Rasterization<br/>geracao de fragmentos]
        PR5[Interpolacao de varyings]
        PR1 --> PR2 --> PR3 --> PR4 --> PR5
    end

    subgraph FS[Fragment Processing / Shaders]
        FS1[Fragment Shader<br/>cor profundidade opcional]
        FS2[Texturas samplers]
        FS3[Uniforms por draw call]
        FS2 --> FS1
        FS3 --> FS1
    end

    subgraph PFO[Per-Fragment Operations]
        PFO1[Scissor Test opcional]
        PFO2[Stencil Test opcional]
        PFO3[Depth Test]
        PFO4[Blending]
        PFO5[Color Mask / Depth Mask]
        PFO1 --> PFO2 --> PFO3 --> PFO4 --> PFO5
    end

    subgraph OM[Output Merger]
        OM1[Framebuffer alvo<br/>color depth stencil]
        OM2[Back Buffer]
        OM3[Swap Buffers<br/>apresentacao]
        OM1 --> OM2 --> OM3
    end

    IA --> VS --> PR --> FS --> PFO --> OM
```

#### Visão detalhada por etapas - sem DirectX

```mermaid
  flowchart LR
    %% OpenGL Core Profile 3.3+ Pipeline

    subgraph APP[Aplicacao CPU]
        APP1[Configura estado GL]
        APP2[glBindVertexArray]
        APP3[glUseProgram]
        APP4[glBindBuffer glBindTexture]
        APP5[glDrawArrays / glDrawElements]
        APP1 --> APP2 --> APP3 --> APP4 --> APP5
    end

    subgraph VERTEX[Vertex Specification + Vertex Shader]
        V1[Vertex Arrays VAO]
        V2[Vertex Buffer Objects VBO]
        V3[Element Buffer Object EBO opcional]
        V4[glVertexAttribPointer / glEnableVertexAttribArray]
        V5[Vertex Shader]
        V6[Saidas em clip space gl_Position + varyings]
        V1 --> V4
        V2 --> V4
        V3 --> V4
        V4 --> V5 --> V6
    end

    subgraph PRIM[Primitive Assembly and Rasterization]
        P1[Primitive Assembly]
        P2[Clipping]
        P3[Perspective Divide]
        P4[Viewport Transform]
        P5[Rasterization]
        P6[Interpolacao de varyings]
        P1 --> P2 --> P3 --> P4 --> P5 --> P6
    end

    subgraph FRAG[Fragment Shader]
        F1[Fragment Shader]
        F2[Uniforms]
        F3[Samplers / Textures]
        F4[Saidas: cor e opcional gl_FragDepth]
        F2 --> F1
        F3 --> F1
        F1 --> F4
    end

    subgraph PERFRAG[Per-Fragment Operations]
        PF1[Scissor Test]
        PF2[Stencil Test]
        PF3[Depth Test]
        PF4[Blending]
        PF5[Color/Depth/Stencil Write Masks]
        PF1 --> PF2 --> PF3 --> PF4 --> PF5
    end

    subgraph FB[Framebuffer]
        FB1[Default Framebuffer ou FBO]
        FB2[Color Attachments]
        FB3[Depth/Stencil Attachments]
        FB4[Double Buffer SwapBuffers]
        FB1 --> FB2
        FB1 --> FB3
        FB2 --> FB4
    end

    APP --> VERTEX --> PRIM --> FRAG --> PERFRAG --> FB
```

<http://www.opengl.org>

![The OpenGL Machine](cg-slides_u2_imagens/slide-06-maquina-opengl.png)

### OpenGL - “Renderizador”

- Primitivas geométricas
  - pontos, linhas e polígonos
- Primitivas de imagens
  - imagens e bitmaps
  - canais independentes: geometria e imagem
    - ligação via mapeamento de textura
- “Renderização” dependente do estado
  - cores, materiais, fontes de luz, etc.

### OpenGL - Sistema de Janelas

- Trata apenas de “renderização”
  - independente do sistema de janelas
    - X, Win32, Mac O/S
  - não possui funções de entrada
- Necessita interagir com o sistema operacional e o sistema de janelas
  - interface dependente do sistema é mínima
    - realizada através de bibliotecas adicionais: GLX, AGL, WGL

### OpenGL - GLU, OpenGL Utility Library

- Funções para auxiliar a tarefa de produzir imagens complexas
  - manipulação de imagens
  - polígonos não-convexos
  - curvas
  - superfícies
  - esferas
  - etc.

### OpenGL - GLUT, OpenGL Utility Toolkit

- API de janelas para o OpenGL
  - independente do sistema de janelas
  - indicado para programas:
    - pequeno e médio porte
  - processamento orientado à chamada de eventos (callbacks)
  - dispositivos de entrada

API: Interface para Programação de Aplicações

### OpenGL - Prefixos

- OpenGL
  - `gl`, `GL`, `GL_`
    - para comandos, tipos e constantes, respectivamente
- GLU
  - `glu`, `GLU`, `GLU_`
- GLUT
  - `glut`, `GLUT`, `GLUT_`

### OpenGL - Passos Básicos

- Configurar e abrir janela (canvas)
- Inicializar o estado do OpenGL
- Registrar funções de entrada de callback
  - desenho (“renderização”)
  - redimensionamento do canvas
  - entrada: mouse, teclado, etc.

### Programação Convencional

![Fluxograma da programação convencional](cg-slides_u2_imagens/slide-13-programacao-convencional.png)

### Programação por Eventos

![Aplicação e gerenciador de callbacks na programação por eventos](cg-slides_u2_imagens/slide-14-programacao-eventos.png)

### OpenGL - Primitivas Geométricas

Especificadas por vértices.

- `GL_POINTS`
- `GL_LINES`
- `GL_LINE_LOOP`
- `GL_LINE_STRIP`
- `GL_TRIANGLES`
- `GL_QUADS`
- `GL_QUAD_STRIP`
- `GL_POLYGON`
- `GL_TRIANGLE_STRIP`
- `GL_TRIANGLE_FAN`

![Primitivas geométricas especificadas por vértices](cg-slides_u2_imagens/slide-15-primitivas-geometricas.png)

### OpenGL - Formato, Especificação do Vértice

`glVertex3fv( v )`

- Número de componentes:
  - 2 - (x,y)
  - 3 - (x,y,z)
  - 4 - (x,y,z,w)
- Tipo do dado:
  - `b` - byte
  - `ub` - unsigned byte
  - `s` - short
  - `us` - unsigned short
  - `i` - int
  - `ui` - unsigned int
  - `f` - float
  - `d` - double
- Vetor:
  - omitir “v” para forma escalar
  - `glVertex2f( x, y )`

![Componentes, tipos e vetor na especificação do vértice](cg-slides_u2_imagens/slide-16-especificacao-vertice.png)

### Splines

- Splines (ou curva polinomial)
  - origem:
    - desenvolvida: De Casteljau em 1957 (P. De Casteljau, Citröen)
    - formalizado: Bézier 1960 (Pierre Bézier)
    - aplicações CAD/CAM
  - pontos de controle
  - bastante utilizada em modelagem tridimensional

![Curva spline e exemplos de código](cg-slides_u2_imagens/slide-17-splines-exemplos.png)

Tudo pode ser modelado por fórmulas, o problema é o custo envolvido.

![Equações e desenho do Batman](cg-slides_u2_imagens/slide-18-batman-equacoes.png)

<https://www.wolframalpha.com/input/?i=batman+equation>

#### Curvas de Bézier e pontos de controle

Com 2, 3, 4, 5 ou n pontos de controle.  
<http://en.wikipedia.org/wiki/B%C3%A9zier_curve>  
<http://www.ibiblio.org/e-notes/Splines/Intro.htm>  

##### 2 pontos de controle

[spline_2ptos](./cg-slides_u2_imagens/spline_2ptos.png "spline_2ptos")  
![spline_2ptos](./cg-slides_u2_imagens/spline_2ptos.mov "spline_2ptos")  

----

##### 3 pontos de controle

[spline_3ptos](./cg-slides_u2_imagens/spline_3ptos.png "spline_3ptos")  
![spline_3ptos](./cg-slides_u2_imagens/spline_3ptos.mov "spline_3ptos")  

----

##### 4 pontos de controle

[spline_4ptos](./cg-slides_u2_imagens/spline_4ptos.png "spline_4ptos")  
![spline_4ptos](./cg-slides_u2_imagens/spline_4ptos.mov "spline_4ptos")  

----

##### N pontos de controle

![spline_Nptos](./cg-slides_u2_imagens/spline_Nptos.mov "spline_Nptos")  

----

#### Splines: exemplo de implementação

![Código e resultado gráfico do exemplo de spline](cg-slides_u2_imagens/slide-24-spline-codigo.png)

### Splines (Bezier)

$$
B(t) = (1-t)^3 P_0 + 3t(1-t)^2 P_1 + 3t^2(1-t)P_2 + t^3P_3, \quad t \in \[0,1].
$$

```text
Bx(0,5) = 0,125 * 30 + 0,375 *   30 + 0,375 * 130 + 0,125 * 130 =   80
By(0,5) = 0,125 * 20 + 0,375 * 100 + 0,375 * 130 + 0,125 *   20 = 100
```

![Fórmula de Bézier, cálculo e tabela de pesos](cg-slides_u2_imagens/slide-25-bezier-formula-tabela.png)

![Tabela e gráficos dos pesos de P0, P1, P2 e P3](cg-slides_u2_imagens/slide-26-bezier-pesos.png)

### Splines: modelagem

![Curvas, superfícies e modelos com splines](cg-slides_u2_imagens/slide-27-splines-modelagem.png)

Ver exemplo: <http://www.ibiblio.org/e-notes/Splines/animation.html>  

### Splines: visualização

![Modos de visualização de um modelo](cg-slides_u2_imagens/slide-29-splines-visualizacao.png)

<!-- ### Box

![Anotações do quadro sobre Box - 1](cg-slides_u2_imagens/slide-30-box-quadro-01.png)

![Anotações do quadro sobre Box - 2](cg-slides_u2_imagens/slide-31-box-quadro-02.png)

### Tabela senos/cosenos e Teorema de Pitágoras

![Tabela trigonométrica, fórmulas e teorema de Pitágoras](cg-slides_u2_imagens/slide-32-trigonometria-pitagoras.png)

```text
radiano:=grau * PI / 180;
```

```java
public double RetornaX(double a){
    return (5 * Math.cos(Math.PI * a / 180.0));
}
public double RetornaY(double a){
    return (5 * Math.sin(Math.PI * a / 180.0));
}
```

### Computational Geometry Algorithms Library - CGAL

<http://www.cgal.org/>

- 2D Convex hulls
- Delaunay Triangulation 2
- Regular Triangulations
- Spatial Searching

![Exemplos de algoritmos da CGAL](cg-slides_u2_imagens/slide-33-cgal.png)

### Tabelas e fórmulas de referência

![Tabelas matemáticas de referência - 1](cg-slides_u2_imagens/slide-34-referencia-matematica-01.png)

![Tabelas matemáticas de referência - 2](cg-slides_u2_imagens/slide-35-referencia-matematica-02.png)

![Tabelas matemáticas de referência - 3](cg-slides_u2_imagens/slide-36-referencia-matematica-03.png)

![Tabelas matemáticas de referência - 4](cg-slides_u2_imagens/slide-37-referencia-matematica-04.png)

![Tabelas matemáticas de referência - 5](cg-slides_u2_imagens/slide-38-referencia-matematica-05.png)
 -->

----

## ⏭ [Unidade 3](../Unidade3/README.md "Unidade 3")  

## Principais Referências Bibliográficas​

O material utilizado nesta disciplina é baseado nessas Referências Bibliográficas​.  

### Links OpenGL

- OpenGL: <https://en.wikipedia.org/wiki/OpenGL>  
- OpenGL (aprendendo): <https://learnopengl.com/>  
- OpenGL (aprendendo, livro): <https://learnopengl.com/book/book_pdf.pdf>
- OpenGL (aprendendo, GitHub): <https://github.com/JoeyDeVries/LearnOpenGL>  
- Khronos: <https://www.khronos.org/api/opengl>  

### Links OpenTK

- OpenTK: <https://opentk.net/>  
- OpenTK GitHub: <https://github.com/opentk>  
- OpenTK FAQ: <https://opentk.net/faq.html>  
- OpenTK FAQ (usando OpenTK): <https://opentk.net/faq.html#installing-and-using-opentk>  
- OpenTK (aprendendo): <https://opentk.net/learn/index.html>  
- OpenTK (aprendendo, GitHub): <https://github.com/opentk/LearnOpenTK>  
- OpenTK (API): <https://opentk.net/api/index.html>  

### Links OpenTK fontes

- OpenTK (fontes, não usar): <https://github.com/opentk/opentk>  

### Links IDE VSCode

<https://github.com/LDTTFURB/site/tree/main/ProjetosEnsino/Topicos/VSCode>  

### Links C\#

<https://github.com/LDTTFURB/site/tree/main/ProjetosEnsino/Topicos/CSharp>  

----------

<!--
TODO: arrumar as fontes bibliográficas  
## Principais Referências Bibliográficas​
-->
