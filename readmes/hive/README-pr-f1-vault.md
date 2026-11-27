# F1 · Extrator do Vault de conhecimento: do zip à PR

Passo a passo para levar a F1 ao repositório `Bradesco-Automation/bia-credit-hive-plugin`. Todos os comandos são
para o **Git Bash**, com Node LTS 22 ou 24.

A PR muda **18 arquivos**: 5 alterados e 13 novos. O `plugin.json` **não** muda: a versão nova vem do título da PR
(`feat:` gera minor), lido pelo release-please depois do merge por squash.

## 1. Baixar o zip

Baixe `bia-credit-hive-plugin-f1.zip` do chat para a pasta Downloads. Confira:

```bash
ls -la ~/Downloads/bia-credit-hive-plugin-f1.zip
```

## 2. Preparar a branch

Ajuste `REPO` para o caminho do seu clone do plugin.

```bash
REPO=~/repos/bia-credit-hive-plugin
ZIP=~/Downloads/bia-credit-hive-plugin-f1.zip

cd "$REPO"
git status --short                 # tem que sair vazio; se não sair, guarde ou descarte antes
git switch main
git pull
git switch -c feat/vault-extractor
```

## 3. Copiar só os arquivos da F1

O zip traz o pacote inteiro, mas só estes 18 arquivos mudam. Copiar a lista evita sobrescrever o que o repositório
tem e o pacote não tem (`.github/`, arquivos do release-please, `CHANGELOG.md`).

```bash
TMP=$(mktemp -d)
unzip -q "$ZIP" -d "$TMP"
NOVO="$TMP/bia-credit-hive-plugin"

# alterados
for f in README.md \
         com.github.copilot/agents/bchive.agent.md \
         scripts/core/hook.mjs \
         scripts/core/lib/comum.mjs \
         testes/smoke.mjs; do
  cp "$NOVO/$f" "$f"
done

# novos
cp "$NOVO"/com.github.copilot/agents/bchive-vault.agent.md \
   "$NOVO"/com.github.copilot/agents/bchive-vault-extrator.agent.md \
   "$NOVO"/com.github.copilot/agents/bchive-vault-revisor.agent.md \
   com.github.copilot/agents/
cp "$NOVO/scripts/core/okf.mjs" scripts/core/
cp "$NOVO/scripts/core/lib/okf.mjs" scripts/core/lib/
cp -r "$NOVO/scripts/core/vendor" scripts/core/
cp -r "$NOVO/skills/vault-plan" "$NOVO/skills/vault-extract" "$NOVO/skills/vault-publish" skills/
```

## 4. Conferir antes do commit

```bash
git status --short
```

Saída esperada (a ordem pode variar):

```text
 M README.md
 M com.github.copilot/agents/bchive.agent.md
 M scripts/core/hook.mjs
 M scripts/core/lib/comum.mjs
 M testes/smoke.mjs
?? com.github.copilot/agents/bchive-vault-extrator.agent.md
?? com.github.copilot/agents/bchive-vault-revisor.agent.md
?? com.github.copilot/agents/bchive-vault.agent.md
?? scripts/core/lib/okf.mjs
?? scripts/core/okf.mjs
?? scripts/core/vendor/
?? skills/vault-extract/
?? skills/vault-plan/
?? skills/vault-publish/
```

Se aparecer outro arquivo, ou o `plugin.json`, pare: o repositório andou depois do pacote de 27/11. Traga a saída
para o chat.

```bash
node testes/smoke.mjs                                        # esperado: 62/62 passos ok
grep -rniE "hwk|henriworks|henrimap" . --exclude-dir=.git     # esperado: nada
ls scripts/core/vendor/okf-catalogo/                          # ORIGEM.md e os 4 arquivos do catálogo
rm -rf "$TMP"
```

## 5. Commit, push e PR

O título vira a mensagem do commit em `main` (merge por squash), e é ele que gera a versão. Use exatamente:

**`feat: add knowledge vault extractor`**

```bash
git add -A
git commit -m "feat: add knowledge vault extractor"
git push -u origin feat/vault-extractor
```

Abra a PR. Com o `gh`, o texto da seção 6 vira o corpo da PR:

```bash
sed -n '/^<!-- pr-inicio -->$/,/^<!-- pr-fim -->$/p' ~/Downloads/README-pr-f1-vault.md | sed '1d;$d' > /tmp/pr-f1.md
gh pr create --base main --head feat/vault-extractor \
  --title "feat: add knowledge vault extractor" --body-file /tmp/pr-f1.md
```

Sem o `gh`: abra
`https://github.com/Bradesco-Automation/bia-credit-hive-plugin/compare/main...feat/vault-extractor?expand=1`,
cole o título acima e o texto da seção 6.

Depois do merge: o unstable recebe uma versão `-dev`; o release-please abre ou atualiza a PR de release com
aumento **minor** a partir da última release, que está no `.release-please-manifest.json`. Por exemplo,
1.1.0 vira 1.2.0, ou 1.0.0 vira 1.1.0 se a release anterior ainda não saiu.

## 6. Texto da PR

<!-- pr-inicio -->
## Resumo

Adiciona o extrator do Vault de conhecimento (F1): o Hive passa a extrair do código o que é conhecimento (arquitetura
lógica e regra de camadas, padrões de código, regras de negócio, comportamento, versões que o time decidiu manter,
glossário e decisões) e a publicar notas no formato OKF 0.2 no clone do Vault da tribo, prontas para PR. Nenhum
código vai para o Vault.

## O que entra

- **Agente `bchive-vault`**: coordena a extração. Não lê código: trabalha com o inventário, o plano, os resumos das
  fatias e os rascunhos.
- **Subagentes `bchive-vault-extrator` e `bchive-vault-revisor`**: o extrator lê só o pacote de uma fatia, num
  contexto novo, e escreve as notas; o revisor confere cada nota contra as linhas de código que ela cita.
- **Skills `vault-plan`, `vault-extract` e `vault-publish`**: uma por fase, para carregar só as instruções da fase
  em curso.
- **`scripts/core/okf.mjs`**: `inventario`, `plano`, `aprovar-plano`, `pacote`, `concluir`, `checar`, `revisao`,
  `publicar`, `checar-vault`, `confirmar` e `estado`.
- **`scripts/core/lib/okf.mjs`**: núcleo OKF (validador em duas camadas, escrita do frontmatter, índices e log no
  formato do template do Vault).
- **`scripts/core/vendor/okf-catalogo/`**: `vault.mjs`, `constants.mjs`, `frontmatter.mjs` e `fs-utils.mjs` do
  catálogo (`reusable-ai`), copiados sem alteração, com o sha256 de cada um em `ORIGEM.md`.
- **Hook**: portões humanos novos e limites dos três agentes do Vault.
- **`bchive.json`**: chave `vault` nova, com padrão (`clone`, `squad`, `tetoTokensFatia` 30.000, `staleMeses` 6,
  `staleMesesDependencias` 3).
- **Agente de entrada `bchive`**: indica o `bchive-vault` quando o pedido é sobre o Vault.
- **README**: seção "Vault de conhecimento" e estrutura do plugin atualizada.

## Como funciona

1. `/bia-credit-hive:vault-plan <pasta> squad: <squad>`: o inventário tira configuração, pipeline, lock e código
   gerado, estima os tokens e agrupa por pasta abaixo do teto de cada fatia. O agente escreve o plano (fatias e notas
   previstas) e o dev aprova no terminal: `okf.mjs aprovar-plano <id> --por <login>`.
2. `/bia-credit-hive:vault-extract <id>`: cada fatia vai para um subagente novo, com um pacote montado por script
   (só os arquivos da fatia, com número de linha). Cada afirmação da nota cita a fonte (`caminho#Lx-Ly`), e toda
   nota tem a seção "Perguntas em aberto". O coordenador junta pelos resumos, e o revisor isolado dá o veredito.
3. `/bia-credit-hive:vault-publish <id> vault: <clone>`: aplica notas, índices e log numa cópia do clone, roda as duas
   camadas de validação e só então grava no clone. O dev abre a PR do Vault (`feat:`). As notas sobem como `draft`,
   sem `verified`.
4. Quem conferiu a nota na fonte roda `okf.mjs confirmar <nota> --por <login>`: grava `verified` (lista, como no
   OKF 0.2) e passa a nota a `stable`.

## Validação em duas camadas

- **Catálogo**: o mesmo `checkVault` que roda no CI do Vault, mais a checagem da entrada "Replace with" do índice
  do template.
- **Hive, mais dura**:
  - nenhum código na nota (bloco, indentado, `pre`/`code` em HTML ou código em linha); só diagrama Mermaid;
  - credencial e infraestrutura (hostname, IP privado, cluster, bucket) são erro, não aviso;
  - CPF, CNPJ, e-mail e telefone são erro;
  - frontmatter completo, com `type` de uma lista fechada: Architecture, Coding Standard, Business Rule, Behavior,
    Dependency Baseline, Glossary, Decision;
  - fonte por afirmação e seção "Perguntas em aberto";
  - caminho em kebab-case, sem acento;
  - aviso de orçamento a partir de 80% dos 1.000 arquivos do Vault.
- O script põe aspas no texto do frontmatter: na validação, o parser do catálogo recusou um valor com ": " sem aspas.

## Limites (hook)

- `okf.mjs aprovar-plano` e `okf.mjs confirmar` são portões humanos: o hook nega para qualquer agente.
- `bchive-vault`: só escreve em `.bchive/vault/`, só chama os dois subagentes do Vault e não roda Git que escreve
  (nem com `git -C <pasta>`). As notas chegam ao clone só pelo `okf.mjs publicar`.
- Extrator: só escreve os próprios rascunhos e o resumo; sem terminal nem subagente.
- Revisor: só escreve o veredito; sem terminal nem subagente.

## Testes

- Teste de fumaça: **62/62** no núcleo e no pacote da marca. Os 9 passos novos ou alterados falham contra o código
  anterior. Os passos novos cobrem:
  - o código do catálogo copiado sem alteração;
  - a entrada "Replace with" do template;
  - inventário, plano, teto, tipo e caminho;
  - aprovação humana do plano e plano editado depois de aprovado;
  - as regras de conteúdo;
  - o conserto de aspas e a normalização que não muda a nota já normalizada;
  - publicação com revisão aprovada, numa cópia, sem escrever nada quando há erro e sem duplicar o log;
  - confirmação;
  - os limites do hook para os três agentes.
- Publicação testada contra uma cópia do template real do `reabilitai-vault` (`plugin.json`, skill leitora,
  CODEOWNERS, índice e log): depois da publicação, o `checkVault` do catálogo dá 0 achados.
- Simulação do grafo (não implementado nesta PR) com um Vault sintético de 929 arquivos: 0 erros e 0 avisos no
  `checkVault`.

## Como testar no unstable

1. Instale a versão `-dev` do `bradesco-marketplace-unstable` e rode `/bia-credit-hive:init` num repositório de
   produto.
2. No agente `bchive-vault`: `/bia-credit-hive:vault-plan <uma pasta pequena> squad: <squad>`, aprove no terminal e
   siga com `vault-extract` e `vault-publish` apontando para um clone do `reabilitai-vault`.
3. No clone: `node <repo do produto>/.bchive/bin/okf.mjs checar-vault .` sem erro antes de abrir a PR do Vault.

## Riscos e pendências

- Se a ferramenta não disparar o hook do frontmatter nos subagentes, os limites do extrator e do revisor ficam na
  lista de ferramentas (sem terminal) e no texto. Conferir com o canário no uso real.
- O commit de origem dos arquivos do catálogo não veio no envio: o `ORIGEM.md` guarda o sha256 de cada um, e o
  teste de fumaça falha se algum for editado.
- O CONTRIBUTING do Vault aceita menos tipos de arquivo que o `vault.mjs` (YAML, TXT e CSV); vale o validador. O
  extrator só gera `.md`.
- Grafo para consulta do Vault pelo Hive: fica para uma evolução; por enquanto o Vault é lido pelo índice.

## Versão

`feat:` gera aumento minor. Sem quebra de compatibilidade: os comandos, as skills e a configuração existentes não
mudam.
<!-- pr-fim -->

>
>
>
Vou tratar os quatro achados como bloqueantes. Antes de mexer: a fonte desta sessão continua aqui, e o teste de fumaça continua em 62/62? Também vou ver como o hook já protege os arquivos dos portões que existem hoje, porque o achado 3 tem um primo que o Copilot não apontou.

O hook já trata como imutáveis o veredito e a mudança arquivada, para qualquer agente. Antes de mexer no terminal do coordenador, vejo como o hook lê o texto do comando.

Agora os testes de cada achado, no teste de fumaça.

63/63. Agora confiro que cada teste novo falha contra o código que está na PR.

O passo novo para no primeiro erro (o CPF). Para mostrar que cada correção é necessária por si, rodo de novo pondo as correções uma a uma sobre o código da PR.

Atualizo a passagem e o zip da fonte.

Os quatro achados eram bloqueantes de verdade, e corrigi todos. O `bia-credit-hive-revisao-copilot.zip` traz só os 4 arquivos que mudam na branch da PR. Com eles, o teste de fumaça passa de 62/62 para 63/63, no núcleo e na marca. Para cada correção, conferi que o teste falha sem ela.

## O que mudou

| Achado | Correção |
|---|---|
| **Terminal do coordenador** (um `echo > clone/vault/nota.md` passava fora das duas camadas) | O terminal do `bchive-vault` passou a aceitar só `node .bchive/bin/okf.mjs` nos subcomandos que não são portão humano, um por vez, sem `>`, pipe, `&&`, `;` nem `$(...)`. O hook lê o campo do comando, não a explicação que o agente manda junto. |
| **CPF e CNPJ sem pontuação** | 11 e 14 dígitos agora são erro quando o dígito verificador é válido. Sem essa conferência, qualquer número de protocolo seria acusado. Com pontuação, o formato basta. |
| **Aprovação velha** (nota editada depois do APROVADO passava sem nova revisão) | O comando `revisao` agora apaga o veredito quando a nota muda. A publicação exige um APROVADO novo. |
| **Erro em nota que já estava no Vault** (impresso, mas não travava) | Ajustei em relação à sugestão do Copilot, que travava qualquer erro. Credencial, infraestrutura e dado pessoal travam a publicação em qualquer nota. Formato e código em linha numa nota de outra origem, como as do Companion, saem como aviso. Senão, uma nota legítima de outra pessoa bloquearia toda publicação do Hive. |

**Um problema da mesma família, que o Copilot não apontou:** o coordenador, ou qualquer agente, podia forjar a aprovação do plano ou um veredito editando o `estado.json` ou o arquivo do veredito. Agora o hook nega isso para todos os agentes; só o `okf.mjs` escreve esses arquivos.

## Aplicar na PR (Git Bash)

```bash
REPO=~/repos/bia-credit-hive-plugin
cd "$REPO" && git switch feat/vault-extractor && git pull
TMP=$(mktemp -d) && unzip -q ~/Downloads/bia-credit-hive-revisao-copilot.zip -d "$TMP"
cp -r "$TMP"/bia-credit-hive-plugin/* .
git status --short          # esperado: só os 4 arquivos, todos com M
node testes/smoke.mjs       # esperado: 63/63 passos ok
git add -A && git commit -m "fix: address copilot review findings" && git push
```

O título da PR continua `feat: add knowledge vault extractor`. Com o squash, é ele que vira o commit em `main`, e o `fix:` intermediário não conta na versão.

## Respostas para as conversas da PR

O merge automático espera todas as conversas resolvidas. Cole uma resposta em cada conversa e clique em Resolve:

1. **Reject shell redirection and chaining:** "Corrigido. O terminal do coordenador agora aceita só o okf.mjs nos subcomandos que não são portão, um por vez, sem redirecionamento, pipe, encadeamento nem substituição. A checagem lê o campo do comando, não a explicação. Teste no smoke, inclusive com `echo > clone/vault`."
2. **Detect unformatted CPF and CNPJ:** "Corrigido, com conferência do dígito verificador para 11 e 14 dígitos, para não acusar qualquer número longo. Com pontuação, o formato basta. Teste no smoke."
3. **Clear stale approval:** "Corrigido. O `revisao` apaga o veredito quando a nota muda, e só pula a nota com veredito válido para o hash atual. Teste no smoke: nota editada depois do APROVADO não é publicada."
4. **Count validation errors from every checked note:** "Corrigido para exposição: credencial, infraestrutura ou dado pessoal em qualquer nota do Vault travam a publicação, e nada é escrito. Regras de formato do extrator, como código em linha, numa nota que não veio do Hive saem como aviso, para não travar a publicação por uma nota legítima de outra origem. Teste no smoke para os dois casos."

**Risco antigo, não tratado nesta PR:** o agente de entrada `bchive` ainda pode escrever pelo terminal, como o coordenador podia. A passagem registra isso como risco aceito desde o E2. Se o Copilot apontar de novo, a mesma lista de comandos permitidos resolve, mas o certo é tratar em outra PR.

A passagem já registra os quatro achados, a decisão sobre as notas de outra origem e o teste de fumaça em 63/63. O `bia-credit-hive-fonte-completa.zip` substitui o arquivo da fonte no projeto.
