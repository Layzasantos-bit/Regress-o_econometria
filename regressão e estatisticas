# =========================================================================
# ANÁLISE DE IMPACTO DA VIOLÊNCIA NA RENDA DE MULHERES (PNS)
# =========================================================================

# -------------------------------------------------------------------------
# 0. Configurações Iniciais e Pacotes
# -------------------------------------------------------------------------
library(dplyr)
library(survey)

# -------------------------------------------------------------------------
# 1. Preparação da Base Geral de Mulheres Selecionadas
# -------------------------------------------------------------------------
pns_novas_variaveis <- pns_mulheres %>%
  # Mantém apenas as moradoras selecionadas (com peso individual)
  filter(!is.na(V00291)) %>%
  
  # Criando as Dummies de Violência (Psicológica, Física e Sexual)
  mutate(
    viol_psicologica = if_else(V00201 == 1 | V00202 == 1 | V00203 == 1 | V00204 == 1 | V00205 == 1, 1, 0, missing = 0),
    viol_fisica      = if_else(V01401 == 1 | V01402 == 1 | V01403 == 1 | V01404 == 1 | V01405 == 1, 1, 0, missing = 0),
    viol_sexual      = if_else(V02701 == 1 | V02702 == 1, 1, 0, missing = 0),
    
    # Variável de interesse global
    sofreu_violencia = if_else(viol_psicologica == 1 | viol_fisica == 1 | viol_sexual == 1, 1, 0)
  ) %>%
  
  # Criando os controles básicos
  mutate(
    idade          = C008,
    idade_quadrado = C008^2,
    raca_cor       = as.factor(C009),
    escolaridade   = as.factor(VDD004A),
    regiao         = as.factor(substr(V0001, 1, 1))
  ) %>%
  
  # Criando variáveis de composição familiar
  group_by(UPA_PNS, V0006_PNS) %>%
  mutate(
    filhos_menores_6 = if_else(any(C008 < 6, na.rm = TRUE), 1, 0),
    mora_com_marido  = if_else(any(C001 == 2, na.rm = TRUE), 1, 0)
  ) %>%
  ungroup()

# -------------------------------------------------------------------------
# 2. Separando e Limpando as Bases dos Modelos
# -------------------------------------------------------------------------

# MODELO 1: Rendimento Domiciliar Per Capita (Sem horas trabalhadas)
pns_modelo_dom <- pns_novas_variaveis %>%
  filter(VDF003 > 0 & !is.na(VDF003)) %>%
  mutate(log_rendimento = log(VDF003))

# MODELO 2: Rendimento Individual do Trabalho (Inclui Horas Trabalhadas E017)
pns_modelo_ind <- pns_novas_variaveis %>%
  filter(E01602 > 0 & !is.na(E01602)) %>%
  filter(E017 > 0 & !is.na(E017)) %>% 
  mutate(
    log_rendimento    = log(E01602),
    horas_trabalhadas = E017
  )

# -------------------------------------------------------------------------
# 3. Desenho Amostral Complexo e Regressões
# -------------------------------------------------------------------------

# --- Execução para o Modelo Domiciliar ---
design_dom <- svydesign(
  id      = ~UPA_PNS, 
  strata  = ~V0024, 
  weights = ~V00291, 
  nest    = TRUE, 
  data    = pns_modelo_dom
)

reg_domiciliar <- svyglm(
  log_rendimento ~ sofreu_violencia + idade + idade_quadrado + raca_cor + 
                   escolaridade + regiao + mora_com_marido, 
  design = design_dom
)

# --- Execução para o Modelo Individual ---
design_ind <- svydesign(
  id      = ~UPA_PNS, 
  strata  = ~V0024, 
  weights = ~V00291, 
  nest    = TRUE, 
  data    = pns_modelo_ind
)

reg_individual <- svyglm(
  log_rendimento ~ sofreu_violencia + idade + idade_quadrado + raca_cor + 
                   escolaridade + regiao + mora_com_marido + horas_trabalhadas, 
  design = design_ind
)

# -------------------------------------------------------------------------
# 4. Resultados das Regressões
# -------------------------------------------------------------------------
cat("\n======= MODELO DOMICILIAR PER CAPITA =======\n")
print(summary(reg_domiciliar))

cat("\n======= MODELO INDIVIDUAL DO TRABALHO =======\n")
print(summary(reg_individual))

# -------------------------------------------------------------------------
# 5. Estatísticas Descritivas
# -------------------------------------------------------------------------

# --- Modelo Individual ---
cat("\n=== MEDIDAS RESUMO: MODELO INDIVIDUAL ===\n")
est_ind <- pns_modelo_ind %>%
  select(E01602, idade, horas_trabalhadas, idade_quadrado, raca_cor, escolaridade, regiao, mora_com_marido, sofreu_violencia)
print(summary(est_ind))

cat("\n[Desvio-padrão corrigido pelo plano amostral - Individual]\n")
print(sqrt(svyvar(~E01602, design = design_ind, na.rm = TRUE)))
print(sqrt(svyvar(~idade_quadrado, design = design_ind, na.rm = TRUE)))
print(sqrt(svyvar(~horas_trabalhadas, design = design_ind, na.rm = TRUE)))
print(svymean(~sofreu_violencia + mora_com_marido + raca_cor + escolaridade + regiao, design = design_ind, na.rm = TRUE))

# --- Modelo Domiciliar ---
cat("\n=== MEDIDAS RESUMO: MODELO DOMICILIAR ===\n")
est_domiciliar <- pns_modelo_dom %>%
  select(VDF003, idade, idade_quadrado, raca_cor, escolaridade, regiao, mora_com_marido, sofreu_violencia)
print(summary(est_domiciliar))

cat("\n=== Desvio Padrão: Variáveis Contínuas e Dummies (Domiciliar) ===\n")
print(sqrt(svyvar(~idade, design = design_dom, na.rm = TRUE)))
print(sqrt(svyvar(~idade_quadrado, design = design_dom, na.rm = TRUE)))
print(sqrt(svyvar(~sofreu_violencia, design = design_dom, na.rm = TRUE)))
print(sqrt(svyvar(~mora_com_marido, design = design_dom, na.rm = TRUE)))
print(sqrt(svyvar(~VDF003, design = design_dom, na.rm = TRUE)))

# -------------------------------------------------------------------------
# 6. Proporções das Variáveis Categóricas (Modelo Domiciliar)
# -------------------------------------------------------------------------
cat("\n=== PROPORÇÃO: MORA COM MARIDO (MODELO DOMICILIAR) ===\n")
print(prop.table(table(pns_modelo_dom$mora_com_marido)))

cat("\n=== PROPORÇÃO: RAÇA/COR (MODELO DOMICILIAR) ===\n")
print(prop.table(table(pns_modelo_dom$raca_cor)))

cat("\n=== PROPORÇÃO: ESCOLARIDADE (MODELO DOMICILIAR) ===\n")
print(prop.table(table(pns_modelo_dom$escolaridade)))
