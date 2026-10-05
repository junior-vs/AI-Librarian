# AI-Librarian

Organização automática de e-books (PDF/EPUB) com apoio de IA: identifica título, autor, resumo e categoria, e move cada arquivo para a pasta do seu tema principal, com revisão, rastreabilidade e reversão.

---

## 1. Visão do Produto

### 1.1 Problema
Bibliotecas digitais pessoais crescem sem padrão: nomes inconsistentes, duplicatas e nenhuma estrutura temática. A triagem manual é lenta e raramente é concluída.

### 1.2 Persona
Leitor individual com acervo local (centenas a milhares de arquivos), em português e inglês, que quer organização sem perder controle sobre seus arquivos.

### 1.3 Métricas de Sucesso
| Métrica | Meta |
| --- | --- |
| Classificações corretas (amostra auditada) | ≥ 90% |
| Arquivos processados sem intervenção | ≥ 70% |
| Arquivos perdidos ou corrompidos | 0 |
| Redução do tempo de triagem manual | ≥ 80% |

### 1.4 Fora de Escopo (MVP)
OCR, deduplicação, enriquecimento externo, monitoramento contínuo, formatos além de PDF/EPUB e LLM local.

---

## 2. Roadmap

| Fase | Escopo |
| --- | --- |
| **MVP** | CLI, PDF/EPUB com texto, metadados embutidos, taxonomia fixa, confiança, dry-run, fila de revisão, log SQLite, undo |
| **R2** | Enriquecimento por ISBN/Open Library, deduplicação, OCR |
| **R3** | Monitoramento contínuo (watcher), LLM local, subcategorias e tags, formatos adicionais (MOBI, AZW3, DJVU, CBZ) |

---

## 3. Requisitos Funcionais

Prioridade: **M** = Must, **S** = Should, **C** = Could.

### RF-01: Entrada de Arquivos
| ID | Requisito | Fase | Prior. |
| --- | --- | --- | --- |
| RF-01.1 | Configurar diretório de entrada e de destino. | MVP | M |
| RF-01.2 | Suportar PDF e EPUB. | MVP | M |
| RF-01.3 | Listar arquivos compatíveis e processar individualmente ou em lote. | MVP | M |
| RF-01.4 | Modo *watcher*: processar automaticamente arquivos depositados na pasta. | R3 | C |

### RF-02: Extração
| ID | Requisito | Fase | Prior. |
| --- | --- | --- | --- |
| RF-02.1 | Ler primeiro os metadados embutidos (EPUB OPF, PDF info). Se título e autor forem confiáveis, dispensar ou reduzir a chamada à IA. | MVP | M |
| RF-02.2 | Extrair texto das primeiras páginas (capa, folha de rosto, sumário, prefácio), limitado a 3.000–5.000 caracteres ou 5 páginas. | MVP | M |
| RF-02.3 | Detectar PDF sem camada de texto e encaminhá-lo à revisão (até a entrega do OCR). | MVP | M |
| RF-02.4 | Aplicar OCR a PDFs baseados em imagem. | R2 | S |
| RF-02.5 | Detectar o idioma do livro. | MVP | S |

### RF-03: Classificação por IA
| ID | Requisito | Fase | Prior. |
| --- | --- | --- | --- |
| RF-03.1 | Enviar a amostra a um LLM e receber JSON estruturado. | MVP | M |
| RF-03.2 | Retornar: título, autor, resumo (1–2 frases), categoria primária, tags opcionais, idioma e **score de confiança (0–1)**. | MVP | M |
| RF-03.3 | A categoria deve pertencer à **taxonomia fixa** configurável. Categoria inexistente gera *sugestão de nova categoria*, que só é criada após aprovação do usuário. | MVP | M |
| RF-03.4 | Prompt e taxonomia personalizáveis. A taxonomia define idioma dos nomes de categoria (padrão: português). | MVP | M |
| RF-03.5 | Livros multidisciplinares: uma categoria primária (define a pasta) e tags secundárias (gravadas no log/metadados). | MVP | S |
| RF-03.6 | Validar título (rejeitar cortados ou genéricos). | MVP | M |
| RF-03.7 | Suportar múltiplos provedores de LLM (ex.: Gemini) via interface única. | MVP | S |
| RF-03.8 | Suportar LLM local (ex.: Ollama). | R3 | C |

### RF-04: Revisão e Segurança Operacional
| ID | Requisito | Fase | Prior. |
| --- | --- | --- | --- |
| RF-04.1 | **Dry-run**: simular o processamento, exibindo o plano de movimentações sem alterar arquivos. | MVP | M |
| RF-04.2 | Confiança ≥ limiar configurável (padrão 0,8): movimentação automática. Abaixo: vai para `/Revisao`. | MVP | M |
| RF-04.3 | Comando de revisão: aprovar, editar ou rejeitar classificações pendentes. | MVP | M |
| RF-04.4 | **Undo**: reverter movimentações individuais ou por lote com base no log. | MVP | M |

### RF-05: Organização de Arquivos
| ID | Requisito | Fase | Prior. |
| --- | --- | --- | --- |
| RF-05.1 | Criar a pasta da categoria se não existir. | MVP | M |
| RF-05.2 | Renomear conforme padrão configurável (ex.: `[Título] - [Autor].ext`), com sanitização de caracteres inválidos. | MVP | S |
| RF-05.3 | Mover ou copiar (configurável) para a pasta da categoria. | MVP | M |
| RF-05.4 | Conflito de nomes: sufixo único (`arquivo_(1).pdf`), sem sobrescrita. | MVP | M |
| RF-05.5 | Subcategorias (ex.: `Tecnologia/Java`). | R3 | C |

### RF-06: Enriquecimento e Deduplicação
| ID | Requisito | Fase | Prior. |
| --- | --- | --- | --- |
| RF-06.1 | Validar/completar metadados via ISBN (Open Library / Google Books). | R2 | S |
| RF-06.2 | Calcular hash (SHA-256) de cada arquivo e detectar duplicatas exatas. | R2 | S |
| RF-06.3 | Detectar prováveis duplicatas (mesmo título/autor, formato ou edição diferente) e sinalizar para revisão. | R2 | S |

### RF-07: Erros e Auditoria
| ID | Requisito | Fase | Prior. |
| --- | --- | --- | --- |
| RF-07.1 | Arquivos corrompidos ou ilegíveis vão para `/Erros`; não classificados, para `/Nao_Classificados`. | MVP | M |
| RF-07.2 | Registrar em **SQLite**: caminho original, caminho final, hash, título, autor, resumo, categoria, tags, idioma, confiança, provedor/modelo, timestamp e status (Sucesso / Revisão / Falha). | MVP | M |
| RF-07.3 | Exportar o log para CSV/JSON sob demanda. | MVP | S |

---

## 4. Requisitos Não Funcionais

### RNF-01: Desempenho e Custo
* **RNF-01.1:** Vazão-alvo: lote de 100 livros com texto legível em até 15 minutos (sujeito a latência da API). Meta indicativa por livro: ≤ 10 s (texto) e ≤ 30 s (OCR).
* **RNF-01.2:** Minimizar tokens: usar metadados embutidos antes do LLM e enviar apenas trechos estratégicos.
* **RNF-01.3:** Respeitar *rate limits* com fila e concorrência configurável.

### RNF-02: Confiabilidade
* **RNF-02.1:** Falhas de API ou rate limit acionam *retry* com recuo exponencial antes de marcar erro.
* **RNF-02.2:** Nenhum arquivo original pode ser excluído ou corrompido; toda movimentação é registrada e reversível (RF-04.4).
* **RNF-02.3:** Processamento idempotente: reprocessar a mesma pasta não duplica nem reclassifica arquivos já organizados (via hash/log).
* **RNF-02.4:** Interrupções não deixam estado inconsistente (movimentação e log atômicos).

### RNF-03: Segurança e Privacidade
* **RNF-03.1:** Chaves de API em variáveis de ambiente (`.env`) ou cofre de segredos; nunca no código ou no log.
* **RNF-03.2:** Apenas o trecho mínimo necessário é enviado a terceiros. Informar no README que trechos de obras (possivelmente protegidas por direitos autorais) são transmitidos ao provedor de IA escolhido.
* **RNF-03.3:** Opção de LLM local para processamento sem saída de dados (R3).
* **RNF-03.4:** Modo "somente metadados" (sem envio de texto ao LLM) quando os metadados embutidos forem suficientes.

### RNF-04: Usabilidade e Manutenibilidade
* **RNF-04.1:** Código modular: extração, cliente de IA, regras de classificação, gestão de arquivos e persistência separados.
* **RNF-04.2:** CLI com comandos `scan`, `plan` (dry-run), `run`, `review`, `undo`, `export`.
* **RNF-04.3:** Configuração via `.env` e arquivo `config.json/yaml` (pastas, taxonomia, prompt, limiar, padrão de nome, provedor).
* **RNF-04.4:** Testes automatizados com conjunto de livros de referência (*golden set*) para medir a acurácia da classificação.

---

## 5. Fluxo do Processo

| Etapa | Entrada | Processamento | Saída |
| --- | --- | --- | --- |
| **1. Ingestão** | Pasta de origem | Varredura (`.pdf`, `.epub`) + hash | Lista de arquivos |
| **2. Metadados** | Arquivo | Leitura de metadados embutidos | Título/autor candidatos |
| **3. Extração** | Arquivo | Texto das primeiras páginas (OCR em R2) | Amostra de texto |
| **4. Análise IA** | Amostra + prompt + taxonomia | Chamada ao LLM (com retry) | JSON: metadados, categoria, tags, confiança |
| **5. Decisão** | JSON + limiar | Confiança alta → auto; baixa → `/Revisao` | Plano de movimentação |
| **6. Destino** | Plano (ou aprovação) | Criação de pasta, renomeação, movimentação | Arquivo organizado |
| **7. Registro** | Resultado | Gravação no SQLite | Log auditável e reversível |

---

## 6. Critérios de Aceite do MVP

- `plan` lista todas as movimentações sem alterar nenhum arquivo.
- Arquivos com confiança abaixo do limiar nunca são movidos para categorias finais sem aprovação.
- `undo` restaura 100% dos arquivos de um lote ao local original.
- Reexecutar `run` sobre a mesma pasta não gera duplicatas nem reprocessamento.
- Nenhuma chave de API aparece em código, log ou saída da CLI.
- Acurácia ≥ 90% no *golden set* definido.
