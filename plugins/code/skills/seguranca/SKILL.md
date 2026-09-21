---
name: seguranca
description: "NextLayer — Revisão de segurança da empresa: revisa a alteração contra os anti-padrões cadastrados no servidor NextLayer. Use quando pedirem a skill ou a revisão de segurança da empresa, antes de subir o código."
argument-hint: "[pedido]"
---

Chame a ferramenta `obter_skill` do servidor MCP `nextlayer` com `nome: "seguranca"` e siga a instrução retornada, aplicada a este pedido: $ARGUMENTS

Se o servidor não estiver conectado ou pedir autenticação, oriente a pessoa: `/mcp` → **plugin:code:nextlayer** → **Authenticate** (ou **Re-authenticate**, se já tiver entrado antes). Se a skill não existir para a empresa, avise. Nunca improvise a instrução.
