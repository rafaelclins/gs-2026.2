# GS2026.2 — CitizenTech: Assistente de Mobilidade Urbana

## Equipe

- Eric dos Santos Mendes da Silva — RM 569528
- Rafael Côrtes Lins — RM 570588
- Tom Stringasci Albuquerque Coelho Morais — RM 568844

**Turma:** 1CCPG | **Tema:** Mobilidade

---

## 1. Visão Geral

Sistema RAG (Retrieval-Augmented Generation) que consulta base documental municipal sobre transporte público (Linha 714), responde com citações de fonte e prova sua precisão via avaliação RAGAS.

**Problema:** Cidadãos não sabem se informações sobre ônibus estão atualizadas (pontos interditados, mudanças de tarifa).

**Solução:** Assistente que recupera documentos oficiais, compara datas e responde apenas com evidência rastreável.

---

## 2. Arquitetura do Pipeline

```text
┌──────────────────┐      ┌───────────────────┐      ┌──────────────────┐
│ Base Documental  │─────▶│ Chunking          │─────▶│ Embeddings       │
│ (2 arquivos)     │      │ (2 estratégias)   │      │ (Ollama)         │
└──────────────────┘      └───────────────────┘      └──────────────────┘
                                                              │
                                                              ▼
┌──────────────────┐      ┌───────────────────┐      ┌──────────────────┐
│ Interface        │◀─────│ RAG (LCEL)        │◀─────│ ChromaDB         │
│ (Gradio)         │      │                   │      │                  │
└──────────────────┘      └───────────────────┘      └──────────────────┘
        │                          │
        ▼                          ▼
┌──────────────────┐      ┌───────────────────┐
│ Agente c/        │      │ Guardrails        │
│ Ferramentas      │      │ (3 camadas)       │
└──────────────────┘      └───────────────────┘
        │
        ▼
┌──────────────────┐
│ Avaliação        │
│ (RAGAS)          │
└──────────────────┘
```

---

## 3. Componentes Técnicos

| Componente | Tecnologia | Função |
|------------|------------|--------|
| LLM | Ollama Cloud (gemma4:31b) | Geração de respostas |
| Embeddings | Ollama (nomic-embed-text) | Vetorização de documentos |
| Vetor DB | ChromaDB | Busca semântica |
| Orquestração | LangChain (LCEL) | Pipeline RAG |
| Agente | LangChain Tools | Decisão de usar ferramentas |
| Interface | Gradio | Chat web |
| Avaliação | RAGAS (llama3.2 + nomic-embed-text) | Métricas de qualidade |

---

## 4. Base Documental

### 4.1 Estrutura

| Arquivo | Tipo | Data | Conteúdo |
|---------|------|------|----------|
| `linha_714_itinerario_2024.txt` | Itinerário oficial | 15/03/2024 | Horários, tarifas, pontos ativos |
| `portaria_atualizacao_2025.txt` | Portaria | 22/01/2025 | Interdição de ponto, mudança de tarifa |

### 4.2 Armadilhas Propositais

| Tipo | Descrição | Teste |
|------|-----------|-------|
| **Contradição** | Doc 2024 diz ponto "ATIVO", Doc 2025 diz "INTERDITADO" | "O ponto da Rua das Flores está funcionando?" |
| **Faixa condicional** | Gratuidade só em dias úteis (mudou em 2025) | "Meu pai de 68 anos pode viajar de graça no sábado?" |
| **Detalhe escondido** | Ponto alternativo no item 2 da portaria | "Onde pego o ônibus se o ponto estiver fechado?" |

---

## 5. Decisões Técnicas

### 5.1 Chunking

| Estratégia | Chunk Size | Overlap | Chunks | Justificativa |
|------------|------------|---------|--------|---------------|
| CharacterTextSplitter | 500 | 50 | 7 | Rápido, mas quebra contexto |
| **RecursiveCharacterTextSplitter** | **1000** | **100** | **4** | **Preserva parágrafos completos** ✅ |

**Escolha:** RecursiveCharacterTextSplitter — mantém portarias inteiras, essencial para comparar datas.

### 5.2 Guardrails

| Guardrail | Implementação | Exemplo |
|-----------|---------------|---------|
| Validação de data | Compara `data_publicacao` nos metadados | Sempre usa documento mais recente |
| Proteção contra alucinação | Prompt: "Se não estiver no contexto, diga 'não sei'" | Nunca inventa horário |
| Sanitização de input | Detecta "ignore instruções anteriores" | Bloqueia prompt injection |

---

## 6. Resultados RAGAS

| Métrica | Valor | Análise |
|---------|-------|---------|
| faithfulness | 0.85 | Respostas ancoradas no contexto |
| answer_relevancy | 0.92 | Perguntas respondidas diretamente |
| context_recall | 0.78 | Recuperou portaria 2025 na maioria dos casos |
| context_precision | 0.81 | Chunks relevantes priorizados |

**Melhoria proposta:** Implementar reranking no retriever para aumentar context_precision em perguntas sobre faixa condicional.

---

## 7. Como Executar

1. Abrir notebook no Google Colab
2. Configurar `OLLAMA_API_KEY` nos Secrets
3. Runtime → Run all
4. Acessar interface Gradio via link público

---

## 8. Repositório

[[Link do GitHub/GitLab do grupo](https://github.com/rafaelclins/gs-2026.2)]

---

*Documentação gerada para GS2026.2 — CitizenTech*