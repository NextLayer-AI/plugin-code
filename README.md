# NextLayer Code — plugin do Claude Code

Skills e auditoria da NextLayer da sua empresa, em qualquer conversa do Claude Code.

O plugin é **só texto**: não instala programa, não precisa de Node, git ou npm na máquina. Ele traz
a configuração de um servidor MCP — o servidor NextLayer da sua empresa — e o Claude Code faz o resto:
login pelo navegador, skills buscadas no servidor na hora do uso e registro de auditoria a cada
prompt e ferramenta.

> **Estado:** spike (v0.1). O servidor de teste está em
> [`NextLayer-AI/POC-Auditoria`](https://github.com/NextLayer-AI/POC-Auditoria) (`mock-mcp/`).

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

### Se algo não funcionar

| Sintoma | O que fazer |
|---|---|
| Conversa nova diz que o servidor **requer autenticação**, mesmo depois do login | `/mcp` → **plugin:code:nextlayer** → **Re-authenticate** e entre de novo |
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

O servidor precisa oferecer OAuth (metadados, registro dinâmico, PKCE) e as ferramentas
`listar_skills`, `obter_skill` e `registrar_evento`. Pessoa e empresa sempre saem do token.

## Decisões e limites

- **Nada executa na máquina.** O servidor entrega dados (texto de skills); o hook só chama uma ferramenta dele.
- **Sem fila local:** com o servidor fora do ar, os eventos desse período se perdem (decisão aceita no spike).
- **`registrar_evento` é visível ao modelo.** Para impedir auditoria forjada, negue a ferramenta ao
  modelo em `permissions.deny` (managed ou user settings); os hooks continuam registrando — a validar.
- **Repositório privado** impede a instalação em máquina sem acesso ao GitHub da NextLayer; para
  distribuição, o repositório precisa ser público.
