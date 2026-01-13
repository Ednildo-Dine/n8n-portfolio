## ⚡ Gateway de Ingestão de Eventos e Roteamento (Webhooks > Message Broker)

#### 🧠 Lógica do Processo: Este padrão arquitetural funciona como um **Roteador de Alta Disponibilidade**. O objetivo é desacoplar a recepção de dados do seu processamento pesado. O fluxo recebe requisições HTTP de diversas **Fontes Externas** (Plataformas de Vendas, CRMs, Formulários), realiza uma normalização rápida dos dados (padronização de telefone, extração de IDs, tratamento de UTMs) e despacha o payload imediatamente para uma fila específica no **Message Broker** (Sistema de Mensageria). Isso garante que a fonte externa receba um "200 OK" instantâneo, evitando timeouts, enquanto assegura a persistência dos dados para processamento posterior (assíncrono).

---

### 🏗️ Diagrama de Arquitetura:

```mermaid
graph TD
    %% Estilos
    classDef source fill:#6f33a3,stroke:#fff,stroke-width:2px,color:#fff
    classDef middleware fill:#FF6D5A,stroke:#fff,stroke-width:2px,color:#fff
    classDef queue fill:#F4B400,stroke:#fff,stroke-width:2px,color:#fff
    classDef endnode fill:#95a5a6,stroke:#333,color:#fff

    subgraph "Origem"
        Source[Fontes Externas]:::source
    end

    Source --> Webhook["Webhook:<br/>Receber Payload"]:::middleware
    
    Webhook --> Normalize["Data Prep:<br/>Tratar/Padronizar"]:::middleware
    
    Normalize --> Router{{"Roteador:<br/>Selecionar Fila"}}:::middleware

    %% Destinos Dinâmicos
    Router --> Q1["Fila A:<br/>Dados Analíticos"]:::queue
    Router --> Q2["Fila B:<br/>Ativação de Cliente"]:::queue
    Router --> Q3["Fila C:<br/>Integração Genérica"]:::queue

    Q1 & Q2 & Q3 --> EndNode([Fim / Ack]):::endnode

```

---

### 🔍 Dicionário de Dados

#### 1. Ingestão (Entrada)

* **Gatilho Webhook** `(Nó: Webhook Receiver)`
* **Função:** Ponto de Entrada. Rotas distintas configuradas para receber JSONs de diferentes eventos (vendas, leads, atualizações).
* **Estratégia:** Configurado para resposta imediata, atuando puramente como coletor de dados brutos sem executar lógica de negócio complexa neste estágio.



#### 2. Normalização (ETL Leve)

* **Tratamento de Dados** `(Nó: Data Transformation)`
* **Função:** Higienização. Antes do envio para a fila, o fluxo padroniza os dados críticos para facilitar o consumo posterior:
* **Contatos:** Remoção de códigos de país duplicados ou formatação de strings numéricas.
* **Identificadores:** Consolidação de IDs de transação ou clientes para rastreabilidade única.
* **Rastreamento:** Mapeamento de parâmetros de origem (UTMs) para atribuição correta.





#### 3. Mensageria (Saída)

* **Produtor de Mensagem** `(Nó: Message Broker Producer)`
* **Função:** Buffer de Segurança.
* **Configuração Padrão:**
* **Alta Disponibilidade:** Uso de filas replicadas (Quorum) para evitar perda de dados.
* **Persistência:** Configuração durável (Durable) para garantir que o evento sobreviva a reinicializações do servidor.


* **Roteamento:** O destino (Fila) é selecionado dinamicamente com base no tipo de evento recebido, segregando responsabilidades (ex: fila de vendas vs. fila de suporte).

