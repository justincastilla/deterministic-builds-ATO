# One Environment, N Machines: Deterministic Builds for Humans and Agents

Demo repository and slide deck for a 20 minute talk at
[All Things Open 2026](https://www.allthingsopen.org/) (Raleigh, October 19 to 20).
An earlier 5 minute version was given at WeAreDevelopers World Congress NA 2026.

**Speaker:** Justin Castilla ([@justincastilla](https://github.com/justincastilla)), Flox

## What is in here

```
.flox/env/manifest.toml   one file: runtime, database, agent harness, services, hermetic build
.flox/env/manifest.lock   the receipt: exact versions and store paths for 3 systems
app.py                    tiny HTTP service that reports exactly what it was built from
scripts/sbom.py           SPDX 2.3 SBOM derived from manifest.lock (not a scan)
slides/index.html         the deck, self contained, no network needed on stage
.github/workflows/ci.yml  "machine 2 of 3": an x86_64-linux runner activating the same lock
```

## Run the demo

Install Flox from [flox.dev/download](https://flox.dev/download) (or `brew install flox`), then:

```sh
git clone https://github.com/justincastilla/deterministic-builds-ATO && cd deterministic-builds-ATO

flox activate -s                       # enter the env and start redis + app
python3 --version                      # Python 3.14.7, on every machine
flox services status
curl -s localhost:8080                 # {"python": "3.14.7", "redis": "PONG", ...}

flox build                             # hermetic (sandbox = "pure") build
./result-app/bin/app --version

flox containerize --runtime docker     # OCI image, no Dockerfile
python3 scripts/sbom.py | jq '.packages[] | {name, versionInfo, licenseConcluded}'
```

Same lock, three systems:

```sh
shasum -a 256 .flox/env/manifest.lock  # 7c18d091b247ea6f…  identical on darwin and linux
jq -r '[.packages[].locked_url] | unique[]' .flox/env/manifest.lock
```

## The deck

Open `slides/index.html` in a browser (or the
[GitHub Pages copy](https://justincastilla.github.io/deterministic-builds-ATO/slides/)).
Terminal slides type each command when you press `→`, then reveal the output.

| Key | Action |
|-----|--------|
| `→` `space` | run next command, or go to next slide |
| `←` | previous slide (terminals shown completed) |
| `n` | speaker notes with per slide time targets |
| `t` | start or reset the talk clock (turns red when a slide runs a minute late) |
| `f` | fullscreen |
| `?` | help |

Every terminal output in the deck was captured from a real run of this
repository on 2026-10-16 with flox 1.18.0. Three things are emulated and
marked as such in the speaker notes:

- The `ci-runner` and `agent-sandbox` panes on the three machine slide. Their
  versions and store paths are the real ones from `manifest.lock`; the machines
  were not on my desk.
- `flox activate -r justin-flox/deterministic-builds-ato`. Run `flox push` from
  this repo before the talk and that step works live.
- The `claude -p ...` transcript. Every command in it is real and `nodejs@22.21.1`
  is what the catalog resolves today.

## Talk outline (20 min)

| Time | Section |
|------|---------|
| 0:00 | Title |
| 1:00 | The problem: where is your software being built? |
| 3:00 | What setup looks like today: four machines, four install stories |
| 4:30 | Two problems: same build everywhere, same run wherever it lands next |
| 6:00 | The fix: declare it, don't script it (manifest, lock, closure) |
| 8:00 | Demo 1: one file |
| 12:00 | Demo 2: three machines, one hash |
| 15:00 | Demo 3: the artifact and the receipt (build, containerize, SBOM) |
| 18:00 | Humans and agents share one definition |
| 19:30 | Takeaway and links |
