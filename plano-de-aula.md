# Plano do Lab 2: pipelines com scikit-learn

**Disciplina**: Prática Avançada em Data Science e Visualização (PADS Insper, turma PADSONL08)
**Data**: sexta, 21 de agosto de 2026, 19h00 às 22h30 (remota)
**Responsável**: Julio Trecenti (monitor)
**Duração planejada**: 2h55, das 19h00 às 21h55

O slot é de 3h30. O plano ocupa 2h55 e deixa **35 minutos de folga** de
propósito: problemas de ambiente vão acontecer (aconteceram no Lab 1), e se não
acontecerem a turma sai mais cedo ou usa o tempo no projeto da integradora.

---

## Onde este lab se encaixa

No Lab 1 (05/08) a turma saiu do notebook solto e chegou a um pacote Python
versionado no GitHub, com `uv`, ambiente virtual, Quarto e GitHub Pages. O
fechamento daquele lab já anunciou este: *"no Lab 2 eu vou falar de pipelines do
scikit-learn, e de quebra um de-para do que a gente tinha de tidymodels no R
para o que a gente vai ter no scikit-learn"*.

A aula tem duas metades que se encaixam:

1. **Slides**: a ponte tidymodels para scikit-learn, com dois modelos concretos
   (lasso e floresta aleatória) sobre a base da SPTrans.
2. **Demo ao vivo**: construir, do zero e digitando, um repositório que faz o
   ciclo inteiro, de dado real a modelo salvo. Sem agente de IA.

## Objetivos de aprendizagem

Ao final da aula, o aluno deve conseguir:

1. **Traduzir** um fluxo de tidymodels para scikit-learn e vice-versa (OV6).
2. **Construir** um `Pipeline` com `ColumnTransformer` e explicar por que o
   pré-processamento precisa morar dentro dele (OV6, prepara Deploy).
3. **Escolher** uma estratégia de validação coerente com o uso do modelo, e
   reconhecer vazamento por grupo (OV6).
4. **Ler** um resultado criticamente: comparar com baseline, distinguir treino
   de teste e dizer o que o número não responde (OV1, OV5).
5. **Organizar** um projeto de modelagem em módulos, com testes e um comando de
   entrada (OV4, prepara Deploy).

Nível de Bloom: Aplicar, Analisar e Avaliar.
Dinâmica: exposição dialogada e programação ao vivo acompanhada.

## Antes da aula

Enviar no Teams, com dois dias de antecedência:

- **Confirmar que `uv` e Git funcionam**, com o comando de teste:
  ```bash
  uv --version
  git --version
  ```
  Quem não tiver, refazer o passo a passo do Lab 1 **antes** da aula. Instalação
  em sala custou uma hora no Lab 1 e não pode custar de novo.
- Aviso de que hoje **todos digitam junto**. Não é aula de assistir.
- Link do repositório de referência
  ([jtrecenti/lab2-sptrans](https://github.com/jtrecenti/lab2-sptrans)) com o
  pedido explícito de **não** abrir antes da aula. Quem lê o gabarito antes
  perde a demo.

::: nota
**Apresentações de gráficos.** A planilha de apresentações
([link](https://docs.google.com/spreadsheets/d/1rUQfvhQMjehs0hrNUdlAZCbUz63O59famTXF9j0WIv0/edit))
não tem linha para 21/08. Criar a linha e abrir para até 4 voluntários. Se
ninguém se inscrever, antecipar os quatro de 26/08 (Willian, Beatriz, Leonardo
Koga e Celso), o que alivia aquela aula. Combinar com a Gabrielle, representante
da turma.
:::

## Roteiro

| Horário | Bloco | Formato | Duração |
| --- | --- | --- | --- |
| 19h00 | Abertura e recados | exposição | 5 min |
| 19h05 | **Apresentações de gráficos** (até 4 alunos) | apresentação + crítica | 20 min |
| 19h25 | **Slides, parte 1**: o mapa e a receita | exposição dialogada | 25 min |
| 19h50 | Intervalo | | 10 min |
| 20h00 | **Demo ao vivo**: construir o repositório do zero | programação ao vivo acompanhada | 80 min |
| 21h20 | **Slides, parte 2**: resultados e armadilhas | exposição dialogada | 15 min |
| 21h35 | Discussão: o que o número diz e o que não diz | plenário | 10 min |
| 21h45 | Fechamento: entrega e o que vem no Lab 3 | exposição | 10 min |
| **21h55** | **Fim** | | **2h55** |

**Slides mais demo: 2h00.** É o coração da aula e não deve ser comprimido.

---

## Detalhamento por bloco

### 19h00 (5 min) Abertura

- Retomada de uma frase do Lab 1: *o Claude Code faria essa estrutura sozinho, e
  é justamente por isso que você precisa saber qual estrutura pedir*. Hoje
  vamos ao contrário: **ninguém usa agente de IA na demo**. A ideia é sair da
  aula sabendo escrever, não só revisar.
- Aula extra (compensação do sábado 01/08): confirmar data com a Gabrielle.
- Combinar a dinâmica: todo mundo com o editor aberto, câmeras abertas na demo,
  e a regra de interromper na hora em que der erro, não dez minutos depois.

### 19h05 (20 min) Apresentações de gráficos

Até quatro alunos, 3 a 5 minutos cada, sobre uma visualização boa ou ruim vista
na mídia. Depois de cada uma, uma pergunta fixa da turma: **qual decisão esse
gráfico apoia?**

Amarra o bloco no tema do dia: o modelo também precisa apoiar decisão, não só
existir.

### 19h25 (25 min) Slides, parte 1

Slides 1 a 12 de `slides/tidymodels-para-sklearn.qmd`:

- o mapa dos oito pacotes do R para uma biblioteca do Python;
- o problema de hoje: prever o headway da rede da SPTrans;
- os dois modelos escolhidos, **lasso** e **floresta aleatória**, e por que
  esses dois (um exige padronização, o outro não; se a mesma receita serve aos
  dois, ela está bem construída);
- a diferença de filosofia: receita por **papel** contra receita por **coluna**;
- dividir por grupo, a receita inteira lado a lado, o de-para dos `step_*` e as
  três armadilhas.

Ritmo: mostrar o R primeiro, deixar a turma reconhecer, e só então o Python.

### 20h00 (80 min) Demo ao vivo

Guião completo em [`roteiro-demo.md`](roteiro-demo.md). São nove checkpoints,
cada um terminando com um comando que roda:

| | Checkpoint | Min |
| --- | --- | --- |
| 0 | o projeto existe (`uv init --package`) | 6 |
| 1 | baixar e olhar o GTFS | 10 |
| 2 | a hora que passa de 24, e o primeiro teste | 10 |
| 3 | a tabela de modelagem | 14 |
| 4 | a receita (`ColumnTransformer`) | 12 |
| 5 | dividir por grupo, pipeline, busca | 15 |
| 6 | `uv run lab2`: o resultado na tela | 8 |
| 7 | o experimento do vazamento | 6 |
| 8 | para o GitHub | 4 |
| 9 | o robô confere e publica (colar, não digitar) | 6 |

O resultado é o repositório
[`lab2-minimo`](https://github.com/jtrecenti/lab2-minimo): dois módulos, cerca
de 130 linhas, o ciclo completo funcionando e um relatório publicado sozinho em
<https://jtrecenti.github.io/lab2-minimo/>.

**O checkpoint 7 é o ponto alto da aula** e não se corta. Trocar
`GroupShuffleSplit` por `ShuffleSplit` faz o R² da floresta ir de **0,21 para
0,62** sem que o modelo tenha melhorado nada.

### 21h20 (15 min) Slides, parte 2

Slides 13 a 20:

- o que o lasso escolheu, com os coeficientes reais;
- a floresta trocando uma linha, com a **mesma** receita;
- a tabela de resultados nos dois ecossistemas, lado a lado;
- como ler essa tabela;
- `rsq()` contra `rsq_trad()`, e o `neg_` do `scoring`;
- onde o scikit-learn não segura sua mão;
- salvar e servir; a cola final.

### 21h35 (10 min) Discussão

Plenário, três perguntas na tela:

1. **O R² deu 0,21. O modelo presta?** Puxar para o uso: para achar linha fora
   do padrão, o resíduo já serve; para prever a oferta de uma linha nova, não
   basta.
2. **Quanto o R² subiu quando quebramos a validação?** Coletar os números da
   turma e fechar: esse é o número que vocês veriam num relatório e
   acreditariam.
3. **Que dado falta?** Demanda (Censo por setor censitário), realizado (coleta
   da API Olho Vivo), contrato de concessão. Fechar com: *o modelo diz quais
   dados faltam, e isso já é um entregável*.

E a pergunta que atravessa para a integradora: **no projeto do seu grupo, qual é
o agrupamento?** Cliente, município, empresa, processo, período. Cada grupo
anota antes de sair.

### 21h45 (10 min) Fechamento

- **O que entregar** (opcional, com feedback): o repositório construído hoje,
  com pelo menos um exercício de [`exercicios.md`](exercicios.md) resolvido.
  Prazo de uma semana, link no Teams.
- **O repositório completo**, [`lab2-sptrans`](https://github.com/jtrecenti/lab2-sptrans),
  liberado agora: é o mesmo problema com cruzamento de bases, relatório em
  Quarto, integração contínua e coleta agendada. Os exercícios levam de um ao
  outro.
- **Os dois documentos irmãos**, `tidymodels.qmd` e `sklearn.qmd`, para quem
  quiser ver o mesmo ajuste nas duas linguagens, lado a lado.
- Próximo lab (31/08): dashboard interativo conectado à saída do modelo, UX e
  WebAssembly.
- Recado sobre o Deploy: o `.joblib` gerado hoje é literalmente o arquivo que
  vira endpoint de API lá.

---

## Planos B

| Se acontecer | Fazer |
| --- | --- |
| Alguém sem `uv` ou Git | Assistir e digitar depois. Não parar a demo para instalar. |
| Internet do Insper cair (aconteceu no Lab 1) | Passar o compartilhamento de tela para um aluno voluntário e conduzir por voz. |
| Download do GTFS lento na sala inteira | Distribuir o zip por outro canal; são 12 MB. |
| Turma lenta para digitar | Usar o esqueleto com as funções declaradas e os corpos vazios (ver `roteiro-demo.md`). |
| Atraso de 15 min | Colar o `montar_tabela()` pronto e explicar lendo, em vez de digitar. |
| Atraso de 20 min | Pular para o checkpoint 7 com o repositório pronto na outra janela. |
| Sobrar tempo | Exercício 4 de `exercicios.md`, em duplas, com mentoria. |

## Material

| Arquivo | O que é |
| --- | --- |
| `slides/tidymodels-para-sklearn.qmd` | os slides, em duas partes |
| `roteiro-demo.md` | o guião da programação ao vivo, checkpoint a checkpoint |
| `exercicios.md` | exercícios, do aquecimento ao desafio |
| `comparacao/tidymodels.qmd` | o mesmo ajuste em R |
| `comparacao/sklearn.qmd` | o mesmo ajuste em Python |
| `lab2-minimo` (repo) | o que sai da demo |
| `lab2-sptrans` (repo) | a versão completa, de referência |
