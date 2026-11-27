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
