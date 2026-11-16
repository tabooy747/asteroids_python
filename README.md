# Asteroids

A classic Asteroids game written with Pygame.

## Features

- 1280x720 window drawn at 60 frames per second on a black background.
- A white triangle ship starts in the center. It rotates in place and thrusts forward or backward along its heading.
- Shots travel in the direction the ship is facing. Another shot cannot be fired until 0.3 seconds have passed.
- Asteroids spawn from outside a random screen edge every 0.8 seconds, in three sizes (radii 20, 40, and 60).
- A shot that hits an asteroid is removed. Asteroids larger than the minimum size split into two smaller, faster pieces; the smallest size is destroyed.
- Collision uses circle distance. If an asteroid hits the ship, the game prints `Game over!` and exits.

## Controls

| Key | Action |
| --- | --- |
| W | Thrust forward |
| S | Thrust backward |
| A | Rotate left |
| D | Rotate right |
| Space | Shoot |

Close the window to quit. The window must be focused for the keyboard input to register.

## Requirements

- Python 3.13 or newer (`requires-python` in `pyproject.toml`; `.python-version` is 3.13)
- pygame 2.6.1 (pinned in `pyproject.toml` and `uv.lock`)

## Install and run

This repository includes a `uv.lock` file. With [uv](https://docs.astral.sh/uv/) installed:

```bash
uv sync
uv run python main.py
```

Alternatively, in a Python 3.13 virtual environment:

```bash
pip install pygame==2.6.1
python main.py
```

## Project structure

| File | Role |
| --- | --- |
| `main.py` | Initializes Pygame, runs the frame loop, draws sprites, and checks collisions |
| `player.py` | Ship shape, rotation, thrust, and shooting |
| `asteroid.py` | Asteroid motion and splitting |
| `asteroidfield.py` | Spawns asteroids along the four screen edges |
| `shot.py` | Projectile motion |
| `circleshape.py` | Shared position, velocity, radius, and circle collision |
| `constants.py` | Screen size, speeds, radii, cooldown, and spawn timing |
| `logger.py` | Writes JSONL state snapshots and gameplay events |
| `pyproject.toml` | Project metadata and the pygame dependency |
| `uv.lock` | Locked pygame version and wheels |
| `.python-version` | Pins the Python version uv uses (3.13) |
| `.gitignore` | Ignores virtual environments, build output, and JSONL logs |

## JSONL logging

`logger.py` records a session as JSON Lines in the working directory. The first write of a process replaces the file; later writes append. Both files are listed in `.gitignore`.

`game_state.jsonl` is a periodic snapshot of the sprite groups in `main`. `log_state()` is called every frame, records a line once per second (every 60 frames), and stops after 16 seconds. Each line includes:

- `timestamp` — local time as `HH:MM:SS.mmm`
- `elapsed_s` — whole seconds since the process started logging
- `frame` — frame counter
- `screen_size` — width and height of the display
- `updatable`, `drawable`, `asteroids`, and `shots` — each group's sprite `count` and up to 10 sprites

A sprite entry includes `type` and, when the object has them, `pos`, `vel`, `rad`, and `rot`.

`game_events.jsonl` records discrete events for the whole session, including after state snapshots stop. Each line has `timestamp`, `elapsed_s`, `frame`, and `type`:

- `asteroid_shot` — a shot collided with an asteroid
- `asteroid_split` — that asteroid was large enough to break into two
- `player_hit` — an asteroid collided with the ship

The state logger reads `main`'s local variables through the call stack, so `log_state()` is called from the frame loop.
