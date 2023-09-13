# Composing Nix

As a Composable Information Machine, how should we model our Nix configuration files in order to define contexts. Nix already has some modules that do parts of this in various ways such as Profiles and Home-Manager. Part of our problem is going to be the repetition of names such as flake.nix and default.nix.  Naming in a CIM is crucial to define a well organized, understandable system.

Nix and the Nix File Structure is simply a pattern we will need to follow in order to use Nix. CIM is not `dependent` on Nix. We chose Nix specifically because it fits our model. Nix already knows about everything we want to do, the Nix Language is designed for functional programming. We are adding some additional structure to help us organize and work with our system, a CIM, as a whole. Nix is totally replaceable, if you have something else that passes the `Fitness Function` we will be defining.

Instead of configuration files for you to edit, this should be viewed as a testable destination.  I mean instead of manually editing a flake or any .nix file, we should be generating this from Templates, our CIM Model, and running `Acceptance Tests` to verify correctness prior to any deployment.

The Base CIM Model consists of:

    CIM
    Entites
    Components
    Commands
    Queries
    Events
    Domains
    Organizations (aka orgs)
    People
    Transactions

This is not a hierarchy. Organizations are not dependent on Domain, they may be `composed` with Domain to make a new Entity with Domain and Organization components. Also, everything is NOT based on `Object` as the root.

CIM is a root item, meaning everything inside this CIM has a relationship to the CIM. 
It's not hierarchical in the standard sense, what we mean is that for an Organization or a Person to participate in the CIM, it MUST reference the CIM, the CIM is not responsible for referencing or containing the Entity. Inside a Transaction, we have a Root Entity, the item we care about the most in the transaction. There may be many other Entities involved, but the Root is the primarily focused Entity. This is also refered to as an `Aggregate` we chose to use the name `Transaction` because that is more understandable to non-DDD-programmers. 

This is very different from the way we look at OOP and Classes.
You can certainly do this with OOP, but it will probably break every pattern you have ever been taught.

We are using a methodology called `Category Theory` 

> a general theory of mathematical structures and their relations 

It lies underneath everything we do.

You do NOT need to be an expert in Category Theory to use this system, it can actually teach it to you.

CIM is a Category, a "category" here means it is: "A system of related objects. The objects do not live in isolation: there is some notion of map between objects, binding them together."

We will try to spare you all the specific math terms and just use Category Theory.

Really, all you need to know is that for A --> B, A and B are mathematical objects and the arrow is a function transforming A into B.

How we define those objects and arrows is quite arbitrary as long as they follow a few rules. Identity being one, and `Structure Preserving` being another.

## Core Model
Entity
    an object with an ID
Component
    an object which may be composed with an Entity
Command
    make the system do something
Query
    ask a question
Event
    something that happened
Transaction
    a state machine encapsulating composed functionality

Events are preserved in an `Event Store` which is simply an immutable sequence of structured Events. The events refer to everything else.  If you build this correctly, having only the Events and the Structures should be enough to recreate an entire system in any state preserved in an Event Stream.

Transactions allow us to compose things, perform functionality, and produce Events.

Getting Started, should be just defining some core structure and applying events to it. You can actually start with the Events and build the structure from the Events.

The entire CIM is simply a system of related objects where we control the relationships and how to map between objects.

We will formalize all this into what we think of as Types in some language later, what we care about RIGHT NOW, is that we have a structure preserving way of looking at something, we need to be able to store this information and retrieve it in the same form no matter where it may be. We chose to do that with `JSON` and `JSON-LD`. This gives us a formal definition we can use that is not difficult to grasp. We can make decisions later on how we want to formalize our structures into usable programs and language objects.

## CIM
The structure of a CIM is a name and an Identifier. Billions of things can refer to this Identifier to be included in the CIM. How they participate is structured with boundaries.

```json
{"cim":{"name":"", "id": ""}}
```

We are going to make a decision here: What is the ID?

We want something unique and not based on the `Name`.
We could potentially want to change the name, so the ID serves as the anchor for the Entity we refer to as a CIM. In our case we will simply use a `uuid`.

You could also use a gpg key pair with the Public Key being the ID and the Private Key being a way to validate it. We use a uuid because the ID is in fact public, not something we need to secure. We do however need to be able to validate it is ours, and we are in control of it.

For this we will be using a `DID` which is a `decentralized identifier` for which there is an [existing standard](https://www.w3.org/TR/did-core/), and is being widely adopted.

This is also something easily generated when we start a new CIM.

We can do it manually, but we really want some templating to help us build out all this structure. the `cim` command line tool will do exactly this.

## Domain

A Domain is simply an Entity, a `set of data` along with an Identity.
How we preserve that is up to us. We will be preserving Content-Addressed Information, so IDs are set by the system through IPLD. These are indeed `natural keys` when you create a domain, you can't change it's name. Doing so would cast a long shadow on your system and cause numerous problems. Instead, a CIM can have multiple participating domains and has the ability to transition items between them.

```json
{"domain":{"name":"example.com"}}

```

## Organizations
```json
{
    "orgs": [
        {"org":{"name":"Example Company, LLC.", "id": "b29bbd85-19a8-4ebb-8b0e-030e09da1e1d"}}
        {"org":{"name":"employees", "id": "a16b8cd7-c3b1-4bb8-8a62-959fd1098e5b"}}
        {"org":{"name":"vendors", "id": "94fa70c7-f7a6-4d7b-ba76-bfd34f05d079"}}
    ]
}

```

Naming is critial, these become critical parts of our system, but the IDs are what MUST be unique. Each Org structure is saved to IPLD and is given a CID. If the Name changes we get a new CID, but the org.id is consistent.  This allows us to transition from CID #1 to CID #2 in a sequence of events, preserving the uuid inside.

This is analogous to changing a `Primary Key` in a relational database.

## People
People are essentially the same, we have a name and ID, the name can change, but there is an Event Stream preserving how and when that happened.

```json
{"person":{"name":"John Doe", "id":"81f1075d-b4fb-4cdb-97f4-74e4aa250211"}}

```

Again, this is our base preservation for an Entity. There are no Components attached yet.
The uuid is simply to define uniqueness and has no associated meaning.

The CID is what we will use for linking, this ID simply makes:
```json
{"person":{"name":"John Doe", "id":"81f1075d-b4fb-4cdb-97f4-74e4aa250211"}}
### AND ###
{"person":{"name":"John Doe", "id":"5c896d64-ee14-4971-b195-b0636d64bc43"}}
```
unique CIDs.  We need to distinguish which "John Doe" we are referring to. 
Person names and other names in the system are not necessarily unique, but their `id` is.
So now we have two people with the same name, but still are unique.  A Query for "person where name equals `John Doe`" will still return 2 objects and the id helps to distinguish them at this level.

### Entity
An Entity is a name and id as well.

Entities are built from Components and each Entity with the same components makes a Type.

### Component
A Component is simply a set of key/value pairs that are invariant.  That means a group of key value pairs that are grouped together and have rules.

This is how we would preserve something like a Street Address:
```json
{"address":{"street":"123 Main St.","locality":"Denver","region":"CO","code":"80208","country":"USA"}}

```

Note that we DO NOT include IDs in the Component as those are not needed with a Component.
A Component can be changed at will, but it is changed all at once. It MUST refer to an Entity to have any contextual meaning outside of itself.

If I try to add 2 of these exact same things, it doesn't fail the second, it simply refers to what we already have, the same CID.

Two copies of the same thing here are really meaningless, I am pointing to the same thing and two copies would be wasteful, instead we simply refer to the CID for the Component.

## Transactions
Transactions are how we combined things and perform `State Transitions`.
This means a transaction is a `State Machine` and we are interested in how the state moves from one state to the other. Each state has distinct meaning to the transaction and Events are thrown on state changes.

