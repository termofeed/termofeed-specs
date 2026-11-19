# Termofeed — Requisitos funcionais

Versão 1.0 · 2026-11-18

Atores: **Criador** (cria e controla a sala), **Participante** (entra na sala e reage) e **Sistema** (ações automáticas).

O criador também é um participante.

---

ID: RF-001

Título: Criar sala

Descrição: O sistema deve criar uma sala virtual quando o criador enviar os dados da sala.

Ator: Criador

Critérios de aceite: Dado que o criador informou uma duração válida, quando ele envia o pedido de criação, então o sistema cria a sala com status `active`, devolve o código da sala, o `hostToken` e o link de entrada.

---

ID: RF-002

Título: Dispensar login

Descrição: O sistema deve permitir o uso completo sem conta quando o usuário criar ou entrar em uma sala.

Ator: Criador, Participante

Critérios de aceite: Dado que o usuário não tem conta, quando ele cria uma sala ou entra em uma sala, então o sistema não pede e-mail, senha nem cadastro.

---

ID: RF-003

Título: Definir duração inicial

Descrição: O sistema deve aceitar a duração inicial da sala quando o criador criar a sala.

Ator: Criador

Critérios de aceite: Dado que o criador informou uma duração entre 5 e 240 minutos, quando ele cria a sala, então o sistema define o fim da sala como o horário de criação mais a duração. Dado que a duração está fora desta faixa, quando ele cria a sala, então o sistema responde `VALIDATION_ERROR` e não cria a sala.

---

ID: RF-004

Título: Definir intervalo entre reações

Descrição: O sistema deve aceitar o intervalo entre reações quando o criador criar a sala.

Ator: Criador

Critérios de aceite: Dado que o criador informou um intervalo entre 1 e 60 segundos, quando ele cria a sala, então o sistema guarda este valor para a sala. Dado que o criador não informou o intervalo, quando ele cria a sala, então o sistema usa 5 segundos.

---

ID: RF-005

Título: Definir paleta de emojis

Descrição: O sistema deve aceitar uma lista de emojis permitidos quando o criador criar a sala.

Ator: Criador

Critérios de aceite: Dado que o criador informou de 2 a 12 emojis, quando ele cria a sala, então o sistema guarda esta lista como a paleta da sala. Dado que o criador não informou a lista, quando ele cria a sala, então o sistema usa a paleta padrão de 8 emojis.

---

ID: RF-006

Título: Definir nome da sala

Descrição: O sistema deve aceitar um nome opcional para a sala quando o criador criar a sala.

Ator: Criador

Critérios de aceite: Dado que o criador informou um nome de até 80 caracteres, quando ele cria a sala, então o sistema mostra este nome na sala. Dado que o nome tem mais de 80 caracteres, quando ele cria a sala, então o sistema responde `VALIDATION_ERROR`.

---

ID: RF-007

Título: Gerar link de entrada

Descrição: O sistema deve gerar um link único para a sala quando o sistema criar a sala.

Ator: Criador

Critérios de aceite: Dado que a sala foi criada, quando o criador recebe a resposta, então o link tem o formato `/r/<código>` e o código tem o formato `abc-defg-hij`. Dado que existem duas salas, então os dois códigos são diferentes.

---

ID: RF-008

Título: Validar link da sala

Descrição: O sistema deve devolver os dados públicos da sala quando o usuário abrir o link da sala.

Ator: Participante

Critérios de aceite: Dado que o código existe, quando o usuário consulta a sala, então o sistema devolve nome, status, horário de fim, intervalo, paleta e número de participantes. Dado que o código não existe, quando o usuário consulta a sala, então o sistema responde `ROOM_NOT_FOUND`.

---

ID: RF-009

Título: Entrar na sala por link

Descrição: O sistema deve registrar o participante quando o participante entrar na sala pelo link.

Ator: Participante

Critérios de aceite: Dado que a sala está ativa e tem vaga, quando o participante entra pelo link, então o sistema devolve um `participantId` e um `participantToken`, e aumenta em 1 o número de participantes.

---

ID: RF-010

Título: Informar apelido

Descrição: O sistema deve aceitar um apelido opcional quando o participante entrar na sala.

Ator: Participante

Critérios de aceite: Dado que o participante informou um apelido de até 30 caracteres, quando ele entra na sala, então o sistema aceita a entrada. Dado que ele não informou apelido, quando ele entra na sala, então o sistema aceita a entrada do mesmo jeito.

---

ID: RF-011

Título: Limitar participantes

Descrição: O sistema deve recusar novas entradas quando a sala atingir o limite de participantes.

Ator: Participante

Critérios de aceite: Dado que a sala tem 200 participantes (limite padrão), quando outro usuário tenta entrar, então o sistema responde `ROOM_FULL` e não registra o usuário.

---

ID: RF-012

Título: Recusar entrada em sala encerrada

Descrição: O sistema deve recusar a entrada quando a sala estiver encerrada.

Ator: Participante

Critérios de aceite: Dado que a sala tem status `ended`, quando um usuário tenta entrar, então o sistema responde `ROOM_ENDED` e não registra o usuário.

---

ID: RF-013

Título: Enviar reação

Descrição: O sistema deve registrar uma reação quando o participante enviar um emoji da paleta.

Ator: Participante

Critérios de aceite: Dado que a sala está ativa e o emoji está na paleta, quando o participante envia a reação com o `participantToken`, então o sistema responde `202` com o horário `nextAllowedAt` e conta a reação no resumo da sala.

---

ID: RF-014

Título: Aplicar intervalo entre reações

Descrição: O sistema deve recusar a reação quando o participante reagir antes do fim do intervalo da sala.

Ator: Participante

Critérios de aceite: Dado que o participante reagiu há menos tempo que o intervalo da sala, quando ele envia outra reação, então o sistema responde `COOLDOWN_ACTIVE` (HTTP 429) com `retryAfterSeconds` e não conta a reação.

---

ID: RF-015

Título: Validar emoji permitido

Descrição: O sistema deve recusar a reação quando o emoji não estiver na paleta da sala.

Ator: Participante

Critérios de aceite: Dado que o emoji não está na paleta, quando o participante envia a reação, então o sistema responde `EMOJI_NOT_ALLOWED` e não conta a reação.

---

ID: RF-016

Título: Exigir token para reagir

Descrição: O sistema deve recusar a reação quando o pedido não tiver um `participantToken` válido da sala.

Ator: Participante

Critérios de aceite: Dado que o pedido não tem token, ou tem um token de outra sala, quando o participante envia a reação, então o sistema responde `UNAUTHORIZED` e não conta a reação.

---

ID: RF-017

Título: Manter reações anônimas

Descrição: O sistema deve ocultar a identidade de quem reagiu quando o sistema enviar reações e resumos aos participantes.

Ator: Participante

Critérios de aceite: Dado que um participante enviou uma reação, quando o sistema envia `reaction.created` ou o resumo aos outros participantes, então a mensagem não contém `participantId` nem apelido.

---

ID: RF-018

Título: Mostrar reações em tempo real

Descrição: O sistema deve enviar cada reação aos participantes conectados quando o sistema aceitar a reação.

Ator: Participante

Critérios de aceite: Dado que o participante está conectado ao WebSocket da sala, quando outro participante reage, então o participante conectado recebe o evento `reaction.created` com o emoji e o horário.

---

ID: RF-019

Título: Consultar resumo por janela

Descrição: O sistema deve devolver o resumo dos emojis quando o usuário consultar uma janela de tempo.

Ator: Participante

Critérios de aceite: Dado que a sala tem reações, quando o usuário consulta a janela `30s`, `1m`, `5m` ou `all`, então o sistema devolve o total, o ranking em ordem decrescente com contagem e porcentagem, e o emoji dominante. Dado que não houve reações na janela, então o emoji dominante é `null` e o ranking é vazio.

---

ID: RF-020

Título: Atualizar resumo em tempo real

Descrição: O sistema deve enviar o resumo atualizado aos participantes conectados quando as reações mudarem o resumo.

Ator: Participante

Critérios de aceite: Dado que o participante está conectado ao WebSocket, quando a sala recebe reações, então o participante recebe `summary.updated` com as janelas `30s`, `1m` e `5m`, e no máximo 1 mensagem a cada 2 segundos.

---

ID: RF-021

Título: Mostrar feedback lúdico

Descrição: O cliente deve mostrar os emojis mais usados de forma animada quando o cliente receber o resumo ou uma reação.

Ator: Participante

Critérios de aceite: Dado que o participante está na sala, quando chega uma reação, então o cliente anima o emoji na tela. Dado que chega um resumo, então o cliente mostra o ranking e destaca o emoji dominante. Dado que o sistema operacional pede "reduzir movimento", então o cliente mostra o mesmo feedback sem animação de movimento.

---

ID: RF-022

Título: Escolher janela de tempo

Descrição: O cliente deve permitir a escolha da janela do resumo quando o participante quiser ver outro período.

Ator: Participante

Critérios de aceite: Dado que o participante está na sala, quando ele escolhe `30s`, `1m` ou `5m`, então o cliente mostra o ranking desta janela.

---

ID: RF-023

Título: Mostrar número de participantes

Descrição: O sistema deve informar o número de participantes quando alguém entrar na sala ou sair da sala.

Ator: Participante

Critérios de aceite: Dado que o participante está conectado, quando outro participante entra ou sai, então o participante recebe `participants.changed` com o novo número.

---

ID: RF-024

Título: Estender a sala

Descrição: O sistema deve somar minutos ao fim da sala quando o criador pedir a extensão.

Ator: Criador

Critérios de aceite: Dado que a sala está ativa e o pedido tem o `hostToken` e `extraMinutes` entre 5 e 60, quando o criador pede a extensão, então o sistema soma os minutos ao horário de fim, devolve a sala atualizada e envia `room.extended` aos participantes.

---

ID: RF-025

Título: Limitar duração total

Descrição: O sistema deve recusar a extensão quando a duração total passar de 480 minutos.

Ator: Criador

Critérios de aceite: Dado que a duração total é de 470 minutos, quando o criador pede 15 minutos a mais, então o sistema responde `DURATION_LIMIT_EXCEEDED` e não muda o horário de fim.

---

ID: RF-026

Título: Restringir extensão ao criador

Descrição: O sistema deve recusar a extensão quando o pedido não tiver o `hostToken` da sala.

Ator: Criador

Critérios de aceite: Dado que o pedido não tem token, ou tem um `participantToken`, quando alguém pede a extensão, então o sistema responde `UNAUTHORIZED` e não muda o horário de fim.

---

ID: RF-027

Título: Avisar o fim próximo da sala

Descrição: O sistema deve avisar os participantes quando faltarem 60 segundos para o fim da sala.

Ator: Participante

Critérios de aceite: Dado que a sala está ativa, quando faltam 60 segundos para o fim, então os participantes conectados recebem `room.expiring` com `secondsLeft` igual a 60. Dado que o criador estende a sala depois do aviso, então o sistema envia `room.extended`.

---

ID: RF-028

Título: Encerrar a sala pelo criador

Descrição: O sistema deve encerrar a sala quando o criador pedir o encerramento.

Ator: Criador

Critérios de aceite: Dado que o pedido tem o `hostToken`, quando o criador encerra a sala, então o sistema muda o status para `ended` com `endedReason = host`, devolve o resumo final e envia `room.closed` aos participantes. Dado que a sala já está encerrada, quando o criador repete o pedido, então o sistema responde com sucesso e não muda nada.

---

ID: RF-029

Título: Encerrar a sala por tempo limite

Descrição: O sistema deve encerrar a sala quando o horário de fim chegar.

Ator: Sistema

Critérios de aceite: Dado que a sala está ativa, quando o horário de fim chega, então o sistema muda o status para `ended` com `endedReason = timeout` e envia `room.closed` aos participantes.

---

ID: RF-030

Título: Fechar conexões ao encerrar

Descrição: O sistema deve fechar as conexões WebSocket quando o sistema enviar `room.closed`.

Ator: Sistema

Critérios de aceite: Dado que a sala foi encerrada, quando o sistema envia `room.closed`, então o sistema fecha todas as conexões WebSocket da sala.

---

ID: RF-031

Título: Mostrar resumo final

Descrição: O sistema deve mostrar o resumo final da sala quando a sala estiver encerrada.

Ator: Participante

Critérios de aceite: Dado que a sala está encerrada, quando o usuário abre o link ou consulta o resumo com `window = all`, então o sistema devolve o resumo de toda a sala e o cliente mostra o estado "encerrada".

---

ID: RF-032

Título: Remover sala depois da retenção

Descrição: O sistema deve remover os dados da sala quando passarem 24 horas do encerramento.

Ator: Sistema

Critérios de aceite: Dado que a sala terminou há mais de 24 horas, quando o usuário consulta a sala, então o sistema responde `ROOM_GONE` (HTTP 410).

---

ID: RF-033

Título: Mostrar controles do criador

Descrição: O cliente deve mostrar os controles Estender e Encerrar quando o dispositivo tiver o `hostToken` da sala.

Ator: Criador

Critérios de aceite: Dado que o dispositivo guardou o `hostToken` do código da sala, quando o usuário abre a sala, então o cliente mostra os botões Estender e Encerrar. Dado que o dispositivo não tem o token, então o cliente não mostra estes botões.

---

ID: RF-034

Título: Retomar o controle da sala

Descrição: O sistema deve permitir que o criador volte a controlar a sala quando ele usar o link de administração.

Ator: Criador

Critérios de aceite: Dado que o criador perdeu o armazenamento local, quando ele abre o link de administração da sala com o `hostToken`, então o cliente guarda o token e mostra os controles do criador.

---

ID: RF-035

Título: Manter a sala sem o criador

Descrição: O sistema deve manter a sala ativa quando o criador sair da sala.

Ator: Participante

Critérios de aceite: Dado que o criador desconectou, quando a sala ainda tem tempo, então a sala continua ativa e os participantes continuam a reagir.
