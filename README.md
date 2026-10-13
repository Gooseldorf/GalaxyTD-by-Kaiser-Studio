# Galaxy TD

A 2D tower-defense project built around Unity Entities. This README is a guide to the codebase: how mission data becomes a running simulation, and where to find its main systems.

**Project date:** April 2023.

## Gameplay and Android demo

Watch gameplay videos on the [Behance project page](https://www.behance.net/gooseldorf). Download the [Android demo build](https://gooseldorf.itch.io/gunztd).

## Architecture at a glance

The project uses two cooperating layers:

- **Authoring and orchestration:** `ScriptableObject` data describes missions, creeps, towers, upgrades, and weapon parts. `GameServices` initializes missions and handles player actions such as building, upgrading, selling, and reloading towers.
- **Simulation:** Unity Entities systems process spawning, navigation, targeting, projectiles, damage, power, and win/loss conditions. Components, jobs, and event entities carry state and results between systems.

The UI and visual systems respond to the simulation; they are not the source of its core rules.

## Follow a mission through the code

1. [`GameInitialization.cs`](Assets/Scripts/GameInitialization.cs) selects the mission and available towers, then calls `GameServices.InitMission`.
2. [`GameServices.cs`](Assets/Scripts/Systems/GameServices.cs) creates simulation singletons, initializes spatial lookups, spawners, rendering, and the map, then instantiates mission objects as entities. It also configures fixed-step simulation.
3. [`Mission.cs`](Assets/Scripts/Systems/Mission.cs) converts tile data into walkable cells and creates entities for spawn zones, exits, drop zones, energy cores, portals, gates, bridges, and other map objects. [`MapCreator.cs`](Assets/Scripts/MapCreator.cs) contains the map authoring and drawing logic.
4. [`SpawnerSystem.cs`](Assets/Scripts/ECSTest/Systems/SpawnerSystem.cs) schedules creeps from mission waves.
5. [`FlowFieldBuildCacheSystem.cs`](Assets/Scripts/ECSTest/Systems/FlowField/FlowFieldBuildCacheSystem.cs) builds inward and outward navigation fields. [`MovingSystem.cs`](Assets/Scripts/ECSTest/Systems/Movings/MovingSystem.cs) combines those directions with local steering, collision handling, knockback, and portals.
6. [`TargetingSystemBase.cs`](Assets/Scripts/ECSTest/Systems/TargetingSystemBase.cs) acquires targets, checks line of sight, controls firing timers, and creates projectiles. [`ProjectileSystemBase.cs`](Assets/Scripts/ECSTest/Systems/ProjectileSystemBase.cs) advances them and resolves penetration or ricochet. Collision events are consumed by [`OnCollisionEventSystem.cs`](Assets/Scripts/ECSTest/Systems/OnCollisionEventSystem.cs) and [`DamageSystem.cs`](Assets/Scripts/Systems/Attakers/Stats/DamageSystem.cs).

## Other systems worth reading

| Area | Starting point | What to look for |
| --- | --- | --- |
| Tower configuration | [`TowerFactory.cs`](Assets/Scripts/Towers/TowerFactory.cs) | Assembly from a prototype, upgrades, parts, ammunition, and directives |
| Power and objectives | [`PowerSystemBase.cs`](Assets/Scripts/ECSTest/Systems/PowerSystemBase.cs), [`WinLoseSystem.cs`](Assets/Scripts/ECSTest/Systems/WinLoseSystem.cs) | Energy cores, connected objects, remaining power cells, and mission outcome |
| Rendering | [`BRGSystem.cs`](Assets/Scripts/BRG/BRGSystem.cs) | `BatchRendererGroup` rendering for creeps, health bars, status indicators, and firing effects |
| UI | [`GameUIManager.cs`](Assets/Scripts/UI/GameScene/Scripts/GameUIManager.cs) | UI Toolkit documents and subscriptions to game events |
| Persistence | [`GameData.cs`](Assets/Scripts/GameData.cs) | Player progression, inventory, currencies, and binary save/load |
| Replay tooling | [`ReplayManager.cs`](Assets/Scripts/Systems/ReplayManager.cs), [`Replay.cs`](Assets/Scripts/Systems/Replay.cs) | Timestamped player commands and their playback |

## Reading notes

- `ECSTest` is the directory name for much of the ECS implementation; it is not limited to tests.
- Navigation is affected by map state: building or selling a tower changes the base flow field, while powered gates and bridges can change traversability.
- Weapon parts can apply static stat changes during tower assembly and attach event-driven effects handled during shooting, hits, kills, reloads, or ricochets.
- The code includes replay recording and playback; this README does not claim deterministic replays or a verified test run.

## Publication note

_Unfortunately, Gyro Games studio has closed, and their projects are no longer available on Google Play. I have received permission from the company's management to publish this demo version of the project on GitHub for personal use._
