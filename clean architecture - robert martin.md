# clean architecture - robert martin

terms
- business requirements = use cases = actions a user might take (delete a blog post, calculate their taxes)
- policy
	- "Software systems are statements of policy. Indeed, at its core, that’s all a computer program actually is. A computer program is a detailed description of the policy by which inputs are transformed into outputs."
		- In most nontrivial systems, that policy can be broken down into many different smaller statements of policy. Some of those statements will describe how particular business rules are to be calculated. Others will describe how certain reports are to be formatted. Still others will describe how input data are to be validated.

## component principles

### common reuse principle (crp)
*don't depend on things you don't need*
- Don' force users of a component to depend on things they don't need
- classes and modules that tend to be reused together belong in the same component
- ie • it should be impossible to depend on some, but not all, of the classes in a component
- this is the generic version of [[ISP]]

### tension diagram

- inclusive principles (make components larger)
	- REP
	- CCP (more important early on, because developability is more important than resuse)
- exclusive principles (make components smaller)
	- CRP 
- resolving this tension is  necessary
- the balance is dynamic (probably will change over time)

![](https://i.imgur.com/px6G5U2.png)

## component coupling
*relationship between components*

### acyclical dependencies principle (ADP)
*allow now cycles in the component dependecy graph*
- "morning after syndrome": happens when many devs are modifying same source file
>  It is not uncommon for weeks to go by without the team being able to build a stable version of the project.

#### solutions
- the weekly build
- acyclic dependencies principle (ADP)

##### weekly build
- everyone ignores each other for four days
- on friday they integrate and build
- large integration penalty on friday
- to maintain efficiency, build schedule has to be lengthened 

##### eliminating dependency cycles
- solution to above:
	- partition dev environment into releasable components
		- component are units of work
		- responsibility of single dev or dev team
		- release components sa they are working, for use by other devs
		- give release number
		- concerns:
			- you have to **manage the dependency structure of the components**
				- Directed Acyclic Graph (DAG)
			- aka the structure of the dependencies has to be a :
				- directed graph (dependencies point in one way only?)
				- acyclic: impossible to end up in place you started
					- a "cycle" means you could follow dependency arrows in a circle, and is bad
					- (removing the arrow from Entities to Authorizer makes it a DAG and eliminates the cycle)
					- ![](https://i.imgur.com/Qe1C2pY.png)
	- how to "break a cycle"
	- 1 • Apply Dependency Inverstion Principle (DIP)
		- create an interface between (in the above) Entities and Authorizer, called eg `IPermissions`
			- this is put into Entities, and inherited into Authorizer (that means, Authorizer implements it, and exports it for Entities to import) (?)
	- 2 • creat a new component that Entities and Authorizer depend on (that is, move the classes they both depend on into that new component)
		- **Jitters** employing #2 implies that you're always having to watch out for cycles, then creating new components, watching for new cycles, ad infinitum

### Top-down design
You might think that the structure made visible when you graph component dependencies would correspond to a graph of (high-level) functional analysis[^1]
*component dependency diagrams have very little do to with describing the function of the application. Instead, they are a map to the buildability and maintainability of the application*
[^1]: analysis in the sense of " the process of separating something into its constituent elements"

**isolation of volatility**
- don't want frequently changing components to affect stable components 

### stable dependencies principle (SDP)
*depend in the direction of stability*

CCP also implies that some components are designed to be volatile, and are expected to change
- it just means that volatile components shouldn't be depended on by stable component
- by conforming to SDP, you (try) to prevent situations (common) where a stable (difficult to change) component  is dependent on a volatile module

**stability**
- doesn't have anything directly to do with *frequency of change*
	- related to "the amount of work required to make a change" ("and still keep things working")
	- factors making it hard to change
		- size
		- complexity
		- clarity
		- make a lot of other components depend on it
			- x is "responsible to" the three components dependent on it
			- x is "independent" since it doesn't depend on anything itself
			- ![](https://i.imgur.com/6OgtZSh.png)
	- we see the opposite, unstable, with Y ("irresponsible" - nothing depends on it, and dependent (depends on 3))
		- ![](https://i.imgur.com/t0W83vp.png)
- **stability metrics**
	- instability = # what it depends on / (all incoming or outgoing dependencies)
		- so, most stable = 0 (depends on nothing)
		- least is 1 (nothing depends on it)

as seen above, creating components that are nothing but an interface is necessary with statically typed languages 

### stable abstractions principle (SAP)
and **dependency management metrics**

- a component should be as abstract as it is stable
	- the software that encapsulates high-level policies of the system  (ie • business and architectural decisions) should be in most stable components 
	- problem: but then these components are hard to change
	- how can component that is maximally stable be flexible enough to change?
	- answer: OCP (extend without requiring modification)
		- aka Abstract classes
- relationship b/t stability and abstractness
	- stable component should be abstract so stability doesn't prevent it from being extended (as it would if it were concrete?) - basically, make it an interface / abstract classes
	- unstable component should be concrete - easily change concrete code

 
The SAP and the SDP combined amount to the DIP for components. This is true because the SDP says that dependencies should run in the direction of stability, and the SAP says that stability implies abstraction. **Thus dependencies run in the direction of abstraction.**

but, unlike classes, components can be varying degrees of abstract / concrete

**abstraction metrics**
Nc - # classes in component
Na - number abstract classes / interfaces
A - abstractness = Na / Nc
(ie • =  abstractions / total)

assuming 3rd axis, "volatility" to be maximal, 1
![](https://i.imgur.com/Xr8pz51.png)
- zone 0: stable and concrete
- rigid
- can't be extended since not abstract
- difficult to change since stable
- examples: 
	- database schema (notoriously volatile - so how are they stable?)
	- nonvolatile: concrete utility library, eg • the String component (in Java): concrete, but depended on by so many things that it's not going to get changed
- so, only volatile components problematic in zone of pain (but by definition they wouldn't be in here , since they aren't volatile... ? )
- zone 1
	- maximally abstract with no dependents
	- useless
	- leftover abstract classes that no one implemented

lessons: most volatilie components should be as far from exclusion zones as possible

unnecessary D metric, since all this stuff is highly subjective anyway at some point

## architecture
 
 ### what is architecture
some thoughts
- A good software architecture  makes the operation of the system readily apparent to the developers
	-  The architecture of the system should elevate the use cases, the features, and the required behaviors of the system to first-class entities that are visible landmarks for the developers.

- cost of maintenance
	- **spelunking**: digging through existing software, finding best place/strategy to add new feature or repair defect
	- risk

The goal of the architect is to create a shape for the system that recognizes policy as the most essential element of the system while making the details irrelevant to that policy. This allows decisions about those details to be delayed and deferred.
- i still don't really know what "policy" means

A good architect maximizes the number of decisions not made.

Conway’s law says: Any organization that designs a system will produce a design whose structure is a copy of the organization’s communication structure.

### independence
[ • • •]

#### decoupling layers
 - we find the system divided into decoupled horizontal layers—
	 - the UI
	 - application-specific business rules
	 - application-independent business rules
	 - and the database

#### decoupling use cases
We keep the use cases separate down the vertical height of the system.
- *what are use cases?*: the (specific) reason someone is using the system at a particular time, like adding an order or deleting a blog post
- use cases are "narrow vertical slices that cut through the horizontal layers", since each one uses some part of the whole code base
- omg what: "we separate the UI of the add-order use case from the UI of the delete-order use case" - this is an insane amount of decoupling

#### decoupling mode
- decoupling for use cases also helps with operations
- but to be helpful, decoupling must have appropriate "mode"

#### independent develop-ability
#### independent deployability
#### duplication
there are different kinds of duplication
- **true duplication**: every change to one instance necessitates the same change to every duplicate of that instance
- **false / accidental duplication**: if two apparently duplicated sections of code evolve along different paths, then they are not ntrue duplicates

#### decoupling modes (again)
his pref: push decoupling to the point where a service *could* be formed ??

A good architecture will allow a system to be born as a monolith, deployed in a single file, but then to grow into a set of independently deployable units, and then all the way to independent services and/or micro-services. Later, as things change, it should allow for reversing that progression and sliding all the way back down into a monolith.

### boundaries: drawing lines
enemy: coupling *(Recall that the goal of an architect is to minimize the human resources required to build and maintain the required system. What it is that saps this kind of people-power? Coupling—and especially coupling to premature decisions.*)

a decision is "premature" if it has nothing to do with business requirements (use cases) of the system, such as for: (should be able to be made at the latest possible moment, without significant impact)
- frameworks
- databases
- web servers
- utility libraries
- dependency injection

All the business rules need to know is that there is a set of functions that can be used to fetch or save data. This allows us to put the database behind an interface.
![](https://i.imgur.com/1sf5GC6.png)
![](https://i.imgur.com/l5fIMaO.png)

this is the persistence / Core division :
![](https://i.imgur.com/rsGZQ3a.png)

**plugin architecture**
![](https://i.imgur.com/aPQ8hU8.png)

term: axis of change (173)

### boundary anatomy
at runtime: boundary anatomy is function '"on one side" calling function "on the other side" and passing data'

### policy and level

terms:
- **level**: "the distance from the inputs and outputs"
	- high = further from inputs and outputs
	- lowest = manage inputs and outputs

We want **source code dependencies to be coupled to level.** (not coupled to data flow)

### business rules
- defs: 
	- **critical business rules (CBR)**:  rules or procedures that make or save the business money (crucial to business, and would exist even without system to automate)
		- **critical business data (CBD)**: the (usually necessary) data that the rules work with
	- **entity**: object that embodies a small set of CBR operating on CBD
		- entity either has easy access to CBD or contains it
			- entity interface is functions that implement CBR
