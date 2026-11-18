# MisterWeapon modpack workspace

Space Engineers 1 weapon mods. This folder is the base git repo `MisterWeaponModpack` (docs only); each mod
folder is its own repo with its own `CLAUDE.md`, pinned here as a **git submodule**. All seven are public on GitHub
and shared with a collaborator, so nothing may depend on one machine: no absolute paths, no Windows-only steps
without a Linux equivalent (see [deployment.md](missil-misterplace/docs/deployment.md#machine-setup)).
- Commit and push inside the mod repo first; then `git add <mod folder>` and commit here to move the pinned version.
- Clone: `git clone --recurse-submodules <url>`; update: `git pull --recurse-submodules`.

## Collaborating (two people, two machines)
- **Start of every session:** `git pull --recurse-submodules` here, then make sure each mod you will touch is on
  `main`, not a detached commit (`git -C <mod> branch --show-current` prints nothing when detached; fix with
  `git -C <mod> switch main` then `git -C <mod> pull`). Never commit on a detached HEAD: that work is easy to lose.
- **End of a change:** commit and push in the mod, then bump the pointer here (`git add <mod>`, commit, push). Pull
  before pushing; if a push is rejected, pull and rebase, never force-push.
- `docs/status.md` is edited by both people: pull before editing it, keep edits small, and push soon after.
- Only the Workshop owner (`steamIdOwner` in `deploy.json`) publishes and runs `deploy.ps1 final`. Collaborators use `dev`.
- **First-time setup on a new machine** (once per person; also in [README.md](README.md#collaborator-setup)):
  Git + Git LFS (`git lfs install`), clone as above, `deploy.ps1 dev` in each mod (brain first), and the `se-dev-*`
  skills from `CometWorks/skills` (`npx skills add CometWorks/skills`, or clone it and run `install/claude.bat` /
  `install/claude.sh`). They prepare themselves on first use (needs Space Engineers installed); set
  `SE_DEV_GRAPHIFY=0` to skip Graphify, as on the owner's machine. If the skills are missing, say so to the user
  instead of guessing Mod API members.
- Claude's memory is per machine and not shared: anything both people's Claude must know goes in these `CLAUDE.md`
  files or `docs/`, not only in memory.

## Docs (start with the status)
- [docs/status.md](docs/status.md): **current state, what is tested, open items, gotchas. Read first when resuming**
- [docs/modpack-plan.md](docs/modpack-plan.md): structure, Workshop IDs, modernization phases
- [docs/testing.md](docs/testing.md): which mods to load and how to test in the game

## Mods
| Folder | What | Docs |
|---|---|---|
| `missil-misterplace/` | Brain: guided-missile logic (git repo) | [CLAUDE.md](missil-misterplace/CLAUDE.md) |
| `missilSLBM-misterplace/` | SLBM blocks (depends on the brain; git repo) | [CLAUDE.md](missilSLBM-misterplace/CLAUDE.md) |
| `IronDome/IronDomeClassic/` | Original Iron Dome turret + launcher, data only; requires the brain (git repo) | [CLAUDE.md](IronDome/IronDomeClassic/CLAUDE.md) |
| `LauncherX/` | LauncherX block + visuals script (git repo) | [CLAUDE.md](LauncherX/CLAUDE.md) |
| `IronDome/IronDomeX/` | IronDomeX block + visuals script (git repo) | [CLAUDE.md](IronDome/IronDomeX/CLAUDE.md) |
| `IronDome/AntennaIronDome/` | Antenna block with rotating radar dish, standalone (git repo) | [CLAUDE.md](IronDome/AntennaIronDome/CLAUDE.md) |

## Conventions (apply to every mod repo)
- Shipped content lives in `<repo>/Mod/`. Deploy with `deploy.ps1 dev|final`, configured by `deploy.json`.
  Details: [missil-misterplace/docs/deployment.md](missil-misterplace/docs/deployment.md).
- Dev and final versions of a mod must never be loaded in the same world.
- After changing a mod (anything under its `Mod/` or its `deploy.json`), always run `.\deploy.ps1 dev` in every repo
  touched, without waiting to be asked. Never run `final` unless the user asks for a release.
- Never delete folders in `<SE data>/Mods` (Windows: `%APPDATA%\SpaceEngineers\Mods`); deploys move replaced folders to `ModBackups\`.
- Use the `se-dev-mod` and `se-dev-game-code` skills for Mod API questions.
- The user works only through Claude Code (no IDE). Edit docs with the Edit tool or Python: PowerShell 5.1
  `Get-Content`/`Set-Content` garbles UTF-8 characters.
- Needs the Water Mod and a water planet to test anything aquatic (SLBM, LauncherX); see [docs/testing.md](docs/testing.md).
- Document every change to the workflow or layout in the docs above, and update [docs/status.md](docs/status.md) when
  the state, test results or open items change.
