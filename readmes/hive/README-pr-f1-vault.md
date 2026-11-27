A trava nova tem um furo real: ela é criada vazia e o conteúdo só é escrito depois. Nesse intervalo, outro processo lê "trava ilegível" e apaga. Além disso, a limpeza na saída apaga o arquivo sem conferir de quem ele é. Vou refazer a trava, primeiro reproduzindo o problema.

O achado High é real e está corrigido. O teste de fumaça passa em 67/67 no núcleo e no pacote da marca, sem termo interno. O `bia-credit-hive-revisao-copilot-5.zip` traz 3 arquivos: `okf.mjs`, `lib/okf.mjs` e `testes/smoke.mjs`. Aplicado sobre a PR, o resultado fica idêntico ao pacote novo.

**O que estava errado na trava** (reproduzi os dois primeiros no código da PR; os dois aparecem como defeito na PR e como OK no novo):
- **Nascia vazia:** a trava era criada e só depois recebia o conteúdo. Nesse intervalo, outro processo lia "trava ilegível", apagava e seguia, e os dois rodavam juntos.
- **A saída apagava sem conferir:** ao terminar, o processo apagava o arquivo de trava sem ver de quem era. Podia apagar a trava de outro processo.
- **Trava de processo morto era tomada:** dois processos podiam tomá-la ao mesmo tempo, e um apagava a trava recém-criada pelo outro. Esse eu não reproduzi; a correção abaixo elimina a tomada.

**Como ficou:**
- **Nasce completa:** o conteúdo (pid e um token único) é escrito num arquivo próprio e entra no lugar da trava de uma vez só, por link. Se a trava já existe, o link falha. Ninguém vê trava pela metade.
- **Nunca é tomada de outro processo:** viva, morta, vazia ou ilegível, a trava existente faz o comando parar. Se ficar uma trava de um processo que caiu (queda de energia, kill -9), a mensagem mostra o `rm "<caminho>"` para você apagar no Git Bash. Ctrl+C e erros normais limpam a trava sozinhos.
- **Na saída, só a própria trava sai:** o processo confere o token antes de apagar. Se não for a dele, devolve o arquivo para o lugar.

**Custo:** depois de uma queda sem saída normal, você roda um `rm` à mão. E a pasta precisa aceitar link de arquivo (NTFS, ext4 e APFS aceitam). Se não aceitar, o comando para e diz o motivo.

**Achado Medium (rodapés):** é a conversa da 4ª rodada, que não está marcada como "New". A correção já está na PR: cada rodapé tem que repetir o `resource` de mesmo id, e rodapé sem fonte ou sem definição é erro. Basta responder e resolver.

**Aplicar (Git Bash):**
```bash
cd ~/caminho/bia-credit-hive-plugin
git switch feat/vault-extractor
mkdir -p /tmp/r5 && unzip -o ~/Downloads/bia-credit-hive-revisao-copilot-5.zip -d /tmp/r5
cp -r /tmp/r5/. .
node testes/smoke.mjs          # esperado 67/67
git add -A && git commit -m "fix: never take a lock from another process" && git push
```

**Respostas para as conversas:**
- **Trava:** "Corrigido. A trava nasce completa (conteúdo com pid e token, posto no lugar por link, que falha se já existe). Nunca é removida de outro processo, esteja viva, morta, vazia ou ilegível: o comando para e mostra o `rm` para o dev. Na saída, só sai se o token for o do próprio processo."
- **Rodapés:** "Corrigido na rodada anterior: cada rodapé tem que repetir o `resource` da fonte de mesmo id, e rodapé sem fonte ou sem definição é erro."

A passagem ganhou o item da 5ª rodada no F1 e as contagens foram atualizadas. A fonte completa vai junto para guardar até a PR passar.