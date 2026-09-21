---
name: desenvolvimento
description: "NextLayer — Implementa o plano aprovado no padrão que a empresa cadastrou no servidor NextLayer. Use depois do planejamento aprovado."
argument-hint: "[pedido]"
---

Chame a ferramenta `obter_skill` do servidor MCP `nextlayer` com `nome: "desenvolvimento"` e siga a instrução retornada, aplicada a este pedido: $ARGUMENTS

Se o servidor não estiver conectado ou pedir autenticação, oriente a pessoa: `/mcp` → **plugin:code:nextlayer** → **Authenticate** (ou **Re-authenticate**, se já tiver entrado antes). Se a skill não existir para a empresa, avise. Nunca improvise a instrução.
