# Status and handoff

Snapshot at the end of the first working session (2026-11-17). Read this first when resuming. Structure and phases are in
[modpack-plan.md](modpack-plan.md); how to test is in [testing.md](testing.md).

## State of the mods

| Mod | Git | Dev link in `Mods\` | Notes |
|---|---|---|---|
| `missil-misterplace` (brain) | Clean | `missil-misterplace-dev` | Has the aquatic ammo registry, `AquaticLauncher`, and the splash crash fix. Final `release/baseline` is the published version |
| `missilSLBM-misterplace` | Clean | `missilSLBM-misterplace-dev` | Data only. Dev item not published yet (no dev Workshop ID) |
| `LauncherX` | Clean | `LauncherX-dev` | Missile is aquatic/SLBM ammo, block is underwater-only, requires the brain. Not published (dev or final) |
| `IronDomeX` | Clean | `IronDomeX-dev` | Block + ammo registered in the brain's Iron Dome logic (requires the brain); ammo stats equal `IDMissile`. Dev deployed, not tested in game |
| `AntennaIronDome` | Clean (baseline commit) | `AntennaIronDome-dev` | Standalone antenna with a spinning radar dish; moved into a repo 2026-11-18 (content unchanged from the plain `Mods\AntennaIronDome` copy). Dev deployed; not tested in game; not published |
| `WaterImpact` | Phase 1 committed locally; **no GitHub repo or submodule yet** | `WaterImpact-dev` | New 2026-11-19: missiles detonate on hitting water, railguns shatter, flares vanish (phase 1: loads, one rocket detonated per the log); artillery/assault shells travel on under water losing damage (phase 2: not tested); multiplayer packet Plan: [water-impact-plan.md](water-impact-plan.md) |
| `IronDomeClassic` | Clean (initial commit) | `IronDomeClassic-dev` | Iron Dome turret/launcher/ammo moved out of the brain (data only; brain scripts still drive them). Dev deployed; not tested in game; not published |

The three Iron Dome repos (`IronDomeClassic`, `IronDomeX`, `AntennaIronDome`) moved into the `IronDome/` folder on 2026-11-18; their dev junctions were re-created and their `deploy.ps1` now finds sibling repos one level up and survives a broken junction.

All six mod repos and the base repo are **public on GitHub** under `Artur-Mi` (2026-11-18); the base repo `MisterWeaponModpack` pins the mods as submodules (relative URLs). Nothing has been published or deployed as final since the repos were created.

`deploy.ps1` (same file in every repo) is machine- and OS-agnostic since 2026-11-18: SE data folder from `-SeDataDir`, `SE_DATA_DIR`, `%APPDATA%` or the Linux Proton prefix; symlink off Windows. Tested on Windows (including a fresh `Mods` folder); **not tested on Linux**.

## What was done

1. One git repo per mod with `Mod/` for shipped content, Git LFS for binaries, `deploy.json` + `deploy.ps1`, docs and `CLAUDE.md`.
2. Dev = junction from `%APPDATA%\SpaceEngineers\Mods\<name>-dev` to the repo's `Mod/`; final = clean-commit snapshot copy plus a
   `release/*` tag ([deployment.md](../missil-misterplace/docs/deployment.md)).
3. `se-dev-mod` skill prepared (Graphify off). The user works only through Claude Code, with no IDE.
4. **LauncherX made an SLBM-style block** in the brain: `LauncherXMissile` registered as aquatic ammo
   (`GuidedMissileSystem.AquaticAmmoSubtypes`); Iron Dome whitelist includes it; LauncherX's own stats were not changed.
5. **LauncherX requires the brain**: `requires` in `deploy.json` (deploy-time warning), runtime warning in `LauncherX.cs`,
   Workshop "Required items" still to be set by hand at publish time.
6. **Crash found and fixed** (first in-game test): `MissileGuider` asked the Water Mod for a splash for any aquatic missile
   that wasn't underwater on its first tick, and `Jakaria.SimulatedSplash` threw a `NullReferenceException` where there is
   no water. Now the splash only happens on a real surfacing, inside a try/catch.
7. **Underwater-only enforcement** (`AquaticLauncher.cs`) for both `LargeMissileLauncherSLBM` and `LauncherX`: the Water Mod only
   tests a block's origin cell and does nothing without water (space), so the brain switches the block off whenever its
   muzzle is not under water (Water Mod loaded). The user confirmed it is working.

## Tested vs not tested

Confirmed by the user or by the game log: scripts compile and load (all four mods); the crash is fixed; the underwater-only
enforcement works.

**Not confirmed yet** (check these next time you test at sea):
- Surfacing effects (smoke and splash) and the speed ramp after surfacing, for LauncherX (3 to 300 m/s) and for SLBM (held at 100).
- Iron Dome blocks ignoring the missile underwater and intercepting it after it surfaces.
- A tall tower with only the base submerged (muzzle sampling).
- A big flame at LauncherX's nozzle underwater (visual only).
- Multiplayer or dedicated server (the enforcement is server-side; untested).
- The "held off" state across save/load (not persisted: a saved-off block stays off until the player switches it on).
- The first final deploy and publish of anything.

## Open items, in rough priority

0. **WaterImpact** (new, 2026-11-19): create its GitHub repo, push, add it here as a submodule; test phase 1 at sea
   ([testing.md](testing.md#checklist-for-waterimpact)), single player then with a second player, including phase 2 (shell penetration);
   then phase 3 (config file). See [water-impact-plan.md](water-impact-plan.md).

1. SLBM and IronDomeX: declare the brain dependency the same way as LauncherX (`requires` in `deploy.json`, a runtime
   warning in a script, and the Workshop required item). IronDomeX has the `requires` entry now; both still need the runtime
   warning and the Workshop item. SLBM is data-only, so it needs a small session script for the warning.
2. Phase 4: replace hard-coded ammo subtypes in the brain with registration (interceptor ammo now has
   `InterceptorAmmoSubtypes`, with `IronDomeXMissile` registered); test IronDomeX in game: it fires at missiles (its ammo now has
   the same stats as `IDMissile`; the weapon still fires 120 rpm, single shot); decide what shared logic
   moves from LauncherX/IronDomeX into the brain ([modpack-plan.md](modpack-plan.md)).
3. First publish of the dev items (SLBM, LauncherX, IronDomeX): record the IDs the game writes into `Mod/modinfo.sbmi`
   into `deploy.json`, and set the dev brain as a required item on each.
4. `MissileTurret.missilesSubtypeIdWhitelist` has the typo `"PMissike"` (should be `"PMissile"`): the Iron Dome never
   intercepts `PMissile`. Not fixed; the user is working in that area.
5. LauncherX and IronDomeX lack `metadata.mod` and a `BlockCategories.sbc` entry.
6. Iron Dome can intercept `LauncherXMissile` once it surfaces (mirrors SLBM). To undo it, remove the `"LauncherXMissile"` line in
   `MissileTurret.cs`.
7. Questions still open: `AntennaIronDome` now has its own repo; should `M61IronDomeX` (in the game's Mods folder only) join the pack too?
8. **Iron Dome split (done, committed, dev deployed, untested)**: test brain-dev + `IronDomeClassic-dev` in game
   ([testing.md](testing.md)). The published final brain
   still contains the blocks, so do not release a new final brain until `IronDomeClassic` is published and set as required
   by worlds that use the Iron Dome. The blocks keep the same subtypes, so existing worlds work once the new mod is loaded.
9. **Iron Dome launcher Range slider (works in game per the user, 2026-11-18)**: classic launcher and Iron Dome X have a Range
   slider; since 2026-11-18 if the AntennaIronDome mod is loaded, a working antenna on the same grid is required and sets the range (slider hidden; also for the classic turret), untested (see [architecture.md](../missil-misterplace/docs/architecture.md)). Still unchecked: that the value survives
   save/load, and that it syncs in multiplayer or on a dedicated server.
10. GitHub sharing (done 2026-11-18, public): still to do: add the friend as a collaborator on all seven repos (`gh repo edit` / Settings > Collaborators), and the friend's first clone and deploy. See [modpack-plan.md](modpack-plan.md#github-sharing).

## Gotchas learned

- **Water Mod:** it only works where there is water. In space or on a dry world its configs do nothing and API splash
  calls throw. It tests one point per block (the origin cell). Test worlds need a water planet
  ([testing.md](testing.md)).
- **Game log:** `%APPDATA%\SpaceEngineers\SpaceEngineers_<date>_<time>.log` (newest file). Search for `Exception`,
  the mod names, and `WaterMod:`.
- **PowerShell 5.1 and UTF-8:** `Get-Content`/`Set-Content` read files with the system code page and garble non-ASCII
  characters (this happened to `modpack-plan.md`). Edit docs with the Edit tool or Python, never a PowerShell read-and-write.
- **Same names in two mods:** the plain copies (`LauncherX`, `IronDomeX`, `missil-misterplace`, `missilSLBM-misterplace`) share
  block and ammo names with the dev mods. Never load both in one world.
- **Backups, never deletes:** deploys move replaced folders to `%APPDATA%\SpaceEngineers\ModBackups\`. Currently there:
  `missil-misterplace-dev-20261117-223554` (the old plain dev copy) and `missil-misterplace-dev-source-20261117-224010`
  (the old pre-git source folder, identical to the repo's `Mod/` at that time).
- **Removing a dev link:** delete it from File Explorer or run `rmdir` on it; it removes only the link. Never use a
  recursive delete on it. `deploy.ps1` already handles this safely.
- **Test worlds:** `Empty World 2026-11-17 20-53` is the user's old LauncherX world (one asteroid, no water, old plain mods).
  Left untouched.
- **Skills:** `se-dev-mod` was prepared with `SE_DEV_GRAPHIFY=0`; re-run its `Prepare.bat` after subscribing to new
  Workshop mods to refresh the index. `se-dev-game-code` is ready.
