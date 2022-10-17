---
title: "Unreal Cheatsheet"
layout: cheatsheet
categories: "cheatsheets"
permalink: /:categories/:title
---

# Classes
Actor: base class for classes present in scene.
ActorComponent: base class for components attached to Actors.
CheatManager: class for gameplay debugging functionalities, cheats. Instanciated in Debug and Development builds. Referenced by Controller. Lifetime: TODO
DataAsset: base class for anything managed by the asset manager that isn't a Blueprint.
GameInstance: class persistent across levels. Single instance throughout the whole runtime of the application. Lifetime: TODO
Controller: class receiving user inputs and containing pawn controlling logic. Consumes inputs by default. Set the default one in your GameModeBase deriving class. Not replicated for networking. References CheatManager. Transform not attached to Pawn by default. Lifetime: TODO
Pawn: class "posessed" by a controller. Set the default one in your GameModeBase deriving class. Lifetime: TODO
PlayerCameraManager: class for overriding default camera behaviour (non springarm + camera setup). Associated with a Controller. Lifetime: TODO
HUD: don't. Use UMG instead.
GameMode: don't. Use GameModeBase instead.
GameModeBase: class defining what managers to use. Only one is used at a time. Set up the default one in project settings. Does not receive inputs. Lifetime: TODO
GameState: class for managing state of the world but NOT specific to any player. Keep things like time of day in there. Lifetime: TODO
PlayerState: class for holding player specific state. Use it for multiplayer projects since Controllers aren't replicated over the network.


# Blueprint Facilities
- Interface: ordinary interface.
- Function: ordinary function.
- Custom events: don't bother. Same as function.
- Event dispatchers: grouping for custom events.
- Set: array of unique elements.
- Map: key-value pairs, one value per key.

# User Input Processing Stack
1. Input-enabled actors
2. PlayerControllers
3. LevelBlueprint
4. Pawns
Check "Block Input" to consume input on the InputComponent. Note "Consume Input" refers to consumption of Event arguments, not user inputs.