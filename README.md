# Assistente Virtual - Supermercado (Agente de IA com RAG)

Agente de inteligência artificial com **RAG (Retrieval-Augmented Generation)** que responde dúvidas de clientes e colaboradores de um supermercado, com base em documentos internos (políticas, regulamentos, tabelas de remuneração, procedimentos operacionais, etc). Projeto desenvolvido para o **Challenge Alura Agente - Oracle Next Education (ONE) - AI for Tech**

## Descrição geral do projeto

O agente lê documentos PDF armazenados no Google Drive, indexa esse conteúdo em um banco vetorial (Pinecone) e usa um modelo de linguagem (Groq) para responder perguntas em linguagem natural, com base **apenas** nas informações encontradas nos documentos oficiais da empresa. O agente atende tanto clientes (dúvidas de atendimento) quanto colaboradores (políticas internas, procedimentos, RH).

## Arquitetura da solução

O projeto é dividido em duas partes: o **agente** (construído em n8n) e a **interface** (página web que conversa com o agente).

### 1. Indexação dos documentos (pipeline de ingestão)

```
Google Drive (pasta "database")
        │
        ▼
Download do arquivo PDF
        │
        ▼
Recursive Character Text Splitter (chunks de ~500 caracteres)
        │
        ▼
Embeddings (Cohere - embed-multilingual-v3.0)
        │
        ▼
Pinecone (índice vetorial: llama-text-embed-v2-index)
```

### 2. Fluxo de resposta a uma pergunta

```
Usuário (interface web)
        │
        ▼
Webhook do n8n (Chat Trigger)
        │
        ▼
AI Agent (LangChain Agent)
   ├── Memória: Buffer Window (histórico da conversa)
   ├── Modelo de linguagem: Groq (openai/gpt-oss-20b)
   └── Ferramenta: busca no Pinecone (top 3 trechos mais relevantes)
        │
        ▼
Resposta gerada em português, com base nos trechos encontrados
        │
        ▼
Exibida na interface web
```

### 3. Interface

Página web estática (HTML/CSS/JS puro, sem framework) que se comunica diretamente com o Webhook público do n8n. Elementos da interface:
- Aviso claro de que o usuário está falando com um agente de IA
- Histórico de conversa da sessão
- Exibição de fontes/documentos usados na resposta (quando disponíveis)
- Botões de feedback (👍/👎) em cada resposta

## Tecnologias e ferramentas utilizadas

| Camada | Ferramenta |
|---|---|
| Orquestração do agente | n8n (self-hosted na n8n Cloud) |
| Modelo de linguagem (LLM) | Groq - openai/gpt-oss-20b |
| Embeddings | Cohere - embed-multilingual-v3.0 |
| Banco vetorial | Pinecone |
| Fonte dos documentos | Google Drive |
| Interface | HTML, CSS e JavaScript puro |
| Deploy da interface | Oracle Cloud Infrastructure (OCI) |

## Como executar o projeto

### Pré-requisitos
- Conta no [n8n](https://n8n.io/) (cloud ou self-hosted)
- Conta no [Pinecone](https://www.pinecone.io/) com um índice criado
- Conta na [Cohere](https://cohere.com/) para embeddings
- Conta na [Groq](https://console.groq.com/) para o modelo de linguagem
- Uma pasta no Google Drive com os documentos PDF da base de conhecimento

### Passos

1. Importe o arquivo `My workflow.json` no seu n8n (`Import from File`).
2. Configure as credenciais de cada serviço (Google Drive, Pinecone, Cohere, Groq).
3. Aponte o nó "Search files and folders" para a pasta do Google Drive com os documentos.
4. Execute manualmente o fluxo de ingestão (Search files → Download → Pinecone Vector Store) para indexar os documentos.
5. Ative o workflow (botão "Active") e copie a URL de produção do Webhook (nó "When chat message received").
6. Abra o arquivo `index.html`, edite a constante `WEBHOOK_URL` com a URL copiada.
7. Abra o `index.html` no navegador (ou hospede como página estática) e comece a conversar com o agente.

## Exemplos de perguntas que o agente consegue responder

- "Onde ficam guardados produtos como o ketchup?"
- "Qual emprego no supermercado recebe o salário mais alto? E qual é esse salário?"
- Outras perguntas sobre políticas internas, procedimentos operacionais e atendimento ao cliente presentes na base de documentos.

## Exemplos de respostas geradas pelo agente

> **Pergunta:** Onde ficam guardados produtos como o ketchup?
>
> **Resposta:** Produtos alimentícios como o ketchup devem ficar sobre estrados de polietileno lavável, nunca em contato direto com o chão. Os estrados precisam estar a no mínimo 20 cm do piso e organizados pelo princípio FIFO (First In First Out) para garantir a rotatividade correta. O local de armazenamento deve ser separado de produtos químicos ou de limpeza, que têm sinalização própria.

> **Pergunta:** Qual emprego no supermercado recebe o salário mais alto? E qual é esse salário?
>
> **Resposta:** Cargo com o salário mais alto: Diretoria Executiva (Comitê de Compras). Faixa salarial: acima de R$ 500.000,00 mensais. Esses valores são os máximos indicados na tabela de remuneração da empresa.

## Manutenção contínua

- **Atualização de documentos**: sempre que um documento na pasta do Google Drive for alterado, o pipeline de ingestão (nós "Search files and folders" → "Download" → "Pinecone Vector Store") deve ser executado novamente para reprocessar e reindexar o conteúdo atualizado. Pode ser automatizado com um Trigger agendado (ex: diário) no n8n.
- **Curadoria de conteúdo**: recomenda-se que um responsável por área (RH, Operações, Compras) revise periodicamente se os PDFs na pasta do Drive ainda são a versão oficial vigente.
- **Monitoramento de qualidade**: acompanhar os feedbacks (👍/👎) dados pelos usuários na interface para identificar respostas mal avaliadas ou perguntas recorrentes sem boa resposta, indicando a necessidade de novos documentos na base.
- **Atualização do modelo**: reavaliar periodicamente se uma nova versão de modelo na Groq traz melhoria de qualidade ou custo, testando antes de substituir em produção.

## Deploy no Github Pages

A interface web (`index.html`) foi implantada na Oracle Cloud Infrastructure.

- **URL da aplicação:** `https://brenoamorim7.github.io/RAG-Oracle/`
- **Print da aplicação em produção:** 

## Estrutura do repositório

```
.
├── My workflow.json     # Workflow completo do n8n (agente RAG)
├── index.html            # Interface de chat web
└── README.md
```

## Autor

Breno — Projeto desenvolvido para o Challenge Alura Agente (Oracle Next Education - AI for Tech).