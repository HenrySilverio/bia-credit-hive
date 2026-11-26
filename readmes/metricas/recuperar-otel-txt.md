O Hive mapeia bem o que acontece dentro do fluxo: agentes, turnos, ferramentas, tempo e negações do hook. Contagem de sessões e tokens ainda não dá para usar, e o export esconde isso: diz `temTokens: true` e mostra 13,7 mil tokens. Os eventos vão de 23 a 26/11 (1.833 linhas) e o consumo de 09 a 26/11 (966 chamadas).

**O que está sendo mapeado**

| Bloco | O que grava | Nos seus dados | Situação |
| --- | --- | --- | --- |
| Sessão | início, fim, novo ou retomado, dentro ou fora do fluxo, plugin | 37 inícios, 186 fins | ❌ 119 sessões só têm o evento de fim, sem início nem turno (98 em `repo-1`, sem branch). O export conta 163 sessões, mas as reais são cerca de 44 |
| Turno | principal, subagente, início de subagente, com pai → filho | 135 / 30 / 31 | ✅ a árvore do subagente funciona |
| Agente do fluxo | qual agente atendeu o turno | implement 21, review 11, bchive 11, discovery 8, plan 5, archive 1 | ⚠️ o mesmo subagente aparece com dois nomes (`bchive-gf` e `bia-credit-hive:bchive-gf`); falta juntar |
| Ferramenta | nome e categoria | 1.396 chamadas: leitura 637, terminal 210, comando do Hive 208, escrita 117, busca 111, mapa 63, subagente 33 | ⚠️ o MCP do GitHub cai em "outra" e "leitura", então `mcp` sai vazio; `rename_chat` conta como escrita |
| Comando e skill | só o nome | comandos `{}`, 6 skills | ❌ só pega comando digitado com `/`. Quem escolhe o agente na lista não aparece, por isso `planos = 0` em tudo, apesar dos 5 turnos do plan |
| Negação do hook | portão e motivo | 18 (17 de mapa antes de busca, 1 de pacote) | ✅ é o dado de guard-rail |
| Repositório, branch, ticket | contexto de cada evento | 6 repositórios, ticket sempre vazio | ⚠️ `porTicket` usa a branch quando não há ticket, então o "ticket" vira `branch-1` ou `main` |
| Tempo | total e ativo (5 min sem atividade encerram o ativo) | 880 min no total, 216 ativos (25 e 26/11) | ✅ |
| Versões da spec | replanejamentos por mudança | 1 | ⚠️ o nome da mudança sai legível; o anonimizador só troca o ticket |
| Selos | cadeia de hash por dia fechado | dia 25 selado | ✅ |

**Tokens (consumo)**

| Origem | Chamadas | Entrada | Período | Liga à sessão do Hive? |
| --- | --- | --- | --- | --- |
| Chat sem modelo identificado (`?`) | 142 | 10,4 mi | 11 a 23/11 | não |
| Chat com modelo (editAgent, subagentes) | 16 | 0,43 mi | 11 a 23/11 | não |
| `gpt-5.6-luna` (`copilotLanguageModelWrapper`) | 17 | 13,7 mil | 25 e 26/11 | não. É o único número de tokens que o export mostra |
| Autocomplete (NES) | 634 | 1,5 mi | 09 a 26/11 | é de fundo, fica fora |
| Outras de fundo (commit, títulos, settings, avaliação de risco) | 157 | 0,21 mi | variado | é de fundo, fica fora |

O que isso mostra:
1. **A telemetria está ligada.** O `importar` lê `~/copilot-telemetry/agent-host-otel.jsonl` e o arquivo de saída do VS Code. Então a F9 não está só esperando a TI: o bloqueio é de captura ou de formato.
2. **Nenhuma chamada do chat foi capturada de 24 a 26/11,** justamente os dias em que o Hive rodou. A chamada de 25 e 26/11 que tem modelo (`copilotLanguageModelWrapper` sem sufixo) parece de fundo: o filtro de fundo só reconhece esse nome com `/` no fim.
3. **Nenhuma chamada de consumo bate com um evento pelo `trace`** (0 de 1.648). Hoje não dá para atribuir tokens a sessão, fase ou agente.

**Vazamentos no pacote.** Não é problema para mim, mas tem que estar resolvido antes de qualquer envio real:
- O `saida-comandos-bash.md` foi colado com o prompt do terminal: tem sua matrícula, o nome do repositório e o nome da branch.
- O `versoesDaSpec` mostra o nome descritivo da mudança.

**Falta um arquivo.** O `otel-campos.txt` veio vazio por causa do erro `stdout is not a tty`: no Git Bash o `node` passa pelo winpty. Rode com `node.exe` apontando para o arquivo certo:

```bash
ls -la ~/copilot-telemetry/
node.exe -e '
const fs=require("fs");const ks=new Set();
const w=(o,p)=>{if(Array.isArray(o))o.slice(0,5).forEach(x=>w(x,p+"[]"));else if(o&&typeof o==="object"){if(typeof o.key==="string"){ks.add(p+"{"+o.key+"}");return}for(const k in o)w(o[k],p?p+"."+k:k)}else ks.add(p)};
for(const f of process.argv.slice(1))for(const l of fs.readFileSync(f,"utf8").split("\n").slice(-3000)){try{w(JSON.parse(l),"")}catch{}}
console.log([...ks].sort().join("\n"))' ~/copilot-telemetry/*.jsonl > ~/Downloads/otel-campos.txt
wc -l ~/Downloads/otel-campos.txt
```

O comando lê as últimas 3.000 linhas, que cobrem de 24 a 26/11. Com essa lista eu vejo se o chat do harness parou de gravar ou se mudou de formato.

**Para a sua decisão,** a F7 se divide em três correções:
- **Contagem:** sessão real, comando ou agente, MCP e ticket. Dá para corrigir já, sem depender de ninguém.
- **Tokens por sessão:** precisa da lista de campos do OTel.
- **Anonimizador:** nome da mudança e prompt colado.

Me diga qual pergunta a métrica tem que responder e para quem, e eu ordeno a partir daí.

```bash
ARQ="$HOME/.henriworks/uso/otel/copilot-otel.jsonl"
ls -la "$ARQ"
node.exe -e '
const fs=require("fs");const f=process.argv[1];const tam=fs.statSync(f).size;const N=Math.min(tam,64*1024*1024);
const b=Buffer.alloc(N);const fd=fs.openSync(f,"r");fs.readSync(fd,b,0,N,tam-N);fs.closeSync(fd);
const ls=b.toString("utf8").split("\n");if(N<tam)ls.shift();
const ks=new Map();const add=k=>ks.set(k,(ks.get(k)||0)+1);
const w=(o,p)=>{if(Array.isArray(o))o.forEach(x=>w(x,p+"[]"));else if(o&&typeof o==="object"){if(typeof o.key==="string"){add(p+"{"+o.key+"}");return}for(const k in o)w(o[k],p?p+"."+k:k)}else add(p)};
let ok=0;for(const l of ls.slice(-3000)){try{w(JSON.parse(l),"");ok++}catch{}}
console.error("arquivo "+(tam/1048576).toFixed(0)+" MB; linhas lidas "+ok);
console.log([...ks].sort().map(([k,n])=>n+"\t"+k).join("\n"))' "$(cygpath -w "$ARQ")" > ~/Downloads/otel-campos.txt
wc -l ~/Downloads/otel-campos.txt
```

```bash
ARQ="$HOME/.henriworks/uso/otel/copilot-otel.jsonl"
EV="$HOME/.bchive/uso/eventos/2026-11.jsonl"
node.exe -e '
const fs=require("fs"),rl=require("readline");
const [f,ev]=process.argv.slice(1);
const sids=new Set();for(const l of fs.readFileSync(ev,"utf8").split("\n")){try{sids.add(JSON.parse(l).sid)}catch{}}
const c={spans:0,logs:0,metricas:0,outras:0},ag={},mod={},op={},dia={},evn={},chaves=new Set(),sess=new Set();let bate=0,comTok=0;
const inc=(o,k)=>{o[k]=(o[k]||0)+1};
const r=rl.createInterface({input:fs.createReadStream(f),crlfDelay:Infinity});
r.on("line",l=>{
 if(l.includes("\"scopeMetrics\"")){c.metricas++;return}
 let o;try{o=JSON.parse(l)}catch{c.outras++;return}
 const a=o.attributes||{};
 for(const k of Object.keys(a)) if(/session|conversation|chat|request_id|turn|interaction|parent/i.test(k)) chaves.add(k);
 if(!o.traceId){ if(o.body!==undefined||a["event.name"]){c.logs++;inc(evn,a["event.name"]||"(sem)")} else c.outras++; return}
 c.spans++;
 inc(ag,a["gen_ai.agent.name"]||"(sem)");inc(mod,a["gen_ai.request.model"]||"(sem)");inc(op,a["gen_ai.operation.name"]||"(sem)");
 if(a["gen_ai.usage.input_tokens"]!=null)comTok++;
 const s=(o.resource&&((o.resource.attributes||{})["session.id"]||(o.resource._memoizedAttributes||{})["session.id"]));
 if(s){sess.add(s);if(sids.has(s))bate++}
 const t=Array.isArray(o.startTime)?o.startTime[0]:null;if(t)inc(dia,new Date(t*1000).toISOString().slice(0,10));
});
r.on("close",()=>console.log(JSON.stringify({linhas:c,spansComTokens:comTok,agentes:ag,modelos:mod,operacoes:op,eventosDeLog:evn,porDia:dia,chavesLigadasASessao:[...chaves],sessionIdsDistintos:sess.size,spansComSessionIdDoHive:bate},null,1)));
' "$(cygpath -w "$ARQ")" "$(cygpath -w "$EV")" > ~/Downloads/otel-diagnostico.json
cat ~/Downloads/otel-diagnostico.json
```

```bash
ARQ="$HOME/.henriworks/uso/otel/copilot-otel.jsonl"
EVD="$HOME/.bchive/uso/eventos"
node.exe -e '
const fs=require("fs"),rl=require("readline"),path=require("path");
const [f,dir]=process.argv.slice(1);
const sids=new Set();for(const n of fs.readdirSync(dir).filter(n=>n.endsWith(".jsonl")))for(const l of fs.readFileSync(path.join(dir,n),"utf8").split("\n")){try{sids.add(JSON.parse(l).sid)}catch{}}
const K=["gen_ai.conversation.id","copilot_chat.session_id","copilot_chat.chat_session_id","copilot_chat.parent_chat_session_id","github.copilot.interaction_id","session.id"];
const r={};for(const k of K)r[k]={presente:0,bateComHive:0};const serv={},porDiaServ={};
const inc=(o,k)=>{o[k]=(o[k]||0)+1};
const it=rl.createInterface({input:fs.createReadStream(f),crlfDelay:Infinity});
it.on("line",l=>{
 if(l.includes("\"scopeMetrics\""))return;
 let o;try{o=JSON.parse(l)}catch{return}
 const a=o.attributes||{},ra=(o.resource&&(o.resource.attributes||o.resource._memoizedAttributes))||{};
 for(const k of K){const v=a[k]!==undefined?a[k]:ra[k];if(v!==undefined&&v!==null&&v!==""){r[k].presente++;if(sids.has(String(v)))r[k].bateComHive++}}
 const s=ra["service.name"]||"(sem)";inc(serv,s);
 const t=Array.isArray(o.startTime)?o.startTime[0]:(Array.isArray(o.hrTime)?o.hrTime[0]:null);
 if(t){const d=new Date(t*1000).toISOString().slice(0,10);if(d>="2026-11-23")inc(porDiaServ,d+" "+s)}
});
it.on("close",()=>console.log(JSON.stringify({sessoesDoHive:sids.size,chaves:r,servicos:serv,servicoPorDiaDesde23:porDiaServ},null,1)));
' "$(cygpath -w "$ARQ")" "$(cygpath -w "$EVD")" > ~/Downloads/otel-ligacao.json
cat ~/Downloads/otel-ligacao.json
env | grep -i otel | sed 's/=.*/=(definida)/'
```