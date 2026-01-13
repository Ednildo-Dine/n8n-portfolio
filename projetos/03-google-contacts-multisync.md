## ⚡ SINCRONIZAÇÃO DE CONTATOS MULTI-CONTA (GOOGLE)

#### 🧠 Lógica do Processo

Este fluxo automatiza a distribuição de contatos a partir de uma base centralizada (Google Sheets) para múltiplas agendas corporativas. O sistema lê novos registros na planilha "Matriz", formata os dados telefônicos e cria os contatos sequencialmente em três contas distintas do Google ("Conta 1", "Conta 2" e "Conta 3"). Após a inserção bem-sucedida, o fluxo escreve de volta na planilha original, marcando a linha com "ok" e a data/hora atual para garantir que o contato não seja processado novamente.

---

### 🏗️ Diagrama de Arquitetura

```mermaid
graph TD
    %% Definição de Classes
    classDef whatsapp fill:#25D366,stroke:#fff,stroke-width:2px,color:#fff
    classDef n8n fill:#FF6D5A,stroke:#fff,stroke-width:2px,color:#fff
    classDef db fill:#3C9CD7,stroke:#fff,stroke-width:2px,color:#fff
    classDef subflow fill:#FF9F89,stroke:#333,stroke-width:1px,stroke-dasharray: 5 5,color:#000
    classDef endnode fill:#95a5a6,stroke:#333,color:#fff

    %% Entrada e Leitura
    Start((Gatilho Manual)):::n8n --> |Iniciar| ReadSheet[(Ler Planilha Matriz)]:::db
    
    %% Processamento
    ReadSheet --> |Lista de Contatos| Batch{Processar Lote}:::n8n
    Batch --> |Formatar Dados| Norm[Normalizar Campos]:::n8n

    %% Sincronização Sequencial
    subgraph GoogleEcosystem [Ecossistema Google Contacts]
        Norm --> |Criar/Atualizar| G_Ferreiras[(Conta 1)]:::db
        G_Ferreiras --> |Criar/Atualizar| G_Info[(Conta 2)]:::db
        G_Info --> |Criar/Atualizar| G_Palavra[(Conta 3)]:::db
    end

    %% Atualização de Controle
    G_Palavra --> |Registrar Sucesso| WriteStatus[(Atualizar Planilha)]:::db
    
    %% Loop
    WriteStatus --> |Próximo Item| Batch
    Batch -- Fim --> EndNode((Concluído)):::endnode

```

---

### 🔍 Dicionário de Dados

#### 1. Planilha Matriz (Google Sheets)

* **Função:** Atua como banco de dados principal e fila de processamento (Queue). Armazena os leads brutos e controla quais já foram importados.
* **Entrada principal:** N/A (Leitura de dados existentes).
* **Saída principal:** Dados do Lead (Nome, Email, Telefone, ID da Linha).
* **Observações:** O fluxo filtra especificamente linhas onde a coluna "Adicionado" está vazia ou diferente de "ok".

#### 2. Processador de Lote (n8n Core)

* **Função:** Itera sobre a lista de contatos recuperada para garantir o processamento individual e sequencial, evitando sobrecarga da API (Rate Limiting) e permitindo o controle de erro por item.
* **Entrada principal:** Array JSON com múltiplos contatos.
* **Saída principal:** Objeto JSON único do contato atual.

#### 3. Contas Google (Integração Google Contacts)

* **Função:** Destinos finais da sincronização. O sistema replica o mesmo contato em três ambientes diferentes para garantir que diferentes departamentos ou dispositivos tenham acesso à agenda.
* *Conta 1:* 
* *Conta 2:* 
* *Conta 3:*


* **Entrada principal:** `Given Name`, `Email`, `Phone`.
* **Saída principal:** Confirmação de criação do objeto Contact (ID do Google).

#### 4. Atualizar Status (Google Sheets)

* **Função:** Mecanismo de "Acknowledge" (ACK). Confirma que o processamento total ocorreu e retira o item da fila de futuras execuções.
* **Entrada principal:** ID da Linha (`row_number`) e Timestamp atual.
* **Saída principal:** Célula da coluna "Adicionado" preenchida com `=ok dd/MM/yy HH:mm`.
