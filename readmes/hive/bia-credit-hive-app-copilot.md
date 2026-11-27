Vou corrigir. Antes de mexer, confiro se a fonte já tem as mudanças do PR aberto (P3 e Node), para não desfazer nada, e rodo o teste de fumaça.

Um bloqueio antes de entregar: a fonte do projeto está **antes** do PR aberto. O teste de fumaça dá 47/47 (o PR dá 50/50) e não existe `pastasLocais` no código. O P3 mexeu em arquivos que eu também vou tocar, no mínimo o `testes/smoke.mjs` e talvez o `hook.mjs`. Se eu entregar sobre esta base, desfaço o P3.

Me manda o `bia-credit-hive-correcao-preparacao.zip`, os 17 arquivos do PR. Serve também um zip do plugin na branch `fix/preparacao-repositorios-e-node`. Enquanto isso, já faço a correção aqui como patch e, quando o zip chegar, aplico em cima do PR e rodo a fumaça de novo.

Now the hook edits.

Fiz a correção. Está no `bchive-limites-agente-entrada.zip`, mas eu não a testei sobre a branch do PR, porque a fonte do projeto está antes do P3. O teste de fumaça passou 49/49 sobre a fonte e sobre o pacote da marca. O patch aplica limpo nesse pacote, e não aparece nenhum termo do núcleo interno no que vai para o banco.

**O que mudou no plugin** (5 arquivos: `bchive.agent.md`, `hook.mjs`, `bchive.mjs`, `skills/start/SKILL.md`, `testes/smoke.mjs`):
- **Hook do agente `bchive`:** um `preToolUse` no frontmatter dele nega edição fora de `.bchive/estado/` e qualquer subagente que não seja o `bchive-gf`. Vale também em repositório sem `.bchive/`.
- **Hook do plugin:** o `bchive-implementador` só pode ser chamado numa sessão em que o `bchive-implement` rodou.
- **Texto do agente e da skill `start`:** descrever o trabalho não é pedido para fazê-lo, e o agente reconhece `/start` sem prefixo.
- **Canário:** passa a mostrar se o hook do agente rodou.
- **Teste de fumaça:** 2 passos novos e uma conferência a mais no canário.

**Para aplicar na branch** (Git Bash, na raiz do repositório do plugin):
```bash
git checkout fix/preparacao-repositorios-e-node
git apply --3way bchive-limites-agente-entrada.patch
node testes/smoke.mjs        # esperado 52/52
git add -A && git commit -m "fix: enforce entry agent limits with an agent-scoped preToolUse hook"
```
Se o `--3way` acusar conflito no `smoke.mjs` ou no `hook.mjs`, foi o P3 mexendo perto. Nesse caso me mande o zip do PR que eu integro aqui.

**Título:** o atual não cobre o E2, e no squash merge ele vira o changelog. Sugestão: `fix: harden repository setup, require Node LTS 22/24 and enforce entry agent limits`.

**Descrição do PR** (também está como `descricao-pr.md` no zip):

```markdown
## Resumo

Corrige a preparação (`init` e `configurar`) em repositórios fora do padrão, exige Node LTS 22 ou 24 e põe em código o limite do agente de entrada `bchive`, que até agora existia só no texto do agente.

## O que muda

### 1. Preparação em repositórios fora do padrão
- `configurar` não sai mais com erro sem `package.json` na raiz, em Node sem `src/` ou em monolito com o front numa subpasta. Ele propõe camadas por pasta, não propõe comando que não sabe provar e mostra "Sem verificação automática". Ao gravar, tira o molde de MFE do `verificacao.json`.
- Pasta ilegível (presa por outro processo, sem permissão, caminho longo no Windows) não derruba mais o `configurar`, o mapa nem as consultas do agente ao mapa. A lista de arquivos vem do Git.
- Pastas de log fora do Git e do `.gitignore` não travam mais `pacote-revisao`, `gravar-veredito` e `arquivar`. O `init --corrigir` grava em `pastasLocais` do `bchive.json`, que não é versionado, as pastas da raiz fora do Git com 50 ou mais arquivos. O Git e o `.gitignore` do dev não mudam.
- Quando um passo falha, o `init` mostra o motivo real (`Error: EPERM…`, "interrompido depois de 4 min") e não a última linha da pilha.

### 2. Node LTS 22 ou 24
- Com Node abaixo de 22, `init` e `configurar` param e os outros comandos avisam. Versão ímpar ou acima de 24 só gera aviso.
- A mensagem aponta para o documento de instalação e configuração no Confluence e pede para fechar e abrir o VS Code se o Node já foi instalado.

### 3. O `bchive` roda o `init.mjs` sem recusar
- Os scripts do fluxo escrevem onde precisam (`.bchive/`, `.git/info/exclude`, hooks locais do mapa). Isso não conta como edição do agente.

### 4. Limites do agente de entrada em código
**Problema.** No GitHub Copilot app, um `/start` com a demanda já descrita fez o `bchive` gerar os arquivos da demanda. Os eventos de uso dessa sessão mostram 15 minutos com buscas, 6 comandos de terminal e 10 escritas no turno principal, sem subagente. Não houve discovery, plano, revisão nem portão humano. O limite "só edita em `.bchive/estado/`" estava só no texto do agente, e o hook não conferia nada disso. A mesma brecha deixava qualquer agente chamar o `bchive-implementador`.

**Correção.**
- **Hook do agente:** o `bchive` ganha um `preToolUse` no frontmatter (`--agente bchive`). Ele:
  - nega edição fora de `.bchive/estado/`. O caminho é lido de qualquer campo de caminho, de listas e de cabeçalho de patch, e não de campo de conteúdo. `..` não escapa da pasta. Escrita sem caminho visível é negada;
  - nega subagente que não seja o `bchive-gf`;
  - vale também em repositório ainda não preparado;
  - não coleta uso, porque o hook do plugin já registra a mesma ferramenta.
- **Hook do plugin:** o `bchive-implementador` só pode ser chamado numa sessão em que o `bchive-implement` rodou.
- **Sessão:** o `userPromptSubmitted` lê a sessão de novo antes de gravar. Os dois hooks rodam no mesmo evento, e a lista de agentes decide quem pode chamar o executor.
- **Texto:**
  - O agente e a skill `start` dizem que descrever o trabalho não é ordem para fazê-lo.
  - O agente reconhece o comando sem prefixo (`/start`), que é como o Copilot app lista os comandos.
- **Canário:** o `canario conferir` agora diz se o hook do agente rodou no `preToolUse` ("limites do agente de entrada: ativos") e grava `travaEntrada` no `canario.json`.

**Fica de fora (risco aceito).** Escrita pelo terminal, por redirecionamento de shell, não é travada. Fazer isso por padrão de texto seria frágil.

## Como testar
    node testes/smoke.mjs                                # esperado 52/52

No Windows, em um repositório real de cada caso:
1. `init`, `configurar` e canário (P3 e Node).
2. No agente `bchive`, no Copilot app e no VS Code:
   - `/start` com a demanda descrita: responde só o próximo passo;
   - "crie o arquivo X": o hook nega com a mensagem do b.IA Credit Hive;
   - canário: mostra "limites do agente de entrada: ativos".
3. No `bchive-implement`, a delegação de uma fatia ao `bchive-implementador` continua funcionando.

Se o canário mostrar o aviso de que o hook do agente não rodou, a ferramenta não dispara o hook do frontmatter no `preToolUse`. Nesse caso os limites do `bchive` voltam a depender só do texto.
```

**Riscos:**
- **O hook pode não disparar no app.** Ainda não está provado que o Copilot app dispara o hook do frontmatter no `preToolUse`. O canário agora acusa isso: rode-o no app antes do merge.
- **Ferramenta desconhecida é barrada.** Uma ferramenta de escrita com nome que o hook não conhece e sem campo de caminho é negada para o `bchive`. Prefiro errar para esse lado.

**Passagem atualizada:**
- **E2:** item novo na Onda 0, com o caso real, a correção, o que falta testar e o risco aceito.
- **P7:** comandos sem prefixo no app.
- **F7:** o app não entrega o comando ao hook.
- **Regra:** limite de agente que importa fica no hook, não só no texto.
- **Fonte desatualizada:** a passagem agora diz que a fonte do projeto está atrás do PR. A próxima mudança em `hook.mjs`, `bchive.mjs` ou `smoke.mjs` precisa da fonte nova.