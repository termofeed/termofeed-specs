# Prompt — Back-end do Termofeed (Go)

## Papel

Você é um engenheiro back-end sênior de Go. Implemente o back-end completo do **Termofeed**.

Este trabalho é do repositório **`termofeed-back`**. O front-end fica em outro repositório, o **`termofeed-front`**. Não coloque código de front-end aqui.

O Termofeed é um "termômetro de humor". Ele funciona como uma sala do Google Meet, mas sem vídeo. Os participantes enviam emojis que mostram o humor deles. O sistema mostra os emojis mais usados em uma janela de tempo.

## Fonte da verdade

Leia todos os arquivos do diretório `cnt` antes de escrever código:

- `requisitos-funcionais.md`: 35 requisitos (RF-001 a RF-035).
- `requisitos-nao-funcionais.md`: 24 requisitos (RNF-001 a RNF-024).
- `openapi.yaml`: contrato REST (OpenAPI 3.1).
- `asyncapi.yaml`: contrato dos eventos WebSocket (AsyncAPI 3.0).
- Os arquivos de logo não são necessários para o back-end.

Regras:

1. O contrato é a fonte da verdade. Não mude nomes de campos, códigos de erro nem códigos HTTP.
2. Se você achar um erro ou uma lacuna no contrato, **pare e liste o problema** em `docs/DUVIDAS.md`. Siga a opção mais simples e documente a escolha.
3. Implemente todos os requisitos funcionais. Atenda aos requisitos não funcionais que se aplicam ao servidor (RNF-001 a RNF-013, RNF-023 e RNF-024).

## Stack obrigatória

- **Go 1.22 ou superior.**
- Roteador: biblioteca padrão `net/http` (com `ServeMux` e padrões de método) ou `chi`.
- WebSocket: `github.com/coder/websocket`.
- Tipos da API: gere a partir do `openapi.yaml` com `oapi-codegen`. Não escreva à mão os tipos que o gerador cria.
- Testes: pacote `testing` e `testify`.
- Logs: `log/slog` em formato JSON.
- Métricas: `prometheus/client_golang`, em `GET /metrics`.
- Configuração: variáveis de ambiente.
- Contêiner: `Dockerfile` multi-stage com imagem final mínima.

Não adicione outras dependências sem motivo. Explique cada dependência extra em `docs/DECISOES.md`.

## Arquitetura

Use camadas simples e interfaces pequenas. Estrutura sugerida:

```
cmd/termofeed/main.go        # ponto de entrada, config e desligamento seguro
internal/
  api/                       # handlers HTTP, middlewares, erros (código gerado em api/gen)
  ws/                        # hub WebSocket, conexões, eventos
  room/                      # regras de negócio: salas, participantes, reações, tempo
  summary/                   # cálculo do resumo por janela
  store/                     # interface Store e implementação em memória
  clock/                     # interface Clock (para testes com tempo falso)
  ratelimit/                 # limitador por IP
  config/                    # leitura das variáveis de ambiente
docs/
Makefile
Dockerfile
```

Decisões de projeto:

1. **Armazenamento em memória** na versão 1.0, atrás da interface `Store`. Troque por Redis ou banco no futuro sem mudar as regras de negócio.
2. **Relógio injetável** (`Clock`). Todo teste de tempo usa um relógio falso. Nunca use `time.Sleep` em teste.
3. **Concorrência segura.** Use `sync.RWMutex` ou um canal por sala. Rode os testes com `-race`.
4. **Uma goroutine por sala** pode cuidar do tempo: aviso `room.expiring` aos 60 s, encerramento no fim e atualização do resumo. Alternativa: um agendador central. Escolha uma e documente.
5. **Desligamento seguro** (`SIGTERM`): pare de aceitar conexões, envie fechamento aos WebSockets e espere até 10 s.

## Regras de negócio que não podem falhar

- Código da sala no formato `abc-defg-hij`, gerado com `crypto/rand`. Verifique colisão.
- `hostToken` e `participantToken`: no mínimo 128 bits, gerados com `crypto/rand`, em codificação URL-safe. Compare tokens em tempo constante (`subtle.ConstantTimeCompare`). Guarde só o hash dos tokens, se possível.
- O criador também é um participante. O `hostToken` só dá o poder de estender e encerrar.
- Intervalo entre reações: por participante, definido na sala. Em violação, responda `429` com `Retry-After` e `retryAfterSeconds`.
- Resumo: janelas `30s`, `1m`, `5m` e `all`. O ranking fica em ordem decrescente. Em empate, ordene pela ordem da paleta da sala. O emoji dominante é `null` quando não há reações.
- `summary.updated`: no máximo 1 mensagem a cada 2 s por sala (agrupe as mudanças).
- Extensão: de 5 a 60 minutos por pedido. Duração total máxima de 480 minutos.
- Encerramento: idempotente. Depois de `room.closed`, feche todas as conexões da sala.
- Retenção: 24 h depois do encerramento a sala responde `410`, e o sistema apaga os dados em até 1 h.
- Reações anônimas: nenhum evento nem resumo contém `participantId` ou apelido.
- Privacidade: não guarde e-mail, nome real nem identificador de dispositivo. O IP só aparece nos registros de acesso, por no máximo 7 dias.

## Segurança e robustez

- Aceite WebSocket só com `participantToken` válido (parâmetro `token`) e da mesma sala.
- Limite de requisições por IP: 120 por minuto, e 10 salas criadas por hora. Resposta `429` com `Retry-After`. Leia o IP real com cuidado quando houver proxy (configurável).
- Limite o tamanho do corpo das requisições (por exemplo, 8 KB).
- Valide todo dado de entrada. Use os limites do contrato.
- Configure CORS por variável de ambiente (`ALLOWED_ORIGINS`).
- Configure `ReadHeaderTimeout`, `ReadTimeout` e `IdleTimeout` no servidor HTTP.
- Use ping/pong no WebSocket e feche conexões mortas. Limite o tamanho da fila de envio por conexão. Se a fila encher, feche a conexão lenta.
- Todo erro devolve o formato `Problem` do contrato (`code`, `message`, e campos extras quando existirem).

## Variáveis de ambiente

| Variável | Padrão | Função |
|---|---|---|
| `PORT` | `8080` | Porta HTTP. |
| `PUBLIC_BASE_URL` | `http://localhost:5173` | Base do link de entrada (`/r/<código>`). Em produção: `https://<seu-domínio>`. |
| `ALLOWED_ORIGINS` | `http://localhost:5173` | Origens permitidas (CORS e WebSocket). Em produção: `https://<seu-domínio>`. |
| `MAX_PARTICIPANTS` | `200` | Limite padrão por sala. |
| `RETENTION_HOURS` | `24` | Retenção depois do encerramento. |
| `LOG_LEVEL` | `info` | Nível de log. |
| `TRUSTED_PROXY` | `false` | Usa `X-Forwarded-For` quando `true`. Em produção: `true`, porque o proxy do front fica na frente. |

Documente todas as variáveis em um arquivo `.env.example`.

## Contêiner e hospedagem (obrigatório)

O back-end deve rodar em contêiner e deve ser fácil de hospedar.

### Endpoints de saúde

- `GET /healthz`: responde `200` quando o processo está vivo.
- `GET /readyz`: responde `200` quando o servidor está pronto para receber tráfego.
- Estes endpoints não exigem token e não entram no `openapi.yaml` de negócio. Documente-os no `README.md`.

### Dockerfile

- Multi-stage: etapa de compilação com a imagem oficial do Go e etapa final mínima (`gcr.io/distroless/static` ou `scratch`).
- Compile com `CGO_ENABLED=0` e `-trimpath`. Use `-ldflags "-s -w"`.
- Execute como **usuário sem privilégios** (não root).
- Use `HEALTHCHECK` com o próprio binário (por exemplo, `termofeed healthcheck`), porque a imagem final não tem `curl`.
- Imagem final com menos de 30 MB.
- Adicione um `.dockerignore`.

### docker-compose.yml (neste repositório)

Serve para rodar o back-end sozinho, em desenvolvimento e em testes.

- Serviço `back` com build local, porta `8080:8080`, `env_file: .env` e `healthcheck`.
- `restart: unless-stopped`.
- Sistema de arquivos somente leitura (`read_only: true`) e `security_opt: ["no-new-privileges:true"]`.
- Limites de recursos razoáveis (por exemplo, 512 MB de memória).

### Imagem para hospedagem

- O `termofeed-front` roda o ambiente completo de produção, com os dois serviços (veja o prompt do front-end). Por isso, este repositório deve gerar uma imagem que o outro repositório consiga usar.
- Adicione um workflow do GitHub Actions (`.github/workflows/ci.yml`) que: roda `go vet`, `go test -race ./...`, valida o contrato, constrói a imagem e publica em `ghcr.io/<dono>/termofeed-back` com as etiquetas `latest` e o hash curto do commit. A publicação só acontece na branch `main`.
- No `README.md`, explique como usar a imagem publicada e como construir a imagem localmente.
- O serviço `back` **não** expõe porta ao público em produção. Só o proxy do front fala com ele, pela rede interna do Compose, em `http://back:8080`.

## Testes (obrigatórios)

1. **Unitários** das regras: tempo, intervalo, resumo, extensão, limites.
2. **Integração** com `httptest` e um cliente WebSocket: criar sala → entrar → reagir → receber eventos → estender → encerrar.
3. **Contrato:** valide as respostas contra o `openapi.yaml` (por exemplo, com `kin-openapi`). Valide os eventos contra o `asyncapi.yaml` com esquemas JSON derivados.
4. **Carga:** um teste simples (`go test -bench` ou um programa em `cmd/loadtest`) com 200 participantes em uma sala e 20 reações por segundo. Meça o p95 da latência (RNF-001).
5. Rode tudo com `go test -race ./...`. Cobertura mínima de 80% em `internal/room` e `internal/summary`.

## Entregáveis

1. Código completo em Go, que compila com `go build ./...`.
2. `Makefile` com os alvos: `run`, `test`, `lint`, `gen` (gerar código da API), `build` e `docker`.
3. `Dockerfile`, `.dockerignore`, `docker-compose.yml` e `.env.example`, como descrito em "Contêiner e hospedagem".
4. `.github/workflows/ci.yml` com testes, validação do contrato e publicação da imagem.
5. `README.md` com: como rodar (com e sem Docker), variáveis de ambiente, endpoints de saúde e exemplos de `curl`.
6. `docs/DECISOES.md` (escolhas técnicas) e `docs/DUVIDAS.md` (lacunas do contrato).
7. `docs/RASTREABILIDADE.md`: tabela que liga cada RF e RNF ao arquivo e ao teste que o cobre.

## Como trabalhar

1. Leia `cnt` e escreva um plano curto. Depois comece.
2. Trabalhe em etapas pequenas: esqueleto → salas → participantes → reações e resumo → WebSocket → tempo e encerramento → limites e segurança → testes e documentação.
3. Rode os testes a cada etapa.
4. Use frases curtas na documentação e nos comentários. Escreva em português do Brasil. Nomes de código, de campos e de erros ficam em inglês, como no contrato.

## Critério de pronto

- Todos os RF-001 a RF-035 passam em teste automático.
- Os testes de contrato passam sem falhas.
- `go vet ./...` e `go test -race ./...` passam sem erros.
- O fluxo completo funciona com `docker compose up` e com os exemplos de `curl` do README.
- A imagem Docker constrói sem erro, roda como usuário sem privilégios, passa no `HEALTHCHECK` e tem menos de 30 MB.
- O workflow de CI roda do começo ao fim sem falhas.
- `docs/RASTREABILIDADE.md` mostra 100% dos requisitos cobertos, ou lista o que ficou de fora e por quê.
