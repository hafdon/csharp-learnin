# Clean Architecture with ASP.NET Core 2.1 | Jason Taylor

https://www.youtube.com/watch?v=_lwCVE_XgqI&t=1s
https://jasontaylor.dev/

- **convention over configuration**[^1]
	- the environment (which could mean the language, or libraries) assumes many situations by default, so you only specify (configure) what is **not** the default
	- 

- core:
	- all logic
- domain
	- enterprise-wide types and logic
- application
	- not shared with other apps
- infrastructure (infrastructure and persistence are both "infrastructure")
	- persistence
		- (here is alternate approach to using data annotations in csharp)
- presentation

infrastructure and presentation both
- depend on Application only
- (thus) don't depend on each other
- either can be replaced with minimal effort

## domain
- entities
	- set collections as private so they can't be initialized by someone else
	- and initialize in constructor or with property injection
- value object
	- as opposed to primitive types
	- custom domain exceptions
- enumerations
- logic
- exceptions
	- (custom domain exceptions)

## application layer
- outermost layer of core
- most business logic here
- work really hard to make sure core will be there in 20 years lol
- everything outside of core is exchangeable

- interfaces
- models
- logic
- commands/queries
- validators
	- fluent validation better than data annotations ![fluent validation](https://i.imgur.com/T6rnZ3s.png)

- exceptions

CQRS
- command query responsibility segreation
- separates reads from writes
- maximize simplicity 

MediatR + CQRS
- pub/sub?
- violates explicit dependencies principle: all dependencies should be explicit
- when use mediatr, there's on dependency: mediatr
- everything is a request (define queries and commands as requests)
- application layer is series of req/res objects
- can attach additional beha ior before/after requests (middleware)

e.g. mediator behavior to validate request using validator

validation has to happen in core; that's a business rule; if we switch out webui for console UI, want validation to still be there • if in asp.net, not as useful

query
- get customer detail query - just dto (info to send to application layer)
	- and query handler (grt customer detail handler)
	- supports dependency injection (e •g • context)

(if you move entities out of core, then create depencies betwen external and internal, and the customerdto desn't contain all details we need (for client) - not a good idea to send entities out , since they are low level; better to send dto's out, cause we can customize them (such as adding whether or not the user can edit something)

### persistence
- dbcontext
- migrations
- configurations
- seeding
- abstractions

- should be implement Unit of Work and Repository Patterns?
	- (with entity framework core 2.1)
	- not always best choice
	- DbContext insulates code from database changes
	- DBContext acts as a "unit of work" ?
	- DBSet acts as a repository
- EF Core (entity framework core) has features for unit testing without repositories

with DbContext, anyone can do what they want
Repositories allow you to control access to the Entities (EF)
- like, wouldn't be able to edit an order detail unless do a whole "update the whole order" thing, if you designed it that way


better to less good
convention
configuration
	(move fluentApi Configuration something api into separate file so you "don't mess up the entity")
data annotation

use an extension to automatically apply all entity type configurations

### infrastructure
- implementation only (abstractions and interfaces contained within CORE)
	- api clients
	- file system
	- email
	- clock
	- anything external
- no layers depend on infrastructure (eg presentation)
- contains classe for accessing external resources (see above)
- implements abstractions/interfaces defined within Application layer

"core dealing with external concerns via dependency inversion" ?inversion
![](https://i.imgur.com/BeSHatW.png)

### Presentation 
- whatever you want

typicaly controller:
![](https://i.imgur.com/5DagfVY.png)

OpenApi


![](https://i.imgur.com/SUycVG9.png)
![](https://i.imgur.com/ghNGkND.png)
![](https://i.imgur.com/RtmPi99.png)
[^1]: https://facilethings.com/blog/en/convention-over-configuration