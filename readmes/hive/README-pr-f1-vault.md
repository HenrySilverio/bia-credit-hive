A quarta rodada trouxe seis achados, cinco novos. Antes de corrigir, confiro o que o Copilot apontou contra o código real. Sobre o zip: se você ainda não trocou o arquivo do projeto, faz sentido. Só leve a fonte completa para o projeto quando a PR passar.

- **Achado 4 (CPF em outros arquivos)** não está marcado como "New". É a conversa da rodada anterior, que continua aberta porque ainda não foi resolvida. A correção já está na PR (a varredura de exposição em todo arquivo de texto do `vault/`). Basta responder e resolver essa conversa.
- **Os outros cinco procedem.** E o primeiro tem um primo mais antigo: os portões do Hive que já existiam (`aprovar-spec`, `qa-ok` e os outros) usam a mesma regex sobre o texto cru e caem do mesmo jeito com aspas no meio do subcomando. Corrijo os dois juntos.

Corrigi a quarta rodada e o teste de fumaça passa: 67/67 no núcleo e no pacote da marca. O pacote não tem nenhum termo interno. O passo novo do teste falha contra o código da PR e passa no novo. O zip `bia-credit-hive-revisao-copilot-4.zip` traz só os 5 arquivos que mudaram. Aplicado sobre a PR, o resultado fica idêntico ao pacote novo (conferido com `diff`).

**Achado 4 (CPF em outros arquivos):** é a conversa da 3ª rodada, já corrigida. É só responder e resolver.

**Os cinco achados novos.** Reproduzi cada um no código da PR (DEFEITO) e no código novo (OK):

| # | O problema | O que mudou |
|---|---|---|
| 1 | O hook lia o texto cru: `con"firmar"`, `aprovar-'plano'` e `"qa-ok"` passavam | O hook agora testa o comando como o shell entrega ao programa, sem aspas, escapes nem `$'...'`. A explicação da chamada não conta mais, então não há bloqueio por engano. `aprovar-plano` é negado em qualquer comando, e `confirmar ... --por` também. `okf.mjs` só roda na forma canônica, para qualquer agente. `bchive.mjs` com variável, `$(...)`, crase ou `eval` é negado. Os portões antigos (`aprovar-spec`, `qa-ok` e os outros) tinham a mesma falha e entraram na correção |
| 2 | O rascunho podia trocar o `type` aprovado no plano | O `type` tem que ser o do plano. Senão sai o erro `okf/plano` |
| 3 | A publicação desfazia edições feitas no clone durante a publicação | Na hora da cópia, a publicação guarda o conteúdo de cada arquivo que vai mudar (nota, índices e log). No fim copia só esses, e para tudo se algum mudou no clone nesse meio tempo. Uma nota de outra squad fica como a outra pessoa deixou. Duas travas impedem dois processos ao mesmo tempo: uma no clone, em `.git/` (vale também no `confirmar`), e uma por extração. A trava de um processo que já morreu é assumida |
| 5 | O rodapé podia apontar para outra faixa de linhas | Cada `[^id]:` tem que repetir o `resource` de mesmo id; o prefixo do repositório é aceito. Um rodapé sem item em `sources` é erro |
| 6 | Uma extração parcial seguia para consolidar e revisar | `checar` e `revisao` recusam enquanto houver fatia não feita. A skill para, lista o que falta e manda continuar num chat novo com `/bia-credit-hive:vault-extract <id>` |

**Aplicar (Git Bash):**
```bash
cd ~/caminho/bia-credit-hive-plugin
git switch feat/vault-extractor
mkdir -p /tmp/r4 && unzip -o ~/Downloads/bia-credit-hive-revisao-copilot-4.zip -d /tmp/r4
cp -r /tmp/r4/. .
node testes/smoke.mjs          # esperado 67/67
git add -A && git commit -m "fix: address fourth copilot review round" && git push
```

**Respostas para as conversas da PR:**
- Portões: "Corrigido. Os portões agora são testados no comando como o shell entrega ao programa, sem aspas, escapes nem `$'...'`. `okf.mjs` fora da forma canônica é negado para qualquer agente. Os portões antigos do Hive, que tinham a mesma falha, entraram na correção."
- Tipo: "Corrigido. O `type` do rascunho tem que ser o aprovado no plano (`okf/plano`)."
- Edição concorrente: "Corrigido. A publicação copia só os arquivos que ela mudou e aborta se algum mudou no clone durante a publicação. Há trava no clone e na extração."
- Rodapés: "Corrigido. Cada rodapé tem que repetir o `resource` da fonte de mesmo id."
- Extração parcial: "Corrigido. `checar` e `revisao` recusam com fatia pendente, e a skill para e orienta a continuar num chat novo."
- CPF (3ª rodada): "Já corrigido na rodada anterior: a varredura de exposição cobre todo arquivo de texto de `vault/`."

**Trade-offs:**
- **Falso positivo possível:** um comando legítimo que contenha a palavra `aprovar-plano` (um `grep` no terminal, por exemplo) agora é negado. Ler o arquivo pela ferramenta de leitura continua livre.
- **Falha que continua:** o hook é guarda-corpo, não sandbox. Um script gravado em arquivo e rodado depois ainda escapa. A defesa final continua sendo os portões humanos.

**Passagem atualizada:**
- **Estado:** quatro rodadas (4 + 5 + 3 + 5 achados) e teste em 67/67.
- **Regra dos portões:** agora diz como o hook lê o comando.
- **F1:** ganhou o item da 4ª rodada.
- **Fonte do projeto:** fica registrado que o zip só é trocado quando a PR passar.

O zip da fonte completa vai junto para você guardar até lá.