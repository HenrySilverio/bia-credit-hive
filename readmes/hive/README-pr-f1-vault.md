São três bloqueantes, e os três procedem. Desta vez corrijo cada um como uma classe inteira, não só o ponto que o Copilot apontou:

1. **Link simbólico na pasta de rascunhos.** Toda leitura e escrita dentro de `.bchive/vault/<extração>/` passa a conferir cada parte do caminho a partir da raiz do repositório: estado, plano, inventário, pacotes, rascunhos, resumos, vereditos e o registro de revisão.
2. **Nota aprovada que sumiu.** O conjunto de rascunhos tem que ser exatamente o do plano, sem nota a mais nem a menos, na conferência, na revisão e na publicação.
3. **CPF fora das notas.** Uma varredura de exposição única para todo arquivo de texto do `vault/` (`.md`, `.json`, `.yaml`, `.yml`, `.txt`, `.csv` e `.svg`), na publicação e no `checar-vault`.

Começo mapeando todo acesso a arquivo no `okf.mjs`.

Agora ligo a varredura na publicação e no `checar-vault`.

As duas falhas são mensagens que mudaram de texto, não de comportamento: a publicação travou e o link foi recusado nos dois casos. Ajusto as duas expectativas do teste.

Agora o passo da terceira rodada no teste de fumaça, com uma extração completa nova para isolar os cenários.

66/66. Agora a prova contrária dos três achados: cada um como cenário isolado, contra o código que está na PR (`2c03a1d`) e contra o novo.

A falha é do meu comando, não do código: passei o caminho do plugin relativo (`plugin`), e o cenário roda o `init` de dentro de outra pasta. Rodo de novo com o caminho absoluto.

Três arquivos mudam em relação à PR. Monto o zip, confiro a aplicação e atualizo a passagem.

Corrigi os três bloqueantes, e desta vez por classe, não no ponto exato do achado. O teste de fumaça dá 66/66 no núcleo e na marca. Rodei cada achado como cenário isolado: os três aparecem no código da PR e somem no novo, e os cinco da rodada anterior continuam corrigidos. O `bia-credit-hive-revisao-copilot-3.zip` traz os três arquivos que mudam.

## O que mudou

**1. Link simbólico na pasta da extração.** O Copilot apontou `rascunhos/<tema>`, mas o mesmo valia para todo o estado da extração. Agora toda leitura e escrita dentro de `.bchive/vault/<extração>/` passa por quatro funções que conferem cada parte do caminho a partir da raiz do repositório. Isso cobre estado, plano, inventário, pacotes, rascunhos, resumos, vereditos e registro de revisão. Um link em qualquer ponto para o comando sem ler nem escrever nada. Também cobre um caso que o Copilot ainda não tinha apontado: veredito APROVADO forjado fora do repositório, alcançado por um link em `revisao/`.

**2. Nota aprovada que sumia.** Checar, revisar e publicar agora exigem exatamente as notas do plano, nem a mais nem a menos. Para tirar uma nota, é preciso mudar o plano e pedir aprovação de novo.

**3. CPF fora das notas.** Uma varredura única de exposição (credencial, infraestrutura e dado pessoal) roda em todo arquivo de texto do `vault/`: `.md`, `.json`, `.yaml`, `.yml`, `.txt`, `.csv` e `.svg`. Para cada tipo, ela usa o leitor do catálogo. Vale para índices e logs de qualquer pasta, tocados ou não, na publicação e no `checar-vault`.

**Da auditoria:** o `checar-vault` lia através de link no clone, inclusive quando o próprio `vault/` era um link. Agora acusa o link e não lê. Conferi um por um os acessos a arquivo que sobraram no `okf.mjs`: são da cópia temporária, criada sem links, ou do clone e da nota do `confirmar`, os dois já checados contra link.

**Limite que fica:** imagem e PDF não têm scanner. Isso é conferência humana, como o próprio CONTRIBUTING do Vault diz. Se o Copilot apontar, a resposta é essa.

## Aplicar na branch (Git Bash, no mesmo terminal)

```bash
REPO=~/repos/bia-credit-hive-plugin
cd "$REPO" && git switch feat/vault-extractor && git pull
TMP=$(mktemp -d) && unzip -q ~/Downloads/bia-credit-hive-revisao-copilot-3.zip -d "$TMP" && find "$TMP" -type f | wc -l   # esperado: 3
cp -r "$TMP"/bia-credit-hive-plugin/. .
git status --short          # esperado: M em scripts/core/okf.mjs, scripts/core/lib/okf.mjs e testes/smoke.mjs
node testes/smoke.mjs       # esperado: 66/66 passos ok
git add -A && git commit -m "fix: address third copilot review round" && git push
```

## Respostas para as três conversas

1. **Concluir pode sobrescrever arquivos externos:** "Corrigido para todo o estado da extração. Toda leitura e escrita em .bchive/vault/<extração>/ confere cada parte do caminho a partir da raiz do repositório; link em qualquer ponto para o comando sem ler nem escrever. Testes no smoke: rascunhos/<tema> como link (o arquivo de fora fica igual) e revisao/ como link para vereditos forjados."
2. **Notas aprovadas ausentes:** "Corrigido. Checar, revisão e publicação exigem exatamente o conjunto de notas do plano aprovado, nem a mais nem a menos. Teste no smoke com uma nota apagada depois de concluir."
3. **CPF em outros arquivos:** "Corrigido. Uma varredura de exposição (credencial, infraestrutura e dado pessoal, com o leitor do catálogo por tipo) roda em todo arquivo de texto de vault/, na publicação e no checar-vault. Testes no smoke com CSV, JSON, log aninhado e índice não tocado. Imagem e PDF seguem com conferência humana."

A passagem registra a terceira rodada e o limite das imagens. O `bia-credit-hive-fonte-completa.zip` substitui o arquivo da fonte no projeto.