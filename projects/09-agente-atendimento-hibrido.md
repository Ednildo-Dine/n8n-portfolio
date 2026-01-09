## ⚡ Orquestrador de IA Híbrida (Chatbase + Gemini)

#### 🧠 Lógica do Processo: Este workflow atua como um gateway de atendimento inteligente e resiliente para o Telegram. O objetivo principal é fornecer respostas baseadas em uma base de conhecimento personalizada (Chatbase). No entanto, o sistema implementa um padrão de "Failover Inteligente": caso o Chatbase esteja indisponível ou sobrecarregado, o fluxo automaticamente aciona um agente secundário baseado no **Google Gemini**, garantindo que o usuário nunca fique sem resposta. Todo o contexto da conversa é persistido em um banco **PostgreSQL**, permitindo continuidade no diálogo. As respostas são higienizadas (conversão de Markdown para HTML) antes do envio final ao usuário.

---

### 🏗️ Diagrama de Arquitetura:

```mermaid
graph TD
    %% Estilos
    classDef whatsapp fill:#25D366,stroke:#fff,stroke-width:2px,color:#fff
    classDef n8n fill:#FF6D5A,stroke:#fff,stroke-width:2px,color:#fff
    classDef db fill:#3C9CD7,stroke:#fff,stroke-width:2px,color:#fff
    classDef subflow fill:#FF9F89,stroke:#333,stroke-width:1px,stroke-dasharray: 5 5,color:#000
    classDef endnode fill:#95a5a6,stroke:#333,color:#fff

    Input((Entrada Telegram)):::whatsapp --> Webhook[Webhook]:::n8n
    Webhook --> ContextLoad[("Carregar Histórico<br/>(PostgreSQL)")]:::db
    
    ContextLoad --> PrepareData[Preparar Payload]:::n8n
    PrepareData --> Chatbase["Consultar IA Primária<br/>(Chatbase)"]:::db
    
    Chatbase --> CheckStatus{Serviço<br/>Disponível?}:::n8n
    
    %% Caminho de Falha/Fallback
    CheckStatus -- "Não/Erro" --> GeminiAgent["Fallback: Agente<br/>Google Gemini"]:::db
    
    %% Caminho de Sucesso
    CheckStatus -- "Sim" --> Formatter
    GeminiAgent --> Formatter[Formatador HTML/Texto]:::n8n
    
    Formatter --> SaveContext[("Salvar Contexto<br/>(PostgreSQL)")]:::db
    
    SaveContext --> SendTelegram[Responder Telegram]:::whatsapp
   
    
    SendTelegram --> Fim((Fim)):::endnode
 
```

---

### 🔍 Dicionário de Dados

#### 1. Entrada e Contexto

* **Webhook Telegram** `(Nó: Webhook)`
* **Função:** Recebe as mensagens de texto dos usuários em tempo real.


* **Memória PostgreSQL** `(Nós: Postgres Chat Memory / Manager)`
* **Função:** Gestão de Estado (State Management). Armazena e recupera o histórico de mensagens (User + AI) usando uma chave de sessão baseada no ID do usuário, permitindo conversas longas e contextuais.



#### 2. Motores de Inteligência Artificial

* **Chatbase API** `(Nó: HTTP Request / Chatbase)`
* **Função:** IA Primária (RAG). Consulta uma base de conhecimento treinada especificamente com dados do negócio para gerar a resposta ideal.


* **Google Gemini** `(Nó: Google Gemini Chat Model)`
* **Função:** IA de Fallback (Resiliência). Entra em ação apenas se a IA primária falhar, garantindo continuidade do serviço com um modelo de linguagem generalista.


* **Agente de IA** `(Nó: AI Agent)`
* **Função:** Executor de Regras. Gerencia o prompt do sistema e a interação com o modelo Gemini quando o fallback é acionado.



#### 3. Tratamento e Saída

* **Formatador de Texto** `(Nó: Formata texto1)`
* **Função:** Middleware de Apresentação. Converte formatação Markdown (comum em LLMs) para tags HTML suportadas pelo Telegram (negrito, listas, quebras de linha), melhorando a legibilidade.



