# MisterWeapon modpack

Space Engineers 1 weapon mods: guided missiles (the "brain"), SLBM, LauncherX and the Iron Dome family.
This repo holds the shared docs and pins each mod repo, as a git submodule, at a version known to work with the others.

| Folder | What |
|---|---|
| `missil-misterplace/` | Brain: guided-missile logic (scripts only) |
| `missilSLBM-misterplace/` | SLBM launcher and missiles (needs the brain) |
| `LauncherX/` | LauncherX block (needs the brain) |
| `IronDome/IronDomeClassic/` | Original Iron Dome turret and launcher (needs the brain) |
| `IronDome/IronDomeX/` | IronDomeX block (needs the brain) |
| `IronDome/AntennaIronDome/` | Antenna with a rotating radar dish (standalone) |

## Get it

Needs [Git](https://git-scm.com) and [Git LFS](https://git-lfs.com) (models and textures are in LFS).

```bash
git lfs install
git clone --recurse-submodules https://github.com/Artur-Mi/MisterWeaponModpack.git
```

Later updates: `git pull --recurse-submodules`.

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

- Commit and push inside the mod you changed (each mod is its own repo).
- To share a combination that works, commit the moved submodule pointer here: `git add <mod folder>` and commit.
- Docs: [docs/status.md](docs/status.md) (state and open items), [docs/testing.md](docs/testing.md),
  [docs/modpack-plan.md](docs/modpack-plan.md). Each mod has its own `CLAUDE.md` for Claude Code.
- Only the Workshop owner can publish; collaborators deploy `dev` and test locally.
