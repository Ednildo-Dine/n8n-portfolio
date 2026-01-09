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
```

---

### 🔍 Dicionário de Dados

#### 1. Núcleo e Roteamento
* **Roteador** `(Nó: Router Principal)`
  * **Função:** Atua como o *gateway* de entrada. Identifica o tipo de mídia (áudio/texto) e segmenta o remetente entre **Leads** (novos usuários para Onboarding) e **Clientes** (usuários recorrentes para o Gerenciador).

* **Subfluxo Onboarding** `(Sub-workflow)`
  * **Função:** Executa a qualificação inicial. Coleta dados (Nome, Dores, Objetivos), valida a coerência das respostas e registra o perfil inicial no banco de dados.

* **Gerenciador** `(Nó: Gerenciador Terapeuta)`
  * **Função:** O cérebro lógico do sistema. Verifica o status da assinatura (Free, Starter, Pro, VIP), controla o acesso a recursos (ex: bloqueia áudio para planos básicos) e decide se o usuário deve receber um desafio ou entrar em conversa livre.

#### 2. Inteligência Artificial (IA)
* **IA Validação** `(OpenAI / LLM)`
  * **Função:** "Guardião" do onboarding. Analisa se o input do usuário responde à pergunta feita (ex: se o usuário digitou um nome válido ou algo sem sentido).

* **IA Gerador** `(Orimon / Gemini)`
  * **Função:** Motor do "Modo Perpétuo". Analisa o histórico dos últimos 30 dias para criar um desafio inédito e personalizado quando as trilhas fixas terminam.

* **IA Terapeuta** `(Chatbase)`
  * **Função:** Chatbot com persona psicológica configurada. Gerencia as conversas livres, oferecendo suporte emocional, validação e *insights* baseados no contexto do usuário.

#### 3. Regras de Negócio e Retenção
* **Recuperação** `(Fluxo: Mensagem Vencimento)`
  * **Função:** Ativado quando a assinatura expira. Envia mensagens persuasivas, imagens personalizadas por plano e links de checkout (Greenn) para renovação.

* **Cobrar Desafio** `(Cron: Notifica 18h/23h)`
  * **Função:** Sistema de retenção (Streak). Verifica periodicamente se o desafio do dia foi cumprido. Se não, dispara lembretes para evitar que o usuário perca o ritmo da jornada.

* **Janela 24h** `(Nó: Lógica de Tempo)`
  * **Função:** Verifica a janela de sessão do WhatsApp Business API. Garante que mensagens proativas sejam enviadas apenas dentro das regras da Meta ou via *Message Templates*.

#### 4. Infraestrutura e Dados
* **Banco Dados** `(NoCodeBackend / Redis)`
  * **Função:**
    * **Redis:** Cache de alta velocidade para contexto imediato da conversa.
    * **NoCodeBackend:** Persistência de longo prazo (perfis, status de assinatura, logs de jornada).
