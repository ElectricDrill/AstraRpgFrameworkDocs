# Limitations

<!--
- OnStatChangedEvent is risen only when modifiers are applied, not on level-ups/base stat changes
- OnAttributeChangedEvent is risen only when modifiers are applied and spendable points are assigned, not on level-ups/base attribute changes
- no support for multi-class per entity
- only one event for stats changed, attributes changed, entity spawned can be assigned to entities
-->

- `OnStatChangedEvent` is risen only when modifiers are applied, not on level-ups/base stat changes
- `OnAttributeChangedEvent` is risen only when modifiers are applied and spendable points are assigned, not on level-ups/base attribute changes
- Currently, multi-classing is not supported. Each entity can only have one class at a time
- Stat modifiers are applied in the following order:
  1. Flat modifiers
  2. Stat-to-stat modifiers
  3. Percentage modifiers
- Stat-to-stat modifiers use the base value and the flat modifiers for the source stat for the calculations. This ensures that the modifiers are applied consistently and predictably, and that circular dependencies can be defined without issues.
- Attribute modifiers do not have a attribute-to-attribute modifier system. This was a design choice to keep the system simpler.
- Attribute modifiers are applied in the following order:
  1. Flat modifiers
  3. Percentage modifiers
- Only one event for stats changed, attributes changed, and entity spawned can be assigned to entities. However, multiple event listeners can be assigned for the same event type, allowing for multiple responses
- Custom game event generators support up to four context parameters. To work around this limitation, users can combine multiple pieces of data into a single user-defined class—a technique known as "parameter packing." This approach is extensible through inheritance and helps maintain a simple, user-friendly API by avoiding telescoping parameters.

