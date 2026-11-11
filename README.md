# 🏦 Gestão de Confiança, Transparência da Comunicação e Contenção de Evasão (Churn) em Crises Bancárias

![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![LGPD](https://img.shields.io/badge/LGPD-Zero%20PII-green)
![Status](https://img.shields.io/badge/Etapa-2%20(Pipeline%20%26%20Logit%20preliminar)-orange)

Proposta de ecossistema analítico em **BI & Analytics** para avaliar o impacto da percepção da transparência da comunicação bancária no comportamento de resgate preventivo de recursos e encerramento de contas (*churn*) em momentos de crise de confiança.

---

## 📌 Pergunta de Negócio & Persona

* **Pergunta Principal:** *Como a percepção do cliente sobre a transparência da comunicação do banco em momentos de crise afeta a sua decisão de resgatar recursos ou encerrar a conta?*
* **Persona Alvo:** Head de Risco, Compliance e Comunicação de Crise.

---

## 🏗 Arquitetura do Pipeline de Dados

```text
[ Google Forms ] (Dados Primários) ──┐
                                     ├──► [ Python / Pandas ] ──► [ JOIN por Banco ] ──► [ Modelagem Logit ] ──► [ Looker Studio ]
[ BACEN T2/2026 ] (Dados Secundários) ┘     (Filtro Anti-GIGO)                            (Statsmodels)            (Etapa 3)
```

1. **Ingestão:** coleta primária via Google Forms + Tabela de Reclamações e Procedência do Banco Central (BACEN T2/2026, 198 instituições).
2. **Governança & Anti-GIGO:** validação de consentimento (TCLE / Zero PII). As regras de tempo mínimo de preenchimento e de variância (*straight-lining*) estão previstas, mas só são aplicadas quando o formulário fornece os dados necessários (ver [Limitações](#-limitações-conhecidas)).
3. **Cruzamento (`JOIN`):** relacionamento das bases pelo nome padronizado da instituição financeira (`instituicao_principal`), já que o CNPJ está vazio em 96% das linhas da tabela do BACEN.
4. **Modelagem Econométrica:** Regressão Logística Binária (Logit) para estimar parâmetros de impacto e Razão de Chances (*Odds Ratios*).

---

## 📊 Especificação do Modelo Econométrico (Logit)

$$ \ln\left(\frac{P(Y = 1)}{1 - P(Y = 1)}\right) = \beta_0 + \beta_1 X_1 + \beta_2 X_2 + \beta_3 Z_1 + \beta_4 Z_2 + \beta_5 Z_3 + \beta_6 Z_4 $$

| Variável | Nome | Tipo | Papel Econométrico |
| :--- | :--- | :--- | :--- |
| **$Y_1$ / $Y_2$** | `flag_resgate_preventivo` / `flag_risco_churn` | Lógico (0/1) | Variável Dependente (Impacto) |
| **$X_1$** | `percepcao_transparencia` | Ordinal (1-5) | Causa Principal (Escala Likert) |
| **$X_2$** | `canal_comunicacao_pref` | Nominal | Causa Secundária (App, E-mail, etc.) |
| **$Z_1$** | `faixa_renda_perfil` | Ordinal | Controle Micro (Socioeconômico) |
| **$Z_2$** | `tempo_relacionamento_anos` | Ordinal | Controle Micro (Fidelidade) |
| **$Z_3$** | `tx_reclamacao_geral_bacen` | Numérico | Controle Macro (reclamações procedentes por milhão de clientes) |
| **$Z_4$** | `taxa_procedencia_bacen` | Numérico | Controle Macro (% de reclamações analisadas que foram procedentes) |

---

## 🔬 Resultados Preliminares (Etapa 2)

Com a amostra atual, o modelo foi usado como **demonstração do pipeline**, não para tirar conclusões.

| Item | Valor |
| :--- | :--- |
| Respostas brutas / amostra válida | 21 / 21 |
| Respostas com dados do BACEN (usadas no modelo) | 19 |
| Casos de intenção de churn na amostra do modelo | 5 |
| Modelo estimado | `flag_risco_churn ~ percepcao_transparencia` |
| Odds Ratio da transparência | ≈ 0,64 |
| p-valor | ≈ 0,47 (não significativo) |

**Leitura:** a direção do efeito é a esperada (mais transparência percebida, menor chance de churn), mas o resultado **não é estatisticamente significativo**. Com poucos casos de churn, o modelo completo (com controles e dados do BACEN) só será estimado quando a amostra crescer.

---

## ⚠️ Limitações Conhecidas

- **Tamanho da amostra:** poucos casos de churn limitam o número de variáveis do modelo (regra prática: ~10 eventos por variável explicativa).
- **Tempo de preenchimento:** o export do Google Forms registra só a hora do envio, então a regra "tempo mínimo" do Anti-GIGO não pôde ser aplicada.
- **Straight-lining:** o formulário tem uma única pergunta em escala, então a verificação de variância não se aplica.
- **Perfil da amostra concentrado:** a maioria dos respondentes está na menor faixa de renda e em poucos bancos, o que reduz a variação dos controles.
- **Banco "Outros":** não possui equivalente na tabela do BACEN, e essas respostas ficam fora do modelo.

---

## 🛡 Governança de Dados & LGPD

- **Zero PII (Personally Identifiable Information):** não são coletados nomes, e-mails, números de telefone, CPFs ou endereços IP.
- **TCLE Obrigatório:** trava lógica inicial no formulário. Respostas com opção *"Não concordo"* são descartadas.
- **Anonimização por Agrupamento:** faixas ordinais amplas para renda e tempo de relacionamento. A base processada não inclui o carimbo de data/hora.

---

## 📁 Estrutura do Repositório

```text
projeto-retencao-crise-bancaria/
├── data/
│   ├── raw/                 # Dados brutos (Forms e BACEN), nunca editados à mão
│   └── processed/           # Base limpa (Anti-GIGO) e base final integrada (JOIN)
├── docs/                    # Documentação e figuras (ex.: eda_perfil_amostra.png)
├── notebooks/               # Notebooks Python (ETL + EDA + Regressão Logística)
├── src/                     # Reservada para scripts reutilizáveis (Etapa 3)
├── .gitignore
├── README.md
└── requirements.txt
```

### Notebooks (executar nesta ordem)

| Notebook | O que faz | Saída |
| :--- | :--- | :--- |
| `01_eda_e_tratamento_anti_gigo.ipynb` | Leitura do Forms, renomeação, filtro Anti-GIGO e EDA | `data/processed/dados_limpos_anti_gigo.csv`, `docs/eda_perfil_amostra.png` |
| `02_join_bacen_e_regressao_logistica.ipynb` | Tratamento do BACEN, JOIN, Logit e Odds Ratios | `data/processed/dataset_final_logit.csv` |

---

## ▶️ Como Reproduzir

```bash
# 1. Clonar o repositório
git clone https://github.com/YasRibeiroo/projeto-retencao-crise-bancaria.git
cd projeto-retencao-crise-bancaria

# 2. Criar e ativar o ambiente virtual
python -m venv .venv
source .venv/Scripts/activate      # Windows (Git Bash)
# source .venv/bin/activate        # Mac / Linux

# 3. Instalar as dependências
pip install -r requirements.txt
```

Depois, abra a pasta no VS Code, selecione o kernel `.venv` e rode os notebooks em `notebooks/`, na ordem indicada acima. Os notebooks usam caminhos relativos e precisam ser executados a partir da pasta `notebooks/`.

---

## 👥 Atribuições da Equipe

| Integrante | Papel no Projeto | Bloco de Apresentação |
| :--- | :--- | :--- |
| **[Nome]** | Perfil da Amostra & Persona | Bloco 1: Contexto, Amostra (EDA) e Pergunta de Negócio |
| **[Nome]** | Governança & Diário de Bordo | Bloco 2: LGPD, Filtro Anti-GIGO e Diário de Bordo |
| **[Yasmin] ** | Pipeline & Modelagem Logit | Bloco 3: Pipeline Python, Regressão Logística & Recomendações |