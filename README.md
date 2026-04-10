# LangChain

# Agente de RH com LangChain — RAG, Memória e Tools

Projeto de aprofundamento em **LangChain**, construindo um agente conversacional de RH capaz de responder perguntas sobre políticas internas da empresa, calcular benefícios e lembrar do contexto da conversa entre turnos.

---

## O que este projeto demonstra

| Conceito | Como é usado aqui |
|---|---|
| **RAG** (Retrieval Augmented Generation) | Leitura de PDF → fatiamento → embeddings → FAISS → retriever |
| **Tools** | `create_retriever_tool` (busca em docs) + `@tool` customizada (calculadora) |
| **Memória** | `InMemoryChatMessageHistory` + `RunnableWithMessageHistory` |
| **Agente** | `create_tool_calling_agent` + `AgentExecutor` com raciocínio em loop |

---

## Arquitetura

```
PDF (política de RH)
       │
       ▼
  PyPDFLoader          ← lê o documento
       │
       ▼
RecursiveCharacterTextSplitter  ← fatia em chunks de 1000 chars
       │
       ▼
  OpenAIEmbeddings     ← transforma chunks em vetores
       │
       ▼
     FAISS              ← banco vetorial em memória
       │
       ▼
   Retriever (k=3)     ← busca os 3 trechos mais relevantes
       │
       ├──────────────────────────────┐
       ▼                              ▼
create_retriever_tool          @tool calculadora_beneficios
(ferramenta de busca)          (ferramenta de cálculo)
       │                              │
       └──────────┬───────────────────┘
                  ▼
     create_tool_calling_agent
      + AgentExecutor (verbose)
                  │
                  ▼
   RunnableWithMessageHistory  ← injeta histórico da sessão
                  │
                  ▼
          Resposta ao usuário
```

---

## Fluxo do Agente

1. Usuário envia uma pergunta
2. O **Gerente de Memória** recupera o histórico da sessão
3. O **Agente** decide qual ferramenta usar (ou nenhuma)
4. Se precisar de informação do PDF → aciona `pesquisa_regras_rh`
5. Se precisar calcular → aciona `calculadora_beneficios`
6. Responde com base nos resultados das ferramentas
7. A conversa é salva no caderno de memórias para o próximo turno

---

## Metodologia FEYNMAN nos comentários

Todos os imports e blocos de código possuem comentários no estilo **Técnica Feynman**: cada biblioteca é explicada com uma **metáfora do mundo real**, facilitando o entendimento sem precisar da documentação oficial.

Exemplos:
- `PyPDFLoader` → *"Tradutor Oficial"* (extrai texto do PDF)
- `FAISS` → *"Arquivo Mágico / Gaveteiro ultra-rápido"*
- `@tool` → *"Crachá de Identificação"* para funções Python
- `RunnableWithMessageHistory` → *"Gerente de Memória"*

---

## Como executar

### 1. Clone o repositório

```bash
git clone git@github.com:LucasBezerraSantos/langchain-rag-memory-tools.git
cd langchain-rag-memory-tools
```

### 2. Crie o ambiente virtual e instale as dependências

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### 3. Configure a chave da OpenAI

Crie um arquivo `.env` na raiz do projeto:

```
OPENAI_API_KEY=sk-...
```

### 4. Adicione seu PDF

Altere o caminho do arquivo no notebook para apontar para o seu documento de políticas internas:

```python
carregador = PyPDFLoader(r"caminho/para/seu/documento.pdf")
```

### 5. Execute o notebook

Abra `FEYNMAN.ipynb` no Jupyter ou VS Code e execute célula por célula.

---

## Dependências principais

```
langchain
langchain-openai
langchain-community
langchain-text-splitters
faiss-cpu
pypdf
python-dotenv
```

Versões completas em [`requirements.txt`](./requirements.txt).

---

## Exemplo de uso

```python
# Pergunta combinando RAG + Tool de cálculo
resposta = agente_com_memoria.invoke(
    {"input": "Quais dias posso fazer home office? E se meu salário é R$ 8.000 com 5% de auxílio, qual o valor?"},
    config={"configurable": {"session_id": "minha_sessao"}}
)

# Pergunta de follow-up — o agente lembra do salário da mensagem anterior
resposta = agente_com_memoria.invoke(
    {"input": "E se eu tiver uma redução de R$ 2.000 no salário?"},
    config={"configurable": {"session_id": "minha_sessao"}}
)
```

---

## Aprendizados

- Como construir um pipeline RAG do zero com LangChain
- Diferença entre `Chain` simples e `Agent` com loop de decisão
- Como o agente decide **quando** e **qual** ferramenta usar
- Como adicionar memória persistente por sessão sem alterar a lógica do agente
- Boas práticas de legibilidade: imports próximos ao uso + comentários explicativos
