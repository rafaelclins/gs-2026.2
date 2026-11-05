# Assistente de Mobilidade — Linha 714 (Vai-e-Vem)

Projeto de **Generative Systems (GS 2026.2)** que implementa um pipeline **RAG** sobre a base documental da Linha 714 de transporte público, com interface de chat em Gradio, agente com ferramentas, guardrails de segurança e avaliação com RAGAS.

**Tema:** Mobilidade  
**Turma/Sala:** 1CCPG  
**Grupo:** Grupo X

## Integrantes

| Integrante | RM |
|---|---|
| Eric dos Santos Mendes da Silva | 569528 |
| Rafael Côrtes Lins | 570588 |
| Tom Stringasci Albuquerque Coelho Morais | 568844 |

## Descrição do projeto

O notebook `GS2026_2_1CCPG_GrupoX.ipynb` constrói um assistente capaz de responder perguntas sobre a Linha 714 usando dois documentos oficiais:

- `linha_714_itinerario_2024.txt` — itinerário, horários, tarifas e pontos (2024).
- `portaria_atualizacao_2025.txt` — interdição do ponto da Rua das Flores e mudança da gratuidade (Portaria 047/2025).

O sistema:

1. Carrega e divide os documentos em chunks com `RecursiveCharacterTextSplitter` (1000/100).
2. Gera embeddings com `nomic-embed-text` (Ollama local) e indexa no ChromaDB.
3. Recupera os chunks mais relevantes (`k=4`) e gera respostas com `gemma4:31b` (Ollama Cloud), citando `doc_id`.
4. Oferece um agente com ferramentas (`calcular_proximo_horario` e `verificar_status_ponto`).
5. Expõe uma interface Gradio com chat, fontes consultadas e nível de confiança.
6. Aplica três guardrails: validação de faixa etária, proteção contra alucinação de horários e sanitização contra prompt injection.
7. Avalia o pipeline com RAGAS (`llama3.2` + `nomic-embed-text`) e exporta `3_avaliacao_ragas.json`.

## Diagrama do pipeline

```mermaid
flowchart LR
    A[Base documental<br/>2024 + 2025] --> B[Chunking<br/>Recursive 1000/100]
    B --> C[Embeddings<br/>nomic-embed-text]
    C --> D[(ChromaDB)]

    E[Pergunta do usuário] --> F{Guardrails}
    F -->|injection| G[Não posso processar]
    F -->|horário não documentado| H[Não tenho essa informação]
    F -->|ok| I[Retriever k=4]

    D --> I
    I --> J[Prompt + LLM<br/>gemma4:31b]
    I --> K[Fontes: doc_id, trecho, score]
    J --> L[Resposta + confiança]
    K --> L

    I --> M[Agente com tools<br/>horários e status]
    M --> L
    L --> N[Interface Gradio]
```

## Prints das telas

> Substitua os arquivos abaixo pelos prints reais após executar a interface Gradio (`demo.launch()`).

### Chat com resposta e confiança

![Chat do assistente](docs/tela-chat.png)

### Painel de fontes consultadas

![Painel de fontes](docs/tela-fontes.png)

### Avaliação RAGAS

![Métricas RAGAS](docs/tela-ragas.png)

## Instruções de execução

1. Abra `GS2026_2_1CCPG_GrupoX.ipynb` no **Google Colab** (Python 3.10+).
2. Execute a célula de setup e instale as dependências:
   ```bash
   !pip install -q chromadb langchain langchain-core langchain-classic langchain-community \
     langchain-text-splitters langchain-ollama langchain-chroma \
     gradio ragas datasets openai python-dotenv
   ```
3. Envie a pasta `2_base_documental` para `/content` (ou monte o Google Drive).
4. Em **Colab > Secrets**, configure `OLLAMA_API_KEY` com a chave da Ollama Cloud.
5. Execute as seções **0 a 7** em ordem. A seção 2 instala o Ollama local e baixa `nomic-embed-text`; a seção 7 baixa `llama3.2` para o RAGAS.
6. A interface Gradio sobe na seção 5:
   ```python
   demo.launch(share=True)
   ```
7. A avaliação da seção 7 gera o arquivo:
   ```
   3_avaliacao_ragas.json
   ```

### Requisitos

- Conta/chave na Ollama Cloud (`OLLAMA_API_KEY`).
- Ollama local disponível em `http://localhost:11434` (instalado pela seção 2).
- Modelos: `gemma4:31b` (Cloud), `nomic-embed-text` e `llama3.2` (locais).

## Link do vídeo

> Adicione aqui o link da demonstração.

[Vídeo de demonstração](https://youtu.be/SEU_LINK_AQUI)