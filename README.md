# Regress-o_econometria
Nesse repositório está os scripts em R para a regressão e estatísticas descritivas do trabalho de econometria.
# Análise do Impacto da Violência Doméstica sobre o Rendimento das Mulheres (PNS 2019)

Este repositório contém o script em linguagem **R** utilizado para o processamento de dados, análise descritiva e estimação dos modelos econométricos baseados na **Pesquisa Nacional de Saúde (PNS 2019)** do IBGE. O objetivo principal é mensurar o impacto da violência doméstica (psicológica, física e/ou sexual) na renda e na inserção laboral das mulheres no Brasil.

## 🛠️ Estrutura e Funcionalidades do Script

O script está organizado de forma sequencial em quatro etapas fundamentais:

### 1. Tratamento de Dados e Criação de Variáveis (*Data Wrangling*)
* **Identificação da Violência:** Construção de variáveis dummy agregadas (`sofreu_violencia`) a partir dos blocos de perguntas específicas da PNS sobre agressões psicológicas, físicas e sexuais.
* **Variáveis de Controle:** Definição de características demográficas e socioeconômicas como idade, idade ao quadrado (para capturar retornos marginais decrescentes da experiência), raça/cor, nível de escolaridade e região geográfica.
* **Variáveis de Estrutura Familiar:** Agrupamento por domicílio (`UPA_PNS` + `V0006_PNS`) para mapear o status de coabitação (`mora_com_marido`) e a presença de filhos menores de 6 anos.

### 2. Segmentação dos Modelos Econométricos
A base foi subdividida para analisar o problema sob duas óticas distintas:
* **Modelo Domiciliar:** Focado no impacto sobre o **Rendimento Domiciliar Per Capita** (`VDF003`), permitindo avaliar o bem-estar financeiro do núcleo familiar.
* **Modelo Individual:** Focado no **Rendimento Individual do Trabalho** (`E01602`) ponderado pelas **Horas Semanais Trabalhadas** (`E017`), aplicando restrição amostral para a população feminina ocupada.

### 3. Especificação do Plano Amostral Complexo (*Survey Design*)
Como a PNS possui amostragem por conglomerados e estratificação, o script utiliza rigorosamente o pacote `survey` para evitar viés nas estimativas e nos erros-padrão:
* Incorporação de pesos amostrais (`V00291`), unidades primárias de amostragem (`UPA_PNS`) e estratos (`V0024`).
* Aplicação do ajuste estatístico `options(survey.lonely.psu = "adjust")` para lidar de forma robusta com estratos que possuem apenas uma única UPA (*Lonely PSUs*).
* Estimação por Modelos Lineares Generalizados Populacionais Média Ponderados (`svyglm`) aplicando a transformação logarítmica na variável dependente de renda.

### 4. Extração de Estatísticas Descritivas Populacionais
* Uso de funções estruturais como `svymean` e `svyvar` para calcular médias, proporções e desvios-padrão populacionais corrigidos pelo efeito de delineamento amostral (EPA).

## 🚀 Tecnologias e Pacotes Utilizados

* **R (v4.0+)**
* `tidyverse` (para manipulação ágil de dados)
* `survey` (para análise de dados amostrais complexos)

---
*Nota: Este repositório cumpre fins estritamente acadêmicos, assegurando a transparência e a reprodutibilidade metodológica da pesquisa.*
