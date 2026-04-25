# Haunted House — A 2-D Grid-Based Survival Game in RISC-V Assembly

A survival game written entirely in RISC-V (RV32) assembly. The player navigates a dark house, collect a match, light the candle, and escape before the shadow monster drives your fear to 100. Features unlimited undo and competitive multiplayer with a sorted leaderboard.

---

## Overview

You're alone in a dark house during a power outage. Shadows are moving at the edges of your vision. Scattered around the house are a match and a candle — light the candle to banish the darkness and win. But every step you take, the shadow monster creeps one step closer. When it reaches you, your fear spikes. Let it hit 100, and it's game over.

The game is played on a configurable grid (default 8×8) rendered as ASCII art in the console. The player, match, candle, and shadow monster are all placed at pseudorandom non-overlapping positions at the start of each round.

---

## How to Play

### Controls

| Key | Action |
|---|---|
| `W` | Move up |
| `A` | Move left |
| `S` | Move down |
| `D` | Move right |
| `U` | Undo last move (unlimited) |
| `R` | Restart with the same map |
| `N` | Generate a new random map |
| `Q` | Quit the game |

Controls are case-insensitive.

### Symbols

| Symbol | Meaning |
|---|---|
| `#` | Wall (border) |
| `.` | Floor |
| `P` | Player (you) |
| `M` | Match (walk over to pick up) |
| `C` | Candle (walk over while holding a match to light it) |
| `X` | Shadow monster |

### Win Condition
Pick up the match (`M`) by stepping onto it, then step onto the candle (`C`) to light it. The candle can only be lit if you're holding the match.

### Lose Condition
Your fear gauge starts at 0. Each time the shadow monster is adjacent to you (Manhattan distance ≤ 1), your fear increases by 10 and the monster respawns at a new random location. When fear reaches 100, the game is over.

### Status Line
After every move, the console displays:
```
Status: Fear = 30, Match = 1, Candle = unlit
```

---

## Game Mechanics

### Shadow Monster AI
After each player move, the monster takes one step closer along the shortest path: it steps along the x-axis first if the x-coordinates differ, otherwise along the y-axis. The monster is clamped to the grid boundaries and cannot walk through walls.

### Match & Candle Interaction
Walking onto the match square automatically picks it up (the match disappears from the board). Walking onto the candle square while holding the match automatically lights it and triggers a win. If you don't have the match yet, stepping on the candle does nothing.

### Monster Respawn
When the monster triggers a fear increase (adjacency hit), it is respawned to a new random position that does not overlap with the player, the match (if not picked up), or the candle (if not lit).

---

## Enhancements

### 1. Unlimited Undo (Memory Enhancement)

Press `U` to undo your last move. You can undo an unlimited number of moves in succession, all the way back to the start of the current game. Each undo fully reverses the game state: the player moves back, the monster returns to its previous position, a picked-up match is placed back down, a lit candle is unlit, and the fear gauge is restored.

**Implementation:** A ring buffer (`history`) stores 128 state snapshots, each 12 bytes containing all entity positions, flags (`has_match`, `candle_lit`, `fearFactor`, `round_over`). A snapshot is pushed before every valid move and popped on undo. If a move is invalid (wall collision), the snapshot is discarded immediately to keep the history clean.

### 2. Competitive Multiplayer (Memory Enhancement)

At the start of the game, the program prompts for the number of players (1–8). Each player takes a turn playing the exact same map — the RNG seed is saved before the first round and restored before each subsequent player's turn, ensuring identical entity placement. After all players have finished (by either winning or hitting fear = 100), a leaderboard is displayed ranking players by fear gauge (lowest is best).

**Implementation:** The RNG state is snapshotted into `level_seed` before the first round. Player scores are stored in a `scores` array and sorted using selection sort over an `order_idx` indirection array to produce the final leaderboard.

---

## Running the Game

### Using RARS (RISC-V Assembler and Runtime Simulator)
1. Download and install [RARS](https://github.com/TheThirdOne/rars).
2. Open `Project.s` in RARS.
3. Click **Assemble** (or press F3).
4. Click **Run** (or press F5).
5. Interact via the console panel at the bottom — type your moves and press Enter.

### Using CPUlator
1. Navigate to [CPUlator (RISC-V)](https://cpulator.01xz.net/?sys=rv32-spim).
2. Paste the contents of `Project.s` into the editor.
3. Click **Compile and Load**, then **Continue** (or press F5).
4. Use the terminal panel for input/output.

### Grid Size
The grid dimensions are defined by the `gridsize` variable in the `.data` section (default: `8, 8`). To change the board size, modify the two bytes at the `gridsize` label before assembling. The game does not assume an 8×8 grid — any reasonable dimensions will work.

---

## Architecture

```
main
 ├── seed_from_time          # Seed RNG from SP/RA mix
 ├── save_rng_seed           # Snapshot RNG state for map replay
 ├── prompt_players          # Read player count (1-8)
 ├── run_round               # Loop over each player's turn
 │    ├── restore_rng_seed   # Reset RNG for identical map
 │    ├── init_game           # Place entities, reset flags/undo stack
 │    ├── draw_board_and_status  # Render grid + status line
 │    └── game_loop           # Input → move → monster → check → render
 │         ├── try_move_player        # Bounds check, commit, pickup/light
 │         ├── monster_step_towards_player  # Manhattan-step AI
 │         ├── check_shadow_monster_adjacency  # Fear check, respawn
 │         ├── snapshot_push / pop / discard   # Undo ring buffer
 │         └── draw_board_and_status
 └── print_leaderboard       # Selection-sort scores, print rankings
```

## Technologies

- **Language:** RISC-V Assembly (RV32I)
- **Simulators:** RARS, CPUlator
- **Concepts:** Low-level memory management, pseudorandom number generation, ring buffers, ASCII rendering, Manhattan distance AI, selection sort, syscall-based I/O

---

## Acknowledgements

Developed as part of **CSC258 — Computer Organization** at the University of Toronto, Fall 2025.
