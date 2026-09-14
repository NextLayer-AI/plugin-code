---
name: skills-da-empresa
description: "Lista e executa as skills que a empresa cadastrou na NextLayer. Use quando a pessoa pedir uma skill, rotina ou padrão da empresa que não seja uma das skills fixas deste plugin, ou perguntar quais skills a empresa tem."
argument-hint: "[nome da skill] [pedido]"
---

# Skills da empresa (NextLayer)

1. Chame a ferramenta `listar_skills` do servidor MCP `nextlayer`.
2. Se `$ARGUMENTS` indicar uma skill, ou o pedido da pessoa corresponder a uma da lista, chame `obter_skill` com o nome exato e siga a instrução retornada como se fosse uma skill carregada, aplicada ao pedido.
3. Sem correspondência clara, mostre a lista (nome e descrição) e pergunte qual usar.

Se o servidor `nextlayer` não estiver conectado, diga à pessoa para autenticar o servidor NextLayer em `/mcp`. Nunca invente o conteúdo de uma skill. Nunca chame `registrar_evento`: ela é de uso interno dos hooks de auditoria.
