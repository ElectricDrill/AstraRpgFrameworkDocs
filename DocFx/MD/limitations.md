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
- Only one event for stats changed, attributes changed, and entity spawned can be assigned to entities. However, multiple event listeners can be assigned for the same event type, allowing for multiple responses

