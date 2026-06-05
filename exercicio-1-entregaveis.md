# Etapas


## Etapa 1 — Visão geral:
Forneça ao Claude apenas os títulos, metadados e resumos dos 5 documentos (não o conteúdo completo). Peça um mapa de temas cobertos e hipóteses de gaps.

### Prompt: ###
Imagine que você é um UX Designer ou analista de negócio na fase inicial de um projeto de consultoria. Antes de tirar dúvidas com os stakeholders, você precisa analisar a documentação sobre o negócio do cliente Novatech para levantar dúvidas e hipóteses.

Contexto: NovaTech é uma empresa de médio porte do setor de logística com 1.200 funcionários. Sua operação depende de um conjunto extenso de documentação interna, e equipe de atendimento ao cliente gasta muito tempo buscando informações nessas fontes para responder dúvidas de clientes sobre prazos, regras de frete, políticas de devolução e procedimentos de reclamação. O projeto irá construir um assistente de IA que permita aos atendentes fazer perguntas em linguagem natural e receber respostas fundamentadas na documentação oficial da empresa, com indicação da fonte.

Estou enviando um resumo do que há em cada documento. Identifique problemas graves, contradições e informações que faltam. Gere uma lista em texto de hipóteses de gaps.

Também crie um diagrama mermaid, estilo mapa mental, dos temas identificados. O diagrama deve ser objetivo, e cada tema pode ter subtemas ou estar relacionado a outros temas. Cada tema deve dizer quais arquivos incluem esse tema (exemplo de tema: "Prazo de entrega (FAQ; POL-001)

### Prompts adicionais ###

não consigo visualizar direito o diagrama. me de um arquivo separado para download ou crie em imagem

tem vários cards do diagrama empilhados um em cima do outro. arrume isso. me devolva apenas o svg

 ### Estratégia ###
Dei contexto sobre o projeto, solicitei um resultado específico e em formatos específicos. Expliquei que estava enviando o resumo dos documentos para que o claude soubesse que não era o cenário completo. 

### Output ### 

Hipóteses de gaps identificados
1. Versionamento e governança documental

PROC-042 v1 e v2 coexistem no SharePoint sem hierarquia formal declarada — nenhum dos dois indica explicitamente que substitui o outro. Um assistente de IA sem regra de desambiguação pode servir respostas de versões diferentes para perguntas similares.
O FAQ é um documento colaborativo sem responsável formal, sem validação por Compliance ou Operações, e acumulado ao longo de 2 anos. Há risco alto de que respostas do FAQ contradigam documentos normativos oficiais — especialmente POL-001 e SLA-2024.
Não há evidência de um índice mestre ou matriz de vigência dos documentos. O assistente precisará de uma camada de priorização explícita (qual documento prevalece em conflito?).

2. Contradições potenciais entre documentos

O FAQ responde sobre frete especial (item 8) sem necessariamente estar alinhado com a PROC-042 v2, que introduziu novos multiplicadores e prazos em novembro/2023.
O FAQ menciona frete expresso com carga perigosa (item 32), mas a POL-001 trata cargas perigosas como exceção ao prazo geral de devolução. Não está claro se existe um procedimento dedicado (a PROC-042 referencia uma PROC-043 para cargas perigosas — esse documento existe no acervo?).
O item 15 do FAQ menciona um tier "Platinum" que não consta no SLA-2024, que define apenas Gold, Silver e Standard. Isso pode indicar um tier descontinuado, um erro do FAQ, ou um contrato especial não documentado aqui.

3. Lacunas de cobertura temática

Seguro de carga (FAQ item 22) não tem documento de origem identificado nos arquivos enviados. Se o assistente for perguntado sobre isso, não terá base documental oficial para responder.
Rastreamento/tracking (FAQ item 27) também não aparece em nenhum dos documentos formais — pode existir um procedimento operacional separado não incluído neste conjunto.
Política de desconto no frete (FAQ item 45) só aparece na PROC-042 v2 como desconto por volume. O FAQ sugere que atendentes já dão (ou não dão) descontos de forma ad hoc — há risco de inconsistência.
Não há documento sobre reclamação formal ou escalonamento de incidentes — o SLA-2024 menciona incidentes críticos e penalidades, mas não um fluxo de abertura de chamado do lado do cliente.

4. Ambiguidade no escopo de aplicação

A POL-001 define prazo geral de 7 dias úteis, mas as exceções (cargas perigosas, cadeia de frio, lacres violados) não têm seus próprios procedimentos detalhados nos documentos disponíveis.
A PROC-042 v2 tem disposições transitórias para chamados abertos antes de 01/12/2023 — isso provavelmente já não é relevante operacionalmente, mas pode confundir um modelo de IA se não houver instrução explícita de ignorar regras transitórias expiradas.
O SLA-2024 classifica clientes por volume mensal e valor de contrato, mas não está claro se atendentes têm acesso fácil a essa classificação no momento do atendimento — o assistente precisará saber se deve inferir o tier ou consultá-lo de outra fonte.

5. Riscos específicos para o assistente de IA

Com dois PROC-042 ativos e um FAQ não validado, o modelo pode produzir respostas conflitantes dependendo de qual chunk recuperar — isso precisa de uma política de precedência documentada antes do desenvolvimento.
Ausência de documentação para PROC-043 (cargas perigosas) é um gap crítico: é um dos temas mais sensíveis operacional e legalmente, e aparece em pelo menos 3 perguntas do FAQ.
Nenhum documento aborda o que o atendente deve fazer quando a resposta não existe na documentação — isso é relevante para o design do assistente (comportamento de fallback).

+ mapa em svg

## Etapa 2 — Análise profunda:
 Com base no mapa da etapa 1, selecione os 2 documentos que mais precisam de análise (ex: os dois PROC-042 contraditórios). Forneça o conteúdo completo apenas desses 2 e peça análise de inconsistências.

 ### Prompt: ###
 Agora estou te enviando os arquivos completos do PROC-042 v1 e PROC-042 v2. Liste as inconsistências entre os arquivos, e em qual título ou subtítulo a inconsistencia está. Exemplo: "Multiplicadores maiores na v2 (2.1. Multiplicadores regionais)"

 ### Estratégia ###
No mapa de risco esses eram os arquivos com mais itens problemáticos. Solicitei que fizesse referencia a qual seção do arquivo o problema se encontra, para garantir rastreabilidade. 

### Output ###

Com base nos dois documentos completos:

**Inconsistências entre PROC-042 v1 e PROC-042 v2**

---

**Multiplicadores regionais maiores na v2 (2.1. Multiplicadores regionais)**
Sul 1.2→1.3 · Sudeste 1.0→1.1 · Centro-Oeste 1.3→1.4 · Nordeste 1.4→1.5 · Norte 1.6→1.8. Sem hierarquia formal entre as versões, o assistente pode usar o multiplicador errado dependendo de qual chunk recuperar.

**Fatores de peso reduzidos na v2 (2. Fórmula de cálculo)**
Faixa 1.001–3.000kg: 1.2→1.15. Faixa acima de 3.000kg: 1.5→1.4. A fórmula base é idêntica, mas os fatores produzem valores diferentes para o mesmo pedido.

**Prazo adicional aumentou na v2 (3. Prazo de entrega para frete especial)**
v1 define +2 dias úteis; v2 define +3 dias úteis. A v2 menciona explicitamente a mudança, mas sem revogar a v1 — ambas permanecem ativas no SharePoint.

**Threshold e estrutura de desconto por volume divergem (4. Condições especiais)**
v1 exige mais de 10 fretes/mês para qualquer desconto, e o valor é negociado caso a caso em aditivo contratual. v2 cria uma tabela automática: 5% a partir de 8 fretes/mês e 10% acima de 15 fretes/mês, com descontos maiores dependendo de aprovação da Diretoria Comercial. São regras estruturalmente incompatíveis.

**Status da PROC-043 diverge (4. Condições especiais)**
v1 referencia a PROC-043 como documento vigente, sem ressalvas. v2 acrescenta que a PROC-043 "está em processo de revisão pelo Compliance e pode sofrer alterações" — informação crítica ausente na v1, e que nenhum dos dois documentos resolve.

**Regra transitória sem data de encerramento (5. Disposições transitórias)**
Exclusiva da v2. Define que chamados abertos antes de 01/12/2023 usam multiplicadores da v1. Essa data já passou, mas a regra nunca foi formalmente encerrada ou removida — risco de o modelo de IA interpretá-la como ainda aplicável.





## Etapa 3 — Cruzamento: ## 
Forneça ao Claude o output das etapas 1 e 2 junto com o FAQ-Atendimento completo. Peça que cruze as inconsistências encontradas com as práticas informais do FAQ.

### Prompt: ###
Cruze os dados dessas inconsistências identificadas, no mapa em anexo, e no FAQ em anexo. 
Então, liste resumidamente os principais riscos ao projeto (3 a 5) e inconsistências graves (2 a 4).
O arquivo do FAQ contextualiza as principais dúvidas dos clientes da Novatech. Ele serve para identificar como a inconsistência de dados pode afetar as respostas dos atendentes da Novatech, e também para identificar quais temas geram mais dúvidas.

### Estratégia ###

Descrevi em etapas o que deveria ser feito, para garantir que todos os arquivos fossem analisados. Contextualizei o motivo do arquivo do FAQ estar na análise para que essa fosse a "âncora" assim como seria numa análise humana. Pedi um número específico de riscos para limitar a resposta. 

### Output ### 

Vou ler o SVG do mapa antes de cruzar os dados.Tenho tudo que preciso. Vou cruzar agora.

---

#### Principais riscos ao projeto

**1. O FAQ é a principal fonte de atendimento hoje — e está errado em pontos críticos**
O FAQ é o documento mais consultado no dia a dia (itens 8, 41, 45 mostram isso), mas não foi validado contra nenhum normativo. Para o assistente de IA, isso significa que a base de conhecimento mais "viva" do negócio é também a mais contaminada. Se o FAQ for indexado sem tratamento, o modelo vai aprender e reproduzir erros como regra.

**2. Frete especial é o tema mais dúvida e tem a inconsistência mais grave**
O FAQ item 8 orienta usar a v2 "na dúvida", mas reconhece que contratos antigos podem seguir a v1. O problema: os multiplicadores e fatores de peso diferem entre versões, e o assistente não tem como saber qual versão se aplica a cada cliente sem acesso ao dado contratual. Toda pergunta sobre valor de frete tem chance real de gerar uma resposta numericamente errada.

**3. Temas sem cobertura documental aparecem no FAQ com respostas inventadas**
Seguro de carga (item 22) e rastreamento (item 27) têm respostas detalhadas no FAQ — inclusive com valores percentuais (0,3% e 0,8%) e critérios de priorização — mas não existem em nenhum documento oficial disponível. O assistente, sem fonte para citar, ou vai se recusar a responder, ou vai usar o FAQ como fonte e apresentar dado não validado como oficial.

**4. Ausência de regra de precedência entre documentos**
O mapa identificou que não há índice de vigência. O FAQ agrava isso: o item 45 orienta o atendente a ver a PROC-042 para descontos, mas a PROC-042 tem duas versões com regras incompatíveis (threshold de 10 vs. 8 fretes, desconto negociado vs. automático). Sem hierarquia declarada, o comportamento do assistente será não-determinístico para o mesmo tipo de pergunta.

**5. Comportamento de fallback não definido**
O FAQ resolve vários gaps com orientações informais ("liga no ramal 4500", "encaminha para sinistros@novatech.com.br"). O assistente precisa de uma política explícita para quando não houver resposta na documentação oficial — caso contrário, ou vai travar, ou vai usar o FAQ como fallback silencioso, sem sinalizar ao atendente que a resposta não tem base normativa.

---

#### Inconsistências graves

**Desconto por volume: regras incompatíveis entre FAQ e PROC-042 v2 (FAQ item 45 × PROC-042 v1 e v2)**
O FAQ diz que o desconto automático começa com mais de 10 fretes/mês — alinhado com a v1, que já está desatualizada. A v2 reduziu o threshold para 8 fretes e criou uma tabela automática (5% e 10%). O atendente usando o FAQ está aplicando a regra errada sem saber, e o assistente pode fazer o mesmo.

**SLA de atendimento: números corretos, mas fonte errada (FAQ item 41 × SLA-2024)**
Os tempos citados no FAQ (Gold: 2h/24h; Silver: 4h/48h; Standard: 8h/72h) coincidem com o SLA-2024, o que é positivo. O risco está no complemento: o FAQ não menciona penalidades por descumprimento nem a definição de incidente crítico — que têm prazos menores e são contratualmente vinculantes. Um atendente orientado só pelo FAQ pode tratar um incidente crítico com o prazo errado.

**Carga perigosa: procedimento referenciado está em revisão, mas o FAQ trata como vigente (FAQ item 32 × PROC-042 v2)**
O FAQ item 32 orienta que carga perigosa com frete expresso precisa de autorização do Compliance e documentação ANTT atualizada — sem ressalvas. A PROC-042 v2 alerta explicitamente que a PROC-043 (que rege exatamente esse fluxo) está em revisão pelo Compliance. O atendente está passando ao cliente uma orientação baseada em um procedimento que pode mudar ou já ter mudado.

**Carga danificada: prazo de 48h no FAQ não tem respaldo em nenhum documento disponível (FAQ item 38)**
O FAQ define um prazo específico de 48h para registro de ocorrência por dano em trânsito, com instrução de encaminhar ao Jurídico. Esse fluxo não aparece na POL-001 nem em nenhum outro documento do acervo analisado. É um procedimento inteiro operando sem base normativa formal — e é exatamente o tipo de situação que gera risco legal se o cliente não for orientado corretamente.



## Análise dos resultados ##

O resultado foi bem detalhista, a nível documental e não de negócio. Os exemplos são úteis para conversar com stakeholders, mas faltou um pensamento mais big picture. 

Um prompt ou skill especificando melhor o que precisava nessa fase de discovery teria ajudado mais. 

Se tivesse adicionado todos os documentos de uma vez, mas gastaria muitos tokens e seria muito mais difícil de direcionar os resultados, precisando de diversas revisões. Também seria mais difícil confiar nos resultados.


# Riscos identificados

Para o discovery, eu levaria esses riscos:

**1. O FAQ tem respostas inventadas.** Isso significa que a base de conhecimento mais "viva" do negócio é também a mais contaminada. 

**2. Respostas incorretas ou variáveis para o mesmo tipo de pergunta** As informações contraditórias, falta de índice de vigência e a falta de regras claras sobre versionamento podem gerar uma resposta numericamente errada. 

**3. Fallback hoje é humano** O FAQ resolve vários gaps com orientações informais ("liga no ramal 4500", "encaminha para sinistros@novatech.com.br"). O assistente precisa de uma política explícita para quando não houver resposta na documentação oficial — caso contrário, ou vai travar, ou vai usar o FAQ como fallback silencioso, sem sinalizar ao atendente que a resposta não tem base normativa.


Transformando em perguntas e sugestões como 

"Quando e como essas inconsistências ocorrem?"
"Precisamos revisar as inconsistências, criar versionamento e definir fallback"
"Como ao time da Novatech irá garantir que as informações fiquem atualizadas?"