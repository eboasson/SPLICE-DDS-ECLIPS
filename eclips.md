This is part three of three. The other parts are the [early history](splice.html) and the [route to standardization](dds.html).

# ECLIPS: returning to the architectural question

This year I have returned to a successor of SPLICE named ECLIPS. The ideas began well before the later DDS developments described in the preceding instalment. In the late 1990s I was thinking about the limitations of a flat data space, the awkwardness of generic durability, the confinement of mutually distrustful subsystems, the removal of keyed objects and the joining of systems whose states had diverged. These were related doubts and possibilities, not yet a design.

Other work took precedence for much of the following decade. With some time in 2008 and 2009, I finally managed to fit the pieces together. In May 2010 I submitted the long paper, [*ECLIPS, a distributed data space with induced structure and capability-based protection*](eclips-20100525.pdf), to *ACM Transactions on Computer Systems*. A much shorter version appeared later that year at the IASTED conference on Parallel and Distributed Computing and Systems in Marina del Rey.

The long paper was explicit: “ECLIPS currently is but a thought experiment.” There was no implementation, performance study or formal proof. No wonder it got rejected! It argued that formalisation should precede implementation because it might reveal a fundamental error before considerable effort had been spent, while implementation would still be required to investigate performance. In particular, testing alone could not establish the security claims.

The assumed environment was deliberately bounded: a local-area network of modest size, perhaps a few hundred nodes on Gigabit Ethernet, with a known upper bound on network latency. That was unfashionable even then, amid enthusiasm for clouds and wide-area systems, but it remained a natural setting for operational systems whose requirements concerned availability, bounded resources and correct behaviour. The design did not claim to work unchanged across millions of intermittently connected devices.

My father patiently listened while the ideas were still nebulous and scrutinised several drafts. ECLIPS was my design, subjected to well-informed criticism by the inventor of SPLICE. Its ancestry is therefore personal as well as technical, but it was not simply a further SPLICE implementation. I wanted structure, retention, protection and deletion to acquire meanings within the data model itself. The resulting interface was smaller than DDS’s; the obligations beneath it were considerably more ambitious.

This year, I realised that I might be able to vibe-code my way to a proof-of-concept implementation. It remains exploratory, but it is starting to look as though ECLIPS is at least implementable. Sixteen years after writing the design down, that is an interesting place to be.

This then provides the second occasion for this history. Forty years after those early SPLICE presentations, I am again thinking about the question from which it all began: how should a distributed system be structured so that useful properties follow from its architecture? SPLICE answered that question with a selectively replicated shared data space. DDS cared little for the question and chose to deal with shuffling data around. ECLIPS asks what the data space itself must represent if it is to carry more of the architectural argument.

The questions become concrete as soon as ordinary operation is interrupted. A producing process disappears: where does its state survive? An object is deleted: what happens to an old reference to it? Two disconnected systems continue operating and later meet: which of their decisions and identities can be reconciled? A process claims authority: who else must agree before it acts? These are related problems of system construction, although middleware commonly presents their supporting mechanisms separately.

ECLIPS attempts to bring them into one model. Whether that model is sound and economical remains to be established. The present experiment makes the attempt tangible; it does not supply the conclusion.

## Another direction: Zenoh

Zenoh provides a useful contemporary contrast that originates in DDS. It brings live publication and subscription, stored values and queries to application services under a common model of names and matching. Key expressions select the resources of interest, and the routing system connects the relevant publishers, subscribers and queryables. The same naming model can span constrained devices, edge systems and cloud services across heterogeneous network topologies.

This includes access to state, not just the next publication. A Zenoh `get` can obtain a reply from storage or from a queryable that computes an answer. The operation reaches out to providers of information. An ECLIPS read, by comparison, examines a local store whose contents are maintained according to the rules of the data space. Both can give an application access to existing information, but the obligations behind that access are different. A shared name through which information can be requested is not quite the same as a relationship among replicas that must have received particular states.

I see the broader contrast as a choice about how much the participants must agree upon. Zenoh makes a common naming, matching and routing model useful across a wide range of participants, without saying much about the semantics. ECLIPS specifies more of the relationships among retained states, object lifecycle and authority, but in a much narrower assumed environment. A small common contract can make participation easier, a stronger contract can support more deductions about what a participating system means. Neither choice removes the questions outside its contract, and this comparison says little about the size or sophistication of the machinery a modern implementation needs.

I was involved in Zenoh’s early development: Angelo Corsaro initiated the work in 2016, we refined the early designs together and I implemented an early version (“Zenoh-He” or just “Zhe”). Subsequent development was done by colleagues in our French office led by Angelo, while I concentrated on Cyclone DDS and eventually withdrew from the Zenoh work. I recognise the attraction of its direction as well as that of ECLIPS. The comparison is between architectural choices, not a claim that one design descended from the other or ought to replace it.

## A graph of information flows

The familiar starting point in ECLIPS is typed, keyed data. An instance describes the state of the object selected by its key, and a newer value may replace an older state of that object in a local store. Applications query these stores. Replication is selective and asynchronous, with eventual rather than transactional consistency.

The difference is that the structure controlling replication becomes an explicit part of the application model. A process publishes at a *nabla* and reads from a *delta*. Nablas and deltas are vertices in a dynamic directed graph, alongside neutral vertices that neither publish nor store data. Each nabla and delta belongs to one data *sort*, principally a type combined with a key. A value published at a nabla can contribute only to deltas for that sort that are reachable through the graph.

This graph describes permitted information flows, not cables, switches or hosts. Moving a process to another computer need not mean changing the architectural boundary through which its information passes. Conversely, two processes on the same computer need not share access to the same state. The graph expresses a relationship between publications and stores that is distinct from the machinery carrying updates between them.

DDS domains and partitions divide a largely flat communication space by names. ECLIPS can reproduce such divisions: a neutral vertex can represent a partition, with publications connected towards it and subscriptions away from it. It can also represent gateways, confined subsystems and selective routes. Instead of inferring these relationships from conventions for naming topics and partitions, an application can make them part of the space’s structure.

A process asks a *herald* to perform operations on its behalf. The herald maintains its local stores and the internal state required by the model, exchanging updates with other heralds. There is no single place containing the shared data space. The abstraction arises from those stores, their update rules and the graph connecting them.

Nablas and deltas resemble publications and subscriptions, but identifying them with messaging endpoints too quickly loses the point—hence why I gave them such abstract names. A delta has stored state, a nabla can update the state of a keyed object, and changing the graph can change what a store is obliged to know. Delivering the next message is only one part of maintaining that model.

## Where memory belongs

Consider a restarted process that needs to recover the current state of the system. If that state survives only in the histories of its original producers, recovery depends on those producers still existing and being able to provide it. If other stores retain it independently, the process may recover even though the producers have gone. The distinction between DDS’s transient-local and transient durability, and SPLICE’s context data before DDS, concerns this allocation of responsibility.

ECLIPS has no primitive durability category. Whether earlier data must be supplied to a newly attached delta follows from its position in the graph and from the state already held in other deltas. Retention becomes a property of the structure and the applications operating its stores.

For a delta *w*, the model defines a *context*: a particular set of upstream deltas for the same sort. Informally, these are the nearest relevant stores from which *w* can be reached. The exact construction first contracts the graph’s strongly connected components and then considers paths that pass through no other delta for that sort. Deltas within one such component are equivalent for this purpose, providing a natural unit of redundant storage without selecting one distinguished repository.

The associated invariant matters more than the terminology. Subject to propagation delay and topology changes, the store at *w* must contain, or must have contained, data at least as recent as the corresponding data held by every delta in its context. The qualification “or must have contained” matters: this is an obligation concerning the state supplied to a store, not a demand that every value remain in that store forever. Adding a context source may create an obligation to copy retained state, even when no producer is publishing anything new.

This makes both the location and policy of memory application-programmable. A process operating deltas can combine information from several sorts, discard a track that has ceased to be meaningful, or retain one whose publisher has disappeared. A generic service cannot decide all those questions merely by knowing the age and key of a sample. In ECLIPS, such a process participates in the retention structure through the same model as the applications using its results.

I had already explored application-controlled retention in SPLICE-lite, in the demonstration with cars on a Möbius strip. The ECLIPS paper’s appendix also shows the converse: SPLICE context data can be constructed on top of this model, using context-daemon processes and strongly connected storage around vertices representing SPLICE partitions. The older facility becomes one possible arrangement of the more general structure.

The cost is substantial. Adding or removing an edge can merge or split a strongly connected component and thereby change contexts far from the edge itself. New obligations can require data movement. Maintaining reachability in a changing directed graph is expensive in the general case. ECLIPS makes the allocation of memory explicit and derivable, but that is a semantic gain whose implementation cost still has to be paid.

## What a later state means

Retaining a value is useful only if the system can determine what that value represents and which later states may replace it. DDS provides types, keys and instance lifecycle, but defines no referential semantics between topics. Disposing of an instance need not supply a final valid value. A later write can make the same key alive again.

That behaviour may be appropriate for some data. It is troublesome if another object still refers to the key and silently begins referring to a different object. Reader-local generation counts can distinguish episodes in that reader’s history, but they are not a stable identity carried by references between topics. Applications with consequential deletion must supply the missing meaning themselves.

An ECLIPS sort consequently describes more than a representation and key. It supplies predicates for validity and obsolescence, an application-defined partial order on successive states, a minimum retention time, and optionally immutability or controlled identity. The implementation extends the partial order to a total one while preserving publication order at a single nabla. A later state can thus be defined using the meaning of the data, rather than only its arrival time or a middleware timestamp.

This does not permit the middleware to discover the meaning on its own. The application supplies the predicates and ordering. Their inclusion in the model makes them part of the contract under which replicas replace values, rather than a separate interpretation that every reader must reconstruct after receiving them.

Obsolescence is especially important. An obsolete value is a final state of the keyed object. It may supersede a current value, but no subsequent current value may supersede it. `local-take` may remove this final value from one local store; the herald retains enough administrative state for a prescribed interval to reject late or reordered updates. Removing something from local storage and declaring that object finished are therefore different acts.

There is a deliberate distinction between ordinary, *regular* sorts and *controlled* sorts. For a regular sort the retained administration amounts to a timed blacklist. Once all such state has expired, attempted resurrection need not produce identical results at every delta. Controlled sorts use explicitly allocated, once-only identifiers, allowing an obsolete controlled object to remain permanently excluded. In the paper’s terminology, regular sorts blacklist obsolete objects, while controlled sorts whitelist current ones.

The stronger finality therefore does not follow merely from attaching an “obsolete” flag to arbitrary keys. It depends on the identity discipline of controlled objects. This is one of the places where the design tries to make an architectural choice visible: a system that needs permanent exclusion must use the model that provides it and accept the associated administration.

## Structure induced by contents

The graph is itself represented in the data space. ECLIPS has no separate administrative interface for creating sorts, vertices and endpoints. Objects in predefined sorts describe those entities. Publishing an edge object induces the corresponding graph edge; publishing descriptions of a nabla, delta or neutral vertex induces those entities. Global deletion, or eventual disappearance from all local stores, changes the operational graph.

That is the *induced structure* in the paper’s title. Configuration participates in the same distribution and protection rules as application data. A partitioning scheme, an access-control manager or a SPLICE-style context service can be constructed from objects and the ordinary operations, rather than introduced as another privileged facility beside them.

It is a useful reversal because the configuration of a distributed system is itself distributed state. Processes may disagree temporarily about a new connection just as they may disagree temporarily about an application object. Giving configuration a separate API does not eliminate the disagreement; it merely leaves its relationship to the data model to be explained elsewhere. ECLIPS deliberately puts that relationship into the design.

The recursion still needs a base. Heralds require a bootstrap environment and private state, and they must maintain sufficiently complete replicas of the predefined sorts. The paper sketches the necessary cooperation and identifies a race in which an application might create and remove structural objects before every herald had received them. The structure through which changes are distributed can itself be changed by those changes.

Different heralds may receive modifications at different times and in different orders. Concurrent updates can alter the contexts of the very sorts describing the graph. Whether every permitted ordering induces compatible structures is therefore a central correctness question, not an incidental matter of optimising propagation. The paper explicitly demanded a formal answer. An implementation must give these descriptions an operational meaning, but making them executable does not by itself establish that all their possible interactions are sound.

## Two histories remain two histories

Suppose a network partition divides a system. Each fragment may continue processing sensor input and issuing commands. When connectivity returns, the fragments contain the results of two histories with a common origin, rather than delayed copies of one history. A newly detected object may have acquired different identifiers on the two sides, while two different objects may have acquired the same identifier. Decisions and physical actions made in the meantime remain part of what happened.

Transport repair (what DDS does) cannot determine what these states ought to mean together and a generic last-writer rule cannot decide whether to identify, preserve or retract an object. The same difficulty arises without a common ancestor when independently administered systems join a coalition: equal names may denote different entities, and different names equivalent ones. Merging namespaces does not reconcile their contents.

ECLIPS specifies no automatic reconciliation. Sort identifiers are hashes of complete sort descriptors, so identical descriptions acquire the same identity without negotiation. Controlled objects have unique identifiers, and each process has a mapping between its private and global identifiers. Global identifiers can consequently be changed during a merge without rewriting the identifiers held in the process’s private state.

Most importantly, the graph is intended to allow two systems to enter one namespace while keeping their information flows separate. Selected edges can then expose selected sorts to reconciliation processes. Those processes can examine both sides and publish an application-defined result. Joining the systems need not immediately cause every store to mix every object that happens to carry a matching name.

This does not solve the semantic problem on behalf of the application. It supplies somewhere to put the solution, and a way to control which existing state is exposed while that solution runs. The same graph that describes ordinary information boundaries can describe the boundaries maintained during reunion.

Restoring a cable cannot undo an actuator movement, recover an already used identifier or reverse a decision made while the other fragment was unreachable. A useful account of recovery must acknowledge that fact. ECLIPS attempts to represent the distinct histories, the permitted flows between them and the construction of a new state, leaving the domain-specific decisions with the processes that understand them.

## Authority and the cost of agreement

Who may construct those flows or modify the objects they expose? In ECLIPS, having an object of a controlled sort in a local store can confer a capability. Depending on its state and the strength of the replica, a process may be permitted to forward it, use it as a name, update or label it, or operate on the entity it denotes.

A weakening edge can propagate an object without propagating its full authority. A monitoring subsystem may receive enough state to observe part of the data space without acquiring the right to alter or extend it. Confinement follows from the graph and its sort-restricted edges: a subsystem can receive precisely the nablas and deltas constituting its interface, without access to unrelated state.

The `newenv` operation creates a private bootstrap-like environment whose identifiers are initially available only to its caller. Revocable access can be constructed by retaining a private intermediate edge or vertex. Deleting it severs the route; it does not attempt the impossible recovery of a capability already disclosed. Authority and information flow are related within the model, but distributing authority still has consequences that cannot simply be recalled.

This is not a complete secure system. Authentication was delegated to the execution environment, while integrity and non-repudiation could require application-level cryptography. The intended contribution was an account of authority, propagation and confinement inside the data space. Those properties would still require formal scrutiny as well as an appropriate implementation environment.

Authority also raises a harder question than access: when must several processes agree? DDS exclusive ownership selects a winner at each reader according to writer strengths, observed liveliness and deadline status. Different readers may temporarily select different owners. That can be entirely acceptable for redundant sources; it does not establish a unique leader for an action requiring agreement across the system.

ECLIPS assigns this stronger obligation to one conspicuous operation. `label` atomically changes the label of every copy of a controlled object to a process identifier, to no owner, or to `delete`. With the appropriate label and name capabilities, ownership can be recovered after the labelled process fails. A controlled object can also sequence a set of nablas: updates written before a label transfer must become visible before the new label, while later updates must not become visible before it. This supplies a basis for token handover and agreement about the visibility of the preceding process’s work.

The paper calls `label` “generally very costly”. It specifies the semantic contract, not the distributed mechanism needed to realise the general operation. That is a substantial omission. Describing an atomic change does not establish when it can complete, or the price of obtaining the necessary agreement. The design puts the obligation in the substrate because applications need it but the obligation does not disappear by being placed there.

## Seven operations—or eight

The interface has only seven headings: `newid`, `newenv`, `write`, `forward`, `read` and `local-take` together, `wait`, and `label`. Counting `read` and `local-take` separately gives eight operations, hence my usual qualification.

The number should not be mistaken for a measure of implementation complexity. A graph modification expressed as an ordinary write can change distant retention obligations. A label transfer can impose agreement on several replicas and order its visibility with respect to publications. Even a local read has meaning only because the heralds maintain the stores and their relationships.

Nor is a count of operations directly comparable with DDS’s list of QoS policies. Many policies govern choices made locally after endpoints have matched: what a reader or writer retains, schedules, orders, expires or presents. Others, notably writer-independent durability and group coherency, require wider cooperation. Naming each choice separately does not make it an independent primitive of a distributed data model.

ECLIPS concentrates such questions in the sort definitions, graph reachability, context rules, state ordering and capabilities. Its interface can be small because those definitions carry much of the contract. Applications still have to choose appropriate state models and information boundaries, and the implementation still has to realise the consequences of those choices.

This was not an attempt to reproduce every DDS facility behind fewer function names. Transactions ("coherent sets"), for example, are absent. Adding atomic groups of application updates while the graph itself may change is a difficult extension. The useful question is whether this collection of concepts gives applications a more coherent basis for constructing the systems they need, at an acceptable cost. Counting calls cannot answer it.

## From the paper to the present experiment

The 2010 paper could offer only a qualitative performance argument for its construction of SPLICE: routes through a stable topology could be cached, and the cost of graph changes amortised over frequent writes. It also identified common cases in which `label` might admit cheaper implementations. These were plausible directions, without evidence of their cost in a working system.

This year’s proof-of-concept work returns to that gap between description and execution. The work is centred on heralds and the small operation set because that is where an application-facing promise must acquire concrete behaviour. The present result is only an exploratory attempt that is starting to look implementable. It by no means validates the model as written down in 2010, but so far it holds up surprisingly well.

One of the problems currently being worked through is deciding when an object has really disappeared. Finding no copy in the stores examined so far is insufficient: a publication may still be waiting for delivery, or retained state may still be on its way to another store. The implementation has to account for that work before drawing a system-wide conclusion from local absence. This is the kind of detail that makes a graph induced by its contents much harder to implement than to describe.

An executable model forces such questions into the open. The context invariant has to survive graph changes, while the objects describing that graph are themselves being distributed. Retention and obsolescence must interact with local removal and delayed updates. Making operational choices about these interactions may preserve the design, require a different formulation or reveal an error.

Proof and measurement remain separate tasks. A convincing demonstration cannot cover every ordering of concurrent structural changes, and passing examples cannot establish confinement. Conversely, a formal argument for the model would not show that its implementation is economical. The general `label` operation is a particularly clear reminder that a useful semantic promise and an affordable mechanism are different achievements.

It is also possible that only parts of ECLIPS deserve to survive: topology-derived retention, explicit finality, private-to-global identity mappings, weakening edges, or a single costly agreement operation among otherwise asynchronous primitives. Exploring the complete design gives those ideas a setting in which their interactions can be examined. It need not end in preserving every decision made in 2010.

That makes this a suitable place to pause the history. ECLIPS did not succeed SPLICE as an operational system or displace DDS. It returned to the architectural question my father had pursued and that I believe to be as relevant as it was then: what structure allows timeliness, consistency, bounded resources, redundancy, recovery and evolution to be considered together? The answer may prove too costly, incomplete or mistaken, but at least it is an attempt to answer the right question.

# Sources

A list of [sources](sources.html) is available. Much of the early history rests on recollections and a few documents my father had kept.
 
*Copyright © 2026 Erik Boasson. Dated 13 September 2026. Licensed under the [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).*
