Desta vez não vou corrigir só os cinco achados. Vou auditar o pacote inteiro atrás dos problemas do mesmo tipo: links simbólicos, fonte que muda, caminho que escapa, perda de metadado, limpeza da pasta temporária e conteúdo que escapa da validação. Começo pela fonte local e pelos helpers de que as correções dependem.

O código local é o mesmo que está na PR (commit `e83dfe8`). As correções mexem em quase todas as funções do `okf.mjs`, então leio o arquivo inteiro antes de editar.

A função `caminhoSeguro` e o laço que tira `status` e `verified` ficaram confusos. Reescrevo os dois de forma direta antes de testar.

Agora a segunda parte da CLI: validação das fontes contra o plano, rascunhos sem link e dentro do plano, revisão presa ao commit, publicação e confirmação.

Os escapes da expressão saíram certos. Agora a terceira parte: publicação e confirmação.

O script parou antes de gravar, então nada da parte 3 entrou. Vejo o trecho exato que não bateu.

Agora os testes novos: um por achado do Copilot nesta rodada, mais um para cada problema que encontrei na auditoria.

Atualizo a passagem e a fonte completa.

Você tem razão: a primeira correção tapou cada achado no ponto exato e não procurou o resto da mesma família. Desta vez, além dos cinco achados novos, auditei o pacote inteiro atrás de problemas do mesmo tipo e achei mais oito. Corrigi todos, cada um com um passo no teste de fumaça, que agora dá 65/65 no núcleo e na marca. Rodei cada um dos cinco achados do Copilot como cenário isolado: os cinco aparecem no código da PR e não aparecem no novo.

## Os cinco achados desta rodada

| Achado | Correção |
|---|---|
| Link simbólico no clone alterava o original | O clone com link é recusado antes de qualquer cópia. A gravação no clone também confere cada caminho. |
| Fontes alteradas reaproveitavam aprovação velha | As fontes vêm sempre do commit do inventário (`git show`), nunca da árvore de trabalho. A aprovação vale para o plano e o inventário juntos; o veredito, para o texto da nota e o commit. Refazer o inventário exige `--recomecar`, e a extração anterior fica guardada. |
| Fonte fora do repositório (`../`) | Cada fonte precisa estar nos arquivos e nas faixas de linhas do plano aprovado. Caminho com `..`, arquivo de configuração e linha fora da faixa dão erro. |
| Republicar apagava a confirmação | Se o conteúdo extraído não mudou, a nota publicada fica como está, com `verified` e `stable`. Só conteúdo novo volta a `draft`. |
| Publicação recusada deixava a cópia temporária | A cópia é apagada em qualquer saída, inclusive Ctrl+C. |

## O que a auditoria pegou além disso

- O `confirmar` reescrevia o frontmatter inteiro e estragava frontmatter aninhado de nota do Companion (`[object Object]`). Agora só troca `status` e `verified`, e confere que o resto ficou igual.
- Rascunho que fosse link simbólico era publicado. Agora é recusado.
- Título e descrição com crase iam para os índices. Agora dão erro.
- Os índices e o log que a publicação escreve não eram conferidos. Agora passam pelas regras de código, credencial, infraestrutura e dado pessoal.
- Rascunho fora do plano aprovado era publicado. Agora a publicação recusa.
- O `--squad` deixava publicar numa squad diferente da aprovada. Saiu.
- Escrever no estado com `node -e` passava pelo hook. Agora qualquer comando de terminal que cite os arquivos do estado é negado.
- Rodar o portão separando o nome do script do subcomando também passava. Agora o portão é reconhecido em qualquer ordem.

**Risco que fica:** o hook é guarda-corpo, não sandbox. Um agente que grave um script num arquivo e depois o rode, sem citar no comando o caminho nem o portão, escapa de qualquer lista de comandos. A defesa final continua sendo humana: aprovar o plano, revisar a PR do Vault e confirmar a nota. Isso está registrado na passagem. Se o Copilot apontar exatamente esse caso, a resposta é essa.

## Aplicar na branch (Git Bash, no mesmo terminal do começo ao fim)

```bash
REPO=~/repos/bia-credit-hive-plugin
cd "$REPO" && git switch feat/vault-extractor && git pull
TMP=$(mktemp -d) && unzip -q ~/Downloads/bia-credit-hive-revisao-copilot-2.zip -d "$TMP" && find "$TMP" -type f | wc -l   # esperado: 7
cp -r "$TMP"/bia-credit-hive-plugin/. .
git status --short          # esperado: 7 arquivos com M
node testes/smoke.mjs       # esperado: 65/65 passos ok
git add -A && git commit -m "fix: address second copilot review round" && git push
```

Os 7 arquivos esperados no `git status`:
- `scripts/core/hook.mjs`
- `scripts/core/okf.mjs`
- `scripts/core/lib/okf.mjs`
- `skills/vault-plan/SKILL.md`
- `skills/vault-extract/SKILL.md`
- `skills/vault-publish/SKILL.md`
- `testes/smoke.mjs`

Se aparecer outro arquivo, pare e traga a saída para o chat.

## Respostas para as cinco conversas da PR

1. **Links simbólicos:** "Corrigido. O clone com link simbólico é recusado antes da cópia, e a gravação no clone confere cada caminho. Teste no smoke: a publicação recusada não escreve no destino do link."
2. **Fontes alteradas:** "Corrigido. As fontes vêm sempre do commit do inventário (git show), nunca da árvore de trabalho. A aprovação vale para o plano e o inventário; o veredito, para o texto da nota e o commit. Refazer o inventário exige --recomecar. Teste no smoke com alteração não commitada e com commit novo."
3. **Fontes fora do repositório:** "Corrigido. Cada fonte precisa estar nos arquivos e faixas do plano aprovado; caminho com .., arquivo de configuração e linha fora da faixa dão erro antes da revisão. Teste no smoke."
4. **Republicar sem mudanças:** "Corrigido. Mesmo conteúdo extraído (hash do conteúdo) preserva a nota publicada com status e verified; só conteúdo novo volta a draft. Teste no smoke nos dois sentidos."
5. **Cópia temporária:** "Corrigido. A cópia é apagada em qualquer saída do processo, inclusive SIGINT e SIGTERM. Teste no smoke conta as cópias antes e depois de uma recusa."

A passagem registra as duas rodadas, o que a auditoria pegou e o risco que fica. O `bia-credit-hive-fonte-completa.zip` substitui o arquivo da fonte no projeto.