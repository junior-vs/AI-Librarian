# AI-Librarian

## Documento de Requisitos: Sistema de Organização Automática de Livros por IA

Este documento especifica os requisitos funcionais e não funcionais para o desenvolvimento do sistema automatizado de extração de informações, análise conceitual e organização de arquivos de e-books.

---

## 1. Visão Geral do Sistema

O objetivo do sistema é automatizar o fluxo de leitura, análise e categorização de arquivos digitais de livros (PDFs, EPUBs, etc.), utilizando inteligência artificial para identificar título, autor e tema central, movendo cada arquivo para uma estrutura de diretórios correspondente ao seu conceito principal.

---

## 2. Requisitos Funcionais (RF)

### RF-01: Monitoramento e Entrada de Arquivos

* **RF-01.1:** O sistema deve permitir a especificação de um diretório de entrada (pasta padrão onde novos arquivos serão depositados).
* **RF-01.2:** O sistema deve suportar nativamente formatos **PDF** e **EPUB**.
* **RF-01.3:** O sistema deve identificar e listar todos os arquivos compatíveis presentes na pasta de entrada, permitindo processamento individual ou em lote (*batch*).

### RF-02: Extração de Conteúdo e Leitura

* **RF-02.1:** O sistema deve extrair o texto contido nas primeiras páginas do arquivo (como capa, folha de rosto, sumário e introdução/prefácio).
* **RF-02.2:** O sistema deve contar com mecanismo de **OCR** (Reconhecimento Óptico de Caracteres) para arquivos PDF baseados em imagem ou sem camada de texto editável.
* **RF-02.3:** O sistema deve limitar a quantidade de texto extraída (ex: primeiros 3.000 a 5.000 caracteres ou primeiras 5 páginas) para otimizar o custo e a velocidade da análise por IA.

### RF-03: Análise e Classificação por IA

* **RF-03.1:** O sistema deve enviar a amostra de texto extraída para uma API de Modelo de Linguagem (LLM, ex: Gemini API).
* **RF-03.2:** A IA deve identificar os seguintes metadados obrigatórios:
* **Título do livro** (com validação para evitar títulos cortados ou genéricos).
* **Nome do autor** (se presente no trecho analisado).
* **Resumo conceitual** (1 a 2 frases sintetizando o tema central do livro).
* **Categoria/Gênero principal** (com base em uma lista pré-definida de categorias ou inferida dinamicamente).


* **RF-03.3:** O sistema deve permitir a personalização do Prompt da IA e da lista de categorias permitidas (ex: *Finanças, Ficção, Tecnologia, Psicologia, Saúde, Negócios, Filosofia*).

### RF-04: Organização de Arquivos e Sistema de Arquivos

* **RF-04.1:** O sistema deve verificar a existência da pasta correspondente à categoria retornada pela IA no diretório de destino. Caso não exista, a pasta deve ser criada automaticamente.
* **RF-04.2:** O sistema deve renomear o arquivo original (opcional) seguindo o padrão pré-configurado pelo usuário (ex: `[Título] - [Autor].pdf`).
* **RF-04.3:** O sistema deve mover (ou copiar) o arquivo processado para a pasta da sua respectiva categoria.
* **RF-04.4:** Caso ocorra conflito de nomes (arquivo já existente na pasta de destino), o sistema deve aplicar uma regra de sufixo único (ex: `arquivo_(1).pdf`) para evitar sobrescrita de dados.

### RF-05: Tratamento de Erros e Logs

* **RF-05.1:** O sistema deve mover arquivos corrompidos ou não reconhecidos pela IA para uma pasta específica de erro (`/Erros` ou `/Nao_Classificados`).
* **RF-05.2:** O sistema deve manter um arquivo de log (`log_processamento.json` ou `.csv`) registrando o histórico de execução com: *Nome original do arquivo, Título extraído, Conceito gerado, Categoria atribuída, Timestamp e Status (Sucesso/Falha)*.

---

## 3. Requisitos Não Funcionais (RNF)

### RNF-01: Desempenho e Eficiência

* **RNF-01.1:** O tempo de processamento por livro não deve exceder 10 segundos para arquivos com texto legível e 30 segundos para arquivos que necessitem de OCR.
* **RNF-01.2:** O uso de tokens na API de IA deve ser minimizado, enviando apenas trechos estratégicos do livro (capa, sumário e introdução).

### RNF-02: Confiabilidade e Tolerância a Falhas

* **RNF-02.1:** Falhas na conexão com a API de IA ou limites de taxa (*rate limits*) devem acionar tentativas automáticas (*retry*) com recuo exponencial antes de marcar o arquivo como erro.
* **RNF-02.2:** NENHUM arquivo original pode ser excluído ou corrompido durante o processo de extração e movimentação.

### RNF-03: Segurabilidade e Privacidade

* **RNF-03.1:** As chaves de API do provedor de IA devem ser armazenadas em variáveis de ambiente (`.env`) ou cofres de segredos, nunca codificadas diretamente nos scripts.
* **RNF-03.2:** Somente o trecho estritamente necessário para a classificação deve ser transmitido via rede.

### RNF-04: Usabilidade e Manutenibilidade

* **RNF-04.1:** O código deve ser modularizado (separando a extração de texto, a chamada à API de IA e a gestão de arquivos no sistema operacional).
* **RNF-04.2:** O sistema deve possuir interface de linha de comando (CLI) ou script configurável via arquivo `.env` ou `.json` simples para definição das pastas de origem, destino e chaves.

---

## 4. Matriz de Mapeamento do Processo

| Etapa | Entrada | Processamento | Saída |
| --- | --- | --- | --- |
| **1. Ingestão** | Pasta de Origem | Varredura de diretório (`.pdf`, `.epub`) | Lista de caminhos de arquivos |
| **2. Extração** | Arquivo digital | Leitura de páginas 1 a 5 + OCR (se necessário) | String contendo o texto da amostra |
| **3. Análise IA** | String de texto + Prompt | Chamada de API de IA (LLM) | JSON contendo Título, Conceito e Categoria |
| **4. Destino** | JSON + Arquivo original | Criação de diretório + Movimentação de arquivo | Arquivo organizado na pasta final |
| **5. Registro** | Status da operação | Gravação em arquivo de log | Registro no log de auditoria |
