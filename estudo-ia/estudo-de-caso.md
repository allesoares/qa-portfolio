# Análise de requisitos e testes com IA

Alexandre Soares · Curso de Engenharia de Qualidade de Software da EBAC · Módulo 30

Neste estudo, usei um agente de IA para analisar uma história de usuário, levantar dúvidas e organizar critérios de aceitação e cenários de teste. Depois revisei as respostas, pedi complementos e identifiquei duplicações que poderiam ser consolidadas.

O material apresenta primeiro a história, os critérios definidos no exercício e o plano de testes. Em seguida, reúne os prompts e as respostas disponíveis no documento original, na ordem em que foram registrados.

Este é um trabalho de análise e planejamento. Não houve implementação nem execução dos cenários neste estudo. As decisões atribuídas a produto foram simuladas durante o exercício; as hipóteses condicionais continuam dependendo de confirmação.

## 1 História e contexto

Como administrador da biblioteca, quero que o sistema bloqueie automaticamente usuários que possuem livros em atraso por mais de 7 dias, para evitar que continuem realizando novas reservas até regularizarem a devolução do livro.

A história inicial fala em reservas. No refinamento registrado no exercício, o comportamento passou a tratar de novas retiradas e da renovação do prazo dos livros já retirados. Essa mudança de vocabulário foi mantida explícita para orientar a leitura dos critérios.

### Como usei o agente

Comecei pedindo que a IA identificasse ambiguidades sem inventar regras. Em seguida, simulei os esclarecimentos com produto, pedi cenários baseados em risco e revisei a cobertura de permissões, falhas e integridade dos dados. A última interação trata da consolidação de cenários repetidos.

## 2 Critérios definidos no exercício

A tabela reúne a versão refinada dos critérios CA01 a CA14, registrada no documento de origem. A contagem considera dias corridos e a virada do dia em GMT−3. O bloqueio se aplica a usuários comuns, e as exceções manuais precisam ser preservadas nas verificações seguintes.

| ID | Situação | Resultado esperado |
| --- | --- | --- |
| CA01 | O usuário possui apenas um livro com até 7 dias de atraso. | Esse atraso não gera bloqueio. |
| CA02 | Um livro sem exceção manual entra no 8º dia de atraso. | O usuário comum é bloqueado à meia-noite em GMT−3. |
| CA03 | O usuário está bloqueado. | Os botões de retirada e renovação ficam ocultos, e essas operações são impedidas. |
| CA04 | O usuário tenta concluir uma retirada ou renovação por uma tela aberta antes do bloqueio. | O sistema impede a operação. |
| CA05 | O perfil de um usuário bloqueado é consultado. | O bloqueio está visível. |
| CA06 | O administrador consulta a lista de bloqueados. | Visualiza o usuário, o motivo e os dias de atraso do livro mais antigo que efetivamente causa o bloqueio. |
| CA07 | O usuário devolve um livro, mas outro sem exceção permanece com mais de 7 dias de atraso. | O bloqueio permanece, e o painel mostra os dados do livro mais antigo que ainda causa o bloqueio. |
| CA08 | O usuário devolve o último livro que mantém o bloqueio. | O bloqueio é removido imediatamente, mesmo que restem atrasos de até 7 dias ou livros com exceção manual. |
| CA09 | O administrador remove manualmente o bloqueio. | O usuário fica desbloqueado e os botões de retirada e renovação voltam a aparecer. |
| CA10 | Após a remoção manual, somente os livros abrangidos pela exceção continuam com mais de 7 dias de atraso. | O usuário não é bloqueado novamente por esses livros. |
| CA11 | Após a remoção manual, outro livro entra no 8º dia de atraso. | O usuário é bloqueado novamente, inclusive se esse livro já estivesse retirado antes da remoção manual. |
| CA12 | O livro que provocou o novo bloqueio é devolvido, restando somente livros com exceção manual. | O bloqueio é removido imediatamente. |
| CA13 | Um administrador possui livros com mais de 7 dias de atraso. | Essa regra não bloqueia o administrador. |
| CA14 | O bloqueio é aplicado ou removido. | O perfil e o painel administrativo refletem a situação atual do usuário. |

## 3 Plano de testes

O plano abaixo foi organizado no exercício a partir dos critérios e dos cenários gerados. Ele descreve a execução proposta, sem representar evidência de testes realizados.

### Escopo do teste

Validar o bloqueio e o desbloqueio de usuários comuns, considerando:

Limite de atraso e processamento à meia-noite em GMT−3.

Impedimento de novas retiradas e renovações.

Exibição do bloqueio no perfil e no painel.

Devoluções totais e parciais.

Desbloqueio manual e preservação das exceções.

Permissões, falhas e integridade dos dados.

As demais regras de retirada e renovação serão usadas como preparação dos testes, sem validação completa neste plano.

### Cenários principais

| Prioridade | Cenários | Referências |
| --- | --- | --- |
| Alta | Não bloquear até o 7º dia; bloquear no início do 8º dia em GMT−3, incluindo finais de semana. | CT01–CT05 |
| Alta | Ocultar os botões e impedir retirada e renovação pela API ou por tela aberta antes do bloqueio. | CT06–CT08, CT24–CT25 |
| Alta | Recusar operações sem alterar prazos, retiradas ou disponibilidade dos livros. | CT24–CT25 |
| Alta | Manter o bloqueio após devolução parcial e remover imediatamente após a última devolução necessária. | CT12–CT15, CT27 |
| Alta | Desbloquear manualmente, preservar os empréstimos e não bloquear novamente pelos livros com exceção. | CT16–CT18, CT28 |
| Alta | Aplicar novo bloqueio por outro livro e preservar as exceções anteriores. | CT19–CT20 |
| Alta | Não aplicar o bloqueio a administradores. | CT21 |
| Alta, condicional | Recusar desbloqueio sem permissão e tratar falhas sem corromper os dados. | CT22–CT23, CT29–CT30 |
| Média | Mostrar bloqueio, motivo e dias corretos no perfil e no painel, desconsiderando livros com exceção. | CT09–CT11, CT26 |

### Estratégia de validação

#### Preparação dos dados

Criar usuários comuns e administradores, com livros sem atraso, com 7, 8 e 12 dias de atraso e com exceção manual. Para testar uma recusa por bloqueio, garantir que a operação seria válida sem ele.

Usar um ambiente de teste com controle de data e hora para verificar a virada do dia sem depender de espera real.

| Tipo de teste | Como testar | O que verificar |
| --- | --- | --- |
| Funcional | Executar os fluxos de bloqueio, devolução parcial, desbloqueio automático e manual. | Se cada ação produz o estado esperado conforme as regras confirmadas. |
| API | Enviar solicitações de retirada, renovação e desbloqueio diretamente, usando diferentes perfis. Pode ser executado com Postman. | Recusa ou sucesso, motivo da resposta e estado dos dados após a operação. Os códigos HTTP devem seguir o contrato da API. |
| Interface | Acessar o perfil, as telas de retirada e renovação e o painel administrativo. Repetir com uma tela aberta antes do bloqueio. | Botões ocultos, bloqueio visível, motivo correto, dias corretos e impedimento de operações pela tela antiga. |
| Integração | Executar o processamento diário e registrar devoluções, consultando depois API e interface. | Consistência entre bloqueio, permissões, perfil e painel. |
| Permissões | Tentar desbloquear com administrador e com usuário comum. | Somente o perfil autorizado consegue executar a ação. A exclusividade administrativa permanece como proposta a confirmar no CT29. |
| Falhas e integridade | Simular falhas antes do registro da devolução, no desbloqueio manual e após uma devolução confirmada. | Preservação dos dados, ausência de duplicidade e identificação de bloqueios inconsistentes. Os comportamentos condicionais dependem de confirmação. |
| Regressão | Reexecutar os fluxos principais após correções. | Usuários liberados continuam realizando operações válidas; bloqueados continuam impedidos. |

API e interface são camadas de validação: os testes funcionais devem cobrir ambas.

#### Técnicas utilizadas

Partição de equivalência: separar usuários comuns e administradores; livros dentro do prazo, atrasados e com exceção.

Valores de fronteira: verificar 7 e 8 dias, além de 23:59:59 e 00:00:00 em GMT−3.

Transição de estados: validar desbloqueado → bloqueado → desbloqueado, incluindo novo bloqueio após exceção manual.

#### Evidências e ordem de execução

Executar primeiro os cenários de criticidade alta. Registrar requisições e respostas, capturas das telas e consultas dos dados antes e depois das operações. Nas recusas, comparar os dados de negócio; um eventual registro de auditoria não representa alteração indevida.

### Riscos identificados

| Risco | Impacto | Validação principal |
| --- | --- | --- |
| Contagem ou fuso incorreto | Bloqueio antecipado ou atrasado. | Fronteiras de data e hora em GMT−3. |
| Bloqueio aplicado apenas na interface | Usuário consegue operar diretamente pela API. | Solicitações diretas e tela antiga. |
| Recusa altera dados | Prazo, disponibilidade ou retiradas ficam incorretos. | Comparação antes e depois. |
| Devolução parcial libera o usuário | Usuário retira livros mesmo com outro atraso impeditivo. | Múltiplos livros atrasados. |
| Exceção manual é perdida | Usuário volta a ser bloqueado indevidamente. | Novas viradas do dia e novo bloqueio. |
| Exceção manual impede qualquer bloqueio futuro | Outros livros atrasados deixam de bloquear. | Livro diferente entrando no 8º dia. |
| Falha após devolução confirmada | Usuário continua bloqueado apesar da regularização. | Injeção de falha e verificação de consistência. |
| Painel mostra livro com exceção | Administrador recebe uma justificativa incorreta. | Conferência do livro que efetivamente causa o bloqueio. |

Proposta de conclusão do teste: considerar a execução aprovada quando os cenários não condicionais passarem e não houver defeitos de alta criticidade em aberto. Os cenários @condicional devem permanecer identificados como pendentes até a confirmação das regras, especialmente a recuperação após falha no desbloqueio.

## 4 Revisão dos cenários

As respostas registradas chegaram a 30 IDs de cenário, de CT01 a CT30. Na revisão, identifiquei sobreposições e pedi a consolidação, mantendo os IDs restantes para não perder a ligação com os critérios.

| Cenários | Consolidação | Cobertura preservada |
| --- | --- | --- |
| CT10 e CT26 | Manter CT10 e remover CT26. | Painel com motivo e dias do livro mais antigo que causa o bloqueio, ignorando exceções manuais. |
| CT08, CT24 e CT25 | Manter CT08 como esquema com exemplos de retirada e renovação; incorporar as verificações adicionais e remover CT24 e CT25. | Acesso direto pela API, operação válida sem o bloqueio e preservação dos dados após recusa. |
| CT12 e CT27 | Manter CT12, incorporando as verificações do CT27. | Devolução parcial registrada, bloqueio mantido e painel atualizado para o livro restante. |
| CT16 e CT28 | Manter CT16, incorporando a preservação dos empréstimos e a criação das exceções. | Desbloqueio manual sem alterar prazos nem registrar devoluções indevidas. A próxima virada do dia continua coberta pelo CT18. |

A proposta final reduz a relação para 25 cenários ou esquemas. Esse número conta os IDs mantidos, não as combinações de exemplos de cada esquema. O retorno final traz a versão completa do CT08 consolidado e orientações para atualizar CT12 e CT16; ele não reapresenta toda a suíte consolidada.

Os cenários CT22, CT23, CT29 e CT30 mantêm hipóteses sobre falhas e permissões. Por isso, a afirmação de ausência de pendências em uma das respostas do agente deve ser lida apenas no contexto daquela etapa; esses quatro cenários continuam condicionais.

Para uma futura execução, ainda seria necessário materializar essas atualizações, confirmar os quatro cenários condicionais e definir o mecanismo e o prazo de recuperação após falha no desbloqueio automático.

## 5 Prompts e respostas do agente

O registro abaixo preserva o conteúdo disponível no documento original, com nova organização visual. O plano de testes foi apresentado antes, na seção 3. Não foram reconstruídos prompts ausentes nem acrescentados resultados de execução.

### Interação 1 Análise inicial da história

<details>
<summary>Abrir interação 1 análise inicial da história</summary>

#### Prompt enviado

Você é um engenheiro de qualidade de software responsável pelo Hub de Leitura, um sistema de biblioteca com usuários comuns e administradores.

Analise a história e as regras abaixo. Identifique ambiguidades, perguntas para o time de produto e pontos que precisam de detalhamento. Não resolva dúvidas inventando regras. Depois, proponha critérios de aceitação claros, em linguagem simples e focada no comportamento do usuário.

Organize a resposta em regras confirmadas, perguntas numeradas com impacto nos testes e critérios de aceitação. Identifique qualquer premissa como proposta a confirmar.

História
Como administrador da biblioteca, quero que o sistema bloqueie automaticamente usuários que possuem livros em atraso por mais de 7 dias, para evitar que continuem realizando novas reservas até regularizarem a devolução do livro.

Regras de negócio enviadas no mesmo prompt

Um usuário deve ser bloqueado automaticamente quando possuir um livro com mais de 7 dias de atraso.

Usuários bloqueados não podem realizar novas reservas nem renovar reservas existentes.

O bloqueio deve aparecer no perfil do usuário e no painel administrativo.

O painel deve permitir visualizar lista de usuários bloqueados, motivo do bloqueio e quantidade de dias em atraso.

Quando o livro for devolvido, o bloqueio deve ser removido automaticamente.

O administrador pode remover manualmente o bloqueio de um usuário, caso necessário.

#### Resposta do agente

#### Regras confirmadas

O bloqueio é automático quando o usuário possui pelo menos um livro com mais de 7 dias de atraso.

Enquanto estiver bloqueado, o usuário não pode fazer novas reservas nem renovar reservas existentes.

O bloqueio deve aparecer no perfil do usuário e no painel administrativo.

O painel deve mostrar os usuários bloqueados, o motivo e a quantidade de dias em atraso.

A devolução do livro deve remover o bloqueio automaticamente. O comportamento com vários livros atrasados precisa ser esclarecido.

O administrador pode remover o bloqueio manualmente.

#### Perguntas para o time de produto

Como os dias de atraso são contados: dias corridos, úteis ou períodos completos de 24 horas? Qual horário marca o vencimento?
Impacto nos testes: define o momento exato do bloqueio e os casos de limite de 7 dias.

Quando o sistema verifica os atrasos e atualiza o bloqueio?
Impacto nos testes: define quanto tempo pode passar entre atingir o limite e aparecer bloqueado.

Se o usuário tiver dois livros com mais de 7 dias de atraso e devolver apenas um, continua bloqueado?
Impacto nos testes: define quando o desbloqueio automático deve acontecer.

Depois de um desbloqueio manual, o usuário pode ser bloqueado novamente pelo mesmo atraso? Quando?
Impacto nos testes: define a duração e o efeito da exceção administrativa.

“Renovar reservas” significa estender uma reserva ou renovar o empréstimo de um livro? O que acontece com as reservas já existentes?
Impacto nos testes: esclarece qual ação deve ser impedida e se as reservas atuais continuam válidas.

Quando houver vários livros atrasados, quais motivos e quantidades de dias o painel deve mostrar?
Impacto nos testes: define os dados esperados na lista de usuários bloqueados.

O que o usuário deve ver ao tentar reservar ou renovar enquanto estiver bloqueado?
Impacto nos testes: define a mensagem e a orientação esperadas na tela.

O bloqueio também se aplica a administradores que pegam livros emprestados?
Impacto nos testes: define quais perfis estão sujeitos à regra.

#### Critérios de aceitação

Os critérios abaixo são propostas baseadas nas regras confirmadas. Os detalhes indicados como pendentes dependem das respostas do produto.

| ID | Situação e comportamento esperado |
| --- | --- |
| CA01 | Dado um usuário sem bloqueio e com um único livro com mais de 7 dias de atraso, quando o sistema verificar o atraso, o usuário deve ser bloqueado automaticamente. Pendente: contagem dos dias e prazo da atualização. |
| CA02 | Dado um usuário sem bloqueio e com um único livro com exatamente 7 dias de atraso, esse atraso não deve gerar o bloqueio. Pendente: definição da contagem dos dias. |
| CA03 | Dado um usuário bloqueado, quando tentar fazer uma nova reserva, o sistema deve impedir a criação da reserva. |
| CA04 | Dado um usuário bloqueado com uma reserva existente, quando tentar renová-la, o sistema deve impedir a renovação. Pendente: significado de “renovar reservas”. |
| CA05 | Dado um usuário bloqueado, quando seu perfil for consultado, a condição de bloqueio deve estar visível. |
| CA06 | Dado um usuário bloqueado, quando o administrador consultar a lista de bloqueados, deve encontrar esse usuário, o motivo e a quantidade de dias em atraso. Pendente: apresentação de vários atrasos. |
| CA07 | Dado um usuário bloqueado por um único livro atrasado, quando a devolução desse livro for registrada, o bloqueio deve ser removido automaticamente e essa mudança deve aparecer no perfil e no painel. Pendente: prazo da atualização. |
| CA08 | Dado um usuário bloqueado, quando o administrador remover manualmente o bloqueio, o usuário deve ficar desbloqueado e essa mudança deve aparecer no perfil e no painel. Pendente: possibilidade de novo bloqueio pelo mesmo atraso. |

Não foi adotada uma regra para as dúvidas em aberto; esses cenários precisam de decisão do produto antes de terem resultados esperados definitivos.

</details>

### Refinamento após esclarecimentos simulados

<details>
<summary>Abrir refinamento após esclarecimentos simulados</summary>



Fiz prompts simulando ter tirado as duvidas com produto sobre os itens mencionados.



Nota de organização: o documento de origem informa que houve novos prompts simulando esclarecimentos com produto, mas não registra o texto desses prompts. Abaixo está a resposta preservada dessa etapa.

#### Regras confirmadas

O bloqueio se aplica somente a usuários comuns.

O atraso é contado em dias corridos. O bloqueio ocorre automaticamente à meia-noite em GMT−3, no início do 8º dia de atraso.

Usuários bloqueados não podem retirar novos livros nem renovar o prazo dos livros retirados. Os botões dessas ações ficam ocultos.

O bloqueio aparece no perfil e no painel administrativo.

O painel lista os usuários bloqueados e mostra o motivo e os dias de atraso do livro mais antigo que efetivamente causa o bloqueio. Livros com exceção manual são desconsiderados.

Ao devolver o último livro que mantém o bloqueio, o usuário é desbloqueado imediatamente.

Devolver apenas um dos livros que mantêm o bloqueio não libera o usuário.

O administrador pode desbloquear manualmente. Os livros abrangidos pela exceção continuam desconsiderados nas verificações posteriores.

Outro livro que entre no 8º dia de atraso pode gerar novo bloqueio, mesmo que já estivesse retirado antes da exceção.

#### Perguntas pendentes e impacto nos testes

Nenhuma pendência identificada para os cenários discutidos. As decisões confirmadas permitem definir os resultados esperados abaixo, sem inventar regras.

#### Critérios de aceitação

| ID | Situação | Resultado esperado |
| --- | --- | --- |
| CA01 | O usuário possui apenas um livro com até 7 dias de atraso. | Esse atraso não gera bloqueio. |
| CA02 | Um livro sem exceção manual entra no 8º dia de atraso. | O usuário comum é bloqueado à meia-noite em GMT−3. |
| CA03 | O usuário está bloqueado. | Os botões de retirada e renovação ficam ocultos, e essas operações são impedidas. |
| CA04 | O usuário tenta concluir uma retirada ou renovação por uma tela aberta antes do bloqueio. | O sistema impede a operação. |
| CA05 | O perfil de um usuário bloqueado é consultado. | O bloqueio está visível. |
| CA06 | O administrador consulta a lista de bloqueados. | Visualiza o usuário, o motivo e os dias de atraso do livro mais antigo que efetivamente causa o bloqueio. |
| CA07 | O usuário devolve um livro, mas outro sem exceção permanece com mais de 7 dias de atraso. | O bloqueio permanece, e o painel mostra os dados do livro mais antigo que ainda causa o bloqueio. |
| CA08 | O usuário devolve o último livro que mantém o bloqueio. | O bloqueio é removido imediatamente, mesmo que restem atrasos de até 7 dias ou livros com exceção manual. |
| CA09 | O administrador remove manualmente o bloqueio. | O usuário fica desbloqueado e os botões de retirada e renovação voltam a aparecer. |
| CA10 | Após a remoção manual, somente os livros abrangidos pela exceção continuam com mais de 7 dias de atraso. | O usuário não é bloqueado novamente por esses livros. |
| CA11 | Após a remoção manual, outro livro entra no 8º dia de atraso. | O usuário é bloqueado novamente, inclusive se esse livro já estivesse retirado antes da remoção manual. |
| CA12 | O livro que provocou o novo bloqueio é devolvido, restando somente livros com exceção manual. | O bloqueio é removido imediatamente. |
| CA13 | Um administrador possui livros com mais de 7 dias de atraso. | Essa regra não bloqueia o administrador. |
| CA14 | O bloqueio é aplicado ou removido. | O perfil e o painel administrativo refletem a situação atual do usuário. |

</details>

### Interação 2 Geração de cenários por risco

<details>
<summary>Abrir interação 2 geração de cenários por risco</summary>

#### Prompt enviado

Você é um engenheiro de qualidade com foco em testes baseados em risco
Gere cenários para os critérios anteriores usando partição de equivalência, valores de fronteira e transição de estados. Inclua positivos, negativos, limites e erros
Retorne Gherkin em português, com ID, tag de criticidade @alta, @media ou @baixa, critério relacionado e resultado verificável. Marque @condicional quando houver hipótese.

#### Resposta do agente

```gherkin
# language: pt

Funcionalidade: Bloqueio de usuários por atraso na devolução
  Para impedir novas retiradas e renovações durante o bloqueio
  Como biblioteca
  Quero controlar o bloqueio conforme os atrasos e as exceções manuais

  Contexto:
    Dado que a contagem de atraso utiliza dias corridos
    E que a virada do dia considera GMT−3

  @alta @CA01 @particao @fronteira @positivo
  Esquema do Cenário: CT01 - Não bloquear antes de ultrapassar o limite
    Dado um usuário comum sem bloqueio e sem exceções manuais
    E que seu único livro está <situacao>
    Quando o sistema verificar os atrasos
    Então o usuário deve permanecer desbloqueado

    Exemplos:
      | situacao                |
      | antes do vencimento     |
      | no dia do vencimento    |
      | com 1 dia de atraso     |
      | com 6 dias de atraso    |
      | com 7 dias de atraso    |

  @alta @CA02 @particao @fronteira @positivo
  Esquema do Cenário: CT02 - Bloquear atrasos acima do limite
    Dado um usuário comum sem bloqueio e sem exceções manuais
    E que seu único livro terá <dias> dias de atraso na próxima virada do dia
    Quando chegar a meia-noite em GMT−3
    Então o usuário deve estar bloqueado

    Exemplos:
      | dias |
      | 8    |
      | 9    |
      | 30   |

  @alta @CA01 @CA02 @fronteira @transicao
  Cenário: CT03 - Aplicar o bloqueio exatamente na virada para o oitavo dia
    Dado um usuário comum desbloqueado
    E que seu único livro está no 7º dia de atraso, sem exceção manual
    Quando forem 23:59:59 em GMT−3
    Então o usuário deve estar desbloqueado
    Quando chegar 00:00:00 em GMT−3, iniciando o 8º dia de atraso
    Então o usuário deve estar bloqueado

  @alta @CA02 @fronteira
  Cenário: CT04 - Utilizar GMT−3 independentemente do fuso do dispositivo
    Dado um usuário comum desbloqueado com o dispositivo configurado em GMT
    E que seu único livro entrará no 8º dia de atraso à meia-noite em GMT−3
    Quando o dispositivo marcar 00:00 em GMT
    Então o usuário deve continuar desbloqueado
    Quando o dispositivo marcar 03:00 em GMT
    Então o usuário deve estar bloqueado

  @alta @CA02 @particao
  Cenário: CT05 - Considerar sábados e domingos na contagem
    Dado um usuário comum desbloqueado e sem exceções manuais
    E que seu único livro está no 6º dia de atraso na sexta-feira
    Quando chegar a meia-noite de sábado em GMT−3
    Então o usuário deve permanecer desbloqueado
    Quando chegar a meia-noite de domingo em GMT−3
    Então o usuário deve estar bloqueado pelo 8º dia de atraso

  @alta @CA03 @negativo
  Cenário: CT06 - Ocultar ações durante o bloqueio
    Dado um usuário comum bloqueado
    Quando acessar as telas de retirada e renovação
    Então não deve visualizar o botão de nova retirada
    E não deve visualizar o botão de renovação

  @alta @CA03 @CA04 @transicao @negativo
  Esquema do Cenário: CT07 - Impedir operação em tela aberta antes do bloqueio
    Dado um usuário comum desbloqueado com a tela de <operacao> aberta
    E que a operação ainda não foi concluída
    Quando o usuário for bloqueado pela virada para o 8º dia de atraso
    E tentar concluir a operação pela tela já aberta
    Então a operação deve ser recusada
    E <resultado>

    Exemplos:
      | operacao | resultado                                  |
      | retirada | nenhuma nova retirada deve ser registrada  |
      | renovação| o prazo original do livro deve permanecer  |

  @alta @CA03 @negativo
  Esquema do Cenário: CT08 - Impedir operação sem utilizar os botões
    Dado um usuário comum bloqueado
    Quando enviar diretamente ao sistema uma solicitação de <operacao>
    Então a operação deve ser recusada
    E <resultado>

    Exemplos:
      | operacao | resultado                                  |
      | retirada | nenhuma nova retirada deve ser registrada  |
      | renovação| o prazo original do livro deve permanecer  |

  @media @CA05 @CA14 @positivo
  Cenário: CT09 - Exibir o bloqueio no perfil
    Dado um usuário comum que acabou de ser bloqueado automaticamente
    Quando seu perfil for consultado
    Então deve existir uma indicação visível de bloqueio

  @media @CA06 @particao @positivo
  Cenário: CT10 - Mostrar o livro mais antigo que causa o bloqueio
    Dado um usuário comum bloqueado
    E que possui o livro A com 12 dias de atraso, sem exceção manual
    E que possui o livro B com 8 dias de atraso, sem exceção manual
    E que possui o livro C com 20 dias de atraso, com exceção manual
    Quando o administrador consultar a lista de bloqueados
    Então o usuário deve constar na lista
    E o motivo deve corresponder ao atraso do livro A
    E a quantidade de dias em atraso deve ser 12

  @media @CA06 @fronteira
  Cenário: CT11 - Atualizar os dias de atraso na virada do dia
    Dado um usuário bloqueado por um único livro com 8 dias de atraso
    Quando chegar a meia-noite em GMT−3, iniciando o 9º dia de atraso
    E o administrador consultar a lista de bloqueados
    Então a quantidade de dias em atraso desse usuário deve ser 9

  @alta @CA07 @transicao
  Cenário: CT12 - Manter o bloqueio ao devolver o livro mais antigo
    Dado um usuário bloqueado pelo livro A com 12 dias de atraso
    E pelo livro B com 8 dias de atraso
    E que nenhum desses livros possui exceção manual
    Quando a devolução do livro A for registrada
    Então o usuário deve permanecer bloqueado
    E o painel deve mostrar o motivo referente ao livro B
    E a quantidade de dias em atraso deve ser 8

  @alta @CA07 @transicao
  Cenário: CT13 - Manter o bloqueio ao devolver o livro menos atrasado
    Dado um usuário bloqueado pelo livro A com 12 dias de atraso
    E pelo livro B com 8 dias de atraso
    E que nenhum desses livros possui exceção manual
    Quando a devolução do livro B for registrada
    Então o usuário deve permanecer bloqueado
    E o painel deve continuar mostrando o motivo referente ao livro A
    E a quantidade de dias em atraso deve ser 12

  @alta @CA08 @CA14 @transicao @positivo
  Cenário: CT14 - Desbloquear imediatamente após a última devolução necessária
    Dado um usuário bloqueado por um único livro com 8 dias de atraso
    Quando a devolução desse livro for registrada às 14:30 em GMT−3
    Então o usuário deve ficar desbloqueado imediatamente, sem esperar a meia-noite
    E seu perfil não deve indicar bloqueio ativo
    E ele não deve constar na lista de usuários bloqueados
    E os botões de retirada e renovação devem estar visíveis

  @alta @CA08 @particao @fronteira @transicao
  Esquema do Cenário: CT15 - Desbloquear com livros restantes abaixo do limite
    Dado um usuário bloqueado pelo livro A com 10 dias de atraso
    E que possui somente mais um livro B com <dias> dias de atraso
    E que nenhum desses livros possui exceção manual
    Quando a devolução do livro A for registrada
    Então o usuário deve ficar desbloqueado imediatamente

    Exemplos:
      | dias |
      | 1    |
      | 6    |
      | 7    |

  @alta @CA09 @CA14 @transicao @positivo
  Cenário: CT16 - Remover o bloqueio manualmente
    Dado um usuário comum bloqueado por dois livros atrasados
    Quando o administrador remover manualmente o bloqueio
    Então o usuário deve ficar desbloqueado
    E os botões de retirada e renovação devem estar visíveis
    E seu perfil não deve indicar bloqueio ativo
    E ele não deve constar na lista de usuários bloqueados

  @alta @CA09 @positivo
  Esquema do Cenário: CT17 - Permitir operações após o desbloqueio manual
    Dado um usuário desbloqueado manualmente
    E que atende aos demais requisitos para <operacao>
    Quando concluir uma <operacao>
    Então <resultado>

    Exemplos:
      | operacao | resultado                                         |
      | retirada | a nova retirada deve ser registrada                |
      | renovação| a extensão solicitada do prazo deve ser registrada |

  @alta @CA10 @transicao @negativo
  Cenário: CT18 - Não bloquear novamente pelos livros com exceção manual
    Dado um usuário desbloqueado manualmente
    E que todos os seus livros com mais de 7 dias de atraso receberam a exceção
    E que nenhum outro livro ultrapassará o limite
    Quando ocorrerem duas novas viradas do dia em GMT−3
    Então o usuário deve permanecer desbloqueado

  @alta @CA11 @transicao @fronteira
  Esquema do Cenário: CT19 - Bloquear novamente por outro livro
    Dado um usuário desbloqueado manualmente por atraso no livro A
    E que o livro B foi retirado <momento>
    E que o livro B não possui exceção manual
    Quando o livro B entrar no 8º dia de atraso à meia-noite em GMT−3
    Então o usuário deve estar bloqueado novamente

    Exemplos:
      | momento                     |
      | antes do desbloqueio manual |
      | após o desbloqueio manual   |

  @alta @CA12 @CA06 @transicao
  Cenário: CT20 - Preservar a exceção manual após um novo bloqueio
    Dado um usuário com o livro A atrasado e abrangido por exceção manual
    E que foi bloqueado novamente pelo livro B com 8 dias de atraso
    E que nenhum outro livro causa bloqueio
    Quando o administrador consultar o bloqueio
    Então o motivo deve corresponder ao livro B
    Quando a devolução do livro B for registrada
    Então o usuário deve ficar desbloqueado imediatamente
    E o atraso do livro A não deve manter o bloqueio

  @alta @CA13 @particao @negativo
  Cenário: CT21 - Não aplicar a regra a administradores
    Dado um administrador sem bloqueio com um livro no 7º dia de atraso
    Quando esse livro entrar no 8º dia de atraso à meia-noite em GMT−3
    Então o administrador não deve ser bloqueado por essa regra
    E não deve constar na lista de bloqueados por esse atraso

  # Hipótese a confirmar: uma devolução que falha antes de ser
  # registrada não altera o empréstimo nem remove o bloqueio.
  @alta @CA08 @condicional @erro
  Cenário: CT22 - Falha ao registrar a devolução
    Dado um usuário bloqueado por um único livro
    Quando ocorrer uma falha que impeça o registro da devolução desse livro
    Então o livro deve permanecer registrado como não devolvido
    E o usuário deve permanecer bloqueado

  # Hipótese a confirmar: uma remoção manual que falha antes de ser
  # registrada não cria a exceção nem altera o bloqueio.
  @alta @CA09 @condicional @erro
  Cenário: CT23 - Falha ao registrar o desbloqueio manual
    Dado um usuário bloqueado por um único livro
    Quando o administrador tentar remover o bloqueio
    E uma falha impedir o registro dessa remoção
    Então o usuário deve permanecer bloqueado
    E nenhuma exceção manual deve ser registrada para esse livro
```

</details>

### Interação 3 Complementos de cobertura

<details>
<summary>Abrir interação 3 complementos de cobertura</summary>

#### Prompt registrado

Falta explicitar a renovação válida, a tentativa sem permissão e a falha após uma devolução já confirmada. Abaixo estão os cenários complementares, incluindo a preservação dos dados nas recusas.

#### Resposta do agente

```gherkin
# language: pt

Funcionalidade: Integridade das operações relacionadas ao bloqueio

  @alta @CA03 @negativo
  Cenário: CT24 - Recusar nova reserva diretamente pela API
    Dado um usuário comum bloqueado
    E uma solicitação de reserva para retirar um novo livro
    E que a solicitação atende aos demais requisitos da operação
    E que foram registrados os dados atuais das retiradas e da disponibilidade do livro
    Quando o usuário enviar a solicitação diretamente pela API
    Então a API deve recusar a operação por bloqueio
    E nenhuma nova reserva ou retirada deve ser registrada
    E a disponibilidade do livro deve permanecer inalterada
    E as retiradas existentes devem permanecer inalteradas

  @alta @CA03 @CA04 @negativo
  Cenário: CT25 - Recusar renovação que seria válida sem o bloqueio
    Dado um usuário comum bloqueado pelo atraso do livro A
    E que possui uma retirada do livro B elegível para renovação
    E que o único impedimento para renovar o livro B é o bloqueio do usuário
    E que foram registrados o prazo e os dados atuais dessa retirada
    Quando o usuário solicitar a renovação do livro B diretamente pela API
    Então a API deve recusar a operação por bloqueio
    E o prazo de devolução do livro B deve permanecer inalterado
    E nenhuma renovação deve ser registrada
    E os dados das retiradas dos livros A e B devem permanecer inalterados

  @media @CA06 @positivo
  Cenário: CT26 - Exibir o motivo e os dias do atraso que causa o bloqueio
    Dado um usuário bloqueado pelo livro A com 12 dias de atraso
    E pelo livro B com 8 dias de atraso
    E que possui o livro C com 20 dias de atraso e exceção manual
    Quando o administrador consultar a lista de bloqueados
    Então o usuário deve aparecer na lista
    E o motivo exibido deve corresponder ao atraso do livro A
    E a quantidade de dias em atraso deve ser 12

  @alta @CA07 @transicao
  Cenário: CT27 - Registrar devolução parcial sem liberar o usuário
    Dado um usuário bloqueado pelos livros A e B
    E que o livro A possui 12 dias de atraso
    E que o livro B possui 8 dias de atraso
    E que nenhum deles possui exceção manual
    Quando a devolução do livro A for confirmada
    Então o livro A deve constar como devolvido
    E o livro B deve continuar registrado como não devolvido
    E o usuário deve permanecer bloqueado
    E o painel deve mostrar o motivo referente ao livro B
    E a quantidade de dias em atraso deve ser 8
    E os botões de retirada e renovação devem permanecer ocultos

  @alta @CA09 @CA10 @transicao
  Cenário: CT28 - Desbloquear manualmente sem alterar os empréstimos
    Dado um usuário bloqueado pelos livros A e B
    E que foram registrados os prazos e os dados atuais dessas retiradas
    Quando um administrador remover manualmente o bloqueio
    Então o usuário deve ficar desbloqueado
    E os livros A e B devem ser abrangidos pela exceção manual
    E os prazos e os registros de retirada devem permanecer inalterados
    E nenhum desses livros deve constar como devolvido
    E o perfil não deve indicar bloqueio ativo
    E o usuário não deve constar na lista de bloqueados
    Quando ocorrer a próxima virada do dia sem outro livro ultrapassar o limite
    Então o usuário deve permanecer desbloqueado

  # Proposta a confirmar: somente administradores podem executar
  # o desbloqueio manual.
  @alta @CA09 @condicional @negativo
  Cenário: CT29 - Recusar desbloqueio manual solicitado sem permissão
    Dado um usuário comum bloqueado
    E que foram registrados seu bloqueio, suas exceções e suas retiradas
    Quando esse usuário solicitar seu desbloqueio manual diretamente pela API
    Então a API deve recusar a operação por falta de permissão
    E o bloqueio deve permanecer inalterado
    E nenhuma exceção manual deve ser criada
    E os dados das retiradas devem permanecer inalterados
    E o perfil e o painel devem continuar indicando o bloqueio

  # Proposta a confirmar: uma falha no processamento do desbloqueio
  # não desfaz uma devolução já confirmada.
  # O mecanismo e o prazo de recuperação ainda precisam ser definidos.
  @alta @CA08 @CA14 @condicional @erro
  Cenário: CT30 - Preservar a devolução confirmada diante de falha posterior
    Dado um usuário bloqueado por um único livro
    E que a devolução desse livro foi confirmada e persistida
    Quando ocorrer uma falha no processamento do desbloqueio automático
    Então a devolução deve permanecer registrada
    E o livro não deve voltar a constar como não devolvido
    E nenhuma segunda devolução deve ser criada
    E os demais registros de retirada devem permanecer inalterados
```

</details>

### Interação 4 Consolidação de duplicações

<details>
<summary>Abrir interação 4 consolidação de duplicações</summary>

#### Prompt enviado

Há cenários repetidos. O CT26 é praticamente igual ao CT10, e o CT24 e o CT25 cobrem o mesmo que o CT08. Ao consolidar essas duplicações, a suíte fica mais enxuta e você mostra a revisão crítica da saída da IA.

#### Resposta do agente

Concordo. Os cenários complementares deveriam ter atualizado os existentes, evitando duplicação.

| Cenários | Consolidação | Cobertura preservada |
| --- | --- | --- |
| CT10 e CT26 | Manter CT10 e remover CT26. | Painel com motivo e dias do livro mais antigo que causa o bloqueio, ignorando exceções manuais. |
| CT08, CT24 e CT25 | Manter CT08 como esquema com exemplos de retirada e renovação; incorporar as verificações adicionais e remover CT24 e CT25. | Acesso direto pela API, operação válida sem o bloqueio e preservação dos dados após recusa. |
| CT12 e CT27 | Manter CT12, incorporando as verificações do CT27. | Devolução parcial registrada, bloqueio mantido e painel atualizado para o livro restante. |
| CT16 e CT28 | Manter CT16, incorporando a preservação dos empréstimos e a criação das exceções. | Desbloqueio manual sem alterar prazos nem registrar devoluções indevidas. A próxima virada do dia continua coberta pelo CT18. |

O CT08 consolidado fica assim:

```gherkin
# language: pt

@alta @CA03 @negativo
Esquema do Cenário: CT08 - Recusar operação pela API preservando os dados
  Dado um usuário comum bloqueado
  E uma solicitação de <operacao> que seria válida sem o bloqueio
  E que foram registrados os dados atuais das retiradas, dos prazos e da disponibilidade
  Quando o usuário enviar a solicitação diretamente pela API
  Então a API deve recusar a operação por bloqueio
  E <resultado>
  E os dados das retiradas, dos prazos e da disponibilidade devem permanecer inalterados

  Exemplos:
    | operacao  | resultado                                 |
    | retirada  | nenhuma nova retirada deve ser registrada |
    | renovação | nenhuma renovação deve ser registrada     |
```

O CT07 permanece separado, pois verifica uma tela aberta antes da mudança de estado. O CT18 também permanece, pois testa a persistência da exceção nas próximas viradas do dia.

Com essa revisão, a suíte passa de 30 para 25 cenários ou esquemas, mantendo os IDs restantes para preservar a rastreabilidade. Isso reduz duplicações sem retirar as verificações adicionais de integridade dos dados.

</details>
