# Análise de Clientes Netflix usando LLM

## Data Analyst LLM

📥 **Baixe a apresentação estilo Power BI em HTML:**
[Clique aqui para acessar o arquivo](https://drive.google.com/file/d/1xn2AwccX7XlEmF_NdsQFVtgpMq4twwzg/view?usp=sharing)

📥 **Baixe a apresentação executiva em HTML:**
[Clique aqui para acessar o arquivo](https://drive.google.com/file/d/11T7c7-9B7dSObjXDkliVOaxYGq5huV6B/view?usp=sharing)

## Problema de Negócio

### Contexto da empresa

**Quem consome:** CEO — visão executiva, não operacional. Isso muda o nível de profundidade: menos "por que a métrica X caiu na tabela Y", mais "o que isso significa para a estratégia da empresa".

**Dor principal:** Entender o perfil e comportamento da base de assinantes — quem são, o que assistem, o que os mantém engajados, e o que impacta retenção — cruzando quatro dimensões: gênero de conteúdo, tempo de tela, forma de pagamento e geografia.

**Decisão a embasar:** Você mencionou "tomar uma decisão com base nos dados" de forma genérica — vale destacar isso, porque um CEO normalmente tem 1-2 decisões concretas em mente (ex: onde investir em conteúdo, qual plano de pagamento promover, qual país priorizar em marketing, se vale simplificar planos). Isso não trava a etapa 1, mas vai moldar bastante quais hipóteses priorizamos na etapa 3.

## Premissas da análise

---

## Estratégia da solução

O método **Fato-Dimensão** foi usado para desenvolver a análise de dados.

### Passo 1: Resumir o contexto em uma pergunta aberta

As perguntas abertas são um tipo de demanda muito comum em análise de dados nas quais a demanda possui **N possíveis soluções** e cabe ao analista de dados avaliar as possibilidades e escolher a alternativa com o maior retorno e o menor esforço possível.

Para essa análise, foi definida a seguinte pergunta aberta:

> **Quem são os clientes da Netflix? Como eles usam nossos produtos? Como está o churn?**

### Passo 2: Transformar pergunta aberta em fechada

As perguntas fechadas são um tipo de demanda muito comum na área de análise de dados. Essa demanda contém todos os detalhes da análise de dados e direciona o analista exatamente para o que precisa ser feito. Geralmente, a pergunta fechada é a escolha de uma solução entre todas as alternativas possíveis, feita por um profissional mais sênior da área.

Para essa análise, foi definida a seguinte pergunta fechada:

> **Pergunta Fechada:** Faça uma análise exploratória de cada aspecto que abrange o consumo do cliente, seja ele quanto tempo ele passa na Netflix por mês, até a playlist favorita, caso tenha.

### Passo 3: Definição da Coluna Fato

O **Fato** é a coluna de interesse que representa o ponto focal da análise.

`churned` — indica se o cliente cancelou a assinatura (Yes/No). É a variável-alvo (target) da análise, pois responde diretamente à pergunta do CEO sobre retenção e abandono.

### Passo 4: Identificação das Dimensões

#### Entendimento da Base de Dados

**Overview estrutural:**

50.000 usuários, 20 colunas, sem nulos, sem duplicados. Cada linha representa um assinante único da Netflix em um "corte" (não há dimensão temporal — é uma fotografia, não uma série histórica).

#### O que cada coluna representa (e seu significado de negócio)

**🧑 Perfil demográfico**

| Coluna | O que é | Por que importa pro negócio |
| :--- | :--- | :--- |
| `user_id` | Identificador único | Chave primária, sem valor analítico direto |
| `age` | Idade (18-64) | Permite segmentar por geração — conteúdo e comportamento variam muito por faixa etária |
| `gender` | Gênero (Male/Female/Other) | Preferências de gênero de conteúdo podem variar por público |
| `country` | País (10 países) | Base para decisões de expansão, localização de catálogo, pricing regional |

**💳 Assinatura e pagamento**

| Coluna | O que é | Por que importa |
| :--- | :--- | :--- |
| `subscription_type` | Basic/Standard/Premium | Indica disposição a pagar e possivelmente nº de telas/qualidade |
| `monthly_fee` | Valor pago mensalmente | Receita direta por usuário — insumo para LTV |
| `payment_method` | PayPal/Cartão/UPI/Débito | Pode revelar padrões de atrito no pagamento (ex: recusas, renovação automática) |
| `account_age_months` | Tempo de casa (1-59 meses) | Proxy de maturidade/lealdade do cliente |

**📺 Comportamento de consumo**

| Coluna | O que é | Por que importa |
| :--- | :--- | :--- |
| `primary_device` | Mobile/Smart TV/Laptop/Tablet | Mostra onde investir em UX e onde está o hábito de consumo |
| `devices_used` | Nº de dispositivos (1-3) | Multi-tela pode indicar maior engajamento familiar/domiciliar |
| `favorite_genre` | Gênero preferido (8 opções) | Direciona investimento em produção de conteúdo |
| `avg_watch_time_minutes` | Tempo médio assistido | Métrica-chave de engajamento |
| `watch_sessions_per_week` | Frequência semanal | Regularidade de uso — proxy de hábito |
| `binge_watch_sessions` | Nº de sessões de maratona | Sinal de "vício" no produto — forte indicador de retenção |
| `completion_rate` | % de conclusão do conteúdo (30-99%) | Qualidade do match conteúdo-usuário (recomendação funcionando?) |

**⭐ Engajamento e satisfação**

| Coluna | O que é | Por que importa |
| :--- | :--- | :--- |
| `rating_given` | Nota média dada (1-5) | Proxy de satisfação declarada |
| `content_interactions` | Nº de interações (likes, listas, etc.) | Engajamento ativo além de assistir passivamente |
| `recommendation_click_rate` | % de cliques em recomendações | Eficácia do algoritmo de recomendação |
| `days_since_last_login` | Dias desde o último acesso | Sinal de risco de abandono — quanto maior, mais frio o usuário |

**🎯 Variável-alvo**

| Coluna | O que é | Por que importa |
| :--- | :--- | :--- |
| `churned` | Cancelou? (Yes/No) — 20% Yes | O resultado que queremos entender e prever |

---

### Passo 5: Hipóteses Analíticas

**H1: Baixo engajamento recente prediz churn**

Lógica: usuários que pararam de logar recentemente provavelmente já "desistiram" mentalmente do produto antes de cancelar formalmente. `days_since_last_login` deveria ser o sinal mais forte e mais imediato de risco.
Como testar: comparar a distribuição de `days_since_last_login` entre `churned = Yes` vs `No` (boxplot, médias). Se houver diferença clara, testar como variável de corte (ex: >X dias = alto risco).

**H2: Baixo binge-watching e baixa completion rate estão associados a maior churn**

Lógica: usuários que "viciam" na plataforma (maratonam, terminam o que começam) enxergam mais valor e têm menos motivo para cancelar. Baixo `completion_rate` pode indicar que o catálogo/recomendação não está entregando o que a pessoa quer.
Como testar: comparar médias de `binge_watch_sessions` e `completion_rate` entre `churned Yes/No`; correlação entre essas variáveis e churn (point-biserial ou regressão logística).

**H3: Planos mais baratos (Basic) têm maior taxa de churn que Premium**

Lógica: quem paga mais pode estar mais comprometido/satisfeito, ou o plano Premium oferece mais valor percebido (mais telas, qualidade). Alternativamente, pode ser o oposto: Basic atrai usuários mais sensíveis a preço e menos fiéis.
Como testar: tabela cruzada `subscription_type` x `churned`, calculando taxa de churn (%) por categoria.

**H4: Certos métodos de pagamento têm maior churn (fricção de cobrança)**

Lógica: métodos de pagamento menos "automáticos" ou com mais falhas de cobrança (ex: PayPal, débito) podem gerar cancelamentos involuntários, diferente de cartão de crédito com renovação automática mais confiável.
Como testar: tabela cruzada `payment_method` x `churned`, taxa de churn por método.

**H5: Contas mais antigas têm menor churn (efeito de lealdade/inércia)**

Lógica: quanto mais tempo o usuário fica, mais hábito e menos propensão a trocar de serviço — mas também pode ser o oposto se houver fadiga de assinatura ao longo do tempo.
Como testar: correlação entre `account_age_months` e `churned`; comparar médias entre grupos.

**📺 Eixo 2 — O que assistem (conteúdo e gênero)**

**H6: Determinados gêneros favoritos têm maior tempo médio de tela**

Lógica: conteúdo seriado (ex: Drama, Thriller) tende a prender mais que conteúdo mais "leve" (Comedy), impactando `avg_watch_time_minutes`.
Como testar: agrupar por `favorite_genre` e comparar médias de `avg_watch_time_minutes`, `binge_watch_sessions` e `completion_rate` (ANOVA ou comparação de médias).

**H7: Gênero preferido varia por perfil demográfico (idade/gênero)**

Lógica: diferentes faixas etárias e gêneros de usuário tendem a ter preferências de conteúdo distintas — relevante para segmentação de catálogo.
Como testar: tabela cruzada `favorite_genre` x `age` (faixas) e `favorite_genre` x `gender`, olhando distribuição percentual.

**H8: Gênero preferido influencia o churn**

Lógica: se um gênero entrega menos satisfação (menor completion, menor rating), pode estar associado a maior cancelamento — sinal de que o catálogo daquele gênero não é forte o suficiente.
Como testar: taxa de churn por `favorite_genre`; cruzar com `rating_given` médio por gênero.

**⏱️ Eixo 3 — O que gera mais tempo de tela / engajamento**

**H9: Recomendações eficazes aumentam completion rate e tempo de tela**

Lógica: se o algoritmo de recomendação "acerta", o usuário assiste mais e termina mais conteúdo — validando o valor do motor de recomendação.
Como testar: correlação entre `recommendation_click_rate` e `completion_rate` / `avg_watch_time_minutes`.

**H10: Multi-dispositivo está associado a maior engajamento**

Lógica: usuários que assistem em vários dispositivos (`devices_used`) provavelmente têm uso mais integrado ao dia a dia (trabalho, casa, mobilidade) — maior `watch_sessions_per_week` e menor churn.
Como testar: correlação entre `devices_used` e `watch_sessions_per_week`/`churned`.

**H11: Dispositivo principal influencia padrão de consumo**

Lógica: Smart TV pode favorecer sessões mais longas e binge-watching; Mobile pode favorecer sessões curtas e mais frequentes.
Como testar: agrupar por `primary_device` e comparar `avg_watch_time_minutes`, `watch_sessions_per_week`, `binge_watch_sessions`.

**🌍 Eixo 4 — Geografia e pagamento**

**H12: Padrões de consumo e churn variam significativamente por país**

Lógica: cultura de consumo de streaming, poder de compra e concorrência local (outros serviços) variam por país, afetando engajamento e retenção.
Como testar: agrupar por `country` e comparar `churned (%)`, `avg_watch_time_minutes`, `monthly_fee` médio, `subscription_type` predominante.

**H13: Países com predominância de planos mais baratos têm maior churn**

Lógica: cruzamento entre poder aquisitivo (proxy: tipo de plano dominante) e retenção — países com mix de planos mais baixos podem indicar sensibilidade a preço, o que aumenta risco de troca por concorrentes.
Como testar: cruzar `country` x `subscription_type` x `churned`.

---

### Passo 6: Critérios de Priorização

- **Critério 1:** Dados disponíveis.
- **Critério 2:** Insights acionáveis.

### Passo 7: Priorização das Hipóteses Analíticas

### Resumo dos veredictos

**🔴 Prioridade Alta**

**H3 — Tipo de plano x churn**

| Plano | Churn rate |
| :--- | :--- |
| Basic | 19,24% |
| Standard | 20,09% |
| Premium | 20,39% |

➜ Não validada. Diferença de ~1 p.p., estatisticamente irrelevante nesse volume.

**H4 — Método de pagamento x churn**

| Método | Churn rate |
| :--- | :--- |
| Credit Card | 19,66% |
| UPI | 19,84% |
| PayPal | 20,03% |
| Debit Card | 20,18% |

➜ Não validada. Praticamente idêntico entre métodos (~0,5 p.p. de amplitude).

**H12/H13 — País x churn e x ticket médio**

Churn varia de 19,25% (USA) a 20,96% (Austrália) — amplitude de 1,7 p.p., sem padrão de correlação com `monthly_fee` médio (que é quase idêntico entre países: R$12,26–R$12,37).

➜ Não validada. Não há país "melhor" ou "pior" de forma consistente.

**🟡 Prioridade Média**

**H1 — Dias sem login prediz churn**

Média de `days_since_last_login`: 29,41 (não-churn) vs 29,43 (churn).

➜ Rejeitada com clareza. Não há diferença nenhuma — o que contraria a lógica de negócio mais básica de churn (normalmente esse é o sinal mais forte em bases reais).

**H2 — Binge/completion rate x churn**

Binge: 7,01 vs 6,97 | Completion: 64,55% vs 64,46% | Watch time: 155,2 vs 154,1 min.

➜ Rejeitada. Diferenças desprezíveis.

**H6 — Gênero x tempo de tela/completion**

Varia de 153,75 min (Horror) a 157,38 min (Comedy) — amplitude de ~2%.

➜ Rejeitada. Sem diferença prática entre gêneros.

**H8 — Gênero x churn**

Varia de 19,26% (Action) a 20,51% (Thriller) — amplitude de 1,25 p.p.

➜ Rejeitada.

**🟢 Prioridade Baixa**

H5, H7, H9, H10, H11 — todos testados via correlação ou crosstab: todas as correlações ficaram entre -0,01 e 0,01, e as distribuições percentuais em crosstabs (ex: gênero de conteúdo por gênero de usuário) ficaram muito próximas de uniformes (~12-13% em cada categoria). Nenhuma validada.

**🚨 Achado central da etapa 4**

Isso não é "13 hipóteses fracas" — é um padrão sistemático: a correlação de `churned` com todas as 12 variáveis numéricas do dataset está entre -0,01 e 0,00. Isso é estatisticamente equivalente a ruído aleatório, não a um comportamento real de usuários.

Isso é uma informação crítica para o CEO: os dados, como estão, não sustentam nenhuma decisão de negócio sobre o que causa churn ou engajamento — tudo indica que essa base foi gerada sinteticamente sem relação causal embutida entre as variáveis (não é incomum em datasets de prática/portfólio do Kaggle).

Isso muda o direcionamento da etapa 5 (insights): em vez de "aqui estão os fatores de churn", o insight real é "os dados atuais não permitem explicar churn — é preciso investigar a qualidade/origem da base antes de qualquer decisão".

---

## Insights da análise

## Data Analyst LLM

📥 **Baixe a apresentação estilo Power BI em HTML:**
[Clique aqui para acessar o arquivo](https://drive.google.com/file/d/1xn2AwccX7XlEmF_NdsQFVtgpMq4twwzg/view?usp=sharing)

📥 **Baixe a apresentação executiva em HTML:**
[Clique aqui para acessar o arquivo](https://drive.google.com/file/d/11T7c7-9B7dSObjXDkliVOaxYGq5huV6B/view?usp=sharing)

---

## Próximos passos

Receber feedback do CEO para saber se o seu desejo foi atendido com base na análise aqui presente, ou se será feito um aprofundamento na análise.
