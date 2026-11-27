Testar em outro repositório não atrapalha. O hook e a telemetria gravam por usuário, em `~/.bchive/uso/`, e não por repositório; o repositório entra só como um rótulo de cada evento.

**O que a saída mostra**
- **Limpeza feita:** 43 itens removidos em dois repositórios. Não sobrou HenriWorks na máquina.
- **A telemetria passou a sair pela variável de ambiente.** Às 21:10 nasceu o `copilot-harness-otel.jsonl` (233 KB). O `copilot-otel.jsonl`, que vem do `settings.json`, parou às 20:24. Ou seja: quando a variável existe, ela vale mais que a configuração do VS Code.
- **Corrige o que eu disse antes.** Antes da mudança, a variável e o `settings.json` apontavam para o mesmo arquivo. Então a falta de cobertura de 24 a 26/11 não vinha de um arquivo separado do harness.
- **O que falta saber:** se a sessão do teste das 21h aparece na telemetria ligada ao id da sessão do hook, com tokens e modelo. Se aparecer, o problema de cobertura está resolvido.

**A conferência final** (só contagens, nomes de modelo e agente, nenhum id):
```bash
node.exe -e '
const fs=require("fs");const [h,ev]=process.argv.slice(1);
const desde="2026-11-26T23:50";
const sids=new Set();let evs=0;for(const l of fs.readFileSync(ev,"utf8").split("\n")){try{const o=JSON.parse(l);if(o.t>=desde){evs++;sids.add(o.sid);if(o.sub&&o.sub.id)sids.add(o.sub.id)}}catch{}}
const r={linhas:0,ligadas:0,chatComTokens:0,chatLigadoComTokens:0,modelos:{},agentes:{},operacoes:{}};const inc=(o,k)=>{o[k]=(o[k]||0)+1};
for(const l of fs.readFileSync(h,"utf8").split("\n")){let o;try{o=JSON.parse(l)}catch{continue}r.linhas++;const a=o.attributes||{};
 const c=a["gen_ai.conversation.id"]||a["copilot_chat.session_id"]||a["copilot_chat.chat_session_id"];const lig=!!c&&sids.has(String(c));if(lig)r.ligadas++;
 inc(r.operacoes,a["gen_ai.operation.name"]||a["event.name"]||(o.scopeMetrics?"(metrica)":"(sem)"));
 if(o.traceId&&a["gen_ai.operation.name"]==="chat"&&a["gen_ai.usage.input_tokens"]!=null){r.chatComTokens++;inc(r.modelos,a["gen_ai.request.model"]||"(sem)");inc(r.agentes,a["gen_ai.agent.name"]||"(sem)");if(lig)r.chatLigadoComTokens++}}
console.log(JSON.stringify({eventosDoHook:evs,sessoesDoHive:sids.size,...r},null,1));
' "$(cygpath -w ~/.bchive/uso/otel/copilot-harness-otel.jsonl)" "$(cygpath -w ~/.bchive/uso/eventos/2026-11.jsonl)"
```
O comando considera só os eventos do hook a partir das 20:50 de hoje (horário de Brasília), que cobrem o seu teste. Também conta as sessões de subagente.

Como vou ler:

| Resultado | Significa |
| --- | --- |
| `chatLigadoComTokens` perto de `chatComTokens` | Cobertura resolvida: cada chamada do teste liga à sessão do Hive, com tokens e modelo |
| `eventosDoHook` maior que zero e `ligadas` igual a 0 | O hook gravou, mas o id da conversa não bate. Aí a ligação precisa de outro campo |
| `eventosDoHook` igual a 0 | O hook não gravou o teste. É problema de instalação no repositório novo, não de telemetria |

Fica uma decisão para quando formos mexer no plugin: hoje há dois caminhos de telemetria (a variável e o `settings.json`), e só a variável vale. O certo é deixar um caminho só, e o `init` do Hive avisar quando eles divergem. Anoto isso na passagem junto com o resultado deste teste.