# Samples

The samples folder contains 2 sub-folders:
- **utils**: contains some utility objects that can be used in your game.
- **examples**: contains resources that are used in the sample scene to showcase the package's features

## Utils
The `Utils` folder contains 2 sub-folders:
- `EventInstances`: containing `Entity Attribute Changed`, `Entity Spawned`, `Entity Stat Changed`, and `On Player Level Up` Game Events. These are instances of the Game Events defined in the package's source code.
- `Player`: containing `Max Level`, `Player Hp`, `Player Max Hp`, and `Player Level` Int and Long variables. These are useful to use Scriptable Object variables in your game, as showcased in the [Workflows](workflows.md) documentation.

## Examples
There are several objects in this folder. Let's start by the Sample Scene.
Here we have two relevant game objects: `Heroes` and `HeroesHUD`. These wrappers contain the entity game objects and the GUI elements that are used to display the heroes' stats and attributes respectively.
There are three heroes in the `Heroes` GO: `Warrior`, `Mage`, and `Rogue`. Each hero has its own game object, with the following components:
- `EntityCore`
- `EntityClass`
- `EntityStats`
- `EntityAttributes`

The objects in the `HeroesHUD` are not as relevant. Feel free to explore them, but they are not the focus of this documentation as they are just used to display the heroes' stats and attributes in the UI.

Along with the Sample Scene, there are a few sprites and UI scripts (in the `Sprites` and `Scripts` folders respectively), some prefabs in the `Prefabs` folder, a few events in the `Events` folder, and some more relevant content in the `Instances -> Classes` folder. Here, we have all the instances of the frameworks's scriptable objects for the Sample Scene.  
For each class we have the following:
- Attributes growth formulas
- Stats growth formulas
- An Int var for the hero's level
- A Max HP growth formula
- The class itself
All these objects are used in the Sample Scene to showcase the package's features.

## Using the Sample Scene

The scene uses `Strength`, `Intelligence`, `Dexterity`, and `Constitution` as attributes, and `Physical Attack`, `Magical Attack`, `Critical Chance`, and `Defense` as stats. Each stat scales with the respective attribute:
- `Physical Attack` scales with `Strength`
- `Magical Attack` scales with `Intelligence`
- `Defense` scales with `Constitution`
- `Critical Chance` scales with `Dexterity`

If you now enter play mode in the Sample Scene, you'll see three heroes: the Warrior (left), Mage (center), and Rogue (right), each with their respective HUD displayed above them. You can use the arrow buttons (`<` and `>`) to toggle between viewing the heroes' stats and attributes.

The Sample Scene is designed to demonstrate how stats and attributes respond to changes made in the inspector.

For example, select the `Warrior` in the hierarchy and change its Level from 1 to 2 using the Entity Core component. You'll observe that the Warrior's `Physical Attack` increases from 28 to 29, and `Defense` rises from 26 to 27.

Increasing the level also grants the Warrior a spendable attribute point (visible in the `EntityAttributes` component). If you allocate this point to `Strength`, the `Physical Attack` will increase from 29 to 30.  
If you revert the level back to 1, `Physical Attack` returns to 28 and the spent attribute point is removed.

Next, let's adjust how `Physical Attack` scales with the `Strength` attribute. By default, each point of `Strength` adds 1 point to `Physical Attack`. Suppose you want each point of `Strength` to contribute 3 points instead.  
To do this, locate the `Physical Attack` stat instance in `Examples -> Instances -> Hero -> Stats`. Select the `Physical Attack` stat, then double-click the `Physical Attack ASC` object in the inspector. This Attribute Scaling Component controls how `Physical Attack` scales with `Strength`.  
You'll notice the scaling value for `Strength` is set to `1`. Change this to `3`. Instantly, the Warrior's `Physical Attack` at level 1 jumps from 28 to 60, since the base value is 12 and the Warrior has 16 points of `Strength`: 12 + (16 × 3) = 60.  
If you increase the Warrior's level to 2, `Physical Attack` becomes 61. Assigning the new attribute point to `Strength` further increases `Physical Attack` from 61 to 64.

After exiting play mode, inspect the modified objects. You'll see that the Warrior's level is reset to 1, `Physical Attack` returns to 28, and the `Physical Attack ASC` (ASC stands for Attribute Scaling Component) scaling for `Strength` is back to 1.  
This happens because changes made to `ScriptableObjects` during play mode are not saved to disk when you exit play mode. This is intentional Unity behavior, not a package bug.  
Persisting such changes would be risky. For example, if a debuff reduces the scaling of `Physical Attack` from `Strength` to zero during play mode and this is saved, you would permanently lose the original scaling—even after restarting without the debuff.  
`ScriptableObjects` in this package are intended to define your game's baseline mechanics. Any modifications during play mode are temporary and not meant to be saved. Fine-tuning these values is crucial in RPGs, and once you achieve the desired balance, you want to ensure those values remain intact.

You can also explore the other heroes and all their components in the scene. Play around with their levels, stats scalings, attributes and their spendable points, and see how they affect the heroes' stats and attributes in real-time.