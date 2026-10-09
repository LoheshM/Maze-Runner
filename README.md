# Maze Runner: a timed find-the-exit browser game

**Get the ghost home before the clock runs out.** A browser maze game with four hand-built maps, a fresh random map every time, a per-map leaderboard of the fastest players, and an animated solution if you run out of time. The front end is plain HTML, CSS and JavaScript with no framework. The back end is Python Flask with Flask-SQLAlchemy 2.5.1 and SQLite, which store players and their times.

> Group project, December 2022. Built by a team of four (see [Team](#team)).

## Team

- **ArunSDhamodhar**: [github.com/ArunSDhamodhar](https://github.com/ArunSDhamodhar)
- **LoheshM**: [github.com/LoheshM](https://github.com/LoheshM)
- **iavinash73**: [github.com/iavinash73](https://github.com/iavinash73)
- **Denis-Robert**: [github.com/Denis-Robert](https://github.com/Denis-Robert)

The original README credits all four as creators. iavinash73 pushed most of the commit history.

## What it does

| | |
|---|---|
| **Intro page** (`/home`) | Explains the game and its rules. You pick a username, which must be unique. |
| **Map picker** (`/maps`) | Shows Map A–D as image cards plus a **Random** option. Each map has a top-3 leaderboard by time used. Maps already played get a red border and their link is disabled. |
| **Game page** (`/level1`–`/level4`, `/random1`) | A 14 × 25 grid of fire walls. You move the ghost with the arrow keys. Walls block movement and play a bass "kick" sound. Stepping back onto your previous square erases that step from your trail. |
| **Time limit** | Map A and D give 40 s, Map B and C give 30 s, and Random gives 60 s. A progress bar counts down. |
| **Win** | Reaching the house posts your time to the server, which stores it in the leaderboard. |
| **Lose** | When time runs out, a ghost trail animates the path from start to exit. |
| **Random map** | A new 14 × 25 maze is generated on the server for every visit. |

## How it works

```
browser                                   Flask (auth blueprint)                      SQLite (maze_game.db)
───────                                   ──────────────────────                      ─────────────────────
intro.html ── POST uname ───────────────► /home ── new usersplayed row ─────────────► usersplayed(id, name)
maps.html  ◄─ top-3 per level ─────────── /maps ◄─ leaderboard query ─────────────── leaderboard(id, name,
   │                                                                                             level, timeused)
   ├─ Map A–D ─────────────────────────► /level1..4 ─► index.html (map name, time limit)
   └─ Random  ─────────────────────────► /random1 ─► generate 14×25 maze ─► index.html (maze as JSON)
index.js: draw grid ─ arrow keys ─ wall check ─ countdown
   ├─ on start:  $.post /usedpages {lvl}        (marks the map as played)
   ├─ on win:    $.post /data {time, lvl} ─────────────────────────────────────────► leaderboard row
   └─ on timeout: recursive path search from start → exit, animated every 250 ms
```

**Notable design decisions:**
- **Fixed maps live in the client.** Maps A–D are 0/1 grids hard-coded in `Js/index.js`. The server only tells the page which map to load and how much time to give.
- **Random maps come from the server.** `/random1` builds the maze in Python with a randomised wall-list method: start from a random cell and repeatedly pick a random wall from a list and open it if it joins a carved cell to an unvisited one without touching two carved cells. It then opens an entrance on the top row and an exit on the bottom row, and passes the grid to the page as JSON.
- **The solution is found by backtracking.** On timeout, `pathfind()` does a depth-first search that never steps straight back the way it came. It stops at the exit and replays the found path cell by cell.
- **One name per player.** `/home` rejects a username that is already taken, so each leaderboard name belongs to one registration.

## Repository layout

```
main.py              entry point: from website import create_app
__init__.py          create_app(): Flask config, SQLAlchemy (sqlite:///maze_game.db), Flask-Login, blueprint
auth.py              routes: /home, /maps, /level1-4, /random1 (maze generator), /data, /usedpages, /logout
models.py            usersplayed and leaderboard tables
HTML/                intro.html, maps.html, index.html (Jinja templates)
Js/index.js          game logic: maps A–D, movement, timer, win posting, path animation
Js/js.js             marks already-played maps on the map picker
CSS/                 style.css, style_intro.css, stylemap.css
media/               logos, map thumbnails, GIFs, fire wall tile, kick-bass.mp3
```

## Quick start

The code expects the usual Flask package layout: `main.py` next to a `website/` package that holds the Python modules, a `templates/` folder and a flat `static/` folder. The templates load assets from `../static/<file>`. The repo keeps these files in separate folders, so copy them into that shape first:

```bash
git clone https://github.com/LoheshM/Maze-Runner.git && cd Maze-Runner
mkdir -p ../maze-app/website/templates ../maze-app/website/static
cp main.py ../maze-app/
cp __init__.py auth.py models.py ../maze-app/website/
cp HTML/*.html ../maze-app/website/templates/
cp CSS/*.css Js/*.js media/* ../maze-app/website/static/
cd ../maze-app
```

The original README notes that the code **won't work with later versions of Flask**. It was written for Flask-SQLAlchemy 2.5.1. The authors didn't record their exact versions. These pins keep a compatible stack:

```bash
pip install "flask<2.3" "werkzeug<3" "flask-sqlalchemy==2.5.1" "sqlalchemy<2" flask-login colorama
python main.py                 # then open http://127.0.0.1:5000/home
```

The SQLite database `maze_game.db` is created on first run. The pages also load Bootstrap, jQuery, Google Fonts and a Font Awesome kit from CDNs, so the browser needs internet access.

## Limitations

- **The repo layout doesn't match the imports.** `main.py` imports a `website` package, so the app needs the restructuring step above.
- **Old dependencies.** It is pinned to Flask-SQLAlchemy 2.5.1 and doesn't run on current Flask.
- **Committed secret key.** The Flask `SECRET_KEY` is hard-coded in `__init__.py`. Replace it before deploying anywhere.
- **Server errors after saving.** `/data` and `/usedpages` redirect to `url_for('auth./maps')`, which isn't a valid endpoint, so those requests return a server error after the database write. The game carries on because they are background requests.
- **Shared progress list.** The list of maps already played is a single global list on the server, shared by every player. It is reset only when a new user registers.
- **Random-map times aren't shown.** They are stored with level 0, but the leaderboard only shows levels 1–4.
- **No passwords.** A player is identified only by a unique username.

## Licence

No licence has been chosen yet; all rights reserved by the authors.
