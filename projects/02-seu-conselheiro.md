## ⚡ ECOSSISTEMA INTEGRAL - SEU CONSELHEIRO

#### 🧠 Lógica do Processo
O sistema opera como um "terapeuta de bolso" contínuo. A entrada principal tria mensagens: novos usuários vão para o Onboarding (qualificação via IA), enquanto usuários recorrentes vão para o Gerenciador Terapeuta. O Gerenciador verifica a assinatura e decide entre conversa livre ou desafio. Se a assinatura expirou, ativa-se a Recuperação. Paralelamente, gatilhos de inatividade (18h/23h) cobram a realização dos desafios para garantir a retenção.

---

### 🏗️ Diagrama de Arquitetura:

```mermaid
graph TD
    %% Definição de Classes
    classDef whatsapp fill:#25D366,stroke:#fff,stroke-width:2px,color:#fff
    classDef n8n fill:#FF6D5A,stroke:#fff,stroke-width:2px,color:#fff
    classDef db fill:#3C9CD7,stroke:#fff,stroke-width:2px,color:#fff
    classDef subflow fill:#FF9F89,stroke:#333,stroke-width:1px,stroke-dasharray: 5 5,color:#000
    classDef endnode fill:#95a5a6,stroke:#333,color:#fff

    %% Nós Principais
    UserMsg((WhatsApp)):::whatsapp --> |Webhook| Router{"Roteador"}:::n8n
    Time18h((Cron 18h/23h)):::n8n --> |Agendamento| CheckDaily{"Fez Desafio?"}:::n8n

    %% Roteamento
    Router -- Lead --> Onboarding["Subfluxo Onboarding"]:::subflow
    Router -- Cliente --> Manager{"Gerenciador"}:::subflow
    
    %% Detalhes Onboarding
    Onboarding --> |Valida| AI_Valid["IA Validação"]:::db
    AI_Valid --> SaveProfile["Salvar DB"]:::db
    
    %% Detalhes Gerenciador
    Manager -- Vencido --> Expired["Recuperação"]:::n8n
    Manager -- Ativo --> Ctx["Carregar Contexto"]:::db
    
    %% Lógica de Vencimento
    Expired --> |Oferta| Offer["Enviar Oferta"]:::whatsapp
    
    %% Núcleo de IA
    Ctx --> Mode{"Modo?"}:::n8n
    Mode -- Desafio --> AI_Gen["IA Gerador"]:::db
    Mode -- Conversa --> AI_Chat["IA Terapeuta"]:::db
    
    %% Notificações
    CheckDaily -- Pendente --> Window["Janela 24h"]:::n8n
    Window --> Notify["Cobrar Desafio"]:::whatsapp
    
    %% Saídas
    AI_Gen --> Reply((Resposta)):::whatsapp
    AI_Chat --> Reply
    Onboarding --> Reply
    
    %% Persistência (Visualmente conectada mas discreta)
    SaveProfile -.-> DB[(Banco Dados)]:::db
    Ctx -.-> DB
