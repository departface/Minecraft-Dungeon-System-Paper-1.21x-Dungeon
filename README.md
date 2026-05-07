# Minecraft-Dungeon-System-Paper-1.21x-Dungeon
# DungeonPlugin v1.2.2 — Patch Release

**Автор / Author:** [@departface](https://t.me/departface)  
**Совместимость / Compatibility:** Paper 1.21.x · Java 21  
**Дата / Date:** May 7, 2026

---

## What's Changed

### 🐛 Bug Fixes

#### [CRITICAL] Chest loot was always empty
- **Root cause 1 — snapshot overwrite:** `BlockState.getInventory()` was called on a *snapshot* block state, then `chest.update()` was flushing the snapshot back to disk — overwriting any items that had been added to the live inventory. Fixed by switching to `block.getState(false)` which returns a **direct tile-entity reference** instead of a snapshot, so inventory writes persist immediately without needing an explicit update call.
- **Root cause 2 — tile entity not ready:** The original delay was `2L` ticks (later patched to `5L`), which is insufficient on servers with multiple dungeons generating simultaneously. Increased to **`20L` ticks (1 full second)** to guarantee the chest tile entity is fully initialised before loot is written.
- **Root cause 3 — physics flag:** `block.setType(Material.CHEST, false)` with `physics=false` could prevent Bukkit from registering the chest's tile entity on some server configurations. Changed to `physics=true`.
- Additionally hardened slot-placement logic: items now try up to 5 random slots before falling back to `addItem()`, preventing silent item loss on a full row.

#### [FIX] Players were teleported to spawn when walking out of dungeon
- **Root cause:** `PlayerMoveEvent` called `onPlayerLeaveDungeon(player, false)` whenever a player crossed the dungeon boundary, which unconditionally invoked `teleportToSpawn()`.
- **Fix:** Introduced a dedicated `onPlayerWalkOutOfDungeon(Player)` method in `DungeonManager` that removes the player from dungeon tracking and sends a notification message **without teleporting**. Teleport-to-spawn is now only triggered by `/dungeon leave` (voluntary) or when the dungeon timer expires / dungeon closes.

---

## Files Changed

| File | Change |
|---|---|
| `loot/LootManager.java` | Removed `chest.update()` calls; new `placeItem()` helper with 5-attempt random slot placement |
| `dungeon/DungeonGenerator.java` | `physics=true` on chest placement; delay `2L→20L`; use `getState(false)` |
| `dungeon/DungeonManager.java` | New `onPlayerWalkOutOfDungeon()` — untracks player without teleport |
| `listeners/DungeonListener.java` | Boundary exit now calls `onPlayerWalkOutOfDungeon()` instead of `onPlayerLeaveDungeon()` |

---

## Full Changelog

```
v1.2.1 → v1.2.2
  fix: chest tile entity snapshot overwrite causing empty loot
  fix: chest tile entity not ready at 2-5 tick delay → raised to 20 ticks
  fix: CHEST block placed with physics=false preventing tile entity init
  fix: boundary exit teleporting players to spawn unconditionally
  feat: new onPlayerWalkOutOfDungeon() for boundary-only untracking
  refactor: LootManager.placeItem() helper for safer slot assignment
```

---

## Installation

1. Stop your Paper server.
2. Delete the old `DungeonPlugin-*.jar` from `plugins/`.
3. Drop `DungeonPlugin-v1.2.2-by-departface.jar` into `plugins/`.
4. Start the server — no config migration needed.

---

*Plugin by [@departface](https://t.me/departface)*
