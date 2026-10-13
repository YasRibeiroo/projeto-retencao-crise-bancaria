# 🏦 Gestão de Confiança, Transparência da Comunicação e Contenção de Evasão (Churn) em Crises Bancárias

Proposta de ecossistema analítico em **BI & Analytics** para avaliar o impacto da percepção da transparência da comunicação bancária no comportamento de resgate preventivo de recursos e encerramento de contas (*churn*) em momentos de crise de confiança.

---

## 📌 Pergunta de Negócio & Persona
* **Pergunta Principal:** *Como a percepção do cliente sobre a transparência da comunicação do banco em momentos de crise afeta a sua decisão de resgatar recursos ou encerrar a conta?*
* **Persona Alvo:** Head de Risco, Compliance e Comunicação de Crise.

---

## 🏗️ Arquitetura do Pipeline de Dados

```text
[ Google Forms ] (Dados Primários) ──┐
                                     ├──► [ Python / Pandas ] ──► [ JOIN por Banco ] ──► [ Modelagem Logit ] ──► [ Looker Studio ]
[ BACEN T2/2026 ] (Dados Secundários) ┘    (Filtro Anti-GIGO)                               (Statsmodels)          (Etapa 3)
```

1. **Ingestão:** Coleta primária via Google Forms ($N \approx 80-100$) + Tabela de Reclamações e Procedência do Banco Central (BACEN T2/2026).
2. **Governança & Anti-GIGO:** Validação de consentimento (TCLE / Zero PII), expurgo de preenchimentos ultrarrápidos ($<25\text{s}$) e verificação de variância (*straight-lining*).
3. **Cruzamento (`JOIN`):** Relacionamento de bases utilizando como chave única o nome padronizado da instituição financeira (`instituicao_principal`).
4. **Modelagem Econométrica:** Regressão Logística Binária (Logit) para estimação dos parâmetros de impacto e Razão de Chances (*Odds Ratios*).

---

## 📊 Especificação do Modelo Econométrico (Logit)

$$ \ln\left(\frac{P(Y = 1)}{1 - P(Y = 1)}\right) = \beta_0 + \beta_1 X_1 + \beta_2 X_2 + \beta_3 Z_1 + \beta_4 Z_2 + \beta_5 Z_3 + \beta_6 Z_4 $$

| Variável | Nome | Tipo | Papel Econométrico |
| :--- | :--- | :--- | :--- |
| **$Y_1$ / $Y_2$** | `flag_resgate_preventivo` / `flag_risco_churn` | Lógico (0/1) | Variável Dependente (Impacto) |
| **$X_1$** | `percepcao_transparencia` | Ordinal (1-5) | Causa Principal (Escala Likert) |
| **$X_2$** | `canal_comunicacao_pref` | Nominal | Causa Secundária (App, E-mail, etc.) |
| **$Z_1$** | `faixa_renda_perfil` | Ordinal (1-4) | Controle Micro (Socioeconômico) |
| **$Z_2$** | `tempo_relacionamento_anos` | Numérico | Controle Micro (Fidelidade) |
| **$Z_3$** | `tx_reclamacao_geral_bacen` | Numérico | Controle Macro (Reputação BACEN) |
| **$Z_4$** | `taxa_procedencia_bacen` | Numérico | Controle Macro (Procedência BACEN) |

---

## 🛡️ Governança de Dados & LGPD
- **Zero PII (Personally Identifiable Information):** Não são coletados nomes, e-mails, números de telefone, CPFs ou endereços IP.
- **TCLE Obrigatório:** Trava lógica inicial no formulário. Respostas com opção *"Não concordo"* são automaticamente descartadas.
- **Anonimização por Agrupamento:** Faixas ordinais amplas para renda e tempo de relacionamento.

---

## 📁 Estrutura do Repositório

```text
├── data/
│   ├── raw/          # Dados brutos (.csv do Forms e BACEN)
│   └── processed/    # Dados limpos pós Anti-GIGO e JOIN
├── docs/             # Esqueleto da apresentação e mapas
├── notebooks/        # Notebooks Python (ETL + Regressão Logística)
├── .gitignore
└── README.md
```

---

## 👥 Atribuições da Equipe

| Integrante | Papel no Projeto | Bloco de Apresentação |
| :--- | :--- | :--- |
| **Integrante B** | Perfil da Amostra & Persona | Bloco 1: Contexto, Amostra (EDA) e Pergunta de Negócio |
| **Integrante A** | Governança & Diário de Bordo | Bloco 2: LGPD, Filtro Anti-GIGO e Diário de Bordo |
| **Integrante C** | Pipeline & Modelagem Logit | Bloco 3: Pipeline Python, Regressão Logística & Recomendações |


