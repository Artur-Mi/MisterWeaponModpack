# Water impact: plan (draft, nothing built yet)

Goal: missiles fired from above water stop treating the sea as air. Explosive missiles (rockets, guided missiles)
**detonate on the water surface**. Kinetic shells (assault cannon, artillery, railgun) **either detonate at the
surface or travel a short way underwater**, losing speed and damage, depending on their speed, mass and impact angle.

Status 2026-11-19 (repo `WaterImpact/`, see its [docs/behaviour.md](../WaterImpact/docs/behaviour.md)): **phase 1**
built and loaded in game (a rocket detonated on the water per the log); **phase 2** built, not tested. Phases 3 to 5 not
started. The user's decisions are at the end.

## What the game gives us (checked in the decompiled code and the Water Mod source)

- **Missile ammo is more than rockets.** In vanilla `Ammos.sbc`, these are `MissileAmmo` (entities the Mod API can touch):

  | Subtype | Weapon | Mass (kg) | Speed (m/s) | Explosion dmg | HealthPool | Gravity |
  |---|---|---|---|---|---|---|
  | `Missile` | rocket launchers | 45 | 100 → 200 | 500 (r 4) | none | no |
  | `MediumCalibreShell` | assault cannon | 300 | 500 | 0 (r 2) | 4 000 | yes |
  | `LargeCalibreShell` | artillery | 800 | 500 | 0 (r 3) | 17 000 | yes |
  | `SmallRailgunSlug` | small railgun | 500 | 1 000 | 0 | 8 000 | yes |
  | `LargeRailgunSlug` | large railgun | 2 000 | 2 000 | 0 | 50 000 | yes |
  | `Flare`, `Firework*` | decoys / fun | 1 | 160–180 | 0 | 3 | yes |

  Shells do their damage with the **HealthPool** (a penetration budget spent on the blocks they hit), not the explosion.
  So "losing damage potential" for a shell means **lowering its HealthPool**; for a rocket it means `ExplosionDamage`.
- Gatling and autocannon rounds are **projectiles**, not missiles: no entity, no position per tick. The Mod API only
  offers hit interceptors and `IMyProjectileDetector` (the hook shields and safe zones use). Out of scope for the
  first version; planned for phase 4.
- `IMyMissile` lets a mod set `LinearVelocity`, `ExplosionDamage`, `HealthPool`, `MaxTrajectory`, `ShouldExplode`, and
  call `Destroy()`. `MyAPIGateway.Missiles` has `OnMissileAdded`, `OnMissileRemoved`, `OnMissileMoved` (fires after
  each move; `Destroy()` must not be called inside it) and `Remove(id)`.
- **Detonating on demand:** `MyMissile` explodes when its distance from `Origin` reaches `MaxTrajectory`. Setting
  `MaxTrajectory` to the current distance and the velocity to zero makes it explode where it is, with its normal
  damage and effects, on the next ticks. The explosion runs on the server only (clients just remove the missile).
- **Removing without explosion (a "dud"):** `Destroy()` / `Missiles.Remove(id)` return the missile to the pool silently.
- The game re-adds gravity to `MissileGravityEnabled` rounds every tick, and accelerates rockets toward `DesiredSpeed`
  until reached, so slowing a round means overriding its velocity **every tick** while it is under water.
- **Water Mod:** the sea has no physics; nothing collides with it. It never touches missiles itself; it only restyles
  explosions near the surface (water blast or underwater effect, client side), so a surface detonation already looks
  right. Useful API: `GetDepth(pos)` (**signed**: positive above the surface, negative below, `null` with no water),
  `IsUnderwater`, `LineIntersectsWater`, `GetClosestSurfacePoint`, `GetUpDirection`, `CreateSplash`,
  `CreatePhysicsSplash`, `CreateBubble`. Calls throw or return nonsense where there is no water: guard them.

## Behaviour

### When a round counts as hitting the water

Only on **entry from above**: the round was above the surface on the previous tick and is below it now. Consequences:

- Rounds fired underwater (SLBM `AMissile`, `LauncherXMissile`) are never affected while they climb out. If one
  surfaces, misses and falls back into the sea, it detonates on entry. No subtype list needed for them.
- Fast rounds move up to 33 m per tick (large railgun), so the entry point is interpolated from the two signed depths:
  `t = d_prev / (d_prev - d_now)`, `entry = prev + t · (now - prev)`.
- Impact angle `θ` = angle between the velocity and the water surface (from `GetUpDirection`); 90° is straight down.

### Outcome per round

Every ammo subtype gets a **mode**: `Auto` (default), `Detonate`, `Penetrate`, `Dud`, or `Ignore`.

`Auto` resolves as:

1. `Flare`, `Firework*` → `Dud` (vanish with a small splash, no explosion).
2. Explosive ammo (no HealthPool, `ExplosionDamage > 0`: rockets, guided missiles, torpedo-like mod ammo) → `Detonate`.
3. Kinetic ammo (HealthPool > 0) faster than `BreakupSpeed` (default 1 100 m/s) at entry → `Detonate`: the impact
   shatters it at the surface (see [Realism](#realism)). With the defaults this is both railguns.
4. Other kinetic ammo → `Penetrate`, unless its predicted underwater travel is under `MinTravel` (default 3 m), then
   `Detonate` at the surface. Light, slow rounds hitting at shallow angles end up here.

Explosive ammo can instead get a `FuzeDelay` (metres, per subtype): it penetrates and detonates after that distance
under water, like a real delay-fuzed shell, unless it hits a hull first.

`Detonate`: move the round to the entry point, zero its velocity, set `MaxTrajectory` so it explodes there. Damage is
the ammo's own (a blast at the waterline still hurts a boat's hull nearby). Plus a splash.

`Penetrate`: a splash at the entry point, then every tick while under water:

- Speed decays with distance (quadratic water drag): `v = v0 · e^(-s / L)`, where `s` is distance travelled in water.
- `L` (the drag length) grows with mass, since a heavier round has more mass per frontal area:
  `L = DragK · m^(1/3)`, default `DragK = 4`. Overridable per ammo.
- Damage follows kinetic energy: `HealthPool = HP0 · (v / v0)²` (and `ExplosionDamage` the same way, if any).
- Direction is kept (gravity is cancelled under water; sinking is too slow to matter at these speeds).
- A bubble trail (`CreateBubble`) every few ticks, client side.
- When damage falls below `StopFraction` of the entry value (default 5 %), the round becomes a dud and is removed.
- If it hits a submerged hull before that, the game's normal collision uses the reduced HealthPool: less penetration.

Examples with the defaults (`v` falls to 22 % of entry speed, damage to 5 %, at `s = 1.5 L`; damage halves at `0.35 L`):

| Round | `L` | Damage halved after | Stops after |
|---|---|---|---|
| Assault cannon (300 kg) | 27 m | 9 m | 40 m |
| Artillery (800 kg) | 37 m | 13 m | 56 m |
| Small railgun (500 kg)* | 32 m | 11 m | 48 m |
| Large railgun (2 000 kg)* | 50 m | 18 m | 76 m |

\* Only if set to `Penetrate`; by default the railguns are above `BreakupSpeed` and shatter at the surface.

So a near miss within about 10 m of a ship's waterline still hurts it; a miss by 50 m does nothing. All numbers are
config values, to be tuned in game.

### Realism

The model follows real behaviour in broad strokes, with simplifications:

- **Explosive rounds detonating on the water: realistic.** At these speeds water hits like a hard surface, and impact
  fuzes fire on it. Near-miss blasts at the waterline did damage real hulls.
- **Shells travelling underwater: realistic, for heavy and not too fast rounds.** Naval armour-piercing shells did
  travel underwater; Japan even designed a "diving" AP shell to hit ships below the armour belt. The distance is tens
  of metres, which is what the model gives.
- **The drag maths is the real one.** Quadratic drag makes speed fall exponentially with distance, `v = v0·e^(-s/L)`
  with `L = 2m / (ρ_water · Cd · A)`. For an 800 kg shell about 38 cm wide that is roughly 20 to 50 m, depending on the
  drag coefficient. The plan's 37 m sits inside that range. (Game masses are large: 800 kg is battleship-shell size.)
- **Faster is not better in water.** Entry pressure is about `½ρv²`: 125 MPa at 500 m/s, which a steel shell survives,
  but 2 GPa at 2 000 m/s, above the strength of steel. Very fast rounds shatter or erode within a few metres; rifle
  bullets famously break up within about a metre while slower pistol bullets go further. Hence `BreakupSpeed`, which
  sends the railguns to the "explode at the surface" branch.
- **Shallow angles skip:** real (ricochet fire with cannonballs). Planned in phase 4.
- **Simplified:** real shells tumble or curve upward under water (they are shaped to fly stably in air, not water) and
  sometimes break on entry at any speed. The model keeps them straight and intact. A shallow-angle upward curve can
  come with the ricochet phase.

### Config

A file in the world's storage (`WaterImpact.xml`, written with the defaults on first load, server side):
global defaults (`DragK`, `MinTravel`, `StopFraction`, effects on/off) plus per-subtype overrides (`Mode`, `DragLength`).
Other mods can also register ammo by mod message later (this matches Phase 4 of [modpack-plan.md](modpack-plan.md)).

## Technical design

- **New repo `WaterImpact/`** at the workspace root, same layout as the others (`Mod/`, `deploy.json`, `deploy.ps1`,
  `CLAUDE.md`, `docs/`), added here as a submodule. **Standalone:** it does not need the brain, so it also works for
  vanilla and other mods' weapons. Needs the Water Mod to do anything; without it (or away from water) it is idle.
- Scripts under `Mod/Data/Scripts/WaterImpact/`:
  - `WaterImpactSession.cs`: session component. Tracks missiles via `OnMissileAdded`/`OnMissileRemoved`; per missile
    keeps the previous signed depth, state (`Air`, `Underwater`, `Done`), entry speed and entry HealthPool/damage.
    Does the work in `OnMissileMoved` (property changes) and queues removals for `UpdateBeforeSimulation`
    (where `Destroy()` is allowed).
  - `ImpactModel.cs`: the pure maths (entry point, angle, mode resolution, decay). No game calls, easy to reason about.
  - `ImpactConfig.cs`: defaults, XML load/save, per-subtype overrides.
  - `APIs/WaterModAPI.cs`: own copy of Jakaria's API (each mod compiles separately, so this does not clash with the brain).
- **Cheap when far from water:** cache the closest water planet per missile; skip the depth check while the round is
  higher above the surface than two ticks of travel.
- **Multiplayer:** see the next section.
- **Brain interplay:** the brain's `MissileGuider` sets the velocity of guided missiles every tick. Explosive missiles
  detonate on entry, so they never need slowing; aquatic missiles are skipped by the entry rule. No brain change
  expected. Iron Dome interceptors that dive into the sea while chasing will detonate on entry (intended).

## Multiplayer

Required (dedicated servers and hosted games). Facts from the game code:

- Missiles exist on every machine: the server broadcasts the spawn (`MyObjectBuilder_Missile`, with its `EntityId`),
  and each machine simulates its own copy. **The entity ID is the same everywhere**, so a packet can name a missile.
- Only the server explodes a missile and applies damage; a client just removes its copy at that point.
- `Destroy()` / `Missiles.Remove` act on the local machine only: a dud removed on the server keeps flying on clients
  unless they remove it too.

Design: **the server decides, clients predict and then obey.**

- **Server:** detects entry, resolves the outcome (mode, entry point, entry velocity; later the random parts of
  ricochet), applies it, and owns everything that changes damage (`HealthPool`, `ExplosionDamage`, `MaxTrajectory`).
  Then it sends a `WaterEntryPacket` (missile ID, outcome, entry point, entry velocity, drag length) to the clients.
- **Clients:** run the same detection and model for **motion and effects only** (splash, slowing, bubbles), so there is
  no lag at the surface: at 500 m/s a 100 ms packet delay is 50 m of travel. When the packet arrives it overrides the
  local guess (snaps the round to the server's entry state, advanced by the time elapsed) or removes the round.
  Clients never touch damage.
- **Removal:** a dud or a stopped round is removed by the server and by the packet on clients (local prediction may
  remove it earlier; that is harmless).
- **Transport:** THDigi's NetworkLib, as the brain already uses (copy its `NetworkLib/` folder into this mod, own
  `PacketBase` registration, own channel ID distinct from the brain's `3496380795 % 65535` and from the Water Mod's
  50271). Packets go to players near the round, not to everyone.
- **Bandwidth:** one small packet per round that hits water, no per-tick traffic. Artillery barrages into the sea are the
  worst case; batch entries per tick into one packet if needed.

## Phases

1. **Detonate on entry** for explosive ammo, duds for flares, splash effect. Proves the hook, the entry interpolation
   and the server-side explosion, plus NetworkLib and the entry packet. Test: rockets and brain-guided missiles fired at
   the sea, at a boat's waterline, in single player and with a second player (or a local dedicated server).
2. **Penetration** for kinetic rounds: decay, HealthPool scaling, removal, bubbles. Test: artillery and railgun at a
   partly submerged target at 5, 20 and 60 m below the surface.
3. **Config file** and per-subtype overrides; register the pack's own ammo (`LauncherXMissile`, `IronDomeXMissile`, ...)
   explicitly where `Auto` is wrong.
4. **Extras:** ricochet off the surface at shallow angles; gatling/autocannon bullets stopped at the surface with a
   projectile detector (check feasibility first).
5. Docs, `status.md`, `testing.md`, dev deploy, Workshop page (requires the Water Mod).

## Decisions (user, 2026-11-19)

1. **Separate mod `WaterImpact`**, standalone (no brain dependency).
2. Penetrating rounds with explosive damage: **`ExplosionDamage` scales down with speed**, same as the HealthPool.
3. **Ricochet: yes, in a later phase** (phase 4): shells under about 6° skip off the surface, losing speed.
4. **Gatling and autocannon bullets: stopped at the surface, in a later phase** (phase 4), after a feasibility check of
   `IMyProjectileDetector`.
5. **Multiplayer is required**, with THDigi's NetworkLib (see [Multiplayer](#multiplayer)). Phase 1 already includes the
   packet, so detonation is tested with a client from the start.
6. **Railguns shatter at the surface by default** (`BreakupSpeed` 1 100 m/s). A world can still set them to `Penetrate`
   in the config.
