# NextLayer Code — plugin do Claude Code

Skills e auditoria da NextLayer da sua empresa, em qualquer conversa do Claude Code.

O plugin é **só texto**: não instala programa, não precisa de Node, git ou npm na máquina. Ele traz
a configuração de um servidor MCP — o servidor NextLayer da sua empresa — e o Claude Code faz o resto:
login pelo navegador, skills buscadas no servidor na hora do uso e registro de auditoria a cada
prompt e ferramenta.

> **Servidor:** a própria plataforma NextLayer da empresa (a partir da versão com o módulo
> `plugin-code`, 2026-09). O mock do spike (`POC-Auditoria/mock-mcp`) ficou só como referência do contrato.

## Instalar (máquina com só o Claude Code)

1. Adicione o marketplace e instale o plugin — pelo app em **Customize → Plugins**, ou no terminal:

   ```bash
   claude plugin marketplace add NextLayer-AI/plugin-code
   ```

   ```bash
   claude plugin install code@nextlayer
   ```

2. Informe o **endereço do servidor NextLayer** da empresa. A instalação não pergunta: abra `/plugin` →
   **Installed** → **code** → **Configure options** e preencha (o mesmo do navegador, sem barra no final).
3. Abra uma conversa nova e autentique: `/mcp` → **plugin:code:nextlayer** → **Authenticate**. O
   navegador abre o login da empresa.
4. Se uma conversa nova ainda disser que o servidor **precisa autenticar**, volte em `/mcp` →
   **plugin:code:nextlayer** → **Re-authenticate** e entre de novo. O Claude Code às vezes guarda a
   marca de "precisa autenticar" da primeira tentativa, e refazer o login é o que a limpa (nesse
   estado o **Reconnect** não aparece).
5. Confira numa conversa nova: "quais skills da empresa eu tenho?". Depois use `/code:planejamento`,
   `/code:desenvolvimento`, `/code:teste`, `/code:seguranca` ou peça uma skill da empresa em
   linguagem natural.

Cada prompt e cada ferramenta usada no Claude Code entram na trilha de **Auditoria** da empresa
(com o repositório e um trecho do prompt; dados pessoais e chaves coladas saem mascarados). A
autorização aparece em **Sessões ativas** como um dispositivo "Claude Code": revogá-la lá derruba o
plugin na hora, e o Claude Code volta a pedir autenticação. Se a empresa exige aceite da política de
uso, aceite na plataforma antes: até lá as skills respondem só com o endereço para aceitar.

### Se algo não funcionar

| Sintoma | O que fazer |
|---|---|
| Conversa nova diz que o servidor **requer autenticação**, mesmo depois do login | `/mcp` → **plugin:code:nextlayer** → **Re-authenticate** e entre de novo |
| Hook falha com 401 no meio de uma conversa que funcionava | A sessão do plugin foi revogada (Sessões ativas) ou venceu (30 dias): `/mcp` → **Re-authenticate** |
| `/code:planejamento` (ou outra fixa) diz que a skill **não existe** | A Biblioteca da empresa ainda não tem uma skill com esse nome; peça ao administrador para criar "Planejamento", "Desenvolvimento", "Teste" e "Segurança" (ou use `/code:skills-da-empresa`) |
| Autorizar no navegador volta para o editor mas o `/mcp` mostra **failed** | O servidor está atrás de um proxy que não encaminha `/mcp`, `/oauth/` e `/.well-known/oauth-*` na raiz do host; ver "Requisitos do servidor" |
| `/mcp` mostra **failed** / conexão recusada | Endereço errado ou servidor fora do ar: confira em **Configure options** |
| O servidor nem aparece em `/mcp` | Endereço não preenchido: `/plugin` → **code** → **Configure options**, e abra uma conversa nova |

### Distribuição pela TI (managed settings)

Com managed settings, o plugin já chega habilitado e com o endereço preenchido; a pessoa só autentica.

```json
{
  "extraKnownMarketplaces": {
    "nextlayer": { "source": { "source": "github", "repo": "NextLayer-AI/plugin-code" }, "autoUpdate": true }
  },
  "enabledPlugins": { "code@nextlayer": true },
  "pluginConfigs": { "code@nextlayer": { "options": { "servidor": "https://nextlayer.suaempresa.com.br" } } }
}
```

## Como funciona

| Peça | Arquivo | O que faz |
|---|---|---|
| Endereço | `plugins/code/.claude-plugin/plugin.json` | `userConfig.servidor`, pedido ao habilitar |
| Servidor | `plugins/code/.mcp.json` | MCP remoto `${user_config.servidor}/mcp`; o login OAuth é do Claude Code |
| Auditoria | `plugins/code/hooks/hooks.json` | hooks `mcp_tool` em `UserPromptSubmit`, `PreToolUse` e `SessionEnd` chamam `registrar_evento` |
| Skills | `plugins/code/skills/` | cascas que carregam a instrução vigente com `obter_skill`; `skills-da-empresa` alcança as criadas pela empresa |

As quatro skills fixas (`planejamento`, `desenvolvimento`, `teste`, `seguranca`) são o catálogo de
fábrica do plugin Code, que toda empresa recebe. O texto de cada uma vem do servidor, com a
personalização da empresa; se a empresa desativar uma delas, a skill avisa em vez de improvisar.

## Requisitos do servidor

A plataforma NextLayer atende tudo isto no módulo `plugin-code`; quem opera precisa só de:

- `PUBLIC_URL` = o endereço do navegador, https, **sem path** (ex.: `https://app.suaempresa.com.br`);
- no proxy, `/mcp`, `/oauth/` e `/.well-known/oauth-authorization-server` /
  `/.well-known/oauth-protected-resource` encaminhados para a API **na raiz do host** (o deploy
  padrão serve a API em `/api/`; o bloco de nginx pronto está em `docs/seguranca-e-operacao.md`
  da plataforma, "Plugin Code atrás do nginx"), com `client_max_body_size 2m` no `/mcp`;
- as skills de fábrica ("Planejamento", "Desenvolvimento", "Teste", "Segurança") cadastradas na
  Biblioteca com escopo empresa, para as quatro skills fixas terem texto.

O contrato que o servidor cumpre: OAuth 2.1 para cliente nativo (metadados RFC 8414/9728, registro
dinâmico só com retorno em loopback, PKCE S256, refresh token rotativo) e as ferramentas MCP
`listar_skills`, `obter_skill` e `registrar_evento`. Pessoa e empresa sempre saem do token.

## Decisões e limites

- **Nada executa na máquina.** O servidor entrega dados (texto de skills); o hook só chama uma ferramenta dele.
- **Sem fila local:** com o servidor fora do ar, os eventos desse período se perdem (decisão aceita no spike).
- **`registrar_evento` é visível ao modelo.** Para impedir auditoria forjada, negue a ferramenta ao
  modelo em `permissions.deny` (managed ou user settings); os hooks continuam registrando — a validar.
- **Repositório público**, de propósito: é o que permite instalar numa máquina sem acesso ao GitHub
  da NextLayer. Ele não carrega nada da empresa — só a configuração; o conteúdo vem do servidor dela.
