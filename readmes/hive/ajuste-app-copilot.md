Duas respostas fechadas e um achado novo.

**Prefixo: confirmado.** No app os comandos não têm prefixo, e a forma `/bia-credit-hive:start` não existe lá. No VS Code é o contrário. Então os 38 arquivos do plugin que mandam o dev digitar `/bia-credit-hive:plan`, `:triagem`, `:init`… dão ao usuário do app um comando que ele não acha. Isso vale também para a saída do `estado` e para o README. É um item próprio do checklist, separado do bug dos arquivos.

**"nenhuma sessao com /start": o app não entrega `/start` ao hook.** O hook roda no app: o teste da F9 de hoje ligou eventos do hook ao id da conversa. Mas o `comandoDe` só reconhece o prompt que começa com `/`. As causas prováveis:
- o app passa ao hook o prompt já expandido com o texto da skill, ou num campo com outro formato;
- a sessão do teste não gravou nada, por estar num repositório onde o hook falhou.

Isso também explica a F7: no app nenhum comando é contado, nem o `/start`.

**Para achar a sessão do teste sem depender do comando**, este resume as últimas 5 sessões com hora local, agente, skills, subagentes e contagem de ferramentas por categoria. Não mostra repositório, branch nem id:

```bash
node -e '
const fs=require("fs"),p=require("path"),os=require("os");
const d=process.env.BCHIVE_USO||p.join(os.homedir(),".bchive","uso");
const L=fs.readFileSync(p.join(d,"eventos","2026-11.jsonl"),"utf8").trim().split("\n").map(JSON.parse);
const S=new Map();for(const e of L){if(!S.has(e.sid))S.set(e.sid,[]);S.get(e.sid).push(e)}
const ult=[...S.values()].filter(v=>v.some(e=>e.ev!=="sessionEnd")).slice(-5);
const hora=t=>new Date(t).toLocaleString("pt-BR",{timeZone:"America/Sao_Paulo"});
const u=a=>[...new Set(a.filter(Boolean))].join(",")||"-";
ult.forEach((v,i)=>{const c={};v.forEach(e=>{if(e.cat){const k=(e.sub?"sub:":"")+e.cat;c[k]=(c[k]||0)+1}});
console.log(`#${i+1} ${hora(v[0].t)} a ${hora(v[v.length-1].t)}`);
console.log("  fluxo:",u(v.map(e=>e.fluxo===undefined?null:String(e.fluxo))),"| agente:",u(v.map(e=>e.agente)),"| comando:",u(v.map(e=>e.comando)));
console.log("  skills:",u(v.map(e=>e.skill)),"| subagentes chamados:",u(v.map(e=>e.chama)),"| subagente ativo:",u(v.map(e=>e.sub&&e.sub.tipo)));
console.log("  ferramentas:",JSON.stringify(c));});
'
```

Testei com dados de exemplo. Para ler a saída:
- **`subagentes chamados: …bchive-implementador`** com `sub:escrita` mostra que o `bchive` delegou a implementação (caminho 1).
- **`escrita`** sem `sub:` mostra que o próprio `bchive` editou (caminho 2).
- **Nenhuma sessão no horário do teste** mostra que o hook não gravou essa sessão. Aí o problema é outro: sem hook, não há guarda nenhuma no app.

Mande a linha da sessão do teste (você reconhece pelo horário) e os nomes dos arquivos gerados.

**O que já dá para dizer sobre a correção do prefixo (sem aplicar)**
- **Hook:** o `comandoDoPlugin` passa a aceitar também `/<nome>` quando `<nome>` é uma skill do próprio plugin (pasta `skills/` do `PLUGIN_ROOT`). O risco é contar o `/start` de outro plugin como do Hive. No app isso some, porque lá só aparece o do Hive. No VS Code o prefixo continua separando.
- **Mensagens ao dev:** a saída curta é escrever as duas formas ("`/plan` no app, `/bia-credit-hive:plan` no VS Code"). Funciona hoje, mas pesa em 38 arquivos. A alternativa é o `bchive.mjs` descobrir a ferramenta e imprimir só a forma certa. É mais limpa, mas depende de achar algo no ambiente que diferencie o app do VS Code. Isso dá para descobrir no canário: gravar os nomes (só os nomes) das variáveis de ambiente que o hook recebe em cada ferramenta.

Nada foi alterado ainda. Quando a origem dos arquivos estiver fechada, registro na passagem os dois itens: a trava do agente de entrada e o prefixo de comando no app.