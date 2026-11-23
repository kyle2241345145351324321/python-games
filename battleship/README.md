Fabian Stachurski i Adam Gawryszczak


# Battleship LAN 海戰棋

Classic 10×10 Battleship for you and a friend on the **same local network** — or
against a three-level AI. Python 3.10+, Tkinter, standard library only.

![fleet](https://img.shields.io/badge/players-1--2-4a6fa5) ![deps](https://img.shields.io/badge/dependencies-none-4ec9a0)

## Features

- **LAN play** — host a room, share the 4-digit room code; the other player
  discovers it automatically over UDP broadcast on the same Wi-Fi/LAN. No
  server, no account, no internet.
- **Bilingual UI** — Traditional Chinese / English, switchable live and
  remembered between runs.
- **Fleet placement** — drag ships from the dock onto your grid (or click),
  rotate with <kbd>R</kbd>/right-click, randomize, clear, re-place before ready.
- **Real battleship rules** — 5/4/3/3/2 fleet, ships may touch, a **hit lets you
  fire again**, a miss passes the turn. First fleet sunk loses.
- **Chat + battle log** — talk while you play; every shot, hit and sinking is
  logged and localised.
- **AI practice** — easy / normal / hard (hard uses a probability-density
  solver). Playable offline in the same UI.
- **Learned AI + watch mode** — a neural-network opponent you train yourself
  (`ai_train.py`), and a viewer where you can watch AI vs AI or AI vs bot.
- **Rematch** — both players vote for a rematch; the starting player alternates.

## Quick start

```bash
cd games-python/battleship
python3 battleship.py            # GUI, main menu
```

1. **Host**: click *Host a room* → a 4-digit code and the host IP appear.
2. **Joiner** (same Wi-Fi): click *Join a room* → the room appears in the
   discovered list, or type the 4-digit code and press Join. If discovery is
   blocked by the network, expand *Manual* and enter the host IP + port.
3. Both players place their fleet and press **Ready** → battle starts.

Command-line shortcuts:

```bash
python3 battleship.py --host                    # open a room immediately
python3 battleship.py --host --port 5555        # fixed port (open it in the firewall)
python3 battleship.py --join 4821               # join by room code
python3 battleship.py --join 4821 --ip 192.168.1.5 --port 53122
python3 battleship.py --ai --difficulty hard
python3 battleship.py --ai --difficulty learned # needs the trained model
python3 battleship.py --watch                   # watch two computer players
python3 battleship.py --lang en
```

## Learned AI and watching AI

The **learned** AI is a small numpy neural network (505 inputs → 256 → 128 →
100 cell scores). It only sees public information and has no legality guard:
it must learn by itself not to shoot a used cell.

```bash
pip3 install numpy                 # only the learned AI needs it
python3 ai_train.py                # train (imitates the hard solver, then self-play)
python3 ai_train.py --test         # 100 real games vs easy / normal / hard
python3 ai_train.py --fresh        # start over from a random network
python3 ai_train.py --minutes 60   # stop after about an hour
```

Optionally train it to **place its own fleet** too (it samples a different
layout every game, rewarded when the opponent needs more shots to find it):

```bash
python3 ai_train.py --place        # ~400 iterations -> battleship_place_model.npz
```

This is **off by default**: the learned AI places its fleet at random like the
bots. To use the trained placement, set `USE_LEARNED_PLACEMENT = True` near the
top of `ai.py`.

The best model so far is saved as `battleship_model.npz` next to the code
(Ctrl-C is always safe). Then pick **Learned** under *Play vs AI*, or open
**Watch AI** on the main menu: choose any two computer players —
Learned vs Learned = AI vs AI, Learned vs Easy/Normal/Hard = AI vs bot — and
pause or change speed while they play.

## Rules

- 10×10 grid, columns **A–J**, rows **1–10**.
- Fleet: carrier 5, battleship 4, cruiser 3, submarine 3, destroyer 2 (17 cells).
- Ships may touch but not overlap.
- Fire at one cell: a **hit gives you another shot**, a miss hands the turn over.
- When a ship's last cell is hit it sinks (the opponent sees the whole hull).
- The first player whose entire fleet is sunk loses.

## Network details (for tinkerers)

- TCP: newline-delimited JSON messages, one object per line. The host's
  `GameServer` is authoritative — it owns the rules engine and broadcasts a full
  state snapshot to both clients after every action.
- Discovery: UDP port **54321**. Clients broadcast
  `{"t":"battleship.discover","ver":1}`; the host answers with the room code and
  TCP port. The game itself runs over TCP on an OS-assigned port.
- No data leaves your LAN. Nicknames and chat messages only go to the two
  players in the room.

## Troubleshooting

| Symptom | Fix |
|---|---|
| Room not found | Be on the same Wi-Fi/LAN. Use **Manual** on the join screen and type the host IP + port. Allow Python through the macOS firewall (System Settings → Network → Firewall), or open UDP 54321. |
| Join says *room full* | Battleship is two-player; the room already has two players. |
| *Connection lost* | The host closed the room or the Wi-Fi dropped. Restart hosting or reconnect. |
| *discovery unavailable* | UDP broadcast is blocked (guest Wi-Fi, VPN). Manual IP entry still works. |
| No window appears | Tkinter is missing: `brew install python-tk` (or use the python.org macOS build). |

Host IP and port are shown on the host screen — that is all the joiner needs in
manual mode.

## Development

```
battleship.py    entry point (CLI, self-test, AI-vs-AI simulation)
engine.py        pure rules engine — no UI, no sockets
i18n.py          Traditional Chinese / English strings
net_server.py    authoritative GameServer + UDP discovery responder
net_client.py    NetClient + LAN discovery probe
ai.py            BattleshipAI (easy / normal / hard / learned)
ai_net.py        learned AI: features, network, model loading (numpy)
ai_train.py      trainer for the learned AI (numpy)
ui.py            Tkinter app (menu, host/join, placement, battle, chat/log)
tests/           unittest suite for every module
```

Run the tests / a headless LAN game:

```bash
python3 -m unittest discover -s tests -t . -v
python3 battleship.py --selftest        # suite + loopback server/client game
python3 battleship.py --simulate 200    # AI vs AI win-rate table
```

Everything is standard library only (Python 3.10, Tkinter): no packages to
install, no build step.

## Screenshots

The screenshots in [`docs/screenshots/`](docs/screenshots) are captured from
the real Tkinter app:

| Menu | Deploy your fleet |
| --- | --- |
| ![Menu](docs/screenshots/menu.png) | ![Placement](docs/screenshots/placement.png) |

| Battle | Victory (Traditional Chinese) |
| --- | --- |
| ![Battle](docs/screenshots/battle.png) | ![Victory](docs/screenshots/victory-zh.png) |

