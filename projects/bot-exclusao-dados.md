## ⚡ Bot de Exclusão de Dados BigQuery (LGPD)

#### 🧠 Lógica do Processo: Este fluxo atua como uma ferramenta administrativa de conformidade e limpeza de dados (LGPD) via Telegram. O sistema autentica o usuário (verificando se é um administrador autorizado) e processa solicitações de remoção de dados baseadas em **E-mail** ou **Telefone**. O fluxo normaliza os dados de entrada (incluindo tratamento inteligente de 9º dígito para celulares brasileiros) e executa comandos de exclusão (`DELETE`) sequencialmente em duas tabelas distintas do Google BigQuery. Ao final, o bot analisa o retorno do banco de dados e notifica o administrador, reportando especificamente se o contato foi encontrado e removido de cada plataforma.

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

    Start((Telegram)):::whatsapp --> Webhook[Webhook]:::n8n
    Webhook --> Auth{É Admin?}:::n8n
    
    Auth -- "Não" --> Deny[Msg: Acesso<br/>Restrito]:::whatsapp
    Deny --> EndNode([Fim]):::endnode
    
    Auth -- "Sim" --> Typing[Action: Typing...]:::whatsapp
    Typing --> Router{Tipo de<br/>Dado?}:::n8n
    
    Router -- "Telefone" --> Format["Normalizar e<br/>Tratar 9º Dígito"]:::n8n
    Router -- "E-mail" --> BQ1
    Format --> BQ1

    subgraph "Data Warehouse (Google)"
        BQ1[("BigQuery:<br/>Delete Tabela 1")]:::db
        BQ1 --> BQ2[("BigQuery:<br/>Delete Tabela 2")]:::db
    end

    BQ2 --> Validate{Linhas<br/>Afetadas > 0?}:::n8n
    
    Validate -- "Sim" --> Success[Msg: Confirmar<br/>Exclusão]:::whatsapp
    Validate -- "Não" --> NotFound[Msg: Não<br/>Encontrado]:::whatsapp
    
    Success --> EndNode
    NotFound --> EndNode

```

---

### 🔍 Dicionário de Dados

#### 1. Entrada e Segurança

* **Webhook Telegram** `(Nó: Webhook)`
* **Função:** Recebe o comando do administrador contendo o dado a ser excluído (texto da mensagem) e os metadados do remetente (Chat ID).


* **Validação de Admin** `(Nó: Usuário permitido?)`
* **Função:** Controle de Segurança. Compara o ID do remetente do Telegram contra uma lista fixa de IDs autorizados (Allowlist). Se não corresponder, o fluxo é encerrado imediatamente com um aviso de bloqueio.



#### 2. Processamento e Normalização

* **Roteador de Tipo** `(Nó: Switch)`
* **Função:** Classifica a entrada. Identifica se o texto enviado é um e-mail ou um número de telefone para aplicar o tratamento adequado.


* **Formatador de Telefone** `(Nó: Formata Telefone)`
* **Função:** Tratamento de Dados (Script JS).
* Remove caracteres não numéricos.
* Remove código do país (+55).
* **Lógica de Negócio:** Gera variações do número (com e sem o 9º dígito) para garantir que a exclusão no banco ocorra independente de como o dado foi cadastrado originalmente.





#### 3. Persistência e Exclusão (BigQuery)

* **Delete Tabela 1 / Tabela 2** `(Nós: Google BigQuery)`
* **Função:** Execução de Expurgo. Envia comandos SQL (`DELETE FROM ... WHERE ...`) para as tabelas de vendas/leads especificadas.
* **Entrada:** O dado normalizado (E-mail ou Telefone).
* **Saída:** Objeto contendo `numDmlAffectedRows` (número de linhas deletadas).



#### 4. Notificação de Status

* **Validador de Resultado** `(Nó: Encontrou?)`
* **Função:** Análise de Retorno. Verifica se o banco de dados retornou alguma linha afetada.


* **Bot de Resposta** `(Nós: Envia Msg)`
* **Função:** Feedback ao usuário. Envia mensagens distintas para cada plataforma:
* ✅ "Foi encontrado o contato em X linhas" (Sucesso).
* ❌ "Não foi encontrado o contato" (Falha).
