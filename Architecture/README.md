# Architecture

* [AppShell](AppShell.md)
* [Modularity](Modularity.md)
  * [Dependency injection](DependencyInjection.md)
* [Interact workflow](InteractWorkflow.md)
  * [Actions](Actions.md)
* [Channels and Notifications](ChannelsAndNotifications.md)
* [Settings and Configuration](SettingsAndConfiguration.md)
* [Error handling](ErrorHandling.md)
* [Launcher and user interaction](LauncherAndInteractive.md)

## General

![mue_general](mue_general.drawio.png)

Each rectangle is a module in the system.
Modules interact with each other through interfaces using dependency injection, actions and notifications.
