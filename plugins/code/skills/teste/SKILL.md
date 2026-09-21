---
name: teste
description: "NextLayer — Roda a suíte e reporta o resultado real, no padrão que a empresa cadastrou no servidor NextLayer. Use antes de entregar."
argument-hint: "[pedido]"
---

Chame a ferramenta `obter_skill` do servidor MCP `nextlayer` com `nome: "teste"` e siga a instrução retornada, aplicada a este pedido: $ARGUMENTS

Se o servidor não estiver conectado ou pedir autenticação, oriente a pessoa: `/mcp` → **plugin:code:nextlayer** → **Authenticate** (ou **Re-authenticate**, se já tiver entrado antes). Se a skill não existir para a empresa, avise. Nunca improvise a instrução.
