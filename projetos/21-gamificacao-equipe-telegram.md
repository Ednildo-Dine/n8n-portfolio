## ⚡ Bot de Gamificação e Coleta de Conteúdo (Telegram)

#### 🧠 Lógica do Processo
Este workflow implementa uma estratégia de **gamificação interna** para incentivar colaboradores (vendas e logística) a documentarem entregas e experiências com clientes. O processo inicia quando o colaborador envia arquivos de mídia via Telegram. O sistema identifica o remetente, valida o conteúdo e classifica o tipo de arquivo recebido, aplicando regras de pontuação distintas (ex: vídeos geram mais pontos que fotos). Esses dados alimentam automaticamente um **Dashboard de Performance** (planilha ou banco de dados), atualizando o ranking da equipe. Por fim, o bot envia um feedback imediato ao colaborador confirmando a pontuação recebida, fechando o ciclo de recompensa e engajamento.

---

### 🏗️ Diagrama de Arquitetura

```mermaid
graph TD
    %% Estilos Obrigatórios
    classDef whatsapp fill:#25D366,stroke:#fff,stroke-width:2px,color:#fff
    classDef n8n fill:#FF6D5A,stroke:#fff,stroke-width:2px,color:#fff
    classDef db fill:#3C9CD7,stroke:#fff,stroke-width:2px,color:#fff
    classDef subflow fill:#FF9F89,stroke:#333,stroke-width:1px,stroke-dasharray: 5 5,color:#000
    classDef endnode fill:#95a5a6,stroke:#333,color:#fff

    Start((Entrada<br/>Telegram)):::whatsapp --> Identify[Identificar<br/>Colaborador]:::n8n
    Identify --> Validate{Mídia<br/>Válida?}:::n8n

    Validate -- "Não" --> ErrorMsg[Alertar<br/>Formato Inválido]:::whatsapp
    Validate -- "Sim" --> MediaType{Foto ou<br/>Vídeo?}:::n8n

    MediaType -- "Vídeo" --> ScoreVideo[Calcular Pontos<br/>Regra Vídeo]:::n8n
    MediaType -- "Foto" --> ScorePhoto[Calcular Pontos<br/>Regra Foto]:::n8n

    ScoreVideo & ScorePhoto --> Dashboard[("Atualizar Dashboard<br/>Planilha e KPIs")]:::db
    
    Dashboard --> Ranking[Verificar<br/>Ranking Atual]:::n8n
    Ranking --> Reply((Confirmar<br/>Pontuação)):::whatsapp
    
    Reply & ErrorMsg --> EndNode([Fim]):::endnode

```

---

### 🔍 Dicionário de Dados

#### 1. Interface de Comunicação

* **Bot Telegram (Entrada/Saída)**
* **Função:** Canal oficial de comunicação interna. Recebe as evidências (mídia) e atua como interface de feedback para o colaborador.
* **Entrada principal:** Arquivos de mídia (foto/vídeo) e ID do usuário Telegram.
* **Saída principal:** Mensagem de texto com saldo de pontos e confirmação de recebimento.



#### 2. Motor de Regras e Gamificação

* **Identificador de Colaborador**
* **Função:** Mapeia o ID do Telegram para o registro do funcionário (Vendas ou Logística) para garantir a atribuição correta dos pontos.
* **Observações:** Filtra usuários não autorizados.


* **Calculadora de Pontuação**
* **Função:** Aplica a lógica de negócio definida para transformar a interação em valor numérico (score).
* **Regra de Negócio:** Vídeos podem ter um peso/multiplicador maior que fotos estáticas.



#### 3. Gestão de Dados

* **Dashboard de Performance (Planilha/DB)**
* **Função:** Base central da verdade para a premiação. Armazena o histórico de envios, links das mídias e o saldo acumulado.
* **Entrada principal:** ID do colaborador, Data/Hora, Link da Mídia, Pontos gerados.
* **Saída principal:** Dados consolidados para visualização da gestão.
