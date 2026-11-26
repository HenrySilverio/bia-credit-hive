## Contexto

A preparação do b.IA Credit Hive quebrava em repositórios fora do padrão e bloqueava a adoção nas squads:

- o agente `bchive` recusava rodar o `init` por causa do próprio limite de edição;
- sem `package.json` na raiz, em Node sem `src/` e em monolito com o front numa subpasta, o `/bia-credit-hive:configurar` saía com erro;
- em workspace com pastas de log fora do Git (massa para mocks), uma pasta ilegível derrubava o `configurar` e o mapa; os arquivos de log contavam como "alteração não commitada" e travavam a revisão, o veredito e o `archive`;
- quando algo falhava, o `init` mostrava só `build falhou: Node.js v22…` ou nada, e o chat pedia para repetir;
- o mínimo de Node era 20, contra a regra do Hive (LTS 22 ou 24).

## O que muda

- **Agente `bchive`:** roda o `init.mjs` e os scripts de `.bchive/bin/` sem recusar por causa do limite de edição.
- **Repositório fora do padrão:** o `configurar` lista os arquivos pelo Git (não varre o disco) e, quando não é projeto Node na raiz, propõe camadas por pasta, sem comando que não consiga provar, com o aviso "Sem verificação automática". Gravar essa proposta tira o molde de MFE Angular.
- **Pastas locais:** o `init` detecta pastas da raiz fora do Git com 50 ou mais arquivos e grava essas pastas em `pastasLocais` no `.bchive/bchive.json`, que não é versionado. Elas saem da árvore suja, do mapa e do `configurar`. O `.gitignore` e o Git do dev não mudam.
- **Mapa:** pasta ilegível não derruba mais o mapa, e arquivo não rastreado de pasta inteira fora do Git não entra nele.
- **Erro visível:** o `init` mostra o motivo real (por exemplo, `Error: EPERM…` ou "interrompido depois de 4 min"). As skills `init` e `configurar` mandam o agente mostrar a saída inteira, sem resumir e sem pedir para repetir.
- **Node LTS 22 ou 24:** abaixo de 22, o `init` e o `configurar` param e os outros comandos avisam; versão ímpar ou acima de 24 só avisa. A mensagem orienta seguir o documento de instalação e configuração do b.IA Credit Hive no Confluence do banco e, se o Node já foi instalado, fechar e abrir o VS Code.
- **Teste de fumaça:** 3 passos novos (repositório sem Node na raiz, workspace com pastas de log, versão do Node). Total: 50/50.

## Como testar (Git Bash)

```bash
node -v                       # v22.x ou v24.x
node testes/smoke.mjs         # esperado: 50/50 passos ok
```

Num repositório real de cada caso (sem `package.json`, workspace com pastas de log, monolito):

```bash
node "<pasta do plugin>/skills/init/scripts/init.mjs"
node .bchive/bin/configurar.mjs verificacao
node .bchive/bin/bchive.mjs arvore
```

Esperado: o `init` termina com o relatório, e a linha `pastas-locais` lista as pastas de log quando houver; o `configurar` sai com código 0; a árvore não mostra arquivo de log.

## Riscos

- Uma pasta nova na raiz com 50 ou mais arquivos e nada commitado vira pasta local e sai da checagem de árvore suja. O `init` lista quais pastas pegou e, para desfazer, basta tirá-las de `pastasLocais`.
- Quem está hoje no Node 20 passa a ser barrado no `init` e no `configurar`. É a regra do Hive, mas vale avisar as squads.
- Testado em Linux. Falta o canário no Windows com a versão publicada.

## Fora deste PR

- Lint, tipos e testes automáticos para front em subpasta e para Java (hoje o `configurar` só avisa).
- O problema do chat que não reconhece o agente no VS Code.