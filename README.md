# Pulsares e Meteoritos: Um mesmo céu, diversos conhecimentos.

Trabalho da disciplina de **Mineração de Dados e Aprendizado de Máquina**. A proposta (`docs/proposta.pdf`) pede duas bases de dados percorridas pelas etapas do processo de descoberta de conhecimento (KDD): hipóteses, seleção, pré-processamento, transformação, mineração e, principalmente, **avaliação e interpretação**. Os modelos são os vistos em aula.

## Por que esses dois temas

Os dois temas são de astronomia, o que deixa o trabalho mais interessante de fazer e de apresentar. A escolha também tem um motivo técnico: as duas bases têm **perfis de dados opostos**, e cada uma testa uma parte diferente do que a proposta pede.

| | Pulsares (HTRU2) | Meteoritos (NASA) |
|---|---|---|
| Fonte | UCI ML Repository, id 372 | NASA Open Data, *Meteorite Landings* |
| Tamanho | 17.898 candidatos | 45.716 registros |
| Atributos | 8 contínuos (estatísticas do sinal) | mistos: massa, ano, lat/lon + classe mineralógica (461 valores) e `fall` |
| Alvo | pulsar real vs. ruído (9,2% positivos) | `Fell` (visto caindo) vs. `Found` (encontrado depois) |
| Desafio principal | desbalanceamento, redundância entre atributos | **viés de observação**, dados categóricos e sujos |

- **Pulsares** é um problema "limpo": sem nulos e com atributos contínuos derivados do próprio sinal. A pergunta é física: *o que distingue um pulsar de interferência?* Serve para comparar classificadores (Logística, LDA/QDA, Naive Bayes, KNN, SVM, MLP) e discutir métricas sob desbalanceamento.
- **Meteoritos** é mais categórico e tem uma tendência forte a apresentar viés. O registro de um meteorito depende de **alguém estar olhando**: quedas testemunhadas aparecem onde há gente, e encontrados aparecem onde é fácil procurar (Antártida, desertos). O modelo pode acertar muito sem aprender nada sobre o meteorito. Por isso a base serve para regressão, árvores sobre atributos categóricos e, sobretudo, para mostrar que um bom score não garante uma conclusão válida.

## O que fazemos e como os dados respondem

Cada base começa com hipóteses explícitas (H1 a H4), e os notebooks tentam **confirmá-las ou refutá-las** com os dados:

- **Pulsares:** perfil de pulso mais "pontiagudo" nos pulsares (H1), perfil integrado mais informativo que a curva DM-SNR (H2), separação quase linear (H3), acurácia enganosa sob desbalanceamento (H4).
- **Meteoritos:** viés de observação espacial e temporal (H1), cluster antártico de pequenos *Found* (H2), quedas observadas mais massivas (H3), metálicos super-representados entre os *Found* (H4).

Algumas hipóteses se confirmaram, outras foram refutadas e reformuladas. A H1 temporal dos meteoritos, por exemplo, deu o oposto do esperado. Esse vai e volta entre a etapa 6 e a etapa 1 é o que a proposta pede. O resumo está em [`CONCLUSOES.md`](CONCLUSOES.md).

## Estrutura

```
.
├── Exploratory_Data_Analysis_pulsares_e_meteoritos.ipynb   # etapas 1–4: hipóteses, limpeza, transformação, EDA
├── Modelagem_e_interpretação_pulsares_e_meteoritos.ipynb   # etapas 5–6: modelos, avaliação, interpretação
├── CONCLUSOES.md                                           # constatações iniciais para o relatório
└── docs/
    ├── Meteorite_Landings.csv                              # base de meteoritos (NASA)
    └── proposta.pdf                                        # enunciado do trabalho
```

A base HTRU2 não fica no repositório: ela é baixada pelo pacote `ucimlrepo` quando o notebook roda.

## Como rodar

**Google Colab (recomendado):** abra os notebooks no Colab e rode em ordem (primeiro o EDA, depois a Modelagem). Quando a célula de meteoritos pedir, faça upload de `docs/Meteorite_Landings.csv`.

**Localmente:**

```bash
python -m venv .venv && source .venv/bin/activate
pip install numpy pandas matplotlib seaborn scikit-learn scipy statsmodels ucimlrepo jupyter
jupyter lab
```

Rode os notebooks a partir da raiz do repositório para que o CSV em `docs/` seja encontrado sem upload. O download da HTRU2 precisa de internet.

A modelagem leva alguns minutos, principalmente por causa da SVM RBF e do MLP com validação cruzada.

## Decisões de metodologia

- Divisão treino/teste estratificada (80/20) e validação cruzada estratificada de 5 folds. Escalonamento e transformações ficam dentro de `Pipeline` para evitar vazamento.
- Métricas principais: **AUC-PR** e **recall**, porque a acurácia esconde o desbalanceamento das duas bases.
- O limiar de decisão é escolhido com previsões *out-of-fold* do treino, nunca olhando o teste.
- Meteoritos são modelados **sem a Antártida**, onde o rótulo é trivialmente *Found*.
- Os modelos de aula que não foram usados têm justificativa no final do notebook de modelagem.
