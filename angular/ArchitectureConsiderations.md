# Architecture Considerations
**You should do what the Angular Style guide says** unless you have a good reason not to

## How should the project be organized?
Modules:
 - Core
	- Will contain shared singleton services
		- Don't inject the same service into multiple modules because each service will inject a new instance and this will lead to issues with state, etc.
		- A good example is a service that handles auth with the current user
	- App level components 
		- nav/sidebar/etc
 - Shared
	- Will be imported into all feature modules
	- Good idea to import/export CommonModule so other modules don't have to
	- Import things like ReactiveFormsModule, etc that will be used frequently across features
	- shared components like spinners, custom buttons, etc
	- shared directives and pipes
	- I think this is a good spot for guards since they will be used across feature modules in their routing modules
 - Feature Modules
	- All feature level services, components, directives, pipes
	- Will have its own routing module to enable ***lazy loading***
	- A feature module might correspond to a base route in the app like '/supplementalData', '/members', '/admin'

## Considerations during planning

- App Overview
	- What's is for?
	- How will it be used?
	- What are the business benefits?

- App Features
	- Features for different levels of users
	- Widget
	- WidgetDetails
	- UserMgmt
	- Login/Logout

- Domain Security
	- What security rules/tech are you tied to?
		- OAuth/OIDC via a third party ID Store
		- AspNet Identity in DB
		- Http Interceptors on client to add auth headers to requests

- Domain Rules
	- Business rules for the app
		- A member who has x should be flagged as whatever, etc
	- Form validation rules, route guards

- Logging
	- Logging services in client
	- Store in DB?
	- Send errors to external service like Sentry? Azure AppInsights?

- Client to Server communications
	- Will you need just HTTP? WebSockets?

- Data Models
	- What data are you receiving from APIs?
	- What objects are you passing between components?

- Feature Modules
	- How will you bust up your app functionality into separate feature modules?

- Shared Components
	- Do you have any components that can be shared all over your app? 

- Some Others
	- Accessibility?
	- I18n / internationalization?
	- CI/CD?
	- Will you use a CDN?
	- Containerization?
	- Unit testing
	- Backend APIs

**You should do what the Angular Style guide says** unless you have a good reason not to