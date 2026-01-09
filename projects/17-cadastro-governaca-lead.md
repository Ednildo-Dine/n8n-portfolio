## ⚡ Sistema de Cadastro e Governança de Leads

#### 🧠 Lógica do Processo: Este workflow atua como um "Processador de Entrada" dedicado a oficializar novos usuários que solicitam o teste gratuito de um produto. Ele consome mensagens da fila **RabbitMQ** para garantir que nenhum pedido seja perdido. A inteligência do fluxo está na **Validação de Unicidade**: ele verifica se o lead já existe na base para impedir duplicidade de cadastro. Uma vez validado, o sistema orquestra a distribuição dos dados para dois destinos estratégicos: o **Google BigQuery** (para inteligência de dados e histórico) e o **Google Sheets** (para controle operacional da equipe), utilizando pausas inteligentes para respeitar os limites das APIs.

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

    Start((Fila RabbitMQ)):::whatsapp --> Normalize["Normalizar Dados"]:::n8n
    Normalize --> CheckDB{"Já Cadastrado?"}:::n8n

    CheckDB -- "Sim" --> EndNode([Fim / Ignorar]):::endnode
    CheckDB -- "Não" --> Prepare["Preparar Registro"]:::n8n

    Prepare --> Wait1["Delay API"]:::n8n
    Wait1 --> BigQuery[("Google BigQuery:<br/>Salvar Histórico")]:::db
    
    BigQuery --> Wait2["Delay API"]:::n8n
    Wait2 --> GSheets[("Google Sheets:<br/>Controle Operacional")]:::db

    GSheets --> EndNode

```

---

### 🔍 Dicionário de Dados

#### 1. Recebimento e Fila

* **RabbitMQ Trigger** `(Nó: RabbitMQ Trigger1)`
* **Função:** Entrada Assíncrona. Recebe as solicitações de novos testes grátis via fila. Isso separa o atendimento (bot/site) do processamento pesado, garantindo fluidez mesmo com alto volume de inscrições.



#### 2. Qualidade e Validação

* **Normalizador** `(Nó: Code)`
* **Função:** Padronização. Limpa e formata os dados brutos (ex: ajusta telefones, padroniza e-mails) para garantir que o cadastro seja feito corretamente.


* **Verificador de Duplicidade** `(Nós: consulta / If2)`
* **Função:** Filtro de Segurança. Consulta o banco de dados antes de registrar. Se o usuário já tiver cadastro, o fluxo encerra a execução, mantendo a base de dados limpa e sem repetidos.



#### 3. Registro e Armazenamento

* **Google BigQuery** `(Nó: Google BigQuery)`
* **Função:** Base Analítica. Grava o lead no Data Warehouse para relatórios de longo prazo e inteligência de negócio.


* **Google Sheets** `(Nó: Google Sheets)`
* **Função:** Controle Diário. Cria uma cópia do registro em uma planilha de fácil acesso, permitindo que a equipe de vendas ou suporte visualize os novos inscritos em tempo real.
