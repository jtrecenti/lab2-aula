# Material do Lab 2: pipelines com scikit-learn

Material de aula do Laboratório 2 de **Prática Avançada em Data Science e
Visualização** (PADS Insper, turma PADSONL08), 21 de agosto de 2026.

O código fica em repositórios próprios; aqui ficam os slides, o roteiro, os
exercícios e o par de documentos que compara R e Python.

## Uma inconsistência de propósito

Os repositórios de código deste laboratório ensinam que **arquivo gerado não
entra no Git**. Aqui o HTML renderizado dos slides e dos dois documentos
**entra**, porque ele é o produto: quem recebe este material abre o arquivo, não
roda o Quarto. Os intermediários (`_files/`, `.quarto/`) continuam ignorados.

Vale saber quando a regra se aplica e quando não se aplica.

## O que tem aqui

| Arquivo | O que é | Para quem |
| --- | --- | --- |
| [`plano-de-aula.md`](plano-de-aula.md) | roteiro da aula inteira, 2h55 | monitor |
| [`roteiro-demo.md`](roteiro-demo.md) | guião da programação ao vivo, 9 checkpoints | monitor |
| [`slides/tidymodels-para-sklearn.qmd`](slides/tidymodels-para-sklearn.qmd) | os slides, em duas partes | turma |
| [`exercicios.md`](exercicios.md) | exercícios, do aquecimento ao desafio | turma |
| [`comparacao/tidymodels.qmd`](comparacao/tidymodels.qmd) | o mesmo ajuste em R | turma |
| [`comparacao/sklearn.qmd`](comparacao/sklearn.qmd) | o mesmo ajuste em Python | turma |
| [`comparacao/dados/dicionario-viagens.xlsx`](comparacao/dados/dicionario-viagens.xlsx) | dicionário da base, em Excel | turma |
| [`comparacao/dicionario.py`](comparacao/dicionario.py) | script que gera o dicionário | monitor |

## Os dois repositórios de código

**[`lab2-minimo`](https://github.com/jtrecenti/lab2-minimo)**: o que sai da demo
ao vivo. Dois módulos, cerca de 130 linhas, construído do zero em 80 minutos,
sem agente de IA. Já faz o ciclo completo: baixa o GTFS da SPTrans, monta a
tabela, ajusta lasso e floresta com validação por grupo e salva o modelo. Tem
também um relatório em Quarto e uma GitHub Action que roda os testes, treina os
modelos e **publica o relatório sozinha** a cada push:
<https://jtrecenti.github.io/lab2-minimo/>.

**[`lab2-sptrans`](https://github.com/jtrecenti/lab2-sptrans)**: a versão
completa, de referência. O mesmo problema com cruzamento com a API Olho Vivo,
mais variáveis, relatório em Quarto, testes, integração contínua e coleta
agendada. É onde os exercícios levam.

A ideia é que a turma **construa** o primeiro e **leia** o segundo.

## Os documentos irmãos

`comparacao/tidymodels.qmd` e `comparacao/sklearn.qmd` ajustam o **mesmo lasso e
a mesma floresta**, sobre a **mesma tabela**, com a mesma sequência de seções.
Servem para abrir lado a lado e ler a mesma etapa nas duas linguagens.

Resultado, no conjunto de teste:

| modelo | MAE (R) | MAE (Python) | R² (R) | R² (Python) |
| --- | --- | --- | --- | --- |
| baseline (mediana) | 690 s | 667 s | -0,26 | -0,22 |
| lasso | 589 s | 594 s | 0,28 | 0,25 |
| floresta | 574 s | 562 s | 0,31 | 0,27 |

Os números não batem na casa decimal porque os geradores de números aleatórios
de R e Python são diferentes, então a divisão treino e teste não é a mesma.

### Como renderizar

```bash
# Python
cd comparacao
uv sync
PYTHONUTF8=1 uv run quarto render sklearn.qmd

# R (precisa de tidymodels, glmnet e ranger)
quarto render tidymodels.qmd

# slides
cd ../slides
quarto render tidymodels-para-sklearn.qmd
```

No Windows, o `PYTHONUTF8=1` evita um `UnicodeDecodeError` do plotnine ao ser
chamado pelo Quarto.

## A base

Os dados em `comparacao/dados/viagens.csv.gz` são a tabela analítica gerada pelo
`lab2-sptrans` (`uv run lab2 transform`), guardada aqui para os dois documentos
rodarem sozinhos.

O **dicionário** está em `comparacao/dados/dicionario-viagens.xlsx`, com quatro
abas:

- **Leia-me**: o que é a base, o que é uma linha da tabela, chave, fontes e três
  cuidados que mudam a leitura dos resultados;
- **Colunas**: as 33 colunas, com papel (alvo, preditora numérica, preditora
  categórica, identificador, auxiliar), unidade, faltantes, faixa e origem;
- **Categorias**: todos os níveis das colunas categóricas, com contagem;
- **Exemplo**: quinze registros reais, para ver a cara do dado.

Ele é gerado por `uv run python dicionario.py`: as descrições são escritas à
mão, as estatísticas saem do arquivo. Se a base for regerada, rode de novo e o
dicionário acompanha.
