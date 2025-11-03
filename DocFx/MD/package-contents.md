# Package Contents

<!--
- Samples folder.
- Events generators outside of the samples folder.
-->

The relevant content for you consists of the package samples and the "utility events" contained in the package folder (located in `/Packages`).

For more information about the samples, see the [Samples](samples.md) documentation.

The "utility events" (Game Event Generators, source code of the generated events, and instances of the Game Events) have been intentionally placed outside the samples folder because they are used directly in the package's source code. Therefore, it would not be correct to place them in the samples folder, as omitting them during package import would cause the source code to not function correctly.

The Game Event Generators are located in the `Packages/AstraRPGFramework/Runtime/Events/EventGeneratorInstances` folder. Here you will find:
- `GeneralPurposeEventGenerator`: defines two game events: `EntityCoreGameEvent` and `IntGameEvent`.
- `AttributesEventGenerator`: defines `AttributeChangedGameEvent`
- `StatEventsGenerator`: defines `StatChangedGameEvent`

> [!WARNING]
> Do not modify these Game Event Generators, as they are an integral part of the package's source code.  
>If you want to add new events, create new Game Event Generators in your Assets and use those to define your custom events.

The source code for the events defined within the aforementioned Game Event Generators is located in the `Packages/AstraRPGFramework/Runtime/Events/GeneratedEvents` folder. The organization follows the rules mentioned in the [Adding new events](workflows.md#adding-new-events) documentation.

The instances of the generated Game Events are provided with the samples of the package instead. Such objects are also used by the Sample Scene (always available in the samples folder). If you want to instantiate new Game Events of type `EntityCoreGameEvent`, `IntGameEvent`, `AttributeChangedGameEvent`, or `StatChangedGameEvent`, feel free to do so.