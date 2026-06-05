# Especificação de Requisitos do Pipeline RAG
**NovaTech — Assistente de IA para Atendimento ao Cliente**

| Campo | Valor |
|---|---|
| Cliente | NovaTech Logística |
| Fornecedor | DB1 Group |
| Versão | 1.0 |
| Data | Junho 2025 |
| Status | Em revisão com cliente |

---

## 1. Introdução e Contexto

Este documento especifica os requisitos funcionais e comportamentais do pipeline RAG (Retrieval-Augmented Generation) do assistente de IA da NovaTech, destinado ao time de atendimento ao cliente. Os requisitos são não técnicos, orientados ao comportamento esperado do sistema, e devem ser testáveis por analista de qualidade.

### 1.1. Problema a resolver

O time de atendimento (45 pessoas) gasta em média 12 minutos por chamado buscando informações em três fontes distintas: SharePoint (~800 documentos), Confluence (~400 páginas) e pasta de rede (planilhas mensais). Isso resulta em respostas inconsistentes, atrasos e 15% de escalonamentos evitáveis. O objetivo é reduzir esse tempo para menos de 2 minutos por chamado.

### 1.2. Como o pipeline RAG funciona (referência)

> **O que é RAG**
> Documentos são divididos em pedaços (chunks), transformados em representações numéricas (embeddings) e armazenados em um banco vetorial. Quando um atendente faz uma pergunta, o sistema recupera os chunks mais relevantes por similaridade semântica e os utiliza como contexto para o modelo de linguagem (LLM) gerar a resposta final. O assistente **nunca responde a partir de conhecimento geral** — apenas a partir do conteúdo indexado.

### 1.3. Base documental identificada

A análise prévia da documentação da NovaTech identificou os seguintes documentos como referência para este projeto:

| ID | Documento | Tipo | Status atual |
|---|---|---|---|
| POL-001 | Política de Devolução de Mercadorias | Normativo | ✅ Vigente — v3.1, jan/2024 |
| PROC-042 v1 | Cálculo de Frete Especial | Procedimento | ⚠️ Sem status formal — ver nota abaixo |
| PROC-042 v2 | Cálculo de Frete Especial (Revisado) | Procedimento | ⚠️ Sem status formal — ver nota abaixo |
| SLA-2024 | Tabela de SLA por Tipo de Cliente | Contratual | ✅ Vigente — v2024.1, jan/2024 |
| FAQ-Atendimento | Perguntas Frequentes do Time de Suporte | Informal | ❌ Não validado — não será indexado (ver RQ-1.5) |

> **⚠️ Nota — PROC-042 v1 e v2:** Nenhuma das duas versões possui status formal de vigência no SharePoint. Ambas serão **bloqueadas da indexação** pelo RQ-1.2 até que a NovaTech defina qual versão é vigente e a marque corretamente. Isso é uma decisão em aberto (ver Seção 9.1, item 3).

---

## 2. Requisitos de Fontes de Dados (RQ-1.x)

Define quais documentos devem ser indexados pelo pipeline, com que critérios, e quais devem ser explicitamente excluídos.

### 2.1. Fontes elegíveis para indexação

| ID | Requisito | Critério de aceite | Observações |
|---|---|---|---|
| RQ-1.1 | O sistema deve indexar documentos provenientes das três fontes oficiais da NovaTech: SharePoint corporativo (PDFs e Word), Confluence (páginas wiki) e pasta de rede (planilhas .xlsx atualizadas mensalmente). | Indexar um documento em cada fonte e confirmar que o assistente o referencia corretamente em uma pergunta relacionada. | Fontes: SharePoint, Confluence e `\\novatech-fs\`. |
| RQ-1.2 | Apenas documentos com metadado de status preenchido como **"Vigente"** devem ser indexados. O sistema deve verificar esse campo antes de indexar qualquer documento. | Tentar indexar documento sem status "Vigente" e confirmar que ele não aparece em nenhuma resposta do assistente. | A curadoria de status é feita manualmente pelos admins antes da publicação. O sistema verifica, mas não define o status. |
| RQ-1.3 | Documentos com status alterado para "Revogado", "Obsoleto" ou "Em revisão" devem ser removidos do banco vetorial em até **60 minutos** após a alteração do status. | Alterar status de um documento indexado para "Revogado" e verificar, após 60 minutos, que ele não aparece em nenhuma resposta. | O prazo de 60 minutos é o SLA máximo. A remoção pode ocorrer antes. Documentos sem status preenchido são tratados como não elegíveis (ver RQ-1.2). |

### 2.2. Documentos excluídos ou restritos

| ID | Requisito | Critério de aceite | Observações |
|---|---|---|---|
| RQ-1.4 | Documentos sem status de vigência preenchido **não devem ser indexados**, mesmo que estejam nas fontes oficiais. O sistema deve gerar um log listando os documentos ignorados por ausência de status. | Publicar documento sem status no SharePoint e confirmar: (a) que não é indexado; (b) que aparece no log de documentos ignorados. | Responsabilidade do admin garantir o preenchimento antes da publicação. |
| RQ-1.5 | O documento FAQ-Atendimento **não deve ser indexado** como fonte de resposta, por ser informal e não validado pelo Compliance ou Operações. | Fazer pergunta cujo conteúdo existe apenas no FAQ (ex: seguro de carga) e confirmar que o assistente retorna a mensagem de ausência de resposta — sem citar o FAQ. | O FAQ contém informações não validadas que podem contradizer documentos oficiais. Indexá-lo introduz risco de resposta incorreta com aparência de credibilidade. |

---

> ### ⚠️ Gaps identificados na base documental
>
> Os tópicos abaixo **não possuem documento formal indexável**. Perguntas sobre eles resultarão em "não encontrei resposta" (ver Seção 5).
>
> | Tópico | Situação atual |
> |---|---|
> | Política de carga danificada em trânsito | Existe apenas no FAQ informal (item 38) — sem documento oficial |
> | Seguro de carga — percentuais e condições | Existe apenas no FAQ informal (item 22) — sem documento oficial |
> | Tabela de fretes padrão (abaixo de 500kg) | Nenhuma cobertura na base atual — PROC-042 cobre apenas acima de 500kg |
> | Procedimento da Gestão de Riscos para cargas perigosas devolvidas | POL-001 menciona o ramal 4500, mas o procedimento não está documentado |
>
> **Recomendação:** A NovaTech deve criar documentos formais para esses tópicos antes ou durante o go-live. Priorizar pelos tópicos de maior volume de chamados (prazos de entrega: 35%, regras de frete: 25%, política de devolução: 20%).

---

## 3. Tratamento de Documentos Contraditórios (RQ-2.x)

A base documental da NovaTech contém contradições identificadas entre versões de um mesmo procedimento. Esta seção define o comportamento esperado do assistente nesses casos.

---

> ### ⚠️ Contradições confirmadas na base atual
>
> As contradições abaixo foram identificadas na análise dos documentos fornecidos pela NovaTech. Todas envolvem PROC-042 v1 (mar/2023) e PROC-042 v2 (nov/2023).
>
> | # | Parâmetro | PROC-042 v1 | PROC-042 v2 |
> |---|---|---|---|
> | C-1 | Multiplicador regional — Sul | 1,2 | 1,3 |
> | C-2 | Multiplicador regional — Centro-Oeste | 1,3 | 1,4 |
> | C-3 | Multiplicador regional — Nordeste | 1,4 | 1,5 |
> | C-4 | Multiplicador regional — Norte | 1,6 | 1,8 |
> | C-5 | Fator de peso — 1.001 a 3.000 kg | 1,2 | 1,15 |
> | C-6 | Fator de peso — acima de 3.000 kg | 1,5 | 1,4 |
> | C-7 | Prazo adicional para frete especial | +2 dias úteis | +3 dias úteis |
>
> **Situação adicional:** O FAQ (item 32) menciona que carga perigosa pode ser enviada com frete expresso "com autorização do Compliance", mas **não existe documento formal** que defina esse processo. Não se trata de contradição entre documentos, mas de informação informal sem respaldo oficial.
>
> **Decisão em aberto:** Nenhuma das duas versões do PROC-042 possui status formal. A NovaTech precisa definir qual é vigente antes do go-live (ver Seção 9.1, item 3). Enquanto isso, ambas ficam bloqueadas da indexação por RQ-1.2.

---

| ID | Requisito | Critério de aceite | Observações |
|---|---|---|---|
| RQ-2.1 | Quando dois ou mais documentos vigentes indexados apresentarem informações divergentes sobre o mesmo assunto, o assistente deve exibir **ambas as versões** na resposta, com identificação clara de cada fonte. Não deve escolher um valor e omitir o outro. | Com PROC-042 v1 e v2 indexados, perguntar sobre o multiplicador regional do Norte. A resposta deve exibir os dois valores (1,6 e 1,8) com a fonte de cada um. | Aplicável a qualquer par de documentos vigentes com conteúdo divergente, não apenas PROC-042. |
| RQ-2.2 | Junto às versões divergentes, o assistente deve exibir um **aviso padrão de conflito** e orientar o atendente a escalar para o responsável pela arbitração antes de informar o cliente. | O aviso de conflito deve aparecer em 100% das respostas onde divergência for detectada. Verificar a resposta sobre multiplicador regional e confirmar presença do aviso. | Texto padrão sugerido: *"Atenção: foram encontradas informações divergentes entre documentos. Consulte o responsável pela arbitração antes de responder ao cliente."* O responsável precisa ser nomeado pela NovaTech antes do go-live (ver Seção 9.1, item 1). |
| RQ-2.3 | O sistema deve registrar em log toda ocorrência de conflito detectado, com: documentos envolvidos, campo divergente e timestamp. O log deve ser acessível aos admins no painel. | Gerar uma consulta que ative conflito e verificar que o log registrou os dois documentos, o campo em conflito e a data/hora. | Conflitos devem ser resolvidos na fonte (um documento revogado ou os dois unificados). O assistente expõe conflitos, não os resolve. |
| RQ-2.4 | Informações presentes apenas em fonte informal (ex: FAQ) e sem correspondência em documento oficial vigente **não devem ser apresentadas como resposta**. O assistente deve tratar esse caso como ausência de resposta (ver Seção 5). | Perguntar sobre "frete expresso para carga perigosa" e confirmar que o assistente não cita o FAQ e retorna a mensagem padrão de ausência de resposta. | Evita que o assistente transmita como oficial uma informação que existe apenas em prática informal não documentada. |

---

## 4. Requisitos de Atualização do Índice (RQ-3.x)

Define o comportamento esperado para ingestão de novos documentos e atualização de existentes, garantindo que o assistente reflita sempre a documentação vigente mais recente.

| ID | Requisito | Critério de aceite | Observações |
|---|---|---|---|
| RQ-3.1 | Quando um admin publicar um documento novo ou atualizado com status "Vigente" e concluir a confirmação dupla (ver RQ-3.2), o conteúdo deve estar disponível no assistente em no máximo **60 segundos**. | Após a confirmação dupla, aguardar 60 segundos e fazer pergunta sobre o conteúdo do documento publicado. O assistente deve referenciá-lo. | O prazo de 60 segundos é o SLA máximo de indexação após a confirmação dupla. Deve ser validado em ambiente de produção com o volume real de documentos. |
| RQ-3.2 | O fluxo de publicação deve exigir confirmação dupla obrigatória: **(1)** admin carrega o documento e preenche metadados (status, área responsável, data); **(2)** sistema exibe prévia dos metadados para revisão; **(3)** admin confirma; **(4)** sistema dispara indexação; **(5)** sistema notifica admin com resultado (sucesso ou erro descritivo). | Executar o fluxo completo e verificar que cada etapa está presente. Tentar indexar sem a confirmação dupla e confirmar bloqueio. | Nenhum documento é indexado sem confirmação dupla. Erros de indexação devem ser descritos em linguagem não técnica para o admin. |
| RQ-3.3 | Quando uma nova versão de documento for publicada como "Vigente" e declarada como substituta da versão anterior — por metadado preenchido pelo admin no ato da publicação — a versão anterior deve ser **removida automaticamente** do índice no momento da indexação da nova versão. | Publicar v2 declarando-a como substituta de v1. Após indexação, confirmar que apenas v2 é referenciada em respostas e que v1 não aparece. | O sistema não infere substituição automaticamente: o admin deve declarar explicitamente qual documento é substituído. Quando duas versões coexistem sem hierarquia declarada (como PROC-042 v1 e v2 atualmente), aplica-se o tratamento de conflito (RQ-2.x). |
| RQ-3.4 | As planilhas da pasta de rede, atualizadas mensalmente, devem seguir o mesmo fluxo de confirmação dupla (RQ-3.2). Apenas admins designados têm permissão para publicar atualizações. | Tentar publicar planilha com perfil de atendente e confirmar bloqueio. Publicar com perfil admin, confirmar dupla confirmação e verificar disponibilidade no assistente dentro do prazo de RQ-3.1. | Apenas a versão atual de cada planilha deve estar indexada. Não há histórico de versões para planilhas. |

---

## 5. Comportamento na Ausência de Resposta (RQ-4.x)

Define como o assistente deve se comportar quando a pergunta do atendente não possui resposta nos documentos indexados.

> **Decisão de produto:** O assistente **não deve** tentar responder com conhecimento geral do LLM quando não encontrar resposta na base. Toda resposta deve ser fundamentada em documento oficial indexado. Na ausência, o assistente informa claramente e orienta o escalonamento.

| ID | Requisito | Critério de aceite | Observações |
|---|---|---|---|
| RQ-4.1 | Quando nenhum documento indexado contiver informação suficiente para responder à pergunta, o assistente deve exibir uma **mensagem padrão** informando que não encontrou resposta na documentação oficial e orientando o escalonamento para o supervisor. | Fazer pergunta sobre tópico ausente da base (ex: seguro de carga). O assistente deve exibir a mensagem padrão sem nenhuma informação factual adicional. | Mensagem padrão sugerida: *"Não encontrei resposta para essa pergunta na documentação oficial da NovaTech. Recomendo escalar para o supervisor ou consultar diretamente a área responsável."* |
| RQ-4.2 | O assistente **nunca** deve utilizar conhecimento geral do LLM para complementar ou substituir informações ausentes na base. Toda afirmação factual na resposta deve ser rastreável a um documento indexado. | Fazer 10 perguntas sobre tópicos ausentes na base e verificar que 10/10 respostas retornam a mensagem padrão, sem nenhuma informação factual adicional. | Respostas com informações não rastreáveis a documentos podem causar erros operacionais e impactos contratuais com clientes. |
| RQ-4.3 | Quando o assistente não encontrar resposta, deve **registrar a pergunta no log de lacunas documentais**, com timestamp e texto exato da pergunta, sem dados pessoais de clientes. O log deve ser acessível aos admins. | Fazer 3 perguntas sem resposta e verificar que o log registrou as 3, com timestamp e texto da pergunta, sem dados pessoais. | O log é insumo para priorizar criação de documentos. Deve ser revisado mensalmente pelos responsáveis de conteúdo de cada área. |
| RQ-4.4 | Respostas parciais — quando o assistente encontrar informação relacionada mas insuficiente para responder com completude — devem ser **identificadas como parciais**, com orientação para que o atendente confirme com a área responsável antes de informar o cliente. Uma resposta é considerada parcial quando cobre o tema geral da pergunta mas não o parâmetro específico consultado (ex: a regra de devolução existe, mas não para aquela categoria de carga). | Fazer pergunta parcialmente coberta pela base. A resposta deve incluir aviso explícito de que a informação é parcial e orientação para confirmação. Verificar que o aviso de "parcial" é visualmente distinto da resposta completa. | Distinguir ausência total (RQ-4.1) de resposta parcial (RQ-4.4) evita que o atendente trate informação incompleta como suficiente. |

---

## 6. Requisitos de Rastreabilidade (RQ-5.x)

Toda resposta do assistente deve ser rastreável à sua fonte. Esta seção define o nível de detalhe exigido na citação de fontes e a forma de exibição ao atendente.

| ID | Requisito | Critério de aceite | Observações |
|---|---|---|---|
| RQ-5.1 | Toda resposta que contenha informação factual deve citar obrigatoriamente: **(a)** nome do documento, **(b)** identificador (ex: POL-001), **(c)** versão, e **(d)** data da última atualização. | Fazer 5 perguntas respondíveis pela base e verificar que 5/5 respostas contêm os quatro atributos. Teste negativo: verificar que a mensagem de "não encontrei" não apresenta fonte falsa. | Sem rastreabilidade, o atendente não consegue validar a informação nem direcionar o cliente à documentação oficial. |
| RQ-5.2 | O assistente deve exibir, junto à resposta gerada, o **trecho original do documento** (chunk recuperado) que fundamenta a resposta, visualmente destacado em relação ao texto sintetizado. | Verificar que a resposta exibe dois elementos distintos: (1) resposta sintetizada; (2) trecho do documento original destacado. Os dois elementos não devem ser misturados num único bloco de texto. | Especialmente crítico em respostas sobre SLA contratual e regras de cálculo, onde o atendente precisa verificar o contexto exato. |
| RQ-5.3 | A citação de fonte deve incluir um **link clicável** para o documento original no SharePoint ou Confluence, abrindo diretamente na página ou seção relevante quando possível. | Clicar no link de fonte em uma resposta e confirmar que leva ao documento correto. Verificar que links de documentos removidos da base não são mais exibidos nas respostas. | Para planilhas da pasta de rede, exibir o caminho de rede (`\\novatech-fs\...`) em vez de link clicável. |
| RQ-5.4 | Quando a resposta for fundamentada em mais de um documento, **todas as fontes devem ser citadas individualmente**, com indicação de qual parte da resposta vem de cada fonte. | Fazer pergunta que exige cruzamento de dois documentos (ex: prazo de devolução + SLA de atendimento). Verificar que ambos os documentos são citados e que cada trecho da resposta indica sua origem. | Respostas com múltiplas fontes devem deixar claro qual informação vem de cada documento, especialmente quando as fontes têm hierarquias diferentes (ex: normativo vs. contratual). |

---

## 7. Requisitos Complementares (RQ-6.x)

### 7.1. Gestão de conteúdo e curadoria

| ID | Requisito | Critério de aceite | Observações |
|---|---|---|---|
| RQ-6.1 | O sistema deve disponibilizar um **painel de administração** onde admins autorizados possam: visualizar documentos indexados com metadados, alterar status, publicar novos documentos e consultar os logs de lacunas e conflitos. | Admin acessa o painel, visualiza a lista de documentos indexados, altera status de um documento para "Revogado" e confirma remoção do índice em até 60 minutos. | O painel deve ser acessível via Microsoft Teams ou como aplicação web integrada ao Azure AD, aproveitando as licenças M365 E3 da NovaTech. |
| RQ-6.2 | Apenas usuários com perfil **"Admin"** podem publicar, atualizar ou remover documentos do índice. Atendentes têm perfil somente leitura. | Tentar publicar documento com perfil de atendente e confirmar bloqueio com mensagem de erro. Confirmar que perfil admin executa todas as operações sem bloqueio. | Perfis gerenciados via Azure Active Directory (AAD). |

### 7.2. Acesso e visibilidade

| ID | Requisito | Critério de aceite | Observações |
|---|---|---|---|
| RQ-6.3 | Todos os 45 atendentes têm acesso a **todos os documentos indexados**, sem restrição por perfil ou área. | Dois atendentes com perfis diferentes fazem a mesma pergunta e recebem a mesma resposta com as mesmas fontes. | Decisão da NovaTech: sem controle de acesso por conteúdo nesta versão. Rever se documentos sensíveis forem incluídos em versões futuras. |

### 7.3. Auditoria e qualidade

| ID | Requisito | Critério de aceite | Observações |
|---|---|---|---|
| RQ-6.4 | O sistema deve registrar todas as interações (pergunta, resposta, fontes citadas, timestamp, identificador anonimizado do atendente) em **log de auditoria**, retido por no mínimo 90 dias. O identificador anonimizado deve ser um hash não reversível — sem nome, matrícula ou e-mail do atendente. | Realizar 10 consultas e verificar que o log registrou todas com os campos obrigatórios. Confirmar que nenhum campo de identificação pessoal (nome, e-mail, matrícula) está presente no log. | O hash não reversível garante conformidade com LGPD ao impossibilitar reidentificação do atendente a partir do log. |
| RQ-6.5 | O assistente deve exibir, em toda resposta, um **mecanismo de feedback** ("Esta resposta foi útil?") que permita ao atendente indicar se a resposta resolveu sua dúvida. O feedback deve ser registrado vinculado à interação. | Responder "não" no mecanismo de feedback e verificar que o registro aparece no log com a pergunta e resposta associadas. O feedback negativo deve ser acessível aos admins no painel. | Dados de feedback são insumo para priorizar melhorias na base documental e no pipeline. |

---

## 8. Consolidado de Requisitos e Testabilidade

| ID | Descrição resumida | Categoria | Tipo de teste |
|---|---|---|---|
| RQ-1.1 | Indexar as três fontes oficiais | Fontes de dados | Funcional — ingestão |
| RQ-1.2 | Apenas documentos vigentes são indexados | Fontes de dados | Funcional — filtro de status |
| RQ-1.3 | Documentos revogados removidos em até 60 min | Fontes de dados | Funcional + tempo de resposta |
| RQ-1.4 | Sem status = não indexado + log gerado | Fontes de dados | Funcional — log |
| RQ-1.5 | FAQ informal não indexado como fonte | Fontes de dados | Funcional — blacklist |
| RQ-2.1 | Exibir ambas versões em caso de conflito | Contradições | Funcional — resposta |
| RQ-2.2 | Aviso de conflito em 100% dos casos | Contradições | Funcional — UI |
| RQ-2.3 | Log de conflitos acessível a admins | Contradições | Funcional — log |
| RQ-2.4 | Fonte informal não exibida como resposta | Contradições | Funcional — blacklist |
| RQ-3.1 | Documento disponível em até 60s após publicação | Atualização | Funcional + latência |
| RQ-3.2 | Fluxo de confirmação dupla obrigatório | Atualização | Funcional — fluxo |
| RQ-3.3 | Versão anterior removida apenas quando substituição é declarada pelo admin | Atualização | Funcional — versionamento |
| RQ-3.4 | Planilhas seguem mesmo fluxo de publicação | Atualização | Funcional — permissão |
| RQ-4.1 | Mensagem padrão quando sem resposta | Ausência | Funcional — resposta |
| RQ-4.2 | LLM não usa conhecimento geral | Ausência | Funcional — confiabilidade |
| RQ-4.3 | Log de lacunas documentais | Ausência | Funcional — log |
| RQ-4.4 | Respostas parciais identificadas com critério definido | Ausência | Funcional — UI |
| RQ-5.1 | Toda resposta cita nome, ID, versão e data | Rastreabilidade | Funcional — UI |
| RQ-5.2 | Trecho original destacado junto à resposta | Rastreabilidade | Funcional — UI |
| RQ-5.3 | Link clicável para o documento original | Rastreabilidade | Funcional — navegação |
| RQ-5.4 | Múltiplas fontes citadas individualmente | Rastreabilidade | Funcional — UI |
| RQ-6.1 | Painel de admin com todas as operações | Gestão | Funcional — admin |
| RQ-6.2 | Apenas admin publica documentos | Gestão | Segurança — controle de acesso |
| RQ-6.3 | Todos os atendentes veem todos os docs | Acesso | Funcional — permissão |
| RQ-6.4 | Log de auditoria por 90 dias, hash não reversível | Auditoria | Funcional — log + LGPD |
| RQ-6.5 | Mecanismo de feedback em toda resposta | Auditoria | Funcional — UI |

---

## 9. Decisões em Aberto e Recomendações

### 9.1. Decisões que precisam ser tomadas antes do go-live

| # | Decisão em aberto | Impacto se não resolvida | Responsável sugerido |
|---|---|---|---|
| 1 | Nomear o responsável por arbitrar conflitos documentais e definir o SLA de resolução (sugerido: 5 dias úteis). | Conflitos exibidos pelo assistente ficam sem resolução. O atendente continua sem resposta definitiva. | Diretoria de Operações / NovaTech |
| 2 | Criar documentos formais para os 4 gaps identificados (carga danificada, seguro, frete padrão, procedimento da Gestão de Riscos). | ~15% das perguntas não terão resposta no assistente mesmo após o go-live. | Operações + Compliance + Comercial |
| 3 | Criar o campo/metadado de status no SharePoint e Confluence e executar curadoria inicial dos ~1.200 documentos existentes — incluindo definir qual versão do PROC-042 é vigente. | Sem curadoria, nenhum documento é indexado (RQ-1.2). O assistente vai ao ar sem conteúdo. | TI + áreas responsáveis / NovaTech |
| 4 | Nomear admins responsáveis por cada área de conteúdo (Operações, Compliance, Comercial) para gestão contínua do índice. | Documentos ficarão desatualizados. O assistente passará a gerar respostas defasadas. | RH + Diretoria / NovaTech |

### 9.2. Recomendações para versões futuras

- **Controle de acesso por conteúdo:** se documentos sensíveis (ex: margens comerciais) forem incluídos no índice, implementar segmentação por perfil de usuário.
- **Processo unificado de revisão documental:** as três áreas publicam sem processo unificado, o que gera conflitos antes mesmo de chegar ao índice. Recomenda-se um workflow de aprovação antes da publicação.
- **Monitoramento de qualidade:** cruzar mensalmente o log de lacunas (RQ-4.3) com o log de feedback negativo (RQ-6.5) para priorizar criação de documentos e melhorias no pipeline.

---

*Documento preparado por DB1 Group para NovaTech Logística — Junho 2025*
*Classificação: Confidencial — não distribuir externamente*
