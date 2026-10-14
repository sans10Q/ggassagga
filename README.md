# Lanterns: Shadows of the Corps (v1.8.0)
**v1.8:** TASKS + task-charged GUARDIAN LIGHT (see "Tasks" below).
**v1.7:** the Power Ring is now a 26-cuboid 3D model (gold signet, emerald bars, round gem), kept small in hand. Editable in Blockbench: `assets/lanterns/models/item/power_ring.json` + `textures/item/power_ring_3d.png`.
**v1.6 (smoothness):** boss-bar countdown for discussion/voting; ability feedback no longer flickers away; "Kill ready"/"Blackout ready" pings; Blackout Core shows a cooldown overlay; meeting lock is a soft radius (`meetingFreeRadius`, default 3) instead of rubber-banding; 3-second beat before vote results; kill particles.
**v1.5:** ring and Blackout Core always go in hotbar slot 9 (never auto-selected); `/lantern forcerole`; discussion 45s; old configs auto-migrate.
**v1.1:** ring only goes to the Lantern after the first meeting; 1 Manhunter by default; Manhunters punch to kill and use a Blackout Core item; role shown at start.
Fabric mod (Minecraft Java **1.21.1**, **Java 21**, Fabric Loader >= 0.16, Fabric API 0.102.0+1.21.1).
A social-deduction murder mystery: 2 hidden Manhunters, bodies, meetings, votes, and one secret Green Lantern ring.

## HONEST STATUS
- **No compiled JAR is included.** The environment this was written in had no network, so Gradle, Minecraft and Fabric
  could not be downloaded. The full source project is complete; build it with one of the options below.
- The Minecraft-facing code (`GameManager`, `LanternsMod`, `RingItem`) was written against Yarn 1.21.1 mappings
  but **has never been compiled or run in-game**. Expect to possibly fix a few small mapping/API mismatches on first build.
  Please treat it as a first full draft, not a play-tested release.

## Build the JAR
**A) Easiest, no local setup:** push this folder to a GitHub repo. The included workflow (`.github/workflows/build.yml`)
builds it; download the JAR from the Actions run's *Artifacts*.

**B) Locally:** install JDK 21 and Gradle 8.8+, then in this folder:
```
gradle wrapper --gradle-version 8.8     # once, creates gradlew
./gradlew build                          # Windows: gradlew.bat build
```
Output: `build/libs/lanterns-shadows-of-the-corps-1.0.0.jar` (ignore the `-sources` jar).
Dev client: `./gradlew runClient`.

## Install
Put the JAR and **Fabric API** in `mods/` on the server **and on every client** (the ring is a custom item, so vanilla clients can't join).

## Setup & play
1. Stand where meetings should happen: `/lantern setmeetingpoint` (OP).
2. `/lantern start` (OP; needs 6+ players, or `/lantern start force` with 3+ for testing).
3. At start everyone sees their role (big title, chat line, and a constant action-bar line; `/lantern role` shows it again).
   **Nobody has a ring at the start.** After the first meeting ends, ONE random living innocent receives the Power Ring (and is told privately).
   Ring: right-click = use, sneak + right-click = change mode. If the Lantern is voted out/disconnects, the ring item moves to the next bearer.
4. **Manhunters** punch a player to kill (cooldown), and get a **Blackout Core** item at start: right-click it for System Blackout (it only works for Manhunters). Default is **1 Manhunter**
   (change `manhunters` in `config/lanterns.json`; if you ran an older version delete that file or edit the value, it is saved on first launch).

## Commands
| Command | Who | Effect |
|---|---|---|
| `/lantern start [force]` | OP | Randomize roles, start |
| `/lantern stop` | OP | End game safely, restore game modes/rings |
| `/lantern status` | all | Public info only, no roles |
| `/lantern forcerole <manhunter\|innocent\|lantern\|clear> [player]` | OP | Guarantee a role (default: yourself) from the next start. Lantern = you get the ring after the first meeting. Cleared with `clear`; forgotten on server restart |
| `/lantern task add <scan\|puzzle\|calibrate>` | OP | Turn the block you are looking at into a task station (saved in config) |
| `/lantern task remove` / `list` / `clear` | OP | Remove the looked-at station / list all / delete all |
| `/lantern tasks` | players | Show your own tasks and team progress |
| `/lantern answer <n>` | players | Answer a puzzle (also clickable in chat) |
| `/lantern role` | players | Shows your own role again (private) |
| `/lantern meeting` | players | Emergency meeting (1 per player, grace + cooldown) |
| `/lantern report` | players | Report nearest body in range (or right-click the body) |
| `/lantern vote <player\|skip>` | players | Cast vote (also clickable in chat); one vote, locked |
| `/lantern setmeetingpoint` | OP | Save position + dimension |
| `/lantern reload` | OP | Reload `config/lanterns.json` |

(`vote` and `reload` are additions; there is deliberately no `/lantern investigate`.)

## Rules as implemented
- **Manhunters** (default 1, configurable; clamped to <= (n-1)/2): know each other. Abilities: *Instant Kill* (punch a player within 3.5 blocks, 60 s cooldown, 30 s opening cooldown)
  and *System Blackout* (right-click the Blackout Core) (10 s darkness on **everyone** incl. the activator, 90 s cooldown, ends when a meeting starts, doesn't bypass kill cooldown).
  Win: living Manhunters >= all living non-Manhunters, or killing the current Lantern.
- **Bodies:** a kill leaves an "Unidentified Body" and a hidden evidence record. Reporting shows the victim and approximate time only; no role, no killer.
- **Meetings:** all abilities off, living players teleported to the meeting point and held there, 45 s discussion (voting can't start before this), 30 s voting (ends early when everyone has voted).
  Most votes ejects; ties, or skips >= top votes, eject nobody. Ejected/dead players become spectators ("ghosts" with a private ghost chat).
  Players return to prior positions afterwards. Roles are **not** revealed on death (config `revealRoleOnDeath`); all roles are revealed at game end.
- **Green Lantern:** none until the first meeting ends; then a random living innocent gets the ring. If the bearer is voted out (or disconnects) the ring secretly
  passes to another random living innocent (never a Manhunter/dead player); ability state belongs to the ring, so it carries over. If killed -> Manhunters win instantly.
  - *Verdict of Will:* look at a player within 30 blocks, accurate private result, **1 charge per game**, no notification to target.
  - *Residual Will:* near a body, 45 s cooldown; gives up to 4 distinct inconclusive clues (rough age, blackout or not, a 3-name pool containing the killer, side the attacker stood on).
  - *Last Light:* look at a player (or at nobody = self) to shield them from one Manhunter kill for 20 s. 1 use per round (refreshed when a meeting ends);
    self-shield only once per game, tracked separately. A blocked kill gives a public green flash. Not usable in meetings; harmless to everyone.
- Participants are put in Adventure mode (no block breaking), take no damage, and don't starve. Game modes are restored on stop/disconnect/shutdown.
- Disconnecting players are removed from the game (ring transfers if they held it); rejoiners during a game spectate.


## Tasks and Guardian Light (v1.8)
**Setup (admins):** look at any block (a lectern, a button, a furnace...) and run `/lantern task add scan`, `puzzle` or `calibrate`. Stations are saved
in `config/lanterns.json`. With **no stations** the game plays exactly as before (tasks off). Each player is assigned `tasksPerPlayer` (default 3) distinct stations.
Stations sparkle (private particles) only for players who still have a task there.
- **scan** - right-click, then stay within 4 blocks for `scanSec` (default 6) seconds. Walking away interrupts it.
- **puzzle** - right-click to get a multiplication question with 3 clickable answers.
- **calibrate** ("Power node" in task lists) - works like a scan for everyone; if the **Lantern** finishes it, Residual Will is recharged. Nobody can tell the difference
  in the task list, and **Verdict of Will is never restored**.
- **Manhunters get fake tasks** that look identical but add nothing to team progress (bluffing tool). A dead innocent's unfinished tasks drop out of the team total.
- If every remaining innocent task is done, **Innocents win** (`tasksCompleteWins`, default true; set false to make tasks only charge Guardian Light).
- The action bar shows your own task count and the team %.

**Guardian Light (task-charged, hidden, automatic):** team progress crossing 25% / 50% / 75% (`guardianThresholds`) banks a charge (max 1 banked) on the ring.
If a Manhunter hits the **Lantern** while it is charged and unlocked, the kill fails (green flash like Last Light, which takes priority if active), and the Lantern is told.
**Nerf:** after it triggers it is locked until **two meetings have ended** (the next meeting and the one after). Charges earned while locked are banked but unusable until the lock expires.
Note: the flash is public, so a failed kill tells nearby players that the target is probably the Lantern.

## Config (`config/lanterns.json`)
`manhunters, minPlayers, killCooldownSec, initialKillCooldownSec, postMeetingKillCooldownSec, killRange, blackoutDurationSec, blackoutCooldownSec,
residualCooldownSec, meetingFreeRadius, tasksPerPlayer, tasksCompleteWins, scanSec, guardianThresholds, tasks, shieldDurationSec, discussionSec, votingSec, emergencyMeetingsPerPlayer, meetingGraceSec, emergencyCooldownSec, reportRange,
revealRoleOnDeath, revealVotesPublicly, returnPlayersAfterMeeting, meetingPoint`. Verdict (1/game) and self-shield (1/game) are hard-coded and not configurable.

## Test report (truthful)
| What | Result |
|---|---|
| Pure rules in `GameLogic` (role count/assignment, forced roles, win conditions, vote ties/skips/dead votes, ring candidate selection, guardian thresholds + 2-meeting lock, cooldown math) | **35/35 checks passed**, compiled with `javac` 21 and run via `tools/LogicCheck.java` |
| Same scenarios as JUnit 5 (`src/test/java/.../GameLogicTest.java`) | Written; **not run** (no JUnit offline). Runs with `gradle test` |
| Gradle build / JAR | **Not run** (no network) |
| In-game behavior (kills, bodies, meetings, teleport, blackout, ring item, commands, disconnect handling) | **Not tested.** No Minecraft server was available |

Run the logic checks without Gradle:
```
javac -d /tmp/out src/main/java/com/lanterns/GameLogic.java tools/LogicCheck.java && java -cp /tmp/out LogicCheck
```

## Known limitations / suggested next steps
- No custom GUI for voting (clickable chat + commands). No player-head skins on bodies. Game state is not persisted across server restarts.
- Chat is open to the living at all times (only the dead are isolated). Clue and balance numbers are first guesses; playtest and tune the config.
- Item art is generated: the ring is a 3D model (32x32 texture), the Blackout Core is a flat 16x16 sprite. Replace the PNGs in assets/lanterns/textures/item/ with your own art any time.
