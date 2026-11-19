# Prompt — Front-end do Termofeed

## Papel

Você é um engenheiro front-end sênior e designer de produto. Implemente o front-end completo do **Termofeed**.

O Termofeed é um "termômetro de humor". Ele funciona como uma sala do Google Meet, mas sem vídeo. Os participantes enviam emojis que mostram o humor deles. O app mostra, de forma lúdica, os emojis mais usados em uma janela de tempo.

Slogan: **o humor da sala, em tempo real**.

Este trabalho é do repositório **`termofeed-front`**. O back-end (em Go) fica em outro repositório, o **`termofeed-back`**. Não coloque código do back-end aqui.

## Fonte da verdade

Leia todos os arquivos do diretório `cnt` antes de escrever código:

- `requisitos-funcionais.md` (RF-001 a RF-035): atenda aos requisitos que o cliente cumpre.
- `requisitos-nao-funcionais.md` (RNF-001 a RNF-024): atenda aos que se aplicam ao cliente (RNF-008, RNF-014 a RNF-022).
- `openapi.yaml` e `asyncapi.yaml`: contratos da API REST e dos eventos WebSocket.
- Arquivos PNG de logo: use-os como referência visual. Recrie o ícone em **SVG** limpo para uso no app (não use os PNGs no app final, exceto o ícone do app e o favicon).

Regras:

1. Não mude o contrato. Gere os tipos TypeScript a partir do `openapi.yaml` (por exemplo, com `openapi-typescript`). Escreva à mão os tipos dos eventos WebSocket, a partir do `asyncapi.yaml`.
2. Se você achar um erro ou uma lacuna no contrato, registre em `docs/DUVIDAS.md` e siga a opção mais simples.

## Stack

- **React 18 + TypeScript + Vite.**
- Roteamento: `react-router`. Rotas: `/` (criar sala) e `/r/:code` (sala).
- Estilo: **Tailwind CSS** com variáveis CSS para as cores do tema.
- Animação: **Framer Motion** (ou CSS puro, se for mais leve).
- Dados: `fetch` com um cliente fino em `src/api/`. Para o estado do servidor, use `@tanstack/react-query` apenas se ajudar.
- Testes: Vitest + Testing Library. Teste de ponta a ponta: Playwright.
- Fontes: **Fredoka** (títulos e números) e **Space Grotesk** (texto). Hospede as fontes no projeto (pacote `@fontsource`) para não depender de rede externa.
- Em desenvolvimento, use o proxy do Vite para o back-end em `http://localhost:8080`.
- A URL base da API vem de `VITE_API_URL`. O valor padrão é **`/api/v1`** (mesma origem). Em produção, o proxy do próprio front encaminha `/api/v1` ao back-end, então não existe CORS.
- A URL do WebSocket é derivada da página: `wss://<host>/api/v1/rooms/{code}/ws` em HTTPS e `ws://` em HTTP.

Mantenha o tamanho inicial de JavaScript, CSS e fontes em até **300 KB** comprimido (RNF-019).

## Direção de design

**Palavras-chave:** limpo, leve, calmo, divertido, lúdico.

### Princípios

1. **Pouca sobrecarga visual.** Cada tela tem uma ação principal. Muito espaço em branco. No máximo 3 níveis de texto.
2. **Lúdico sem bagunça.** O divertimento vem dos emojis, do termômetro e das animações suaves. Não vem de muitos enfeites.
3. **Um elemento herói por tela.** Na tela da sala, o herói é o termômetro de humor e a paleta de emojis.
4. **Cantos arredondados, formas macias, sombras discretas.**
5. **Nada de painéis cheios de números.** Mostre o humor com formas e emojis. Os números ficam pequenos e secundários.

### Cores (do logo)

| Uso | Hex |
|---|---|
| Fundo | `#FFF6EA` (creme) |
| Texto e contornos | `#1B2340` (azul-marinho) |
| Destaque principal | `#E8442F` (coral) |
| Destaque secundário | `#FFB627` (amarelo) |
| Superfícies | `#FFFFFF` |

Use o gradiente amarelo → coral só no termômetro e em um botão principal por tela. Contraste mínimo de 4,5:1 para texto normal. Texto coral sobre creme só em tamanho grande, com a variante `#D63A26`.

### Tipografia

- Logo e títulos: Fredoka 600.
- Texto e rótulos: Space Grotesk 500 ou 400.
- Poucos tamanhos: 14, 16, 20, 32 e 56 px.

### Movimento

- Emojis recebidos **flutuam** de baixo para cima, com leve balanço, e somem em cerca de 2,5 s. Limite de 40 emojis ao mesmo tempo na tela. Descarte os mais antigos.
- O mercúrio do termômetro sobe e desce com suavidade conforme o humor da janela escolhida.
- O botão de emoji dá um pequeno "pulo" ao toque.
- Depois de reagir, o botão mostra um **anel de espera** que enche durante o intervalo da sala. Não bloqueie a tela inteira.
- Quando o emoji dominante muda, ele cresce um pouco e volta.
- Respeite `prefers-reduced-motion`: sem movimento, mostre só as trocas de estado (RNF-015).
- Mantenha 50 quadros por segundo ou mais com 20 reações por segundo (RNF-020).

## Telas

### 1. Início (`/`)

- Logo grande no centro, com o slogan.
- Um cartão simples "Criar sala" com:
  - nome da sala (opcional);
  - duração (seleção rápida: 15 min, 30 min, 1 h, 2 h, ou valor livre de 5 a 240 min);
  - intervalo entre reações (controle deslizante de 1 a 60 s, padrão 5 s);
  - paleta de emojis (a padrão já vem marcada; permita trocar de 2 a 12 emojis em uma linha de opções);
  - botão principal "Criar sala".
- Mostre as opções avançadas (intervalo e paleta) dobradas por padrão, para manter a tela limpa.
- Um campo pequeno "Já tem um link? Cole aqui" leva à sala.
- Erros de validação aparecem ao lado do campo, em português.

### 2. Sala criada (logo depois de criar)

- Mostre o link de entrada com botão **Copiar**. Confirme com um aviso discreto.
- Mostre o **link de administração**, com a explicação em uma frase: "Guarde este link. Ele devolve o controle da sala se você trocar de dispositivo." (RF-034)
- Botão "Entrar na sala".
- Guarde o `hostToken` no armazenamento local, por código de sala (RF-033).

### 3. Sala (`/r/:code`) — a tela principal

Ao abrir:

1. Consulte a sala (`GET /rooms/{code}`). Em `404`, `410` ou sala encerrada, mostre a tela de estado correspondente.
2. Chame `join` com um apelido opcional (um campo pequeno e amigável: "Como quer ser chamado? (opcional)"). Guarde o `participantToken` por código de sala.
3. Abra o WebSocket e mostre a sala.

Layout (celular em primeiro lugar):

- **Topo:** nome da sala, tempo restante em contagem regressiva suave (`mm:ss`), número de participantes.
- **Centro (herói):** o **termômetro de humor**. O mercúrio mostra o humor da janela escolhida. A carinha no bulbo muda com o emoji dominante. Emojis flutuam ao redor.
- **Abaixo do termômetro:** um **seletor de janela** com 3 opções (30 s, 1 min, 5 min) e um **ranking curto** com os 3 emojis mais usados, em barras arredondadas simples.
- **Rodapé fixo:** a **paleta de emojis** em botões grandes (no mínimo 44×44 px, ideal 56 px), com o anel de espera.
- **Controles do criador** (só quando o dispositivo tem o `hostToken`): um botão discreto de menu abre uma folha inferior com **Estender** (opções de +5, +15 e +30 min) e **Encerrar sala** (com confirmação em duas etapas).

Comportamentos:

- Ao receber `room.expiring`, mostre um aviso suave: "A sala termina em 1 minuto". O criador vê o atalho "Estender".
- Ao receber `room.extended`, atualize o relógio e mostre "Mais X minutos!".
- Ao receber `room.closed`, vá para a tela de resumo final.
- Em `COOLDOWN_ACTIVE`, não mostre erro vermelho. Mostre o anel de espera e ignore o toque.
- Em perda de conexão, mostre uma faixa discreta "Reconectando…". Reconecte em até 5 s com recuo exponencial (máximo de 10 s) e busque o resumo por REST (RNF-008).
- Em `ROOM_FULL`, mostre uma tela amigável com a explicação.

### 4. Sala encerrada / resumo final

- Título "A sala terminou".
- O termômetro congelado, com o **emoji campeão** em destaque.
- Ranking completo da sala (`window = all`), em barras simples.
- Botão "Criar nova sala".
- Para `ROOM_GONE`: "Esta sala não existe mais".

### 5. Estados de erro

- Sala não encontrada (`ROOM_NOT_FOUND`), sala removida (`ROOM_GONE`), sala cheia, sem conexão com o servidor.
- Cada estado tem um desenho simples (o termômetro com outra expressão), uma frase curta e uma ação.

## Textos

- Português do Brasil (RNF-022). Frases curtas, tom simpático e leve. Sem gíria pesada.
- Todos os textos ficam em um arquivo único (`src/i18n/pt-BR.ts`) para facilitar a revisão.
- Mensagens de erro: use o `code` do contrato para escolher o texto. Nunca mostre o `message` bruto do servidor.

## Acessibilidade (obrigatório)

- Contraste mínimo de 4,5:1 para texto normal e 3:1 para texto grande (RNF-014).
- Alvos de toque de no mínimo 44×44 px.
- Todas as ações funcionam só com o teclado. Mostre foco visível (RNF-016).
- Use `<button>`, `<a>` e `<label>` reais. Todo botão só com ícone ou emoji tem `aria-label` (por exemplo, "Enviar reação 😂").
- Anuncie mudanças importantes com `aria-live="polite"` (por exemplo, "A sala termina em 1 minuto"). Não anuncie cada emoji que chega.
- O termômetro tem uma alternativa em texto: "Humor da sala no último minuto: 😂 domina, 50%".
- Não dependa só de cor para mostrar informação.

## Desempenho

- Divida o código por rota (`React.lazy`).
- Anime com `transform` e `opacity`.
- Agrupe as atualizações de tela para no máximo 1 por quadro.
- Tamanho inicial de até 300 KB comprimido. LCP de até 2,5 s em 4G simulado (RNF-019).

## Contêiner e hospedagem (obrigatório)

Este repositório contém o **ambiente completo de produção**: o front-end e o back-end sobem juntos com um único `docker compose`. O objetivo é que uma pessoa hospede o Termofeed inteiro com poucos comandos.

### Imagem do front-end

- `Dockerfile` multi-stage:
  1. Etapa de compilação: Node LTS, `npm ci` e `npm run build`.
  2. Etapa final: imagem oficial do **Caddy** com os arquivos de `dist/` e o `Caddyfile`.
- O Caddy faz três coisas:
  1. Serve os arquivos estáticos. Responda com `index.html` para qualquer rota do app (`try_files {path} /index.html`).
  2. Encaminha `/api/*` ao back-end em `http://back:8080` (inclui o WebSocket, que o Caddy já encaminha sem configuração extra).
  3. Emite e renova o **certificado HTTPS** sozinho, com o domínio da variável `DOMAIN`.
- Cabeçalhos de cache: arquivos com hash no nome (`/assets/*`) com `Cache-Control: public, max-age=31536000, immutable`. O `index.html` com `no-cache`.
- Ative compressão (`encode zstd gzip`).
- Cabeçalhos de segurança: `Strict-Transport-Security`, `X-Content-Type-Options: nosniff`, `Referrer-Policy: no-referrer` e uma `Content-Security-Policy` restritiva. A política permite só a mesma origem, e `wss:` para o WebSocket.
- Em desenvolvimento local (`localhost`), o Caddy deve funcionar sem HTTPS público. Use `DOMAIN=localhost` e a configuração padrão de HTTP local.

### docker-compose.yml (desenvolvimento local)

- Serviço `front` com build local e porta `5173:80`, e o serviço `back` ligado à imagem do back-end.
- O serviço `back` usa a variável `BACK_IMAGE` (padrão: `ghcr.io/<dono>/termofeed-back:latest`). Como alternativa, permita um build a partir de `../termofeed-back` por meio de um arquivo de sobreposição `docker-compose.override.yml.example`.

### docker-compose.prod.yml (hospedagem)

Serviços:

| Serviço | Imagem | Porta pública | Observações |
|---|---|---|---|
| `front` | build deste repositório (Caddy) | `80` e `443` | Único serviço exposto ao público. Guarda os certificados no volume `caddy_data`. |
| `back` | `${BACK_IMAGE}` | nenhuma | Fala só pela rede interna do Compose. |

Regras:

1. `restart: unless-stopped` nos dois serviços.
2. `healthcheck` nos dois. O `front` só sobe depois que o `back` fica saudável (`depends_on` com `condition: service_healthy`).
3. Rede interna dedicada. O `back` não publica portas.
4. Volumes nomeados: `caddy_data` e `caddy_config`.
5. `read_only: true`, `no-new-privileges` e limites de memória em cada serviço, quando o serviço permitir.
6. Variáveis vindas de `.env` (veja `.env.example`):

| Variável | Exemplo | Função |
|---|---|---|
| `DOMAIN` | `termofeed.exemplo.com` | Domínio público. O Caddy emite o certificado para ele. |
| `BACK_IMAGE` | `ghcr.io/<dono>/termofeed-back:latest` | Imagem do back-end. |
| `ACME_EMAIL` | `voce@exemplo.com` | E-mail para o certificado. |

   O Compose repassa ao `back` estes valores, derivados de `DOMAIN`:
   `PUBLIC_BASE_URL=https://${DOMAIN}`, `ALLOWED_ORIGINS=https://${DOMAIN}`, `TRUSTED_PROXY=true`.

7. Comandos documentados no `README.md`:
   - subir: `docker compose -f docker-compose.prod.yml up -d --build`
   - ver logs: `docker compose -f docker-compose.prod.yml logs -f`
   - atualizar: `docker compose -f docker-compose.prod.yml pull && docker compose -f docker-compose.prod.yml up -d`
   - parar: `docker compose -f docker-compose.prod.yml down`

### Verificação

Antes de terminar, confirme em um teste real com Docker:

1. `docker compose -f docker-compose.prod.yml up -d --build` sobe os dois serviços saudáveis.
2. `https://<DOMAIN>/` (ou `http://localhost` em teste local) mostra a tela inicial.
3. Criar sala, entrar e reagir funcionam pelo proxy, **incluindo o WebSocket**.
4. `docker compose ps` mostra o `back` sem porta publicada.

### Integração contínua

Adicione `.github/workflows/ci.yml` com: `npm ci`, verificação de tipos, testes unitários, build, testes de acessibilidade e a construção da imagem. Publique a imagem em `ghcr.io/<dono>/termofeed-front` na branch `main`.

## Estrutura sugerida

```
src/
  api/            # cliente REST e tipos gerados
  ws/             # cliente WebSocket com reconexão
  features/
    create-room/
    room/         # termômetro, paleta, ranking, controles do criador
    summary/
  components/     # botão, cartão, folha inferior, aviso, logo SVG
  i18n/pt-BR.ts
  styles/         # tokens de tema (variáveis CSS)
  routes/
docs/
```

## Testes

1. **Unitários:** formatação do tempo, lógica do anel de espera, ranking, reconexão.
2. **Componentes:** paleta, termômetro, seletor de janela, controles do criador (aparecem só com `hostToken`).
3. **Ponta a ponta (Playwright):** criar sala → copiar link → entrar com outro contexto → reagir → ver o resumo mudar → estender → encerrar → ver a tela final. Use o back-end real ou um servidor simulado fiel ao contrato.
4. **Acessibilidade:** rode `axe` em todas as telas, sem falhas.
5. **Visual:** capturas em 320, 390, 768, 1280 e 1920 px, sem rolagem horizontal (RNF-018).
6. Navegadores: Chrome, Firefox, Safari (WebKit) e Edge (RNF-017).

## Entregáveis

1. Projeto completo que roda com `npm install && npm run dev`.
2. `Dockerfile`, `Caddyfile`, `.dockerignore`, `docker-compose.yml`, `docker-compose.prod.yml` e `.env.example`, como descrito em "Contêiner e hospedagem".
3. `.github/workflows/ci.yml`.
4. `README.md` com: como rodar (com e sem Docker), como hospedar em produção, variáveis (`VITE_API_URL`, `DOMAIN`, `BACK_IMAGE`, `ACME_EMAIL`) e como gerar os tipos da API.
5. Ícone, favicon e ícone do app a partir do logo (`manifest.webmanifest` simples).
6. `docs/DECISOES.md` e `docs/DUVIDAS.md`.
7. `docs/RASTREABILIDADE.md`: tabela que liga cada RF e RNF aplicável ao componente e ao teste que o cobre.

## Como trabalhar

1. Leia `cnt` e escreva um plano curto. Depois comece.
2. Comece pelos tokens de tema e pelos componentes base. Depois faça as telas na ordem: Sala → Início → Sala criada → Resumo → Erros.
3. Faça o termômetro e a animação dos emojis com cuidado extra. Eles são a alma do produto.
4. Rode os testes a cada etapa.
5. Escreva a documentação e os comentários em português do Brasil, com frases curtas.

## Critério de pronto

- Os fluxos dos RF aplicáveis funcionam com o back-end real, de ponta a ponta.
- Não há falhas de acessibilidade no `axe`.
- O tamanho inicial fica em até 300 KB comprimido.
- `docker compose -f docker-compose.prod.yml up -d --build` sobe front e back saudáveis, com HTTPS pelo Caddy, e o WebSocket funciona pelo proxy.
- Só o serviço `front` expõe portas ao público.
- A tela parece **limpa, leve e divertida**: uma ação principal por tela, bastante espaço em branco, o termômetro como herói, e emojis que flutuam com suavidade.
- `docs/RASTREABILIDADE.md` mostra os requisitos aplicáveis cobertos, ou lista o que ficou de fora e por quê.
