Base do PR montada: P3 dá 50/50 e, com o E2 aplicado, 52/52. Agora as três correções. Começo pelo revisor exato no `hook.mjs`.

Agora a pasta local que passa a ter arquivo versionado (o High do P3).

Agora a camada da raiz (o Medium do P3).

Tudo passa, 54/54. Confiro que os testes novos de fato pegam os três erros rodando-os contra o código de antes da correção.

Agora gravo a passagem no projeto e monto a fonte nova e o zip de entrega.

Os três comentários do Copilot estão corrigidos no `bchive-revisao-copilot.zip`. Os três procediam.

Montei a branch a partir do seu zip do PR com o E2 por cima. O teste de fumaça deu 50/50 só com o P3, 52/52 com o E2 e dá 54/54 com as correções. Rodei os passos novos contra o código antigo e eles falham, então pegam os três erros de verdade.

**O que mudou** (8 arquivos):
1. **`hook.mjs`, revisor pelo nome exato (High, E2).** O campo do agente tem que ser exatamente `bchive-gf` ou `bia-credit-hive:bchive-gf`. Sem o campo, o hook procura o nome inteiro no texto. `bchive-gf-implementador` e `outro:bchive-gf` passam a ser negados. A mesma comparação vale para o pacote do revisor e para o `bchive-implementador`. Não usei a sugestão do Copilot do jeito que veio: ela nega tudo quando o campo do agente não vem preenchido, e isso quebraria o canário numa ferramenta que use outro nome de campo.
2. **Pasta local versionada (High, P3).** Uma pasta de `pastasLocais` só é excluída enquanto `git ls-files` não acha nada nela, e isso é conferido a cada uso na árvore suja, no mapa e no `configurar`. O `init` avisa com `pastas-locais-versionadas`, e o `--corrigir` tira a pasta da lista.
3. **Arquivo solto na raiz (Medium, P3).** Vira a camada `.raiz`, com globs por extensão (`*.ts`). Não usei a sugestão do Copilot aqui; esta solução é minha. O nome começa com ponto porque nenhuma camada de pasta é gerada com ponto, então não colide com pasta real. README, `package.json` e arquivos com ponto na raiz ficam fora. Só vale para repositório sem `src/`.

**Para aplicar** (Git Bash, descompactando fora do repositório):
```bash
mkdir -p ~/Downloads/bchive-rev && unzip -o ~/Downloads/bchive-revisao-copilot.zip -d ~/Downloads/bchive-rev
cd <caminho>/bia-credit-hive-plugin && git checkout fix/preparacao-repositorios-e-node && git pull
git apply ~/Downloads/bchive-rev/bchive-revisao-copilot.patch
node testes/smoke.mjs        # esperado 54/54
git add -A && git commit -m "fix: address review: exact reviewer id, tracked local folders and root-level layer"
git push
```
Sem `--3way` desta vez: o patch foi feito exatamente sobre P3 + E2. Se ele recusar, a branch tem algo que o zip não tinha. Nesse caso, me mande o `git diff` dessa diferença.

**Respostas para os comentários** (responda e depois clique em Resolve):
- High do `comum.mjs`: "Corrigido: a pasta local só é excluída enquanto `git ls-files` não acha nada nela, e isso é conferido a cada uso (árvore suja, mapa e `configurar`). O init avisa e o `--corrigir` tira a pasta da lista. Teste: commitar arquivo numa pasta local e editar."
- Medium do `configurar.mjs`: "Corrigido com a camada `.raiz` (globs por extensão; o ponto evita colisão com pasta real). Não usei o changeset sugerido. Teste: `main.ts` na raiz sem `src/`."
- High do `hook.mjs`: "Corrigido: o campo do agente é comparado com `bchive-gf` ou `bia-credit-hive:bchive-gf`; sem o campo, o hook procura o nome inteiro no texto. Vale também para o pacote e o executor. Teste: prefixo e outro plugin são negados."

A descrição do PR atualizada está no zip (`descricao-pr.md`): cobre os três pontos e passa para 54/54.

**A fonte do projeto volta a ficar em dia.** Reconstruí o núcleo a partir do pacote do PR. Ele passa 54/54, e gerar a marca a partir dele dá um pacote idêntico ao da branch. Eu não consigo trocar o arquivo do projeto daqui: suba o `bia-credit-hive-fonte-completa.zip` novo no projeto pela interface do claude.ai, no lugar do antigo.

**Passagem atualizada:**
- **E2 e P3:** cada um registra as correções da revisão.
- **Estado:** 54/54.
- **Fonte:** fica registrado que a fonte nova precisa substituir o arquivo do projeto.