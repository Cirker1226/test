# Guia de Estudo: HTTP, APIs RESTful e Node.js/Express

Material de revisão para a prova.

## Sumário

1. [Protocolo HTTP](#1-protocolo-http)
2. [Estrutura de requisição e resposta](#2-estrutura-de-requisição-e-resposta)
3. [Segurança e autenticação](#3-segurança-e-autenticação)
4. [Cookies](#4-cookies)
5. [APIs RESTful](#5-apis-restful)
6. [Node.js](#6-nodejs)
7. [Express](#7-express)
8. [Middlewares](#8-middlewares)
9. [Resumo para revisar](#9-resumo-para-revisar)
10. [Simulado básico](#10-simulado-básico)
11. [Simulado avançado](#11-simulado-avançado)
12. [Gabarito do simulado avançado](#12-gabarito-do-simulado-avançado)

---

## 1. Protocolo HTTP

**HTTP** (HyperText Transfer Protocol) é o protocolo da camada de aplicação que define como **clientes** (navegadores, apps) e **servidores** trocam dados na web.

- **Modelo cliente-servidor:** o cliente envia uma *requisição*, o servidor devolve uma *resposta*.
- **Sem estado (stateless):** cada requisição é independente; o servidor não "lembra" da anterior. É por isso que existem cookies e sessões.
- **Roda sobre TCP** (porta 80). Com TLS vira **HTTPS** (porta 443).
- **Versões:**
  - HTTP/1.1: texto, conexões persistentes (keep-alive).
  - HTTP/2: binário, multiplexação (várias requisições na mesma conexão).
  - HTTP/3: roda sobre QUIC/UDP, mais rápido em redes instáveis.

---

## 2. Estrutura de requisição e resposta

### Requisição

```
GET /produtos?id=5 HTTP/1.1
Host: loja.com
User-Agent: Mozilla/5.0
Accept: application/json

(corpo, se houver)
```

Partes: **linha de requisição** (método + caminho + versão), **cabeçalhos**, linha em branco, **corpo** (opcional).

### Resposta

```
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 42

{"nome": "Camiseta", "preco": 59.9}
```

Partes: **linha de status** (versão + código + texto), **cabeçalhos**, linha em branco, **corpo**.

### Métodos

| Método | Uso | Seguro* | Idempotente** |
|---|---|---|---|
| GET | Buscar recurso | Sim | Sim |
| POST | Criar recurso / enviar dados | Não | Não |
| PUT | Substituir recurso inteiro | Não | Sim |
| PATCH | Alterar parte do recurso | Não | Não (em geral) |
| DELETE | Remover recurso | Não | Sim |
| HEAD | Igual ao GET, sem corpo | Sim | Sim |
| OPTIONS | Consultar métodos permitidos (usado em CORS) | Sim | Sim |

\*Seguro = não altera o estado do servidor.
\*\*Idempotente = repetir N vezes tem o mesmo efeito que 1.

### Códigos de status

- **1xx** informativo (ex: 101 Switching Protocols)
- **2xx** sucesso: 200 OK, 201 Created, 204 No Content
- **3xx** redirecionamento: 301 (permanente), 302 (temporário), 304 Not Modified (cache)
- **4xx** erro do cliente: 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 405 Method Not Allowed, 429 Too Many Requests
- **5xx** erro do servidor: 500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable

> **Pegadinha clássica:** **401** = não autenticado (não sabemos quem você é); **403** = autenticado, mas sem permissão.

### Cabeçalhos importantes

- **Requisição:** `Host`, `User-Agent`, `Accept`, `Authorization`, `Cookie`, `Content-Type`, `Origin`
- **Resposta:** `Content-Type`, `Content-Length`, `Set-Cookie`, `Location` (redirecionamento), `Cache-Control`, `WWW-Authenticate`
- **Segurança:** `Strict-Transport-Security`, `Content-Security-Policy`, `X-Content-Type-Options`, `Access-Control-Allow-Origin` (CORS)

### Corpo (body)

Contém os dados transmitidos. O tipo é descrito por `Content-Type`:

- `application/json`
- `application/x-www-form-urlencoded` (formulários simples)
- `multipart/form-data` (upload de arquivos)
- `text/html`

GET normalmente não tem corpo; os parâmetros vão na URL (*query string*). POST/PUT/PATCH levam dados no corpo.

---

## 3. Segurança e autenticação

### HTTPS

HTTP puro trafega em **texto claro**, então qualquer um na rede pode ler ou alterar os dados. O **HTTPS** adiciona **TLS**, que garante:

- **Confidencialidade** (criptografia)
- **Integridade** (dados não adulterados)
- **Autenticidade** (certificado digital confirma a identidade do servidor)

### Autenticação vs. autorização

- **Autenticação:** quem é você? (login)
- **Autorização:** o que você pode fazer?

### Esquemas de autenticação HTTP

1. **Basic:** envia `Authorization: Basic base64(usuario:senha)`. Base64 **não é criptografia**, só codificação. Só é aceitável sobre HTTPS.
2. **Digest:** envia um *hash* da senha com um *nonce* do servidor, sem mandar a senha em claro. Mais seguro que Basic, mas pouco usado hoje.
3. **Bearer Token:** `Authorization: Bearer <token>`. Muito usado em APIs, com **JWT** (JSON Web Token: header.payload.signature) ou tokens opacos.
4. **API Key:** chave enviada em cabeçalho ou query string.
5. **OAuth 2.0:** framework de **autorização delegada** (ex: "Entrar com Google"). O usuário autoriza um app a acessar seus dados sem dar a senha a ele.

**Fluxo do desafio:** o servidor responde `401` com `WWW-Authenticate: Basic realm="..."`; o cliente reenvia a requisição com `Authorization`.

### Ameaças comuns

- **MITM (man-in-the-middle):** interceptação, combatida com HTTPS.
- **XSS:** injeção de script no site; mitigação com CSP e escape de saída.
- **CSRF:** o site malicioso faz o navegador do usuário enviar uma requisição autenticada; mitigação com token CSRF e cookie `SameSite`.
- **CORS:** política do navegador que restringe requisições entre origens diferentes. O servidor libera via `Access-Control-Allow-Origin`.

---

## 4. Cookies

**O que são:** pequenos dados que o servidor pede ao navegador para guardar e reenviar em requisições futuras. Resolvem o problema do HTTP ser stateless.

**Fluxo:**

1. Servidor responde: `Set-Cookie: sessao=abc123; HttpOnly; Secure`
2. Navegador guarda.
3. Nas próximas requisições ao mesmo site: `Cookie: sessao=abc123`

**Usos:** manter sessão/login, preferências (idioma, tema), carrinho de compras, rastreamento/analytics.

### Atributos

| Atributo | Função |
|---|---|
| `Expires` / `Max-Age` | Define a validade. Sem eles, é cookie de **sessão** (apaga ao fechar o navegador). |
| `Domain` | Domínios que recebem o cookie |
| `Path` | Caminhos que recebem o cookie |
| `Secure` | Só enviado via HTTPS |
| `HttpOnly` | Inacessível por JavaScript (protege contra roubo via XSS) |
| `SameSite` | `Strict`, `Lax` ou `None`: controla envio em requisições entre sites (protege contra CSRF) |

### Tipos

- **De sessão** vs. **persistentes** (têm data de expiração)
- **Primários (first-party)** vs. **de terceiros (third-party)**: terceiros são definidos por outro domínio (ex: anúncios) e usados para rastrear; vêm sendo bloqueados pelos navegadores.

### Cookies vs. outras opções

- **Sessão no servidor:** o cookie guarda só um **ID**, e os dados ficam no servidor (mais seguro).
- **localStorage/sessionStorage:** armazenamento do navegador, **não** é enviado automaticamente nas requisições.
- **Limite:** cerca de 4 KB por cookie.

---

## 5. APIs RESTful

**REST** (Representational State Transfer) é um **estilo arquitetural** para APIs sobre HTTP. Uma API é *RESTful* quando segue as restrições dele.

### Princípios (restrições)

1. **Cliente-servidor:** front e back separados.
2. **Stateless:** cada requisição leva tudo o que o servidor precisa (ex: token). Ele não guarda sessão do cliente.
3. **Cache:** respostas indicam se podem ser armazenadas em cache.
4. **Interface uniforme:** recursos identificados por URL, manipulados por métodos HTTP padrão.
5. **Sistema em camadas:** o cliente não sabe se fala direto com o servidor ou com proxy/gateway.
6. **Código sob demanda** (opcional).

### Recursos e URLs

Tudo é um **recurso**, identificado por uma URL com **substantivos no plural**, não verbos:

| Ação | Errado | Certo |
|---|---|---|
| Listar usuários | `GET /listarUsuarios` | `GET /usuarios` |
| Buscar um | `GET /buscarUsuario?id=3` | `GET /usuarios/3` |
| Criar | `POST /criarUsuario` | `POST /usuarios` |
| Atualizar | `POST /atualizar/3` | `PUT/PATCH /usuarios/3` |
| Remover | `GET /deletar/3` | `DELETE /usuarios/3` |

Recursos aninhados: `GET /usuarios/3/pedidos`.
Filtros e paginação vão na query string: `GET /produtos?categoria=livros&page=2`.

### Mapeamento CRUD → HTTP

- **Create** → `POST` (retorna **201 Created**)
- **Read** → `GET` (**200**; **404** se não existir)
- **Update** → `PUT` (substitui tudo) / `PATCH` (altera parte) (**200**)
- **Delete** → `DELETE` (**204 No Content**)

### Boas práticas

- Usar o **status code correto** (400 dados inválidos, 401/403 acesso, 404 não encontrado, 500 erro interno).
- Trocar dados em **JSON** (`Content-Type: application/json`).
- **Versionar** a API: `/api/v1/usuarios`.
- Mensagens de erro padronizadas: `{ "erro": "Usuário não encontrado" }`.
- Paginação, filtros e ordenação em listagens.
- **HATEOAS** (nível mais alto do modelo de Richardson): a resposta traz links para as próximas ações possíveis.

---

## 6. Node.js

**Node.js** é um ambiente de execução JavaScript **fora do navegador**, baseado no motor **V8** do Chrome.

- **Single-thread com event loop:** uma thread principal trata eventos.
- **I/O não bloqueante (assíncrono):** operações lentas (arquivo, banco, rede) não travam o programa, e o resultado volta por *callback*, *Promise* ou *async/await*.
- Ótimo para APIs e aplicações com muitas conexões simultâneas; ruim para cálculos pesados de CPU.
- **npm:** gerenciador de pacotes; `package.json` lista dependências e scripts.
- **Módulos:** CommonJS (`require`/`module.exports`) ou ES Modules (`import`/`export`).

---

## 7. Express

**Express** é o framework web mais usado do Node. Simplifica rotas, requisições, respostas e middlewares.

### Servidor mínimo

```js
const express = require('express');
const app = express();

app.use(express.json()); // middleware: lê corpo JSON

app.get('/usuarios', (req, res) => {
  res.json([{ id: 1, nome: 'Ana' }]);
});

app.listen(3000, () => console.log('Rodando na porta 3000'));
```

### Objetos `req` e `res`

- **`req.params`:** parâmetros de rota (`/usuarios/:id` → `req.params.id`)
- **`req.query`:** query string (`?page=2` → `req.query.page`)
- **`req.body`:** corpo da requisição (precisa de `express.json()`)
- **`req.headers`:** cabeçalhos
- **`res.status(201).json({...})`:** define o status e envia JSON
- **`res.send()`, `res.sendStatus()`, `res.redirect()`**

### Rotas

```js
app.post('/usuarios', (req, res) => {
  const novo = req.body;
  res.status(201).json(novo);
});

app.get('/usuarios/:id', (req, res) => {
  res.json({ id: req.params.id });
});
```

Para organizar, use **`express.Router()`** e monte em um prefixo: `app.use('/usuarios', usuariosRouter)`.

---

## 8. Middlewares

**Middleware** é uma função que roda **entre a chegada da requisição e a resposta final**. Tem acesso a `req`, `res` e `next`:

```js
function logger(req, res, next) {
  console.log(`${req.method} ${req.url}`);
  next(); // passa para o próximo
}
app.use(logger);
```

### Como funciona o "gerenciamento" da cadeia

O Express mantém uma **pilha ordenada** de middlewares e rotas e a percorre **na ordem em que foram registrados**:

```
Requisição → [logger] → [express.json] → [autenticação] → [rota] → Resposta
```

Cada middleware pode:

1. **Executar código** (log, validação).
2. **Alterar `req`/`res`** (ex: preencher `req.usuario`).
3. **Encerrar o ciclo** enviando resposta (`res.json(...)`).
4. **Chamar `next()`** para seguir adiante.

> **Regra crítica:** se não chamar `next()` nem responder, a requisição **fica travada** (pendurada).

### Tipos de middleware

- **De aplicação:** `app.use(fn)` (todas as rotas) ou `app.get('/x', fn, handler)` (rota específica)
- **De roteador:** `router.use(fn)`
- **Embutidos:** `express.json()`, `express.urlencoded()`, `express.static()`
- **De terceiros:** `cors`, `helmet`, `morgan` (logs), `cookie-parser`
- **De tratamento de erros:** têm **4 parâmetros** `(err, req, res, next)` e vão **por último**:

```js
app.use((err, req, res, next) => {
  console.error(err);
  res.status(500).json({ erro: 'Erro interno' });
});
```

Para disparar o erro: `next(new Error('falha'))`.

### Exemplo: autenticação como middleware

```js
function auth(req, res, next) {
  const token = req.headers.authorization;
  if (!token) return res.status(401).json({ erro: 'Não autenticado' });
  req.usuario = verificarToken(token);
  next();
}
app.get('/perfil', auth, (req, res) => res.json(req.usuario));
```

> **Observação:** "Middleware Manager" não é um termo oficial do Express. Pode ser o nome usado em aula para o gerenciamento da cadeia de middlewares; vale conferir nos slides.

---

## 9. Resumo para revisar

- HTTP é **stateless**; cookies/sessões criam o "estado".
- Requisição = linha + cabeçalhos + corpo; resposta = status + cabeçalhos + corpo.
- Saiba de cor: métodos, **idempotência**, famílias de status e **401 vs 403**.
- Basic ≠ criptografia; **HTTPS** é o que protege.
- Cookies: `Secure`, `HttpOnly`, `SameSite`.
- REST: **recursos** (substantivos) + **métodos HTTP** + **stateless** + **status codes corretos**.
- Node: JavaScript no servidor, **event loop** e **I/O assíncrono**.
- Express: rotas + `req`/`res`, `express.json()` para ler o corpo.
- Middleware: função `(req, res, next)`; a **ordem de registro importa**; sem `next()` ou resposta, a requisição trava; middleware de erro tem **4 parâmetros**.

---

## 10. Simulado básico

### Múltipla escolha

**1.** Qual afirmação sobre o HTTP é verdadeira?

- a) Mantém o estado entre requisições automaticamente
- b) É stateless: cada requisição é independente
- c) Só funciona com HTTPS
- d) Usa UDP por padrão no HTTP/1.1

**2.** Qual método é **idempotente** e usado para **substituir** um recurso inteiro?

- a) POST
- b) PATCH
- c) PUT
- d) GET

**3.** O usuário está logado, mas tenta acessar uma área só de administradores. Qual status o servidor deve retornar?

- a) 401
- b) 403
- c) 404
- d) 500

**4.** Sobre a autenticação **Basic**:

- a) A senha é criptografada com AES
- b) A senha vai em Base64, que é só codificação
- c) É segura mesmo sem HTTPS
- d) Usa o cabeçalho `WWW-Authenticate` na requisição

**5.** Qual atributo de cookie impede que o JavaScript leia o cookie?

- a) Secure
- b) SameSite
- c) HttpOnly
- d) Path

**6.** Qual atributo de cookie ajuda a mitigar ataques **CSRF**?

- a) Domain
- b) SameSite
- c) Expires
- d) HttpOnly

**7.** Qual é a URL mais adequada a uma API RESTful para **buscar o pedido 7 do usuário 3**?

- a) `GET /buscarPedido?usuario=3&pedido=7`
- b) `POST /usuarios/getPedido/3/7`
- c) `GET /usuarios/3/pedidos/7`
- d) `GET /pedido_do_usuario/3/7/buscar`

**8.** Ao criar um recurso com sucesso via `POST`, o status mais adequado é:

- a) 200 OK
- b) 201 Created
- c) 204 No Content
- d) 302 Found

**9.** No Express, um middleware recebe `(req, res, next)`. O que acontece se ele **não** chamar `next()` e **não** enviar resposta?

- a) O Express chama o próximo automaticamente
- b) A requisição fica pendurada, sem resposta
- c) O Express retorna 404
- d) O servidor reinicia

**10.** Qual assinatura identifica um middleware de **tratamento de erros** no Express?

- a) `(req, res)`
- b) `(req, res, next)`
- c) `(err, req, res, next)`
- d) `(next, err)`

### Dissertativas

**11.** Explique por que cookies são necessários se o HTTP é stateless, e descreva o fluxo `Set-Cookie` → `Cookie`.

**12.** Por que a **ordem** em que os middlewares são registrados no Express importa? Dê um exemplo do que dá errado se `express.json()` for registrado **depois** da rota POST.

**13.** Cite 4 princípios/boas práticas de uma API RESTful.

### Gabarito (múltipla escolha)

1-b, 2-c, 3-b, 4-b, 5-c, 6-b, 7-c, 8-b, 9-b, 10-c.
As dissertativas são respondidas nas seções 2, 4, 5 e 8 deste guia.

---

## 11. Simulado avançado

### Múltipla escolha

**1.** Veja o código:

```js
app.get('/perfil', (req, res) => res.json({ ok: true }));
app.use(autenticacao);
```

O que acontece numa requisição `GET /perfil`?

- a) `autenticacao` roda primeiro, pois `app.use` tem prioridade
- b) `autenticacao` nunca roda para essa rota, pois a resposta já foi enviada antes
- c) O Express retorna erro de ordem de registro
- d) `autenticacao` roda depois da resposta, apenas para log

**2.** Um cliente envia `DELETE /usuarios/3` duas vezes. Na 1ª recebe `204`, na 2ª recebe `404`. Qual afirmação é correta?

- a) DELETE não é idempotente, pois as respostas diferiram
- b) DELETE é idempotente: idempotência se refere ao **estado do servidor**, não à resposta
- c) A API está errada: deveria retornar 204 nas duas
- d) DELETE é idempotente só se usar PUT antes

**3.** Qual configuração de cookie é **rejeitada** pelos navegadores modernos?

- a) `SameSite=Lax; HttpOnly`
- b) `SameSite=Strict; Secure`
- c) `SameSite=None` sem `Secure`
- d) `SameSite=None; Secure`

**4.** Sobre **JWT**:

- a) O payload é criptografado, então pode guardar senhas
- b) O payload é apenas codificado em Base64URL; a assinatura garante **integridade**, não sigilo
- c) A assinatura impede que qualquer um leia o conteúdo
- d) JWT dispensa HTTPS, pois é assinado

**5.** Qual requisição **cross-origin** dispara um **preflight** (`OPTIONS`) no CORS?

- a) `GET` simples sem cabeçalhos customizados
- b) `POST` com `Content-Type: application/x-www-form-urlencoded`
- c) `PUT` com `Content-Type: application/json` e `Authorization`
- d) `HEAD` simples

**6.** Sobre `Cache-Control`:

- a) `no-cache` proíbe armazenar a resposta
- b) `no-store` exige revalidação antes de usar
- c) `no-cache` permite armazenar, mas **exige revalidação** com o servidor; `no-store` proíbe armazenar
- d) Os dois são sinônimos

**7.** Analise o middleware:

```js
function auth(req, res, next) {
  if (!req.headers.authorization) {
    res.status(401).json({ erro: 'Sem token' });
  }
  next();
}
```

Qual o problema?

- a) Falta `return` antes de `res.status`: o `next()` ainda executa e a rota seguinte tenta responder de novo (erro de cabeçalhos já enviados)
- b) `401` deveria ser `403`
- c) `json()` não pode ser chamado em middleware
- d) Não há problema

**8.** Em um middleware, você chama `next(new Error('falha'))`. O que o Express faz?

- a) Continua para o próximo middleware normal
- b) **Pula** os middlewares/rotas normais e vai direto ao primeiro de tratamento de erros `(err, req, res, next)`
- c) Encerra o processo Node
- d) Reinicia a cadeia do início

**9.** Qual rota bloqueia o **event loop** e trava todos os outros clientes?

- a) `app.get('/a', async (req,res) => res.json(await db.buscar()))`
- b) `app.get('/b', (req,res) => { fs.readFile(..., cb) })`
- c) `app.get('/c', (req,res) => { const d = fs.readFileSync('grande.bin'); res.send(d) })`
- d) `app.get('/d', (req,res) => setTimeout(() => res.send('ok'), 5000))`

**10.** Comparando onde guardar o token de autenticação:

- a) Cookie `HttpOnly` é imune a XSS e a CSRF
- b) `localStorage` é imune a XSS
- c) Cookie `HttpOnly` protege contra roubo por XSS, mas exige defesa contra CSRF (ex: `SameSite`, token CSRF); `localStorage` é legível por qualquer script injetado
- d) Não há diferença de segurança

**11.** Qual resposta está **incorreta** do ponto de vista do protocolo HTTP?

- a) `304 Not Modified` sem corpo
- b) `401 Unauthorized` com `WWW-Authenticate`
- c) `204 No Content` com um JSON no corpo
- d) `405 Method Not Allowed` com cabeçalho `Allow`

**12.** Qual afirmação sobre **PUT** vs **PATCH** vs **POST** está correta?

- a) PATCH sempre é idempotente
- b) PUT substitui o recurso por completo e é idempotente; POST criando recursos normalmente não é
- c) POST é idempotente porque cria um recurso só
- d) PUT altera só os campos enviados

### Dissertativas

**13.** Projete uma API REST para uma biblioteca com **livros** e **empréstimos**. Liste as rotas (método + URL + status de sucesso). Inclua a ação "**renovar um empréstimo**", que não é um CRUD puro, e justifique como modelá-la sem usar um verbo na URL.

**14.** Descreva o fluxo completo de **login com sessão via cookie** (do POST de credenciais até uma requisição autenticada posterior). Indique quais **atributos do cookie** usaria e **que ataque cada um previne**.

**15.** Explique por que o Express é descrito como uma **pilha/cadeia de middlewares**. O que muda se uma rota for registrada **antes** de `express.json()`? E se o middleware de erros for registrado **antes** das rotas?

---

## 12. Gabarito do simulado avançado

### Múltipla escolha

1. **b)** O Express percorre a pilha **na ordem de registro**. A rota `/perfil` responde sem chamar `next()`, então `autenticacao` nunca roda e a rota fica **desprotegida**.
2. **b)** Idempotência se refere ao **estado do servidor** após N chamadas iguais (o recurso continua removido), não à resposta devolvida.
3. **c)** `SameSite=None` **exige** `Secure`. Sem ele, o navegador rejeita o cookie.
4. **b)** O payload do JWT é só **Base64URL**, qualquer um lê. A assinatura garante **integridade e autenticidade**, não sigilo. Nunca coloque senhas nele, e use HTTPS.
5. **c)** `PUT`, `Content-Type: application/json` e `Authorization` fogem da "requisição simples" e disparam o **preflight**. Na (b), o content-type de formulário é considerado simples.
6. **c)** `no-cache` = pode armazenar, mas **revalida** antes de usar. `no-store` = **não armazena**.
7. **a)** Falta `return`. Depois de responder 401, o `next()` ainda executa, e a rota seguinte tenta responder de novo (*Cannot set headers after they are sent*). O correto é `return res.status(401)...`.
8. **b)** `next(err)` **pula** os middlewares/rotas normais e vai direto ao primeiro middleware de erro `(err, req, res, next)`.
9. **c)** `readFileSync` é **síncrono** e bloqueia o event loop. As demais são assíncronas (`async/await`, callback, `setTimeout`).
10. **c)** Cookie `HttpOnly` impede roubo via XSS, mas exige defesa contra **CSRF** (`SameSite`, token CSRF). O `localStorage` é legível por qualquer script injetado.
11. **c)** `204 No Content` **não pode ter corpo**.
12. **b)** PUT substitui por completo e é idempotente. POST de criação normalmente não é. PATCH, em geral, não é garantidamente idempotente, e PUT não altera só campos parciais.

### Dissertativas (respostas-modelo)

#### 13. API da biblioteca

| Método | URL | Sucesso |
|---|---|---|
| GET | `/livros` | 200 |
| GET | `/livros/:id` | 200 |
| POST | `/livros` | 201 |
| PUT/PATCH | `/livros/:id` | 200 |
| DELETE | `/livros/:id` | 204 |
| GET | `/emprestimos` | 200 |
| GET | `/emprestimos/:id` | 200 |
| POST | `/emprestimos` | 201 |

**Renovar (sem verbo na URL):** há duas modelagens aceitas.

- **Alterar o estado/atributo:** `PATCH /emprestimos/:id` com corpo `{ "dataDevolucao": "2026-11-10" }` → **200**.
- **Tratar a ação como recurso:** `POST /emprestimos/:id/renovacoes` → **201**. A renovação vira um sub-recurso (substantivo) e fica registrada no histórico.

A ideia é transformar a ação em **substantivo** ou **mudança de estado** de um recurso.

#### 14. Login com sessão via cookie

1. `POST /login` com `{email, senha}` **sobre HTTPS**.
2. O servidor valida (comparando o **hash** da senha) e cria uma **sessão** (ID aleatório e imprevisível) no servidor.
3. Responde: `Set-Cookie: sid=abc123; HttpOnly; Secure; SameSite=Lax; Path=/; Max-Age=3600`.
4. O navegador guarda e reenvia `Cookie: sid=abc123` automaticamente nas próximas requisições.
5. O servidor busca a sessão pelo ID, identifica o usuário (`req.usuario`) e responde.
6. No logout, destrói a sessão e limpa o cookie.

**Atributos e o que previnem:**

- `HttpOnly` → roubo do cookie via **XSS**
- `Secure` → interceptação em texto claro (**MITM/sniffing**)
- `SameSite` → **CSRF**
- `Max-Age/Expires` → limita a janela de uso de um cookie roubado
- Bônus: **regenerar o ID da sessão** no login previne *session fixation*

#### 15. Pilha de middlewares

- O Express guarda uma **lista ordenada** de camadas (middlewares e rotas). Cada requisição a percorre **na ordem de registro**, e `next()` passa para a camada seguinte.
- **Rota antes de `express.json()`:** o corpo ainda não foi interpretado, então `req.body` fica **`undefined`** e o handler quebra ou valida errado.
- **Middleware de erros antes das rotas:** erros só se propagam **para frente** na pilha. Erros de rotas registradas depois **não chegam** a ele e caem no tratador padrão do Express (resposta HTML com stack trace). Por isso o middleware de erros vai **por último**.