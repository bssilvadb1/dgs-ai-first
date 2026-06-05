
## Fluxo do Atendente — Assistente de IA NovaTech

---

### 1. Fluxo Principal (Cenário Feliz)

O atendente recebe uma dúvida do cliente durante o chamado e, sem sair do Teams, abre o assistente de IA. Ele digita a pergunta em linguagem natural — por exemplo, "qual o prazo de entrega para cliente Platinum no modal rodoviário?" O assistente processa a pergunta, localiza a informação na documentação oficial e devolve uma resposta objetiva acompanhada da fonte (nome do documento, versão e área responsável).

O atendente lê a resposta, avalia se ela responde à dúvida do cliente e a utiliza no atendimento. O chamado segue normalmente. Ao final, o atendente tem a opção de marcar a resposta como útil, o que alimenta o histórico de interações.

---

### 2. Fluxo de Fallback

**2a. O assistente não encontra resposta**

O assistente informa explicitamente ao atendente que não localizou nenhuma informação sobre o tema na base de documentos disponível. Ele não tenta inferir nem completar com informações externas. O atendente então tem duas opções: reformular a pergunta com outros termos, ou escalar diretamente para o supervisor via chat no Teams — o assistente oferece um atalho para iniciar essa conversa sem sair da interface.

**2b. O atendente considera a resposta confusa ou incompleta**

O atendente sinaliza ao assistente que a resposta não ficou clara. O assistente reapresenta a informação de forma estruturada, desta vez exibindo o trecho exato do documento de origem, com indicação do título, seção e data de atualização. Se ainda assim o atendente não conseguir resolver, o assistente oferece o atalho para escalonamento com o supervisor.

**2c. O assistente identifica informações conflitantes entre documentos**

Quando a base de documentos contém duas ou mais informações contraditórias sobre o mesmo tema, o assistente não escolhe uma delas silenciosamente. Ele informa ao atendente que identificou um conflito, apresenta as duas versões lado a lado com suas respectivas fontes e datas, e indica qual delas tem maior probabilidade de ser a vigente — com base em critérios como data de atualização mais recente ou hierarquia de área responsável. Em seguida, recomenda confirmar com o supervisor antes de usar a informação no atendimento, oferecendo o atalho de escalonamento.

---

### 3. Fluxo de Feedback

Ao receber uma resposta do assistente, o atendente pode sinalizar que ela está errada, desatualizada ou incompleta. Essa ação está disponível em todas as respostas, sem interromper o fluxo do atendimento.

Ao sinalizar, o atendente escolhe o tipo de problema (errada, desatualizada, incompleta ou outro) e tem a opção de adicionar uma observação em texto livre — por exemplo, "a tabela de SLA foi atualizada semana passada e o prazo correto é de 5 dias úteis". O assistente registra a sinalização vinculada à pergunta feita, à resposta gerada e ao documento de origem, e confirma ao atendente que o registro foi enviado para revisão.

A sinalização não altera a base de documentos imediatamente. O documento permanece ativo até que a revisão seja concluída por quem for designado como responsável — decisão que ainda será definida no discovery.

---

### 4. Cenários de Erro e Exceções

**Base de documentos indisponível.** Se o assistente não conseguir acessar as fontes (SharePoint, Confluence ou planilhas), ele informa ao atendente que está temporariamente sem acesso à documentação e sugere acesso direto às fontes ou escalonamento para o supervisor. O atendente não recebe uma resposta parcial ou fabricada.

**Pergunta fora do escopo da documentação.** Se o atendente fizer uma pergunta que claramente não pertence à base de conhecimento da NovaTech (por exemplo, uma dúvida sobre o sistema de CRM ou sobre RH), o assistente informa que o tema não está coberto pela documentação disponível e sugere o canal adequado, se esse mapeamento existir.

**Documento desatualizado sem versão mais recente disponível.** Se o assistente localizar a informação, mas o documento tiver data de atualização antiga e não houver versão mais recente na base, ele apresenta a resposta com um aviso explícito indicando a data do documento e recomendando confirmação antes de usar a informação com o cliente.

**Resposta usada e posteriormente identificada como incorreta.** Se o atendente perceber, após o atendimento, que uma resposta estava errada, ele pode retornar à interação no histórico e acionar o fluxo de feedback normalmente.

---

Há algum ponto que queira ajustar antes de avançar para a representação visual do fluxo?