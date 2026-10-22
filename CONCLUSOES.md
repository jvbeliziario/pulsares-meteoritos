# Conclusões e constatações iniciais

Notas para o relatório, tiradas das saídas dos dois notebooks. Os números vêm da última execução registrada; se os notebooks forem rodados de novo, confira se nada mudou.

**Resumo**

| Base | Hipótese | Resultado |
|---|---|---|
| Pulsares | H1: perfil de pulso mais pontiagudo | ✅ confirmada |
| Pulsares | H2: IP separa melhor que DM | ✅ confirmada |
| Pulsares | H3: separação quase linear | ✅ confirmada (aproximada, não perfeita) |
| Pulsares | H4: acurácia enganosa | ✅ confirmada |
| Meteoritos | H1: viés de observação (espaço) | ✅ confirmada |
| Meteoritos | H1: viés de observação (tempo) | ❌ refutada, efeito no sentido oposto |
| Meteoritos | H2: cluster antártico | ✅ confirmada |
| Meteoritos | H3: *Fell* mais massivos | ✅ confirmada, inclusive controlando confundidores |
| Meteoritos | H4: metálicos entre os *Found* | ❌ global / ✅ sem Antártida (confundimento) |

---

## Parte A: Pulsares (HTRU2)

### Dados
- 17.898 candidatos, 8 atributos contínuos, sem nulos nem duplicatas. Nenhuma linha removida.
- Prevalência de pulsares: **9,2%**. Um classificador que sempre responde "ruído" tem 90,8% de acurácia e recall zero.
- Dois blocos de atributos muito redundantes: `ip_kurt`–`ip_skew` (r ≈ 0,95) e `dm_kurt`–`dm_skew` (r ≈ 0,92).
- Os outliers da classe 1 **são o sinal**: removê-los apagaria os pulsares mais evidentes.

### H1: perfil mais pontiagudo (confirmada)
- Mediana da assimetria do perfil integrado: **≈ 2,96** em pulsares e **≈ 0,19** no ruído. Mediana da curtose: **≈ 11,6** e **≈ 0,12**.
- O ruído fica perto de zero, como um sinal sem pico. Os pulsares se espalham por uma faixa larga, o que reflete a variedade de perfis reais.

### H2: IP é mais informativo que DM (confirmada)
- AUC-PR univariada média: **IP 0,78** vs. **DM 0,38**. O melhor atributo isolado é `ip_skew` (AUC-ROC 0,972, AUC-PR 0,908).
- Logística só com IP: AUC-PR 0,913. Só com DM: 0,350. Com todos: 0,922. **Adicionar DM ao IP quase não acrescenta nada.**
- Detalhe interessante: só com DM, a Random Forest chega a 0,782 e a Logística fica em 0,350. A informação de DM existe, mas é **não linear**.

### H3: separação quase linear (confirmada, com ressalva)
- Melhor linear (SVM linear) com AUC-PR 0,924 vs. melhor não linear (MLP) com 0,929. O ganho de **+0,005** é menor que o desvio-padrão entre folds (≈ 0,013), então não é significativo.
- Uma Logística usando **só a PC1** já chega a AUC-PR 0,842. A PCA, sem usar rótulos, encontra a direção que separa as classes.
- **Margem rígida não existe:** mesmo com C = 1000, sobram 3.153 violações de margem e 310 erros de treino. Os dados não são linearmente separáveis, só *quase*.
- Uma árvore de profundidade 3, quase só com `ip_skew` e `dm_std`, tem AUC-ROC 0,976 no teste. O problema é essencialmente unidimensional.

### H4: desbalanceamento e limiar (confirmada)
- No limiar padrão, o MLP tem recall 0,835 e acurácia 0,980.
- Com o limiar ajustado no treino para recall ≥ 0,95, o recall no teste vai a **0,945**, mas a precisão cai para **0,38** (acurácia 0,854).
- Na Logística, `class_weight="balanced"` dá um meio-termo: recall 0,921 e precisão 0,784.
- O ponto de operação é uma **decisão do problema**: um falso positivo custa segundos de inspeção humana, um falso negativo é um pulsar que não será descoberto.

### Regressão entre atributos
- Curtose em função da assimetria: R² linear de 0,889 (IP) e 0,845 (DM), contra **0,978 e 0,985** com base polinomial de grau 2. A base RBF fica no meio (0,933 e 0,973).
- O resíduo do modelo linear tem padrão curvo. Um r de 0,95 escondia uma relação não linear.

### Naive Bayes e redundância
- É o pior em precisão no teste (0,638, com 161 falsos positivos), porque a independência condicional não vale.
- O Yeo-Johnson ajuda: a AUC-PR em CV sobe de 0,855 para 0,898. Gaussianizar as caudas melhora um modelo que assume normalidade.

### Interpretação dos coeficientes
- Na Logística completa, `ip_skew` tem coeficiente +6,08 e `ip_kurt` tem −3,05, embora a curtose seja *positivamente* associada a pulsar. É **multicolinearidade**: o sinal não deve ser lido isoladamente.
- Com um representante por par redundante, o coeficiente de `ip_skew` fica em +4,42 e a AUC-PR praticamente não muda (0,921 vs. 0,922).

### Perfil dos erros
- Medianas dos falsos negativos: `ip_skew` 0,68 e `dm_kurt` 69,7, muito mais próximas do ruído (0,18 e 90,2) que dos pulsares detectados (3,59 e 1,4).
- Os pulsares perdidos são **fracos ou atípicos**: com estes 8 atributos eles parecem ruído. **O limite está nos dados, não no modelo.** Melhorar exigiria novos atributos extraídos do sinal bruto.

### Melhor modelo
No teste, SVM linear e Logística empatam em AUC-PR (0,933). A Random Forest tem o melhor recall no limiar padrão (0,869, com 43 pulsares perdidos). Pela simplicidade e pela interpretabilidade, a **Logística** é a recomendação. A H3 justifica essa escolha.

---

## Parte B: Meteoritos (NASA)

### Dados
- 45.716 registros. 7.315 sem coordenadas e mais 6.214 em (0, 0), que é placeholder de "desconhecido". Há uma longitude de 354°, 19 massas iguais a zero e 461 valores distintos de `recclass`.
- `recclass` foi agrupada em 8 famílias mineralógicas. Isso é uma simplificação e deve ser documentado.
- `Fell` é só **2,4%** do total e **10,9%** fora da Antártida.

### H1: viés de observação
- **Espacial (confirmada):** só 3,4% dos georreferenciados foram vistos caindo, e eles se concentram em regiões povoadas (Europa, Índia, leste da Ásia, leste dos EUA). Os *Found* se concentram em desertos e na Antártida.
- **Temporal (refutada):** o esperado era *Fell* mais recentes. Os dados mostram o contrário. Ano mediano: **1924 para *Fell*** e **1998 para *Found***. Proporção de *Fell* por época: 54% antes de 1900, 31% em 1900–1968, 5% em 1969–1999 e 1,6% de 2000 em diante.
- Leitura: antes do século XX, quase só se conheciam quedas testemunhadas. A partir de 1969, as campanhas sistemáticas na Antártida e nos desertos geram dezenas de milhares de *Found*. **O viés existe, mas no sentido oposto ao previsto.** É o caso que a proposta descreve na etapa 6: os resultados obrigam a revisar a hipótese.

### H2: Antártida (confirmada)
- 22.096 meteoritos (69% dos georreferenciados) estão abaixo de −60°. **Todos são *Found*, nenhum *Fell*.**
- Massa mediana: ≈ 13 g na Antártida contra ≈ 350 g fora dela. São fragmentos pequenos, achados porque se destacam no gelo.
- Decisão: a Antártida fica fora da modelagem de `fall`, porque nela a latitude determina o rótulo sozinha.

### H3: *Fell* mais massivos (confirmada)
- Fora da Antártida, a massa mediana é **2.905 g para *Fell*** e **278 g para *Found***. Mann-Whitney unilateral: p ≈ 5 × 10⁻¹⁴⁷.
- Regressão `log10(massa) ~ fell + ano + família`: o coeficiente de `fell` é 0,547 (IC95% de 0,47 a 0,62). Um meteorito visto caindo tem massa **≈ 3,5× maior**, mantendo ano e família constantes.
- O R² de 0,26 é baixo, como esperado: a massa depende de fatores que não estão nos dados. O que interessa é o **sinal e o tamanho** do coeficiente, e ambos se mantêm depois do controle.
- Bônus: metálicos (*Iron*) são ≈ 21× mais massivos que condritos ordinários.

### H4: metálicos entre os *Found* (refutada globalmente, confirmada sem Antártida)
- No geral, *Iron* representa 2,9% dos *Found* e 4,4% dos *Fell*, o oposto do esperado.
- Sem a Antártida, *Iron* sobe para **8,8% dos *Found*** e continua em 4,4% dos *Fell*. **A hipótese se confirma.**
- A coleta antártica, dominada por condritos ordinários pequenos, diluía a proporção de metálicos. É um caso de **confundimento**, ou paradoxo de Simpson: a conclusão se inverte quando se controla a região.

### Classificação de `fall`: o modelo aprende quem estava olhando
- A Random Forest chega a AUC-PR **0,888** (referência aleatória: 0,108). A Logística fica em 0,714.
- Comparação de conjuntos de atributos (AUC-PR, Random Forest):
  - completo: 0,888
  - **só contexto (ano + local): 0,812**
  - **só meteorito (massa + família): 0,323**
- Quase todo o poder preditivo vem de **quando e onde** o meteorito foi registrado, não do meteorito em si. O score alto mede o viés de observação, não a física.
- Na Logística, sem o ano a AUC-PR cai de 0,714 para 0,459. A Random Forest compensa com lat/lon (0,858). A informação temporal e a espacial são parcialmente substituíveis, porque as duas codificam o esforço de busca.
- Odds ratios da Logística: ser *Iron* multiplica a chance de ser *Fell* por 0,021, e cada desvio-padrão a mais no ano multiplica por 0,196.

### Árvore sobre atributos categóricos
- Entropia de `fall` fora da Antártida: 0,495 bits. Ganho de informação de cada atributo: **época 0,141**, faixa de massa 0,048, família 0,011, hemisfério ≈ 0.
- A árvore categórica tem AUC-ROC de 0,905 e AUC-PR de 0,599. A regra principal é legível: *antes de 1900 e não metálico → Fell*.
- Proporção de *Fell* por faixa de massa: 2,9% abaixo de 100 g e 23% acima de 10 kg.

---

## Síntese para o passo 6

1. **Dois perfis, duas lições.** Em pulsares, o modelo aprende física: a forma do pulso separa sinal de ruído, e o limite de desempenho está em pulsares fracos que os atributos não capturam. Em meteoritos, o modelo aprende **sociologia da observação**. Um AUC alto não significa que se entendeu o fenômeno.
2. **Modelos simples bastam quando a estrutura é simples.** Logística e SVM linear empatam com MLP e Random Forest em pulsares. Em meteoritos, árvores ganham com folga porque as regiões de busca são retângulos em lat/lon/ano.
3. **Acurácia engana nas duas bases.** AUC-PR e recall contam a história real. A escolha do limiar depende do custo de cada erro.
4. **Hipóteses refutadas foram as mais informativas.** A H1 temporal e a H4 global só fizeram sentido depois de identificar o confundidor (a Antártida e as campanhas a partir de 1969).
