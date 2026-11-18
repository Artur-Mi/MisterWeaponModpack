# Testing the dev mods in the game

## Deploy

```powershell
cd <repo>            # missil-misterplace, LauncherX, missilSLBM-misterplace, IronDomeX, IronDomeClassic
.\deploy.ps1 dev     # once; the link is live afterwards, so edits show on the next world load
```

## Which mods to load

| Load | Why |
|---|---|
| `missil-misterplace-dev` | the brain |
| the block mod's `-dev` folder (`LauncherX-dev`, `missilSLBM-misterplace-dev`, `IronDomeX-dev`, `IronDomeClassic-dev`) | what you are testing |
| `watermod-misterplace` (or the Workshop Water Mod) | the underwater behaviour needs it |

Never load the plain copies (`LauncherX`, `IronDomeX`, `IronDomeClassic`, `missil-misterplace`, `missilSLBM-misterplace`) or the Workshop brain in the
same world as the dev mods: they share block and ammo names.

## The world

It needs a **planet with water**: the Water Mod does nothing in space or on a dry planet. Use a Water Mod planet (for example
Teal). Duplicate a world rather than editing the one you play.

## Checklist for the underwater launchers (SLBM, LauncherX)

1. The world loads without errors. If not, read the newest log in `%APPDATA%\SpaceEngineers\` (search `Exception`).
2. Muzzle above water: the block is switched off and won't fire. Switching it on does nothing.
3. Muzzle submerged: the block comes back on and fires.
4. Missile: leaves slowly (about 10 m/s, slowing), travels straight, surfaces with smoke and a splash, then accelerates and
   steers to the target.
5. Iron Dome (optional; load `IronDomeClassic-dev` too, the blocks are no longer in the brain): ignores the missile
   underwater, can intercept it after it surfaces.
6. Brain disabled: LauncherX shows a red "falta o mod principal" notification at world start.
7. A tall tower (LauncherX) with only its base submerged: should stay off until the muzzle end is under water.

## Checklist for the Iron Dome split (brain + IronDomeClassic)

1. Load `missil-misterplace-dev` + `IronDomeClassic-dev`: the world loads with no `Exception` in the log.
2. The turret and the launcher appear in the weapons category and can be built; the `IronDomeMissile` magazine can be
   produced and loaded.
3. Both intercept an enemy missile as before (the scripts are in the brain).
4. Load only the brain: the Iron Dome blocks are absent and nothing errors.

## When something fails

Read the newest `%APPDATA%\SpaceEngineers\SpaceEngineers_<date>_<time>.log` and search for `Exception`, the mod names
(`GuidedMissile`, `LauncherX`) and `WaterMod:`. A script compile error also stops the mod from loading.
