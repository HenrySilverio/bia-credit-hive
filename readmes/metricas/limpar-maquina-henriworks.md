Antes de rodar, um aviso: o script de limpeza apaga a pasta `~/.henriworks` inteira. É ali que está o arquivo de telemetria de 09 a 26/11, a única cópia bruta. O importador do Hive ainda vai ser corrigido, e depois vai precisar reimportar esse arquivo. Por isso a ordem abaixo salva a telemetria primeiro e só então limpa.

Ao copiar, o arquivo perde as métricas. Elas são 97% das linhas e o Hive não usa nenhuma; ficam só spans e logs.

**0. Feche o VS Code (todas as janelas) e confira**
```bash
tasklist | grep -i "code.exe" || echo "VS Code fechado"
```
Com ele aberto, o Windows trava o arquivo de telemetria.

**1. Ver o que tem na pasta antiga**
```bash
du -sh ~/.henriworks/* 2>/dev/null
ls ~/.henriworks/uso 2>/dev/null
```
Se aparecer algo além de `uso/otel`, me avise antes do passo 4.

**2. Salvar a telemetria sem as métricas, no lugar que o Hive já lê**
```bash
ANT="$HOME/.henriworks/uso/otel/copilot-otel.jsonl"
NOVO="$HOME/.bchive/uso/otel/copilot-otel.jsonl"
mkdir -p "$(dirname "$NOVO")"
if [ -e "$NOVO" ]; then echo "PARE: $NOVO já existe"; else
  LC_ALL=C grep -v '"scopeMetrics"' "$ANT" > "$NOVO"
  ls -la "$ANT" "$NOVO"; wc -l "$NOVO"
fi
```
O esperado é um arquivo novo bem menor, com perto de 3.600 linhas.

**3. Apontar a telemetria do VS Code para a pasta nova**
```bash
S="$APPDATA/Code/User/settings.json"
cp "$S" "$S.bak-hive"
sed -i '/github\.copilot\.chat\.otel\.outfile/s/\.henriworks/.bchive/' "$S"
grep -n 'github.copilot.chat.otel' "$S"
```
A linha do `outfile` tem que mostrar `.bchive` e não `.henriworks`. O `captureContent: false` continua como está. Se algo der errado, o backup está em `settings.json.bak-hive`.

**4. Limpar o HenriWorks** (com o `limpar-henriworks.sh` em `~/Downloads`)
```bash
cd ~/Downloads
bash limpar-henriworks.sh --procurar /c/caminho/dos/seus/projetos           # prévia, não apaga nada
bash limpar-henriworks.sh --procurar /c/caminho/dos/seus/projetos --aplicar  # depois de conferir a prévia
```
Troque o caminho pela pasta onde ficam seus repositórios. Duas linhas da prévia não precisam de ação:
- **`confira ... github.copilot.chat.otel`:** essas chaves ficam, porque agora apontam para `.bchive`.
- **`confira ... COPILOT_OTEL_`:** se aparecer essa variável de ambiente, ela também fica.

Se a prévia mostrar algo marcado `NO GIT`, não apague à mão; me mande a linha.

**5. Conferir que voltou a gravar**

Abra o VS Code e faça uma pergunta qualquer no chat. Depois rode:
```bash
ls -la ~/.bchive/uso/otel/
ls ~/.henriworks 2>/dev/null || echo ".henriworks removida"
```
O arquivo tem que ter crescido.

**6. Manutenção até o plugin fazer isso sozinho** (com o VS Code fechado, uma vez por mês)
```bash
F="$HOME/.bchive/uso/otel/copilot-otel.jsonl"
LC_ALL=C grep -v '"scopeMetrics"' "$F" > "$F.tmp" && mv "$F.tmp" "$F" && ls -la "$F"
```

**O que esta limpeza não resolve.** Pelo próprio código do Hive, a telemetria do harness Copilot só existe se a TI ligar na configuração gerenciada. O caminho que o Hive espera é `~/copilot-telemetry/agent-host-otel.jsonl`, e ele não existe na sua máquina. Isso explica a falta de cobertura das sessões do fluxo: depois desta limpeza, o chat do VS Code fica coberto, mas o harness continua sem telemetria. É um pedido para a TI. Quando você quiser, eu escrevo o pedido com o que exatamente precisa ser ligado.

O passo 4 ficou confuso por culpa minha: deixei um caminho de exemplo e as duas linhas juntas. Não estragou nada. O script não achou nenhum repositório nesse caminho, que não existe, e na sua pasta de usuário respondeu "Nada a limpar". Nada foi apagado.

A tela mostra duas coisas importantes:

1. **O passo 3 deu certo.** A linha do `outfile` agora aponta para `.bchive`.
2. **O script não achou a pasta `~/.henriworks`.** Mas às 19:39 o arquivo de telemetria estava lá. Antes de continuar, preciso saber se você rodou o passo 2 e onde a telemetria está agora.

Rode este bloco e me mande a saída:

```bash
ls -la ~/.henriworks/uso/otel/ 2>&1 | head -5
ls -la ~/.bchive/uso/otel/ 2>&1 | head -5
echo "COPILOT_OTEL_FILE_EXPORTER_PATH = $COPILOT_OTEL_FILE_EXPORTER_PATH"
echo "COPILOT_OTEL_ENABLED = $COPILOT_OTEL_ENABLED"
```

As duas variáveis de ambiente que o script mostrou com "confira" são uma boa notícia. `COPILOT_OTEL_FILE_EXPORTER_PATH` e `COPILOT_OTEL_ENABLED` são as que o harness Copilot usa para gravar a própria telemetria. Elas podem ser a peça que faltava para cobrir as sessões do fluxo. **Não apague nenhuma das duas.** Preciso ver para onde a primeira aponta.

**Passo 4, explicado direito.** O script procura restos do HenriWorks dentro dos seus repositórios. O `--procurar` recebe a pasta onde ficam os repositórios. Como não sei qual é, use a sua pasta de usuário inteira:

```bash
cd ~/Downloads
bash limpar-henriworks.sh --procurar ~
```

Esse comando só mostra, não apaga nada. Pode levar um minuto.
- **"Nada a limpar":** acabou o passo 4.
- **Linhas com "apagaria":** me mande a saída e eu digo se pode rodar de novo com `--aplicar` no fim.

Nas duas situações, ignore as três linhas "confira": as chaves de telemetria do VS Code e as duas variáveis ficam.

Ao terminar, me mande a saída dos passos 2, 4 (a prévia) e 5.