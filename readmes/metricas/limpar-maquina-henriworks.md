A variável ficou certa (o PowerShell e o Git Bash mostram o caminho novo). Mas o `copilot-otel.jsonl` continua com 4,6 MB, gravado às 20:24. O VS Code ainda não gravou nada depois da mudança: ou ele não foi fechado e reaberto, ou ainda não teve nenhuma conversa no chat.

Também apareceram restos do HenriWorks num repositório (`recr-fed-agc-posvenda`): a pasta `.henriworks`, o hook, a skill e os agentes `hwk-*` em `.github/`. Estão todos marcados "apagaria", nenhum "NO GIT": não estão versionados (o `.github/` nunca é), então podem ir.

**1. Aplicar a limpeza**
```bash
cd ~/Downloads
bash limpar-henriworks.sh --procurar ~ --aplicar
```
No fim tem que aparecer "N item(ns) removido(s)".

**2. Reiniciar o VS Code de verdade**
1. Feche todas as janelas do VS Code.
2. Confira que ele fechou:
```bash
tasklist | grep -i "code.exe" || echo "VS Code fechado"
```
3. Abra o VS Code de novo no `recr-fed-agc-posvenda` (ou em outro repositório com o Hive).

**3. Gerar uso**

No chat, selecione o agente `bchive` e faça uma pergunta qualquer, por exemplo "qual o estado do repositório?". Espere a resposta terminar.

**4. O que eu preciso ver no final**
```bash
ls -la ~/.bchive/uso/otel/
for f in ~/.bchive/uso/otel/*.jsonl; do echo "== $(basename "$f")"; tail -n 300 "$f" | grep -o '"service.name":"[^"]*"' | sort | uniq -c; done
```
Esse comando mostra só o tamanho e a hora dos arquivos e quantas linhas recentes vêm de cada serviço; nenhum conteúdo.

Como vou ler:

| Resultado | Significa |
| --- | --- |
| Aparece `copilot-harness-otel.jsonl` com linhas `github-copilot` | O harness grava pela variável. A cobertura das sessões do fluxo está resolvida |
| Só o `copilot-otel.jsonl` cresceu, com hora depois de 20:53 | O VS Code grava, o harness não. Tokens das sessões do fluxo dependem da TI |
| Nenhum arquivo mudou | O VS Code não pegou a configuração nova. Aí investigamos o `settings.json` |

Me mande a saída dos passos 1 e 4.