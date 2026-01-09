## ⚡ Sistema de Disparo Otimizado com Humanização (WhatsApp)

#### 🧠 Lógica do Processo: Este workflow é um motor robusto para campanhas de marketing via WhatsApp, focado em alta entregabilidade e segurança contra bloqueios. O processo inicia via agendamento ou fila (RabbitMQ), buscando uma lista de leads no Google Sheets. A inteligência central reside na distribuição de carga: o sistema valida se o número possui WhatsApp ativo, rotaciona entre múltiplas instâncias de envio (balanceamento de carga) e altera dinamicamente o texto da mensagem (Variação de Copy) baseando-se no histórico salvo no Redis para evitar repetições. Antes do envio, aplica um delay aleatório (humanização) para simular comportamento real. O status final (Enviado, Inválido ou Erro) é gravado de volta na planilha para relatórios.

---

### 🏗️ Diagrama de Arquitetura:

```mermaid
graph TD
    %% Estilos Obrigatórios
    classDef whatsapp fill:#25D366,stroke:#fff,stroke-width:2px,color:#fff
    classDef n8n fill:#FF6D5A,stroke:#fff,stroke-width:2px,color:#fff
    classDef db fill:#3C9CD7,stroke:#fff,stroke-width:2px,color:#fff
    classDef subflow fill:#FF9F89,stroke:#333,stroke-width:1px,stroke-dasharray: 5 5,color:#000
    classDef endnode fill:#95a5a6,stroke:#333,color:#fff

    Start(("Agendamento/<br/>RabbitMQ")):::n8n --> Config["Carregar Configs"]:::n8n
    Config --> GSheets_Get[("Google Sheets:<br/>Buscar Leads")]:::db
    GSheets_Get --> Batch["Split Batch:<br/>Processamento"]:::n8n
    
    Batch --> CodeLogic["Code: Validação,<br/>Variação de Copy<br/>e Balanceamento"]:::n8n
    
    CodeLogic --> RedisGet[("Redis:<br/>Ler Histórico")]:::db
    RedisGet --> CheckAPI["API: Verificar<br/>WhatsApp Válido"]:::whatsapp
    
    CheckAPI --> Switch{"Válido?"}:::n8n
    
    %% Caminho Válido
    Switch -- "Sim" --> RandomDelay["Wait: Delay<br/>Aleatório (Humanização)"]:::n8n
    RandomDelay --> SendMsg["API: Enviar<br/>Mensagem"]:::whatsapp
    SendMsg --> RedisSet[("Redis:<br/>Salvar Histórico")]:::db
    RedisSet --> GSheets_Ok[("Sheets: Update<br/>Status OK")]:::db
    
    %% Caminho Inválido/Erro
    Switch -- "Não/Erro" --> GSheets_Fail[("Sheets: Update<br/>Inválido/Erro")]:::db
    
    GSheets_Ok --> Loop(("Próximo")):::n8n
    GSheets_Fail --> Loop

```

---

### 🔍 Dicionário de Dados

#### 1. Entrada e Gatilhos

* **RabbitMQ / Schedule** `(Nós: RabbitMQ Trigger / Schedule Trigger)`
* **Função:** Iniciadores do processo. Permite que o disparo ocorra em horário fixo ou via mensagem em fila para escalabilidade horizontal.


* **Google Sheets (Leitura)** `(Nó: Buscar Leads)`
* **Função:** Fonte de Dados. Recupera os contatos a serem processados, filtrando apenas aqueles que ainda não receberam mensagem ("Enviado" vazio ou pendente).



#### 2. Processamento Inteligente

* **Motor de Decisão (Code)** `(Nó: Code1)`
* **Função:** Cérebro da Operação.
* **Normalização:** Trata números de telefone (DDI+DDD+Número).
* **Load Balance:** Alterna o envio entre diferentes instâncias conectadas.
* **Anti-Spam:** Seleciona variações de texto (A/B testing) e verifica no Redis qual foi a última mensagem enviada para aquele número, evitando repetições.




* **Memória Redis** `(Nós: Redis_Get / Redis)`
* **Função:** Cache de Contexto. Armazena o histórico de variações de mensagens enviadas por número, garantindo rotação de conteúdo e evitando comportamento robótico.



#### 3. Execução e Saída

* **Gateway WhatsApp** `(Nós: Whatsapp válido? / Envia mensagem)`
* **Função:** Interface de Comunicação.
* Primeiro valida se o número existe no WhatsApp (economiza custos e evita banimento).
* Realiza o disparo efetivo da mensagem com delay simulado.




* **Google Sheets (Escrita)** `(Nós: ATUALIZAR / Erro1 / Atualizar Planilha)`
* **Função:** Log de Auditoria. Registra o resultado de cada tentativa (Enviado, WhatsApp Inválido, Erro) e a data/hora do processamento, permitindo gestão visual do progresso.


