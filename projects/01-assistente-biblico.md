## ⚡ Sistema de Assistente Bíblico Multicanal com Gestão de Assinaturas

#### 🧠 Lógica do Processo: Este sistema atua como um orquestrador inteligente de atendimento via WhatsApp, especializado em conteúdo teológico. Ao receber uma mensagem, o fluxo principal consulta uma base de dados externa para identificar o status da assinatura do usuário (Gratuito, Limitado ou Ilimitado). Com base nessa validação, a requisição é direcionada para o sub-fluxo correspondente, garantindo o controle de uso. O núcleo de processamento utiliza o modelo Google Gemini para gerar respostas, apoiado por um banco Redis que mantém o contexto da conversa (memória). Simultaneamente, todo o histórico e gestão de contatos são sincronizados com o Chatwoot, permitindo transbordo humano e visualização centralizada do atendimento.

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

    Start((whatsapp)):::whatsapp --> Webhook[Webhook]:::n8n
    Webhook --> Validate["Consultar API<br/>de Assinaturas"]:::db
    Validate --> Switch{Status?}:::n8n

    %% Correção: Nomes curtos e quebras de linha
    Switch -- "Ilimitado" --> SubUnlimited["Sub: Cliente<br/>Ilimitado"]:::subflow
    Switch -- "15 Msgs" --> Sub15["Sub: Cota<br/>Limitada"]:::subflow
    Switch -- "Grátis" --> SubFree["Sub: Teste<br/>Gratuito"]:::subflow
    
    %% Tratamento de erro movido para baixo para economizar largura
    Switch -- "Bloqueado" --> Bloqueio["Msg: Bloqueio<br/>e Oferta"]:::whatsapp
    Bloqueio --> EndNode([Fim]):::endnode

    subgraph "Motor de IA"
        SubUnlimited & Sub15 & SubFree --> Redis[("Redis<br/>(Memória)")]:::db
        Redis --> Gemini["Google<br/>Gemini 2.0"]:::db
        Gemini --> Chatwoot["Sync CRM<br/>Chatwoot"]:::n8n
    end

    Chatwoot --> Reply((Responder)):::whatsapp
    Reply --> EndNode
