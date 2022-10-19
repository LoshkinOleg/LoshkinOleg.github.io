---
title: "Unreal Cheatsheet"
layout: cheatsheet
categories: "cheatsheets"
permalink: /:categories/:title
---

# Classes
Actor: base class for classes present in scene. Lifetime: level.
ActorComponent: base class for components attached to Actors. Lifetime: level.
CheatManager: class for gameplay debugging functionalities, cheats. Instanciated in Debug and Development builds. Referenced by Controller. Lifetime: level.
DataAsset: base class for anything managed by the asset manager that isn't a Blueprint.
GameInstance: class persistent across levels. Single instance throughout the whole runtime of the application. Lifetime: runtime.
Controller: class receiving user inputs and containing pawn controlling logic. Consumes inputs by default. Set the default one in your GameModeBase deriving class. Not replicated for networking. References CheatManager. Transform not attached to Pawn by default. Lifetime: level.
Pawn: class "posessed" by a controller. Set the default one in your GameModeBase deriving class. Lifetime: level.
PlayerCameraManager: class for overriding default camera behaviour (non springarm + camera setup). Associated with a Controller. Lifetime: level.
HUD: don't. Use UMG instead.
GameMode: don't. Use GameModeBase instead.
GameModeBase: class defining what managers to use. Only one is used at a time. Set up the default one in project settings. Does not receive inputs. Lifetime: level.
GameState: class for managing state of the world but NOT specific to any player. Keep things like time of day in there. Lifetime: level.
PlayerState: class for holding player specific state. Use it for multiplayer projects since Controllers aren't replicated over the network. Lifetime: level.

# Blueprint Facilities
- Interface: ordinary interface.
- Function: ordinary function.
- Custom events: don't bother. Same as function.
- Event dispatchers: grouping for custom events.
- Set: array of unique elements.
- Map: key-value pairs, one value per key.

# User Input Processing Stack
1. PlayerControllers
3. Pawns
Check "Block Input" to consume input on the InputComponent. Note "Consume Input" refers to consumption of Event arguments, not user inputs.

# Initialization / Destruction Order
1. GameInstance: Init

2. CheatManager: InitCheatManager

3. Pawn: Possessed
4. PlayerController: OnPossess

5. GameModeBase: BeginPlay
6. LevelBlueprint: BeginPlay
7. GameState: BeginPlay
8. PlayerController: BeginPlay
9. PlayerState: BeginPlay
10. PlayerCameraManager: BeginPlay
11. Pawn: BeginPlay

12. Game runs...

13. Pawn: Unpossessed
14. PlayerController: OnUnPossess

15. PlayerCameraManager: EndPlay
16. PlayerState: EndPlay
17. PlayerController: EndPlay
18. GameModeBase: EndPlay
19. LevelBlueprint: EndPlay
20. GameState: EndPlay
21. Pawn: EndPlay

22. GameInstance: Shutdown

2 though 21 is called for every level. 1 and 22 is only called once per runtime.
Note: Shutdown is never called on CheatManager for god knows what reason.