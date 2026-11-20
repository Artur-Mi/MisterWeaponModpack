# MisterWeapon modpack

Space Engineers 1 weapon mods: guided missiles (the "brain"), SLBM, LauncherX, the Iron Dome family and WaterImpact.
This repo holds the shared docs and pins each mod repo, as a git submodule, at a version known to work with the others.

| Folder | What |
|---|---|
| `missil-misterplace/` | Brain: guided-missile logic (scripts only) |
| `missilSLBM-misterplace/` | SLBM launcher and missiles (needs the brain) |
| `LauncherX/` | LauncherX block (needs the brain) |
| `IronDome/IronDomeClassic/` | Original Iron Dome turret and launcher (needs the brain) |
| `IronDome/IronDomeX/` | IronDomeX block (needs the brain) |
| `IronDome/AntennaIronDome/` | Antenna with a rotating radar dish (standalone) |
| `WaterImpact/` | Missiles and shells hitting water (standalone, needs the Water Mod) |

## Get it

Needs [Git](https://git-scm.com) and [Git LFS](https://git-lfs.com) (models and textures are in LFS).

```bash
git lfs install
git clone --recurse-submodules https://github.com/Artur-Mi/MisterWeaponModpack.git
```

Later updates: `git pull --recurse-submodules`.

## Collaborator setup

Once per machine:

1. Install Git and Git LFS, run `git lfs install`, and clone as above.
2. Put the mods on a branch: `git submodule foreach git switch main` (a fresh clone leaves them on a detached commit).
3. Deploy each mod you want with `deploy.ps1 dev` (see below), the brain first.
4. If you use Claude Code: install the Space Engineers dev skills from
   [CometWorks/skills](https://github.com/CometWorks/skills) (`npx skills add CometWorks/skills`, or clone it and run
   `install/claude.bat` on Windows / `install/claude.sh` on Linux). They prepare themselves on first use and need the
   game installed. Optional: set `SE_DEV_GRAPHIFY=0` to skip the Graphify index. Claude reads the rules in `CLAUDE.md`;
   its memory is per machine and not shared, so shared rules belong in `CLAUDE.md` or `docs/`.

Every session: `git pull --recurse-submodules` first, and check the mod you will edit is on `main`
(`git -C <mod> switch main`, then `git -C <mod> pull`).

## Load the mods in the game

In each mod folder you want, run its deploy script once. It links the game's `Mods` folder to the repo, so edits are live:

```powershell
.\deploy.ps1 dev                 # Windows PowerShell
pwsh ./deploy.ps1 dev            # Linux/macOS (PowerShell 7), game under Steam Proton
```

Deploy the brain (`missil-misterplace`) too, and enable it in the world. Never load the dev and the Workshop (final)
versions of a mod in the same world. If the script can't find the game's data folder, set `SE_DATA_DIR` to the folder
that holds `Mods`. Details: [missil-misterplace/docs/deployment.md](missil-misterplace/docs/deployment.md#machine-setup).

## Working together

- After cloning, the mod folders are on a detached commit. Before changing one, run `git switch main` inside it
  (or once for all: `git submodule foreach git switch main`).
- Commit and push inside the mod you changed (each mod is its own repo).
- To share a combination that works, commit the moved submodule pointer here: `git add <mod folder>` and commit.
- Docs: [docs/status.md](docs/status.md) (state and open items), [docs/testing.md](docs/testing.md),
  [docs/modpack-plan.md](docs/modpack-plan.md). Each mod has its own `CLAUDE.md` for Claude Code.
- Only the Workshop owner can publish; collaborators deploy `dev` and test locally.
