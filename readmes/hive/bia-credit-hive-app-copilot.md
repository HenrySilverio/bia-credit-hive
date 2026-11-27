Para ajustar o enquadramento: o Copilot diz "No open issues". Os 4 itens são "previously missed", ou seja, ele achou agora em código que já estava lá (do E2 e do P3). Não são quebras do pacote de ontem. Mesmo assim os quatro procedem, e vou corrigir todos. Começo conferindo o código.

Os quatro estão corrigidos, com 54/54, e os casos novos falham contra o código anterior. Agora gero o patch, a fonte do núcleo e a descrição.

Os quatro estão corrigidos no `bchive-revisao-copilot-2.zip`. O pacote de ontem não quebrou nada: o Copilot marca "No open issues" e diz que os 4 estão "em código que não mudou desde a última revisão". São brechas que já existiam no E2 e no P3 e que ele só viu agora. Os quatro procediam.

O teste de fumaça continua 54/54. Os casos novos falham contra o código de ontem, e o caso do link eu confirmei à mão: antes passava, agora é negado.

**O que mudou** (3 arquivos: `hook.mjs`, `configurar.mjs`, `smoke.mjs`):
1. **Link em `estado/` (High).** O hook agora compara o caminho real. Um link simbólico ou uma junction do Windows dentro de `.bchive/estado/` que aponte para o código é negado, inclusive para arquivo novo criado pelo link.
2. **Cabeçalho de patch (Medium).** O `*** Update File:` só é lido do campo do patch (`input`, `patch`, `diff`). Um arquivo em `estado/` com um patch de exemplo no conteúdo não é mais negado por engano.
3. **Sessão concorrente (Medium).** A presença do agente vai numa marca própria, um arquivo vazio em `sessoes/<sessão>.agentes/<agente>`, que o outro hook do mesmo evento não apaga. O JSON da sessão fica só para a contagem de uso e é gravado por troca de arquivo. O teste simula o JSON sobrescrito e a delegação legítima continua passando.
4. **Arquivo novo na raiz (Medium).** Um `main.ts` ainda não commitado também entra na camada `.raiz`.

Um risco segue de pé, e vale ter claro: o link e o patch são brechas de edição, mas o `bchive` ainda pode escrever pelo terminal, e isso continua como risco aceito no E2. O hook fecha as ferramentas de edição, não o shell.

**Para aplicar** (Git Bash, descompactando fora do repositório):
```bash
mkdir -p ~/Downloads/bchive-rev3 && unzip -o ~/Downloads/bchive-revisao-copilot-3.zip -d ~/Downloads/bchive-rev3
cd <caminho>/bia-credit-hive-plugin && git checkout fix/preparacao-repositorios-e-node && git pull
git apply ~/Downloads/bchive-rev3/bchive-revisao-copilot-3.patch
node testes/smoke.mjs        # esperado 54/54
git add -A && git commit -m "fix: reject a redirected state folder in the entry agent path guard"
git push
```

No Windows, o teste do link cria uma junction (não pede admin). Se o sistema recusar, esse caso é pulado e o resto roda.

**Respostas para o comentário de resumo do Copilot**, se quiser registrar:
- Link: "Corrigido: a comparação usa o caminho real (link e junction, inclusive para arquivo novo). Teste no smoke."
- Patch: "Corrigido: o cabeçalho só é lido do campo do patch. Teste com `file_text` e `new_str` em `estado/`."
- Sessão: "Corrigido com uma marca por agente que um hook não apaga do outro. O JSON da sessão é gravado por troca de arquivo. Teste simula a sobrescrita."
- Raiz: "Corrigido: arquivo novo na raiz entra na `.raiz`."

A descrição do PR atualizada está no zip (`descricao-pr.md`), com os quatro pontos.

**Fonte e passagem:**
- **Fonte:** o `bia-credit-hive-fonte-completa.zip` já vem com esta rodada. O núcleo passa 54/54, e gerar a marca a partir dele dá um pacote idêntico ao da branch. O arquivo do projeto ainda é o antigo: suba este no lugar dele pela interface do claude.ai.
- **Passagem:** o E2 e o P3 registram a 2ª rodada, e o estado aponta o patch novo.