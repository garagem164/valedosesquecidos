# 🎮 Vale dos Esquecidos — CONTEXTO COMPLETO

## Como usar este arquivo
Copie este arquivo INTEIRO junto com o `index.html` na próxima conversa.
Cole tudo no primeiro prompt. A IA entenderá 100% do projeto.

---

## VISÃO GERAL

**Nome:** Vale dos Esquecidos
**Tipo:** Jogo de fazenda/idle em HTML5 Canvas 2D
**Plataforma:** Navegador (desktop + mobile)
**Engine:** Vanilla JS (ES5), sem frameworks, sem imagens externas
**Idioma:** PT-BR (strings SEM acentos, para evitar encoding)
**Estado:** v15 funcional, single-player, salva em localStorage

---

## ESTADO TÉCNICO

### Mundo
- Tile: 32×32 px
- Mapa: 49 × 49 tiles = 2.401 tiles
- 49 áreas de 7×7 (49 tiles cada), grid 7×7 perfeito
- Área inicial: ID 24 (centro, col=3, row=3)
- Sobra: zero (mapa 100% coberto)
- Save key: `fazendinha_v15`

### Biomas (grid 7×7, linha por linha)
```
tree  tree  rock  rock  holy  holy  holy
tree  tall  rock  rock  holy  holy  holy
soil  tall  tall  soil  holy  holy  holy
soil  soil  tall  INIC  holy  holy  holy
soil  soil  tall  soil  tree  holy  holy
rock  rock  tall  soil  tree  rock  holy
rock  rock  soil  soil  tree  rock  holy
```
- INIC = área inicial (gi === 24)
- Custos: 0 (inicial) até 100.000 (lendárias)

### Enum de tiles
```js
T = {
  GRASS:0, TALL:1, WATER:2, TREE:3, SOIL:4, PLANTED:5,
  FENCE:6, PATH:7, HOUSE:8, ROCK:9, STUMP:10, BARN:11, EXPANSION:12
}
```

---

## SISTEMAS IMPLEMENTADOS

### Movimento e input
- WASD + setas + joystick virtual (touch)
- Aceleração suave (PACCEL=12)
- Colisão AABB (PW=18, PH=20)
- Animação 4 frames por direção (p.af, p.dir)

### Câmera e zoom
- Câmera com lerp (CAM_LERP=6)
- Zoom: 0.6 a 2.5, padrão 1.0
- Controles: scroll do mouse, pinch (touch), botões +/- no canto sup. esq., teclas + e -
- Zoom centrado no cursor: setZoom(newZoom, anchorX, anchorY)

### Cultivo
- GROW=15 segundos para crescer
- Solo arado expira em 300s (tillExpiry)
- Regador (watering) acelera crescimento
- Bônus de colheita dupla (crop_bonus)

### Recrutamento
- Trigger: andar em T.TALL com cooldown (encCd=3)
- Raridades: comum/incomum/raro/epico/lendario/mitico
- Minigame de barra oscilante + zona verde
- Chance base + bônus por precisão, limitado por raridade
- Contrato (state.contracts) previne fuga

### Moradores (IA)
- Estados: idle → walking → working → idle
- Tarefas por profissão (Agricultor, Pescador, Cuidador, Extrator, Alquimista, Comerciante, Mestre Geral)
- Fallback: agricultor sem sementes passa a colher
- Reservam tiles para não colidir entre si

### Loja (3 abas)
- Melhorias: 8 upgrades com nível crescente
- Itens: ferramentas, sementes, contratos, essência, machado real
- Vender: madeira, pedra, ovos, leite, cenouras

### Celeiro
- Lista moradores com raridade, tarefa atual, botão trabalho/parado
- Troca de tarefa em tempo real (com release de tiles)

### Fusão (Merge)
- Comum: 5x + 100 moedas → Incomum
- Incomum: 4x + 500 moedas → Raro
- Raro: 3x + 2.500 moedas → Épico
- Lendário e Mítico NÃO fundem

### Expansão de áreas
- Só compra ADJACENTES (cima/baixo/esq/dir) a uma já desbloqueada
- Áreas bloqueadas = tile T.EXPANSION (bloqueia passagem)
- Visual: fundo escuro opaco + padrão diagonal + painel central com:
  - 🔒 cadeado cinza = não adjacente
  - 🔓 cadeado dourado = adjacente + pode pagar
  - Custo em vermelho (sem $) ou dourado pulsante (pode pagar)
- Sem sinais de + nem cerca dentro
- Conteúdo 100% oculto até desbloquear

### Save/Load
- Auto-save a cada 5s + beforeunload
- Salva TUDO: coins, essence, recruits, plants, MAP INTEIRO, TIME, zoom, playerX/Y, tillExpiry, upgrades, áreas
- ATENÇÃO: salvar map e time é CRÍTICO (sem isso, bugs voltam)

---

## BUGS CORRIGIDOS (NÃO REGREDIR)

1. Sementes no HUD não atualizavam → data-seedq + update direto em updHUDText
2. Arado/plantado voltava a grama ao recarregar → save() inclui state.map
3. Plantas cresciam instantâneas → save() inclui state.time
4. Agricultor não colhia → fallback em findWorkTarget + doWork se sem sementes
5. Contador de sementes só atualizava ao trocar ferramenta → resolvido no item 1
6. Plantas órfãs após load → filtro no boot: só mantém se tile ainda é T.PLANTED

---

## ESTRUTURA DO CÓDIGO ATUAL

Arquivo único: index.html (autocontido, ~5000 linhas)

Ordem das seções no <script>:
1. Constantes (TILE, MAP_W, MAP_H, T, TOOLS, RECRUTAS, RECRUT, PROFISSOES, TAREFAS, AREAS, UPGRADES, NPCS)
2. Estado global (state) + BON
3. Helpers (showToast, addFloatText, formatNum, markHudDirty, reserveTile)
4. Render de personagens (drawMoradorPortrait, drawRecruitOnMap)
5. Mapa (genMap, tileAt, isBlocked, isWalkableTile, tryMove)
6. Ações do jogador (actOnTile, action, onMapClick, tryExpand)
7. Input (keyboard, joystick, zoom, pinch, click/touch no canvas)
8. Hotbar (renderHotbar, selSlot)
9. HUD (updHUDText com data-seedq)
10. Diálogos (openDialog, advanceDialog, talkToNPC)
11. Cutscenes (playCutscene, advanceCutscene)
12. Bônus (computeBonuses, renderBonus)
13. IA dos moradores (findWorkTarget, findWorkHome, doWork, updateRecruitsAI)
14. Recursos (runIdle, respawnResources, expireTilledSoil)
15. Update principal (update, updateTarget)
16. Render de tiles (drawTile)
17. Render de personagens (drawPlayer, drawNPC, drawAnimals)
18. Áreas bloqueadas (drawLockedArea, drawAreaBorder, isAreaAdjacentToUnlocked)
19. Draw principal (draw)
20. Recrutamento (rollRecruitRarity, triggerRecruit, startMiniGame, resolveRecruit)
21. Merge (openMerge, renderMerge, doMerge)
22. Modais (openModal, renderShopContent, renderBarnContent, renderExpandContent)
23. Save/Load (save, load)
24. Inicialização (resize, load, genMap, renderBonus, loop)

---

## PRÓXIMOS PASSOS

### CURTO PRAZO

1) MODULARIZAR em ES6
Estrutura planejada:
```
vale-dos-esquecidos/
├── index.html
├── css/style.css
├── js/
│   ├── main.js               (ponto de entrada + loop)
│   ├── config.js             (constantes)
│   ├── state.js              (estado global + BON)
│   ├── firebase.js           (init Firebase - futuro)
│   ├── data/                 (areas, recruits, upgrades, tools, tasks, npcs, balance)
│   ├── systems/              (map, player, expand, recruits, bonuses, resources)
│   ├── cloud/                (auth, cloud-save, friends, gifts, ranking, fair, visit - futuro)
│   ├── ui/                   (hud, shop, barn, merge, battle, dialog, cutscene, toast, login, friends-panel, ranking-panel, fair-panel)
│   ├── render/               (tiles, characters, effects, locked-areas)
│   ├── input/                (keyboard, touch, zoom)
│   └── utils/                (math, storage, hash)
├── functions/                (Cloud Functions - futuro)
├── firestore.rules           (futuro)
└── docs/                     (CONTEXTO.md, MULTIPLAYER.md, CHANGELOG.md)
```

2) HOSPEDAR no GitHub Pages
- Criar repo `vale-dos-esquecidos`
- Ativar Pages (branch main, root)
- URL: https://usuario.github.io/vale-dos-esquecidos/

3) BALANCEAMENTO CENTRALIZADO
Criar js/data/balance.js com TODOS os números num só lugar:
```js
export const BALANCE = {
  growSeconds: 15,
  tillExpirySeconds: 300,
  seedCost: 10,
  bonusPorProfissao: { ... },
  custosAreas: [ ... ]
};
```

### MÉDIO PRAZO - MULTIPLAYER LEVE (Firebase)

Objetivo: multiplayer assíncrono (tipo Hay Day/FarmVille), NÃO co-op real-time.

Etapas (8 semanas):
- Semana 1-2: Modularização + GitHub Pages
- Semana 3: Firebase Auth (anônimo + Google) + save na nuvem
- Semana 4: Sistema de amigos (adicionar por código)
- Semana 5: Visitar fazenda de amigos (read-only)
- Semana 6: Presentes diários (1 recurso/dia)
- Semana 7: Ranking semanal (top 100 por moedas/colheita/áreas)
- Semana 8: Feira coletiva (meta global + recompensa)

Filosofia:
- Cliente otimista (mostra ação na hora)
- Servidor autoritativo em ações críticas (compras, presentes)
- Não salvar mapa inteiro no Firestore — só diffs (tileChanges) + áreas + plantas
- Auto-save local a cada 5s, nuvem só em eventos-chave

Estrutura Firebase:
```
firestore/
├── balance/main               (config global, read-only pro cliente)
├── players/{userId}/
│   ├── profile                (nome, código amigo)
│   ├── progress               (coins, áreas, tileChanges, plantas)
│   ├── friends                (lista de userIds)
│   ├── giftsInbox             (presentes recebidos)
│   └── giftsSent              (controle diário)
├── rankings/
│   ├── weekly_coins
│   ├── weekly_crops
│   └── weekly_areas
└── fair/current               (meta coletiva da semana)
```

Regras Firestore:
- balance: read-only pro cliente
- players/{userId}: cada um só mexe no próprio
- rankings, fair: só Cloud Functions escrevem

Cloud Functions necessárias:
- sendGift (envia presente)
- updateScore (atualiza ranking)
- resetWeeklyRankings (cron toda segunda)
- contributeFair (contribui na feira)
- buyArea (compra área com validação)

---

## DECISÕES DE DESIGN (FIXAS)

- Sem imagens externas — tudo desenhado em Canvas 2D
- Sem frameworks — vanilla JS puro
- Mobile-first — joystick virtual e pinch habilitados
- Zero dependências — abre direto no navegador
- PT-BR sem acentos em strings de código
- Modais ocupam tela cheia em mobile
- Multiplayer leve — assíncrono, não co-op real-time
- Cliente otimista + servidor autoritativo quando tiver backend

---

## ESTILO VISUAL

- Pixel art programática (formas geométricas)
- Paleta: verdes para grama, marrons para solo, cinza para pedra
- Personagens: cabeça redonda, corpo retangular, chapéu (jogador)
- Raridades: cinza → verde → azul → roxo → dourado → magenta
- Áreas bloqueadas: escuras, opacas, com cadeado e painel de custo
- Animações sutis: vento em grama/árvores, bob do jogador, pulse em cadeado

---

## COMANDOS ÚTEIS

Reset de save (quando mudar estrutura):
```js
localStorage.removeItem('fazendinha_v13');
localStorage.removeItem('fazendinha_v14');
localStorage.removeItem('fazendinha_v15');
location.reload();
```

Desenvolvimento local:
- Instalar extensão Live Server no VSCode
- Botão direito em index.html → "Open with Live Server"
- Auto-refresh quando salvar arquivo

Deploy GitHub Pages:
```bash
git init
git add .
git commit -m "v15 - single-player funcional"
git remote add origin https://github.com/SEU_USUARIO/vale-dos-esquecidos.git
git push -u origin main
# Ativar Pages em Settings → Pages → Source: main / root
```

---

## RESUMO EXECUTIVO

| Item | Estado |
|------|--------|
| Versão atual | v15 |
| Formato | HTML único autocontido |
| Modo | Single-player |
| Save | localStorage (fazendinha_v15) |
| Mapa | 49x49, 49 áreas de 7x7 |
| Área inicial | Centro (gi=24) |
| Sistemas | Movimento, zoom, cultivo, recrutamento, IA dos moradores, loja, celeiro, merge, expansão, save/load |
| Bugs conhecidos | Nenhum (todos corrigidos) |
| Próximo passo | Modularizar em ES6 + hospedar no GitHub Pages |
| Futuro | Multiplayer leve com Firebase (login, save nuvem, amigos, presentes, ranking, feira) |

---

## COMO RETOMAR EM NOVA CONVERSA

Prompt sugerido:
```
Olá! Estou desenvolvendo um jogo chamado Vale dos Esquecidos —
fazenda/idle em HTML5 Canvas, vanilla JS, single-player, PT-BR sem acentos.

Anexei o index.html completo (versão v15, ~5000 linhas) e este CONTEXTO.md.

Estado atual: funcional, mapa 49x49, 49 áreas de 7x7, área inicial no centro,
sistema de expansão por adjacência com visual de cadeado, zoom, recrutamento
com minigame, IA de moradores, save/load completo.

Próximos passos planejados:
1. Modularizar em ES6
2. Hospedar no GitHub Pages
3. Adicionar multiplayer leve com Firebase

Por favor, leia o HTML anexo e o CONTEXTO.md e me ajude a continuar.
Começando por: [SUA TAREFA ESPECÍFICA AQUI].
```

---

*Última atualização: v15 — single-player funcional, pronto para modularização e multiplayer leve*
