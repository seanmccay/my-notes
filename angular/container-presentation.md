[Back](/my-notes/angular/overview)

# Container / Presentation Component Structure

This is a pattern for structuring components that lends itself to easier testing and consistency of state

*Also a good structure to use with NgRx*

You have a smart/container component that handles all state and state changes, within that container you have presentational components which are only concerned with presenting state / rendering stuff and firing events.

- Containers
	- Get and manage state and handle events
	- Good idea to have these as the comp served for a route
- Presentation
	- Just have @Input and @Output
	- A good place to use ChangeDetectionStrategy.OnPush to improve performance
		- Will make it so change detection only fires when input or output changes
		- Writing immutable code will make sure you don't screw this up
		- Setting this will also help keep devs from writing code that doesn't stick to the container/presentation paradigm as well because state changes in these components won't be reflected in the UI