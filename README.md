# Shattered New World

> [!WARNING]
> **This mod was written entirely by an AI coding agent (Claude Code). No human has reviewed the code.**
> It has been played and works in the author's games, but it hasn't been checked line by line by a person, or tested with every bookmark, DLC or mod combination. It could break saves, behave unexpectedly, or stop working after a game update. Use it at your own risk, and please [report bugs](../../issues).

A Crusader Kings III mod (built against game version 1.20.1) that keeps the real-world map and removes the real history. When a new game starts, historical realms are broken up to the level you choose and every landholder is replaced with a newly generated character. Each one has a name from their culture's name list, a plausible age, random traits and skills, and their own dynasty and family. Every historical character still alive at the start is then removed. Each new game produces a different world.

## Installing

**From the Steam Workshop (recommended):** subscribe to the mod, then enable **Shattered New World** in a playset in the Paradox launcher.

**Manually from this repository:**
1. Copy this folder into your CK3 `mod` folder (`Documents/Paradox Interactive/Crusader Kings III/mod/` on Windows, `~/.local/share/Paradox Interactive/Crusader Kings III/mod/` on Linux).
2. Next to it, create `shattered_new_world.mod` with the contents of `descriptor.mod` plus a line `path="<full path to the copied folder>"`.
3. Enable **Shattered New World** in a playset in the Paradox launcher.

**Don't enable it together with Random New World.** Both rebuild the world at game start and will conflict.

## Using it

1. Start a new game from any bookmark and pick any character. Who you pick only decides *where* you start: you take over the new ruler who replaces them.
2. Open **Game Rules → Game Modes** and set the `SNW:` rules (described below).
3. Press Play. The world is generated in the first moment of the game, before you unpause. Expect a few seconds of freeze.

| Rule | Options (default first) | What it does |
|---|---|---|
| Shattered New World | **On** / Off | Turns the whole mod on or off. |
| Break Up Realms | **Kingdoms and Empires** / Empires / Duchies, Kingdoms and Empires / Every County | How far the map is broken up: Empires leaves kingdoms as the largest realms, Kingdoms and Empires leaves duchies, Duchies makes every count independent (a count can keep several counties), Every County gives each county its own ruler. |
| Ruler Ages | **Standard** (18–60, mostly 30s–40s) / Young / Mature / Any (child rulers possible) | Age range of generated rulers. |
| Ruler Families | **Large** / Small / None | Large: spouse, children, siblings and a deceased parent. Small: spouse and children. |
| Ruler Culture and Faith | **Same as Realm** / Local | Whether a new ruler takes the culture and faith of the historical ruler they replace, or of their capital county. |
| Related Neighbours | **Some** (15%) / Many (35%) / None | Chance that a new ruler joins the dynasty of a neighbouring new ruler of the same culture. |
| Starting Resources | **Normal** / Rich / None | Gold, prestige and piety by rank. |

Leave the vanilla **Generate Families** rule off. Otherwise vanilla may add a second set of children to the new rulers.

## What changes and what stays

**Replaced or removed**
- Every landholder, from emperors down to barons, mayors and bishops, plus the leaders of mercenary companies and holy orders.
- Every other historical character: courtiers, family members, wanderers. They die off-screen ("vanished") once the new rulers are in place, so no historical dynasty has a living member. The game refills the pool of unlanded characters over time.
- Noble-family titles and landless adventurer camps, since they belong to historical houses and people.
- Titles above the chosen tier. They're destroyed rather than deleted, so they can be created again in play.

**Kept**
- The map, holdings, development, de jure structure, cultures, faiths, and holy sites.
- Faith-head titles such as the Papacy, so religious heads keep working. Clerical regions, holy orders and mercenary companies are also kept, each with a new leader.
- Each realm's government type, plus its crown, tribal or nomadic authority law. Administrative-style governments (Byzantine, Chinese, Japanese, Korean, steppe admin) become feudal, because they depend on an intact empire and noble families.
- Artifacts, which pass to the new ruler of the same realm.
- New rulers get the same game-start setup vanilla gives historical ones: legitimacy, coronation for kings, a regency for child rulers, the nomadic trait for nomads, and mandala government setup.

## How it works

Everything runs once from `on_game_start_after_lobby`, appended without overriding vanilla's own block:

1. `snw_remove_historical_only_titles_effect`: removes noble-family and adventurer titles.
2. `snw_dissolve_high_titles_effect` and `snw_fix_vassalage_effect`: destroy titles above the cap, then make vassals independent where their position is no longer valid.
3. `snw_split_counties_effect` (Every County only): gives each extra county to a new ruler.
4. `snw_replace_all_rulers_effect`: for each historical title holder, creates a new character, transfers their titles, vassals and liege, and copies government and authority. If that holder was a player, control moves to the new ruler.
5. `snw_finalize_ruler_effect`: adds family, 2–4 courtiers, resources and the game-start setup.
6. `snw_remove_historical_characters_effect`: removes everyone without the `snw_generated` flag who no longer holds a title.

Files:
- `common/scripted_effects/snw_setup_effects.txt`: steps 1–4.
- `common/scripted_effects/snw_ruler_effects.txt`: character generation, government and laws, families, courts, cleanup.
- `common/scripted_triggers/snw_triggers.txt` and `common/script_values/snw_values.txt`: helpers.
- `common/game_rules/snw_game_rules.txt` and `localization/english/snw_l_english.yml`: game rules and their text.

## Relationship to Random New World

None of Random New World's code is used. It has no licence, so its code isn't ours to copy, and it was last updated in March 2024. Several of its assumptions no longer hold in 1.20: gender doctrines moved from faith to rite, and administrative, nomadic, mandala and clerical systems were added. The general approach is similar (hook `on_game_start_after_lobby`, destroy high titles, transfer counties with `create_title_and_vassal_change`, generate dead parents so siblings exist), but that approach is also how vanilla's own scripted events do it, and this mod follows vanilla 1.20's code directly.

## Status and troubleshooting

Every effect, trigger and parameter name has been checked against vanilla 1.20.1's script files, and braces are balanced. The first in-game test (1.0.0) started cleanly, and the errors it logged (families, courtiers and artifacts not being created, capital-barony transfer errors) are fixed in 1.0.1. After each test start, check
`logs/error.log` (inside the same `Paradox Interactive/Crusader Kings III` folder as `mod`) for lines mentioning `snw_` and report them.

Known risks to watch on the first runs:
- The start freeze can take several seconds, depending on the bookmark.
- Situations that assume a specific empire, such as China's dynastic cycle or the Great Steppe, may behave oddly once their empire is gone.
- A ruler whose titles couldn't be transferred is kept rather than removed. Search `error.log` for `snw_` to find them.

## Licence

MIT. See [LICENSE](LICENSE).
