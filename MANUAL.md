# randol_carmeet — Manual

Réplica do LS Car Meet do GTA Online: recria a cena do encontro (veículos, peds, Mimi e o organizador de corridas) dentro do IPL, sem depender de framework.

---

## Sumário

1. [Dependências](#dependências)
2. [Instalação](#instalação)
3. [Configuração](#configuração)
4. [Cena e performance](#cena-e-performance)
5. [Teleporte](#teleporte)
6. [Configs alternativas (MLOs)](#configs-alternativas-mlos)
7. [Estrutura de arquivos](#estrutura-de-arquivos)

---

## Dependências

| Recurso | Obrigatório | Observação |
|---|---|---|
| `ox_lib` | Sim | `lib.points`, `lib.zones`, Text UI, `requestModel`, `requestAnimDict` |
| `bob74_ipl` | Sim | Carrega o IPL padrão do car meet. Sem ele, o interior não existe |
| Framework | Não | O recurso é 100% client-side e não usa framework nenhum |

---

## Instalação

1. Copie a pasta `randol_carmeet` para `resources/`.
2. Adicione ao `server.cfg`:
   ```
   ensure randol_carmeet
   ```
3. Garanta que o `bob74_ipl` está rodando — o recurso assume o IPL padrão do car meet.
4. Se for usar um MLO em vez do IPL padrão, substitua o `config.lua` da raiz por uma das configs alternativas (ver seção correspondente).

---

## Configuração

Arquivo: `config.lua` (carregado como script de cliente).

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `EnableTeleport` | bool | Sim | Liga as zonas de entrada/saída do interior. `false` se o acesso for feito por outro recurso (MLO com porta própria, por exemplo) |
| `Teleports` | array | Sim | Zonas de teleporte. Cada entrada tem `label` (texto da Text UI), `zone` (vec4 da caixa 6x6x2 de ativação) e `set` (vec4 de destino) |
| `CenterZone` | vec3 | Sim | Centro da cena. Toda a cena é criada quando o jogador chega a 80 m e destruída quando sai |
| `Prize` | tabela | Sim | Veículo-prêmio em cima do reboque. `slamtruck` e `zr350`, cada um com `model` e `coords` (vec4). O ZR350 é anexado ao Slamtruck |
| `Mimi` | tabela | Sim | Ped da Mimi: `models` (ped + prop do celular), `coords` (vec4) e `rot` (vec3 da synchronized scene) |
| `RaceOrg` | tabela | Sim | Organizador de corridas: `model`, `coords` (vec4) e `rot` (vec3 da synchronized scene) |
| `Vehicles` | array | Sim | Carros da cena (52 no config padrão). Ver campos abaixo |
| `Peds` | array | Sim | Peds ambientes (45 no config padrão). Ver campos abaixo |

### Campos de `Vehicles`

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `model` | hash | Sim | Modelo do veículo |
| `coords` | vec4 | Sim | Posição e heading |
| `colors` | `{primária, secundária}` | Sim | Cores principais |
| `extraColors` | `{perolizado, roda}` | Sim | Cores extras |
| `intColor` | number | Sim | Cor do interior |
| `livery` | number | Não | Livery (aplicada como mod 48 e via `SetVehicleLivery`) |
| `modKits` | tabela `[modType] = id` | Sim | Modificações. Os ids são gravados **+1** no config (o código aplica `id - 1`) |
| `neons` | `{r, g, b}` | Não | Liga os 4 neons com essa cor |
| `door` | number | Não | Deixa essa porta aberta |

Todos os carros ficam congelados, invencíveis, trancados, com vidro fumê, sem sujeira e com a placa `RANDOLIO`.

### Campos de `Peds`

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `model` | hash | Sim | Modelo do ped |
| `coords` | vec4 | Sim | Posição e heading (o ped é criado 1.0 abaixo do Z informado) |
| `dict` / `anim` / `flag` | string / string / number | Não | Animação em loop. Alternativa ao `scenario` |
| `scenario` | string | Não | Scenario do GTA V (`WORLD_HUMAN_*`). Alternativa à animação |
| `animType` | `'phone'`, `'smoke'` ou `'drink'` | Não | Cria e anexa o prop correspondente na mão do ped |
| `default` | bool | Não | `true` usa a roupa padrão do modelo; caso contrário, componentes e props são sorteados a cada spawn |

Por isso, a aparência dos peds sem `default` varia entre jogadores; já os carros são idênticos para todo mundo.

---

## Cena e performance

Todas as entidades são **client-side e não networked**. Elas são criadas de uma vez quando o jogador entra no raio de 80 m de `CenterZone` e deletadas ao sair (e no `onResourceStop`).

A ordem de criação é encadeada: organizador de corridas → Mimi → veículos → peds → veículo-prêmio. Mimi e o organizador usam synchronized scenes em loop (`ANIM@SCRIPTED@CARMEET@TUN_MEET_IG1_MIMI@` e `ANIM@SCRIPTED@CARMEET@TUN_MEET_IG2_RACE@`), com o celular da Mimi animado junto.

Na primeira entrada da sessão do FiveM, a cena pode demorar a aparecer porque os modelos ainda estão sendo carregados; nas entradas seguintes é praticamente instantâneo.

---

## Teleporte

Com `EnableTeleport = true`, cada entrada de `Teleports` cria uma caixa de 6x6x2 que mostra a Text UI e aceita `E`. O teleporte faz fade-out, força o carregamento da cena e da colisão no destino, corrige o Z pelo chão e faz fade-in.

O jogador **mantém o veículo** ao teleportar (`SetPedCoordsKeepVehicle`), e o heading do veículo é ajustado se ele estiver no banco do motorista.

Os dois teleportes padrão são a entrada (rampa em Los Santos) e a saída (dentro do interior).

---

## Configs alternativas (MLOs)

As pastas `GABZ CONFIG/` e `KIIYA CONFIG/` trazem `config.lua` com todas as coordenadas ajustadas para MLOs de terceiros. Para usar, **substitua o `config.lua` da raiz** pelo arquivo escolhido.

| Pasta | MLO | Observação |
|---|---|---|
| `GABZ CONFIG/` | Gabz Underground Car Meet | Coordenadas fornecidas por Arkose, não testadas pelo autor. `EnableTeleport = false` |
| `KIIYA CONFIG/` | Car Meet Parking Lot Interior (MLO gratuito) | Coordenadas fornecidas por `.delok`. `EnableTeleport = false`. Sem a cena de áudio do IPL padrão |

Ambas assumem que o MLO cuida do acesso ao interior, por isso desligam o teleporte embutido.

---

## Estrutura de arquivos

```
randol_carmeet/
├── cl_carmeet.lua        — teleporte, ponto de 80 m, criação/limpeza da cena, synchronized scenes
├── config.lua            — IPL padrão: teleportes, prêmio, Mimi, organizador, 52 carros e 45 peds
├── GABZ CONFIG/
│   └── config.lua        — mesma estrutura, coordenadas do MLO Gabz
├── KIIYA CONFIG/
│   └── config.lua        — mesma estrutura, coordenadas do MLO gratuito de car meet
└── fxmanifest.lua
```
