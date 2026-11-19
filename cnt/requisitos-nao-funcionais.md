# Termofeed — Requisitos não funcionais

Versão 1.0 · 2026-11-18

Os números abaixo são valores propostos. Ajuste cada valor conforme o orçamento e o público real do produto.

Termos: **p95** é o valor abaixo do qual ficam 95% das medições. **RTT** é o tempo de ida e volta da rede.

---

ID: RNF-001

Título: Latência da reação

Descrição: O sistema deve entregar cada reação aos participantes conectados em tempo curto em condições normais de rede.

Ator: Participante

Critérios de aceite: O p95 do tempo entre o `POST /reactions` e o recebimento de `reaction.created` é de no máximo 1 segundo, em uma sala com 200 participantes conectados e 20 reações por segundo, em teste de carga de 10 minutos.

---

ID: RNF-002

Título: Latência da API

Descrição: O sistema deve responder às chamadas REST em tempo curto em carga normal.

Ator: Criador, Participante

Critérios de aceite: O p95 do tempo de resposta de cada endpoint REST é de no máximo 300 ms, sem contar o RTT da rede, com 100 salas ativas e 50 requisições por segundo, em teste de carga de 10 minutos.

---

ID: RNF-003

Título: Capacidade por sala

Descrição: O sistema deve suportar o limite de participantes em cada sala sem perda de desempenho.

Ator: Participante

Critérios de aceite: Com 200 participantes conectados à mesma sala, o RNF-001 e o RNF-002 continuam dentro dos limites, em teste de carga de 10 minutos.

---

ID: RNF-004

Título: Capacidade total

Descrição: O sistema deve suportar várias salas ativas ao mesmo tempo.

Ator: Criador, Participante

Critérios de aceite: Com 100 salas ativas e 2.000 conexões WebSocket no total, a taxa de erros 5xx é menor que 0,1% das requisições e nenhuma conexão cai sem motivo, em teste de carga de 30 minutos.

---

ID: RNF-005

Título: Disponibilidade

Descrição: O sistema deve ficar disponível durante o horário de uso.

Ator: Criador, Participante

Critérios de aceite: A disponibilidade medida é de no mínimo 99,5% em 30 dias corridos, medida por uma verificação de `GET /rooms/{code}` a cada 1 minuto.

---

ID: RNF-006

Título: Precisão do fim da sala

Descrição: O sistema deve encerrar a sala no horário de fim definido.

Ator: Criador, Participante

Critérios de aceite: A diferença entre o horário de fim (`expiresAt`) e o envio de `room.closed` é de no máximo 2 segundos, em 100 salas com horários de fim diferentes.

---

ID: RNF-007

Título: Precisão do intervalo entre reações

Descrição: O sistema deve aplicar o intervalo entre reações com precisão em cada participante.

Ator: Participante

Critérios de aceite: Em 1.000 tentativas de reação em 1 segundo de espaço entre elas, com intervalo de 5 segundos na sala, 100% das tentativas dentro do intervalo recebem `429` e 100% das tentativas depois de 5 segundos recebem `202`, com tolerância de 200 ms.

---

ID: RNF-008

Título: Reconexão automática

Descrição: O cliente deve reconectar ao WebSocket quando a conexão cair.

Ator: Participante

Critérios de aceite: Dado que a rede ficou fora por 10 segundos, quando a rede volta, então o cliente reconecta em no máximo 5 segundos e busca o resumo atual por REST. O teste usa 20 quedas simuladas e 100% devem reconectar.

---

ID: RNF-009

Título: Segurança dos tokens

Descrição: O sistema deve gerar tokens impossíveis de adivinhar quando o sistema criar uma sala ou registrar um participante.

Ator: Criador, Participante

Critérios de aceite: Cada `hostToken` e `participantToken` tem no mínimo 128 bits de aleatoriedade, gerados por um gerador criptográfico. Uma revisão de código confirma este requisito, e o teste de 1.000.000 de tokens não mostra repetição.

---

ID: RNF-010

Título: Transporte seguro

Descrição: O sistema deve proteger o tráfego com criptografia em produção.

Ator: Criador, Participante

Critérios de aceite: 100% das requisições em produção usam HTTPS e 100% das conexões usam WSS (TLS 1.2 ou superior). Uma requisição HTTP simples é redirecionada ou recusada, em teste com um scanner de TLS.

---

ID: RNF-011

Título: Proteção contra abuso

Descrição: O sistema deve limitar as requisições por origem para reduzir abuso, já que não existe login.

Ator: Criador, Participante

Critérios de aceite: Um mesmo IP pode criar no máximo 10 salas por hora e fazer no máximo 120 requisições por minuto. A requisição que passa do limite recebe `429` com `Retry-After`, em teste com 200 requisições em 1 minuto.

---

ID: RNF-012

Título: Privacidade dos dados

Descrição: O sistema deve guardar o mínimo de dados pessoais em todas as salas.

Ator: Participante

Critérios de aceite: O banco de dados não guarda e-mail, nome real, telefone nem identificador de dispositivo. O endereço IP aparece só nos registros de acesso por no máximo 7 dias. Uma auditoria do esquema de dados confirma este requisito.

---

ID: RNF-013

Título: Retenção dos dados

Descrição: O sistema deve apagar os dados da sala depois do período de retenção.

Ator: Participante

Critérios de aceite: Em até 1 hora depois de 24 horas do encerramento, 100% dos dados da sala (reações, participantes e tokens) estão apagados, em teste com 50 salas encerradas.

---

ID: RNF-014

Título: Acessibilidade visual

Descrição: O cliente deve ter contraste e tamanho de alvo suficientes em todas as telas.

Ator: Participante, Criador

Critérios de aceite: O contraste de cor é de no mínimo 4,5:1 para texto normal e 3:1 para texto grande, e todo botão e emoji clicável tem no mínimo 44×44 px, em auditoria automática (por exemplo, axe) sem falhas e em revisão manual de todas as telas.

---

ID: RNF-015

Título: Acessibilidade de movimento

Descrição: O cliente deve reduzir as animações quando o usuário ativar "reduzir movimento".

Ator: Participante

Critérios de aceite: Com a preferência `prefers-reduced-motion: reduce` ativa, 0 animações de movimento rodam e o feedback continua visível como texto e ranking, em teste nos 4 navegadores do RNF-017.

---

ID: RNF-016

Título: Acessibilidade por teclado e leitor de tela

Descrição: O cliente deve permitir todas as ações sem mouse e com leitor de tela.

Ator: Participante, Criador

Critérios de aceite: Entrar na sala, reagir, estender e encerrar funcionam só com o teclado (Tab, Enter e Espaço). Todo botão sem texto tem `aria-label`. Um teste com um leitor de tela (NVDA ou VoiceOver) conclui os 4 fluxos sem ajuda.

---

ID: RNF-017

Título: Compatibilidade de navegadores

Descrição: O cliente deve funcionar nos navegadores comuns em todas as funções.

Ator: Participante, Criador

Critérios de aceite: Todos os casos de teste dos requisitos funcionais passam nas duas últimas versões estáveis de Chrome, Firefox, Safari e Edge, em computador e em celular.

---

ID: RNF-018

Título: Responsividade

Descrição: O cliente deve ajustar o layout a diferentes larguras de tela.

Ator: Participante, Criador

Critérios de aceite: Em larguras de 320 px a 1920 px, a tela não tem rolagem horizontal e nenhum controle fica cortado, em teste visual com 5 larguras (320, 390, 768, 1280 e 1920 px).

---

ID: RNF-019

Título: Desempenho de carregamento

Descrição: O cliente deve carregar rápido em redes móveis.

Ator: Participante

Critérios de aceite: O tamanho inicial de JavaScript, CSS e fontes é de no máximo 300 KB comprimido (gzip), e o conteúdo principal aparece em até 2,5 segundos (LCP, p75) em uma rede 4G simulada de 1,6 Mbps e 150 ms de RTT.

---

ID: RNF-020

Título: Desempenho da animação

Descrição: O cliente deve animar as reações sem travar a tela.

Ator: Participante

Critérios de aceite: Com 20 reações por segundo, a taxa de quadros é de no mínimo 50 quadros por segundo (p95) em um celular de nível médio, medida por 60 segundos no perfilador do navegador.

---

ID: RNF-021

Título: Facilidade de entrada

Descrição: O cliente deve permitir a entrada na sala com poucas ações.

Ator: Participante

Critérios de aceite: Um participante com o link entra na sala e envia a primeira reação em no máximo 2 toques ou cliques e 0 campos de texto obrigatórios. Em teste com 10 usuários novos, 9 ou mais concluem a tarefa em até 15 segundos.

---

ID: RNF-022

Título: Idioma

Descrição: O cliente deve mostrar todos os textos em português do Brasil.

Ator: Participante, Criador

Critérios de aceite: 100% dos textos da interface e das mensagens de erro aparecem em português do Brasil, em revisão de todas as telas e de todos os códigos de erro.

---

ID: RNF-023

Título: Observabilidade

Descrição: O sistema deve registrar métricas e erros para permitir o diagnóstico.

Ator: Equipe de operação

Critérios de aceite: O sistema expõe métricas de salas ativas, conexões, reações por segundo, latência (p95) e taxa de erros. Um alerta dispara em no máximo 5 minutos quando a taxa de erros 5xx passa de 1% por 5 minutos, em teste com falha simulada.

---

ID: RNF-024

Título: Aderência ao contrato

Descrição: O servidor deve seguir o contrato da API em todas as respostas.

Ator: Equipe de desenvolvimento

Critérios de aceite: 100% das respostas dos testes de contrato passam na validação contra `openapi.yaml` e `asyncapi.yaml`, a cada mudança de código, na esteira de integração contínua.
