# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

## [1.0.0] - 2025-10-XX
### Added
- Initial release of SOAP RPG Framework.
- ScriptableObject-based toolkit for core RPG mechanics.
- MonoBehaviour-based components for easy integration.

#### ScriptableObject Features
- Statistics & Attributes: Define core stats like Strength and Intelligence, as well as derived attributes like Health or Mana.
- StatSets & AttributeSets: Group related stats and attributes together for organization. Sets can be nested for complex structures.
- Experience & Leveling System: Custom experience curves and character progression.
- Classes: Define character classes with unique progressions and abilities.
- Progression System: Control how stats and attributes grow as characters level up.
- Scaling Formulas: Custom formulas to calculate final values of stats and attributes.
- Scaling Components: Modular scaling formulas reusable across stats and attributes.
- Game Events: Design and trigger custom game events.
- Growth Formulas: Determine how values progress based on a character's level.
- Game Event Generators: Define custom game event types.
- Variables: Use Long and Int variables as ScriptableObjects for persistent data storage.

#### MonoBehaviour Features
- EntityCore: Manage a character's core data, including experience and level.
- EntityStats & EntityAttributes: Assign statistics and attributes to any entity. Components can retrieve values from the assigned EntityClass based on level.
- EntityClass: Assign a class to an entity, linking it to specific progression paths.

#### API & Modifiers
- Comprehensive API for advanced customization (see API Docs).
- Dynamic application of modifiers to stats and attributes:
  - Flat & Percentage Modifiers for attributes.
  - Flat, Percentage, & Stat-to-Stat Modifiers for statistics.

