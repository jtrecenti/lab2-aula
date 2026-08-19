# Roteiro da demo: construir o `lab2-minimo` ao vivo

Este é o guião do bloco de código da aula. São **9 checkpoints em 80 minutos**,
cada um terminando com um comando que roda e mostra alguma coisa na tela. O
resultado final é o repositório
[`lab2-minimo`](https://github.com/jtrecenti/lab2-minimo): dois módulos, cerca
de 130 linhas de código, e o ciclo completo de dado real a modelo salvo.

**Regra da demo**: nunca fique mais de 10 minutos sem rodar alguma coisa. Se um
checkpoint atrasar, corte o próximo pela metade, não o final.

**Antes de começar**: tenha o `lab2-minimo` pronto numa segunda janela, já
rodado, e o GTFS já baixado em `data/`. Se a internet cair, você copia o zip
para os alunos por outro caminho e continua.

---

## Checkpoint 0: o projeto existe (6 min)

```bash
uv init --package --name lab2-minimo --python 3.12 lab2-minimo
cd lab2-minimo
code .
uv add pandas scikit-learn requests joblib
uv add --dev pytest
```

**Falar enquanto instala:** é o mesmo começo do Lab 1. `--package` cria a pasta
`src/`, que é o que separa um projeto de uma pasta com notebooks.

**Checar:** `ls src/lab2_minimo/` mostra o `__init__.py`.

---

## Checkpoint 1: baixar e olhar o GTFS (10 min)

Criar `src/lab2_minimo/dados.py` com o topo do arquivo e duas funções:
`GTFS_URL`, `PASTA_DADOS`, `baixar_gtfs()` e `ler()`.

**Perguntar antes de escrever:** onde vocês colocariam o arquivo baixado? Deixar
a turma chegar em "numa pasta do projeto, não no Downloads".

**Falar:** três decisões nessas dez linhas.

1. A URL é constante no topo, não escondida no meio de uma função.
2. `if not destino.exists()` faz o download acontecer **uma vez**. Numa aula com
   trinta pessoas, isso é a diferença entre trinta downloads e trinta e zero.
3. `dtype=str` na leitura: o código da linha `1012-10` não é número, e
   `encoding="latin-1"` porque o feed da SPTrans não é UTF-8. Ler tudo como
   texto e converter depois é mais seguro do que deixar o pandas adivinhar.

**Rodar:**

```bash
uv run python -c "from lab2_minimo.dados import ler; print(ler('routes').head())"
uv run python -c "from lab2_minimo.dados import ler; print(ler('frequencies').head())"
```

**Falar sobre o que apareceu:** `frequencies.txt` é o alvo. Cada linha diz
"desta hora até aquela, sai um ônibus a cada tantos segundos". É a oferta
programada da cidade inteira, em 37 mil linhas.

---

## Checkpoint 2: a hora que passa de 24 (10 min)

Escrever `para_minutos()`.

**Antes de escrever, mostrar o problema:**

```bash
uv run python -c "from lab2_minimo.dados import ler; import pandas as pd; \
s = ler('stop_times')['arrival_time']; print(s[s > '24:00:00'].head())"
```

**Falar:** o GTFS permite hora maior que 24. Uma viagem que sai 23h50 e chega
00h20 do dia seguinte é registrada como `24:20:00`. Se você jogar isso no
`pd.to_datetime`, quebra. Por isso convertemos na mão.

Escrever o teste em `tests/test_dados.py` **junto** com a função, e rodar:

```bash
uv run pytest
```

**Falar:** este é o momento de dizer por que testar. O teste não está aqui para
provar que `1 + 1 = 2`. Ele está aqui porque daqui a três meses alguém vai
"simplificar" essa função usando `to_datetime`, e o teste vai avisar. Teste bom
guarda uma **armadilha conhecida do domínio**, não uma linha de código.

---

## Checkpoint 3: a tabela de modelagem (14 min)

Escrever `montar_tabela()`. É o pedaço mais longo; vá por partes e comente cada
junção.

1. **Agregar** `stop_times` por viagem: número de paradas, início e fim.
2. **Preparar** `frequencies`: headway em minutos, hora de início.
3. **Juntar** as quatro tabelas.
4. **Derivar** as variáveis do código da linha.
5. **Limpar** duração zero.

**Parar no `.str.strip()` e insistir:** sem ele, `"10"` e `"10 "` viram duas
categorias diferentes no one-hot, com a mesma informação repartida em duas
colunas. É o tipo de bug que não dá erro, só piora o modelo em silêncio.

**Parar no código da linha e insistir:** `8000-10`. O primeiro dígito é a área
de operação, o sufixo é o tipo de atendimento. Isso é **feature engineering**:
a informação estava lá, mas não em nenhuma coluna.

**Rodar:**

```bash
uv run python -c "from lab2_minimo.dados import montar_tabela; \
t = montar_tabela(); print(t.shape); print(t.head())"
```

**Checar:** 37357 registros.

---

## Checkpoint 4: a receita (12 min)

Criar `src/lab2_minimo/modelo.py` e escrever `construir_receita()`.

**Falar, com o slide da receita ainda fresco:** isto é o `recipe()` do
tidymodels. Um caminho para numérica, um para categórica, cada um sendo ele
mesmo um `Pipeline`.

Quatro decisões, uma por linha:

- mediana para o que falta nas numéricas, porque resiste a valor extremo;
- padronizar, porque o lasso penaliza coeficiente e sem escala comum ele castiga
  quem tem unidade grande;
- categoria `desconhecido` para o que falta nas categóricas, porque ausência é
  informação;
- `min_frequency` e `handle_unknown`, porque em produção aparece categoria nova.

**Rodar, para ver a receita funcionando:**

```bash
uv run python -c "
from lab2_minimo.dados import montar_tabela, NUMERICAS, CATEGORICAS
from lab2_minimo.modelo import construir_receita
X = montar_tabela()[NUMERICAS + CATEGORICAS]
print('antes: ', X.shape)
print('depois:', construir_receita().fit_transform(X).shape)
"
```

**Falar sobre o número que apareceu:** 8 colunas viraram dezenas. As categóricas
explodiram em dummies. E repare que **este `fit_transform` é só para olhar**: no
ajuste de verdade quem chama `fit` é o `Pipeline`, dentro de cada dobra.

---

## Checkpoint 5: dividir, montar o pipeline, buscar (15 min)

Escrever `separar()`, `treinar()` e `avaliar()`.

**Pergunta para a turma antes de escrever `separar()`:** a mesma linha de ônibus
aparece umas trinta vezes nesta tabela, uma por faixa horária. Se eu sortear 25%
das linhas da tabela para teste, o que acontece?

Deixar alguém chegar em "a mesma linha cai nos dois lados". Aí escrever a
divisão sorteando **linhas**, e não registros:

```python
linhas = tabela[GRUPO].unique()
linhas_treino, _ = train_test_split(linhas, test_size=0.25, random_state=SEMENTE)
e_treino = tabela[GRUPO].isin(linhas_treino)
```

**Se alguém perguntar por que não usar o `train_test_split` direto na tabela:**
porque ele não tem `groups=`. A biblioteca tem o `GroupShuffleSplit` para isso,
mas ele é um validador cruzado, então o `.split()` devolve um gerador e precisa
de `next()` para pegar a primeira divisão. Dá o mesmo resultado; três linhas
legíveis ensinam mais que uma linha opaca. Na validação cruzada não tem
alternativa: ali é `GroupKFold`.

**Falar:** `groups` viaja **por fora** do `X`. É diferente do R, onde o
`route_id` está dentro do `data.frame`. Esquecer o `groups=` no `.fit()` é o
erro número um de quem vem do tidymodels.

Ao escrever `treinar()`, parar no dicionário do grid:

```python
{"modelo__alpha": [0.01, 0.1, 1.0]}
```

**Falar:** o `__` é um caminho. `modelo__alpha` significa *o parâmetro `alpha`
do passo chamado `modelo`*. É o equivalente do `tune()` do R, só que apontado de
fora em vez de marcado por dentro. E ele alcança a receita também:
`receita__num__imputar__strategy`.

E no `scoring="neg_mean_absolute_error"`: o scikit-learn sempre maximiza, por
isso o `neg_`.

---

## Checkpoint 6: rodar tudo (8 min)

Escrever `main()`, com o baseline primeiro.

**Falar sobre o baseline:** uma linha. `DummyRegressor(strategy="median")`. Sem
ele você não sabe se MAE de 10 minutos é bom ou péssimo.

Registrar o comando no `pyproject.toml`:

```toml
[project.scripts]
lab2 = "lab2_minimo.modelo:main"
```

**Rodar:**

```bash
uv run lab2
```

Saída esperada:

```
37357 registros | 1142 linhas de ônibus
baseline  teste  MAE 11.12 min  R2 -0.216
lasso     teste  MAE 10.77 min  R2  0.111   (treino R2 0.206)
floresta  teste  MAE  9.94 min  R2  0.208   (treino R2 0.625)
```

**Três perguntas para a turma, nesta ordem:**

1. O modelo ganhou do baseline? Quanto, em minutos?
2. Por que o R² do baseline é negativo?
3. Por que o R² de treino da floresta é o triplo do de teste?

A resposta da 2 é a boa: o R² compara com prever a **média**, e o baseline prevê
a **mediana**, que é o chute constante certo sob MAE. Métrica e baseline
precisam falar a mesma língua.

---

## Checkpoint 7: o experimento do vazamento (6 min)

Este é o ponto alto da aula, e agora ele cabe em **uma linha**. Em `separar()`,
sortear registros em vez de linhas:

```python
treino, teste = train_test_split(tabela, test_size=0.25, random_state=SEMENTE)
```

e, em `treinar()`, trocar `GroupKFold` por `KFold(n_splits=5, shuffle=True,
random_state=SEMENTE)`.

**Antes de rodar, mostrar o estrago:**

```python
len(set(treino[GRUPO]) & set(teste[GRUPO]))   # 1115 de 1142 linhas nos dois lados
```

```bash
uv run lab2
```

| | MAE | R² |
| --- | --- | --- |
| floresta, divisão por grupo | 9,94 min | **0,208** |
| floresta, divisão aleatória | 6,40 min | **0,624** |

O R² **triplicou**. O modelo não melhorou em nada.

**Falar:** este é o número que vocês veriam num relatório e acreditariam. O
modelo passou a reconhecer a linha em vez de aprender o padrão de oferta. Se a
SPTrans usar esse modelo para estimar a frequência de uma linha **nova**, ele
entrega o número de baixo, não o de cima.

**Desfazer a mudança** (`git checkout` ou Ctrl+Z) e seguir.

**Fechar com a pergunta que vale para a integradora:** no projeto do seu grupo,
qual é o agrupamento? Cliente, município, empresa, processo, período? Anota
agora, porque isso muda a avaliação inteira do modelo de vocês.

---

## Checkpoint 8: para o GitHub (4 min)

```bash
cat > .gitignore    # .venv/, data/, __pycache__/
git add -A
git commit -m "pipeline minimo do lab 2"
gh repo create lab2-minimo --public --source=. --push
```

**Falar:** `data/` está no `.gitignore`. O repositório guarda o **código que
recria** os dados, não os dados. Quem clonar roda `uv sync && uv run lab2` e
chega no mesmo lugar.

---

## Checkpoint 9: o robô confere e publica (6 min)

Este checkpoint é para **colar, não digitar**. O arquivo está pronto no
repositório de referência; abra, leia em voz alta e cole.

Criar `notebooks/relatorio.qmd` (colar) e `.github/workflows/ci.yml` (colar),
depois:

```bash
git add -A && git commit -m "relatorio e integracao continua" && git push
```

Abrir a aba **Actions** no navegador, ao vivo, e acompanhar.

**Falar enquanto roda:** são dois trabalhos em sequência.

1. `testes` roda a suíte. Doze segundos, não baixa nada.
2. `relatorio` só começa se o primeiro passar (`needs: testes`). Ele baixa o
   GTFS, treina os dois modelos, renderiza o relatório e publica no GitHub
   Pages. Um minuto e quarenta e cinco.

**Falar sobre o relatório, abrindo o `.qmd`:** repare no que ele **não** faz.
Não define função, não limpa dado, não treina modelo na mão. Ele importa o
pacote e chama as mesmas funções que o `uv run lab2` chama. Notebook conta
história; pacote guarda código. É o oposto do notebook de 400 células que a
gente tinha antes do Lab 1.

**Abrir a página publicada:** <https://jtrecenti.github.io/lab2-minimo/>

**Fechar:** ninguém rodou nada. O relatório no ar reflete o último commit, e vai
continuar refletindo. Isso é o que a palavra "reprodutível" quer dizer na
prática, e é o degrau que falta antes do Docker e da API, que são a disciplina
de Deploy.

**Se a aba Actions demorar**, siga para a discussão e volte a olhar depois. O
robô não precisa de plateia.

---

## Se atrasar

| Atraso | O que cortar |
| --- | --- |
| 5 min | O teste do checkpoint 2 vira "olhem no repositório depois". |
| 10 min | Escrever só o lasso; a floresta entra como "troquem uma linha". |
| 15 min | Colar o `montar_tabela()` pronto e explicar lendo, em vez de digitar. |
| 20 min | Pular para o checkpoint 7 com o repositório pronto na outra janela. O experimento do vazamento **não** se corta. |
| 25 min | Cortar o checkpoint 9: mostrar a aba Actions e a página já publicadas no repositório de referência, sem colar nada. |

## Plano B: o esqueleto

Se a turma estiver lenta para digitar, distribua o repositório com as funções já
declaradas e os corpos vazios:

```python
def para_minutos(hora: pd.Series) -> pd.Series:
    """Converte "HH:MM:SS" em minutos desde a meia-noite."""
    ...
```

Preencher corpo por corpo dá o mesmo aprendizado e economiza metade do tempo de
digitação. Deixe o esqueleto pronto na pasta Labs do Blackboard, mesmo que não
use.
