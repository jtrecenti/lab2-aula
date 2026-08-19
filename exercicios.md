# Exercícios do Lab 2

Estes exercícios partem do repositório que você construiu na aula, o
`lab2-minimo`. Cada um tem uma pergunta a responder, não só um código a rodar.
Anote as respostas em `docs/respostas.md`: elas são o entregável.

Ponto de partida:

```bash
uv sync
uv run lab2
```

---

## 1. Trocar de modelo (aquecimento, 10 min)

O dicionário `MODELOS` em `modelo.py` define quais modelos rodam. Acrescente
mais dois:

```python
from sklearn.dummy import DummyRegressor
from sklearn.ensemble import HistGradientBoostingRegressor
from sklearn.tree import DecisionTreeRegressor

MODELOS = {
    ...,
    "arvore": (DecisionTreeRegressor(random_state=SEMENTE),
               {"modelo__max_depth": [4, 8, 12, None]}),
    "boosting": (HistGradientBoostingRegressor(random_state=SEMENTE),
                 {"modelo__learning_rate": [0.05, 0.1]}),
}
```

**Perguntas**

1. Quantas linhas você precisou mudar para adicionar dois modelos? O que isso
   diz sobre a separação entre receita e modelo?
2. Monte a tabela dos quatro modelos, com MAE e R² de teste. A diferença entre o
   melhor e o pior é maior ou menor que a diferença entre o baseline e o pior?
   O gargalo está no algoritmo ou nos dados?
3. Qual modelo tem a maior distância entre treino e teste? Por quê?

---

## 2. Medir o vazamento direito (o principal, 20 min)

Na aula fizemos o experimento uma vez. Agora faça direito, e com número.

Transforme a escolha em parâmetro, para poder rodar as duas versões sem editar
o código toda vez:

```python
def separar(tabela, por_grupo: bool = True):
    ...
```

Rode as duas e monte a tabela: MAE e R² de teste, para lasso e floresta, com e
sem agrupamento.

**Perguntas**

1. Quanto o R² da floresta subiu? E o do lasso? Por que o efeito é muito maior
   na floresta do que no lasso?
2. Escreva em uma frase o que exatamente o modelo passou a fazer que antes não
   fazia.
3. A SPTrans quer usar o modelo para estimar a frequência de uma **linha nova**,
   que nunca operou. Qual das duas avaliações descreve o que ele vai entregar?
4. E se o uso fosse outro: preencher uma faixa horária faltante de uma linha que
   **já existe**. Qual avaliação estaria certa aí?
5. No projeto da integradora do seu grupo, existe um agrupamento parecido?
   Cliente, município, empresa, processo, período? Escreva qual é e por quê.

::: nota
A pergunta 4 é a mais importante da lista. Não existe validação "correta" no
abstrato: existe validação que imita o uso.
:::

---

## 3. Adicionar uma variável (25 min)

O GTFS tem informação que o `lab2-minimo` ainda não usa. Escolha **uma**:

- **Extensão da linha em km**: está em `shapes.txt`, na coluna
  `shape_dist_traveled` (em metros, acumulada, então o último ponto do traçado
  dá o total). Junte por `shape_id`, que está em `trips.txt`. Isso permite
  calcular também a velocidade programada.
- **Hora como variável cíclica**: `sin(2*pi*hora/24)` e `cos(2*pi*hora/24)`. A
  hora 23 e a hora 0 são vizinhas, e o modelo não sabe disso.
- **Tamanho do nó na rede**: quantas linhas distintas atendem a primeira parada
  da viagem. Proxy de terminal ou ponto de troca.
- **Cor da linha**: `route_color`, em `routes.txt`. A SPTrans usa a cor para
  classificar o tipo de serviço. Entra como categórica.

Passos:

1. Crie a coluna em `dados.py`, com um comentário explicando o que ela mede.
2. Registre o nome em `NUMERICAS` ou `CATEGORICAS`.
3. Escreva **um teste** para a lógica nova.
4. Rode `uv run pytest` e depois `uv run lab2`.

**Perguntas**

1. O MAE de teste melhorou? Quanto, em minutos?
2. Você precisou mexer em `construir_receita()`, `separar()` ou `treinar()`?
   Por que não?
3. `shapes.txt` tem 53 MB. Quanto tempo o ETL passou a levar? Isso mudaria a sua
   decisão sobre onde guardar a tabela pronta?

---

## 4. Do resíduo para a decisão (20 min)

A predição não é o produto. O que vira decisão é o **resíduo**: linhas cuja
oferta programada é muito mais espaçada do que a de linhas parecidas.

Monte a tabela de resíduos do teste e olhe as dez maiores:

```python
residuos = X_teste.assign(
    observado=y_teste,
    predito=modelo.predict(X_teste),
).assign(residuo=lambda d: d["observado"] - d["predito"])
```

Escolha um caminho:

- **Visualização**: os resíduos mudam por período do dia? Existe um horário em
  que a rede é sistematicamente pior do que o modelo espera?
- **Investigação**: pegue as três linhas com maior resíduo, procure o itinerário
  no site da SPTrans e diga se o modelo achou um problema real ou uma
  peculiaridade de cadastro.
- **Recorte**: os maiores resíduos se concentram em alguma área de operação?

**Pergunta única**: escreva o parágrafo que você mandaria para um gestor da
SPTrans. Uma decisão, um número, uma ressalva.

---

## 5. Comparar com o R (20 min, opcional)

Abra `comparacao/tidymodels.qmd` e `comparacao/sklearn.qmd` lado a lado. Eles
ajustam o mesmo lasso e a mesma floresta, sobre a mesma tabela.

**Perguntas**

1. A receita do R termina com 44 colunas e a do Python com 62, com o mesmo dado.
   Por quê? (Dica: `step_dummy()` e `OneHotEncoder`.)
2. O `rsq()` do `yardstick` devolve `NA` para o baseline e o `r2_score` devolve
   um número negativo. Qual dos dois é o "R²" que você aprendeu? Qual função do
   R corresponde ao do Python?
3. Rode o `tidymodels.qmd` mudando `mixture = 1` para `mixture = 0`. Que modelo
   virou? Qual é o equivalente em Python?

---

## 6. Desafio: trocar a tarefa (para quem terminar antes)

Em vez de prever o intervalo em minutos, preveja se a linha é de **alta
frequência** (intervalo até 15 minutos) ou não.

O que muda:

- o alvo vira binário (`headway_min <= 15`);
- `Lasso` vira `LogisticRegression(penalty="l1", solver="liblinear")`;
- `RandomForestRegressor` vira `RandomForestClassifier`;
- `scoring` vira `"roc_auc"` ou `"balanced_accuracy"`;
- as métricas viram matriz de confusão, precisão e revocação.

**O que não muda**: `construir_receita()`, o `Pipeline`, o `GroupKFold` e a
estrutura do projeto inteiro.

**Pergunta**: quantas linhas você precisou mudar? Se foram muitas, alguma coisa
estava no lugar errado.

---

## 7. Para quem quiser ir até o fim

O repositório completo,
[`lab2-sptrans`](https://github.com/jtrecenti/lab2-sptrans), é este mesmo
problema com tudo que não coube na aula: cruzamento com a API Olho Vivo por
proximidade geográfica, mais variáveis, relatório em Quarto, integração contínua
e coleta agendada da frota.

Compare os dois repositórios e responda: **o que o completo tem que o mínimo não
tem, e qual dessas coisas você levaria para o projeto da integradora?** Nem tudo
vale a pena em todo projeto, e saber cortar também é parte do trabalho.

---

## Entrega (opcional, com feedback)

O seu `lab2-minimo` no GitHub, com:

- o exercício 2 e mais um resolvido;
- as respostas em `docs/respostas.md`;
- os testes passando (`uv run pytest`).

Prazo: uma semana. Manda o link no Teams.
