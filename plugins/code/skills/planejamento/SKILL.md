---
name: planejamento
description: "NextLayer — Planeja a alteração antes de escrever código, no padrão que a empresa cadastrou no servidor NextLayer. Use no início de uma tarefa de desenvolvimento."
argument-hint: "[pedido]"
---

Chame a ferramenta `obter_skill` do servidor MCP `nextlayer` com `nome: "planejamento"` e siga a instrução retornada, aplicada a este pedido: $ARGUMENTS

Se o servidor não estiver conectado ou pedir autenticação, oriente a pessoa: `/mcp` → **plugin:code:nextlayer** → **Authenticate** (ou **Re-authenticate**, se já tiver entrado antes). Se a skill não existir para a empresa, avise. Nunca improvise a instrução.
