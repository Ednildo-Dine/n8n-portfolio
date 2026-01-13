## ⚡ Bot de Relatórios de Tráfego Pago (Facebook Ads & Telegram)

#### 🧠 Lógica do Processo: Este workflow automatiza a geração do DRE e métricas de tráfego, funcionando de forma híbrida: **Automática** (todos os dias às 05:00) e **Sob Demanda** (via comando no Telegram). Ao ser acionado, o sistema define o período de análise (geralmente D-1/"Ontem"), extrai os custos do Facebook Ads e cruza com a receita registrada no Google Sheets. O diferencial é a interface via Telegram: o gestor pode solicitar uma atualização fora do horário padrão enviando um comando, e o fluxo processa, consolida os dados (`Merge`) e atualiza as planilhas de gestão, garantindo números sempre frescos para tomada de decisão.

---

### 🏗️ Diagrama de Arquitetura:

```mermaid
graph TD
    %% Estilos
    classDef sheets fill:#1FA463,stroke:#fff,stroke-width:2px,color:#fff
    classDef telegram fill:#2CA5E0,stroke:#fff,stroke-width:2px,color:#fff
    classDef time fill:#F4B400,stroke:#fff,stroke-width:2px,color:#fff
    classDef n8n fill:#FF6D5A,stroke:#fff,stroke-width:2px,color:#fff
    classDef endnode fill:#95a5a6,stroke:#333,color:#fff

    subgraph "Gatilhos de Entrada"
        Cron((Cron: 05:00)):::time
        TgHook((Telegram:<br/>Comando Manual)):::telegram
    end

    Cron & TgHook --> SetVars["Set Variáveis<br/>(Definir Data D-1)"]:::n8n
    SetVars --> DateCalc["Calcular:<br/>Janela de Tempo"]:::n8n
    
    DateCalc --> GSheets_Read["Google Sheets:<br/>Ler Metas/Vendas"]:::sheets
    
    GSheets_Read --> Merge{{"Merge:<br/>Consolidar ROI"}}:::n8n
    Merge --> Split["Split:<br/>Segmentar Dados"]:::n8n
    
    subgraph "Atualização de Dashboard"
        Split --> Sheet1["GSheets:<br/>Atualizar Geral"]:::sheets
        Split --> Sheet2["GSheets:<br/>Atualizar Criativos"]:::sheets
        Split --> Sheet3["GSheets:<br/>Atualizar Vendas"]:::sheets
    end

    Sheet1 & Sheet2 & Sheet3 --> EndNode([Fim]):::endnode

```

---

### 🔍 Dicionário de Dados

#### 1. Entradas (Híbridas)

* **Telegram Webhook** `(Nó: Webhook)`
* **Função:** Gatilho Manual (ChatOps). Permite que o gestor de tráfego force a geração do relatório a qualquer momento enviando um comando no chat. O fluxo recebe a requisição, ignora o agendamento e roda a lógica de consolidação imediatamente.


* **Cron Trigger** `(Nó: Todo dia 05:00)`
* **Função:** Rotina Agendada. Garante que, antes do início do expediente, os dados do dia anterior já estejam processados na planilha, sem dependência humana.



#### 2. Lógica de Negócio

* **Definição Temporal** `(Nós: Set Variáveis / Dados de Ontem)`
* **Função:** Padronização. Independente da origem do disparo (Telegram ou Cron), o sistema calcula a data de "Ontem" para buscar o fechamento correto das métricas de ads e vendas.


* **Consolidação (ETL)** `(Nós: Merge / Separa os dados)`
* **Função:** Cruzamento de Dados. Une as informações de **Gasto** (Facebook Ads) com **Faturamento** para calcular métricas compostas como ROAS (Retorno sobre Investimento em Anúncios) e CPA (Custo por Aquisição).



#### 3. Saída e Visualização

* **Google Sheets** `(Nós de Escrita)`
* **Função:** Banco de Dados Frontend. O n8n atua apenas como processador; os dados finais são persistidos em abas específicas da planilha mestre, que alimenta os dashboards visuais da equipe de marketing.


