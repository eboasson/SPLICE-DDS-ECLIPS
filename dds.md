This is part two of three. The other parts are the [early history](splice.html), and a [plea for aiming higher](eclips.html).

# DDS: the common boundary and the space becoming a bus

In a Stanford robotics laboratory in the early 1990s, a graphical simulator could stand in for a real robot. The user interface, motion planner, controllers, sensors and simulator ran on a mixture of workstations and VME-based real-time processors. The simulator and the robot could publish the same logical information; when the real machine became available, its data took precedence without requiring the other components to connect to a different server. A proposed teleoperation component could similarly override commands from the planner.

The mechanism was the Network Data Delivery Service, or NDDS. Gerardo Pardo-Castellote and Stan Schneider described it in two papers published in 1994. Pardo-Castellote was at Stanford’s Aerospace Robotics Laboratory, Schneider at Real-Time Innovations, founded in 1991 by researchers from that laboratory. Stanford and RTI were developing the work jointly, with support from ARPA’s Domain-Specific Software Architectures programme.

This is the other principal line of descent of DDS. SPLICE had arisen from the problem of structuring information in a large naval command-and-control system. NDDS arose among robots, sensors and controllers: components producing repetitive measurements, consuming them at different rates, and being replaced or moved between machines while the system was running. It had an independent history and a different starting question. The overlap lay in a consequential conclusion: the identities and addresses of communicating processes were the wrong things to represent in application software.

## Data ubiquity

The early NDDS papers use the terms *producer* and *consumer*. A producer registered the named data objects it could supply and sent updates at its own discretion. A consumer subscribed to objects it required, without specifying which process would supply them. Declarations were distributed dynamically and expired unless refreshed; there was no central name server through which all communication had to pass.

The intended effect was called *data ubiquity*. A measurement ordinarily required by a low-level controller might later also be useful to a planner or diagnostic program. New consumers should obtain it without modifications to the producer, and a replacement producer should not require every consumer to be redirected. Named information, rather than network topology, determined communication.

The physical-control setting shaped the details. Recovery of an old sensor value could be worse than its loss if a newer value was already available. Computation often had to be triggered by fresh data. Several sources could provide one logical item, while consumers required it at different rates. The complete set of participants was not fixed at design time.

A producer therefore had a *strength* and a *persistence*. Strength established precedence among sources; persistence bounded the interval for which the last value from a source retained that precedence. The real robot could have greater strength than the simulator. Once its values ceased and the persistence interval elapsed, the simulator could again supply the effective value. No permanent arbiter was required, nor did a failed producer have to announce its own failure.

This resembles a simplified version of SPLICE’s quality-and-decay rule. SPLICE had allowed a quality measure attached to information to diminish with age; Signaal found the mechanism unhelpful in practice because letting the sources publish with different key values was a more general solution. SPLICE-lite dropped it completely.

Consumers supplied two complementary time bounds. *Minimum separation* limited notification frequency, allowing a slow consumer to use a fast sensor without processing every update. A *deadline* caused a callback if no fresh update arrived within the stated interval. Notification could be immediate, handled in an NDDS task, or polled in the application’s own execution context. Sending could similarly favour immediate delivery or group updates to reduce overhead. These facilities made a coherent answer to distributed control: anonymous typed communication, source arbitration, rate control, failure indication and explicit compromises between latency and cost.

## Named flows and keyed populations

An NDDS data instance had a system-wide name and a registered type. Two instances that any client had to distinguish required different names. Later manuals called this name a *publication topic* and recommended hierarchical strings such as `/sensor/location/type/id`. Pattern subscriptions could select sets of names or types, allowing tools to observe broad classes of traffic without being configured separately for every object.

This resembles a topic in later DDS only up to a point. A SPLICE sort, and subsequently a keyed DDS topic, may contain a population of objects: fields in each datum form a key, so that tracks 17 and 42 coexist within the same sort or topic, while successive values for track 17 remain associated with the same instance. Publicly surviving NDDS manuals confirm my recollection from the 1990s that NDDS then had no support for keys.

Like SPLICE, NDDS maintained administrative databases containing publications, subscriptions and matching state. Utility routines also provided a local object store indexed by type and name. The main difference is that NDDS presented named real-time flows, whereas SPLICE made selected portions of a distributed data space available in receiver-side, keyed and queryable data stores.

NDDS did have histories. A polled consumer could buffer several updates between polls, and reliable delivery required a producer to retain values pending acknowledgement. Send and receive queues, sequence numbers and acknowledgements permitted recovery in order. This was bounded, operational history: it bridged a polling interval or repaired loss in one named stream, rather than maintaining a separate history for every keyed object in a collection. Best effort remained a sensible default for periodic measurements, while commands might require recovery of every omission.

The implementation made connectionless UDP serviceable across heterogeneous workstations and real-time processors, using XDR for typed data and exposing controls for reliability, discovery, multicast and resource use. Its manuals increasingly addressed the practicalities of a commercial middleware product: installation, supported platforms, generated code, deployment and diagnosis. SPLICE, by contrast, deliberately hid all these controls from the applications because it concerned deployment configuration an independent activity. SPLICE-lite, moreover, was a research implementation whose facilities could remain open to experiment. The difference says more about their purposes than their relative sophistication.

By the end of the decade NDDS had regularised its API around publishers, publications, subscribers and subscriptions. Individual flows carried their own properties, while publishers and subscribers grouped work for efficient processing. The outline would be recognisable in DDS, though neither the data model nor its semantics was yet that of the standard.

## A boundary customers could own

The two DDS lineages overlapped in many ways. Robotics and naval systems were both concerned with timeliness, bounded resources, multiple sources and components appearing or disappearing. Both technologies replaced addresses with typed descriptions and maintained distribution state dynamically. Their different starting points had produced overlapping solutions.

There is documented contact between the ideas by 1995. Pardo-Castellote’s Stanford dissertation treats my father’s 1993 IEEE article as related work and compares aspects of SPLICE with NDDS, including keys, source arbitration, configuration and update mechanisms. It establishes engagement with the published design by then, not when he first encountered it or influence on NDDS’s original design. NDDS already had a history of its own when the article appeared.

The comparison also shows how one design could be read through the concerns of the other. Pardo-Castellote contrasted SPLICE’s polling interface and lack of deadlines or multiple update rates with NDDS. From the SPLICE perspective, these mix architectural and implementation choices. A latest-state store already decoupled production and consumption rates; TACTICOS dealt with supervision above the data space; polling or notification was an access mechanism. The differences were real, but their significance depended on the question being asked.

By the end of the 1990s, customers had another question in common. NDDS was middleware with interfaces owned by RTI. SPLICE was infrastructure beneath TACTICOS, and Signaal/Thales had no ambition to build a general middleware business around it, but important interfaces beneath the combat system remained proprietary to one supplier. Success made both dependencies harder to accept.

The United States Navy’s open-architecture work supplied important pressure. RTI’s later retrospective describes a Navy testbed evaluating NDDS and SPLICE against Aegis requirements. The mention of SPLICE here is interesting, as Signaal/Thales wasn’t in the middleware business. This is almost certainly thanks to Norm Howes of the Institute for Defense Analyses (IDA) who pursued a sustained effort to bring SPLICE—and, more broadly, publish–subscribe—into US defence systems.

John Sarkesain recalls how already in the 1990s, Norm Howes was arguing for publish-subscribe and presented SPLICE to DISA engineers working on the Global Command and Control System (GCCS). At the time, GCCS took a client–server approach but it eventually did switch to publish-subscribe. In 1999, Sarkesain proposed a DoD Advanced Concept Technology Demonstrator for a Cyber Operations and Information Warfare System, which received funding. The project followed Howes’ suggestion to use a publish–subscribe architecture, with my father and me consulting on the architecture. Howes furthermore advocated its use in the Missile Defense Agency’s Command and Control, Battle Management and Communications system (C2BMC), which also ended up adopting it. At Signaal, I remember Jan Willem Lokin adding ballistic missiles to the air-defence demo at Howes’ request in that period.

Other open technologies occupied neighbouring ground. CORBA standardised remote invocation, its Notification Service distributed events, and the High Level Architecture supplied a federation model for simulation. None offered quite the application contract and operational combination already demonstrated by NDDS and SPLICE. The OMG became the venue for extracting a common contract from those working systems. RTI and Thales supplied the two principal technological lineages, with contributions from other participants; describing it as their joint work should not erase the others.

## The common model

By the time standardisation began I had left Signaal—and the field—and did not participate in the original negotiations; I returned in late 2010, on joining PrismTech, so this part of the account is reconstructed from documents and participants’ recollections.

The proposal was adopted in 2003 and formally published as DDS 1.0 in December 2004. Adoption, finalisation and formal publication were separate stages, which explains the different dates sometimes given for the first standard. Its immediate subject was the service presented to an application. Networking and discovery would require a later specification.

At the centre of the mandatory Data-Centric Publish–Subscribe layer, DCPS, is a rather nebulous *global data space*. Within a domain, a participant contains publishers and subscribers, which in turn contain writers and readers referring to named, typed topics. Compatible readers and writers match without either application knowing the address or identity of the other. The decomposition accommodates NDDS’s separation between individual flows and the entities grouping them.

A DDS topic, however, is more than the externally named flow of early NDDS. Fields in its data type can form a key. Values with the same key are successive samples of one *instance*; different keys denote different instances within the topic. One topic can therefore contain the population of tracks, vehicles or sensors that SPLICE represented with a keyed sort. A topic without a key contains a single instance and can stand in for NDDS’s named flows.

The two levels of naming—topic and key—accommodate both a publish–subscribe interpretation, in which writers send successive samples, and a shared-space interpretation, in which those samples update replicated values. This works remarkably well in simple cases, but difficulties arise when one combines settings originating in the two different interpretations.

The standard deliberately stopped short of prescribing an implementation architecture. SPLICE implementations had placed local administration and application data in a common shared-memory database: the hardware left little room for extra copies and context switches. NDDS had evolved libraries, agents, queues and network mechanisms around a different arrangement. DDS specified observable entities and behaviour, leaving layouts, daemons and transports to the implementation.

This was essential to opening the proprietary boundary. A standard requiring one participant to rebuild its product in the image of the other would have defeated much of the purpose. Applications acquired a common vocabulary without suppliers having to agree on every internal mechanism. The first specification was nevertheless incomplete even at that boundary: it used keys without standardising how key fields were designated in the source code.

## Quality of service as a treaty

The most conspicuous part of the negotiated design is its collection of quality-of-service policies. Reliability, history, deadlines, liveliness and many other became named settings. Many policies express requirements already encountered in one or both parent systems. Deadlines and time-based filtering cover periodic production and rate limitation. Reliability, histories and resource limits describe compromises between recovery, buffering and finite memory. Durability concerns availability to late subscribers. Source-timestamp ordering recognises that an observation’s time may matter more than its arrival time.

For several policies, a requested/offered compatibility rule compares a reader’s request with a writer’s offer. An incompatible pair is not matched. That makes the contract checkable without applications naming their counterparts, albeit at the cost of adding substantial machinery. I have serious objections to making this the general model because data doesn’t become irrelevant just because the writer has a different quality-of-service, but its historical role is clear: the policies served as a treaty between products and communities with different expectations.

Assigning each policy to SPLICE or NDDS would be temptingly tidy and mostly speculative. Both had encountered many of the same operational problems. The standard gave those problems common names and negotiated semantics. It also placed an unusually large number of choices at the application boundary. Whether the choices compose into a satisfactory account of a complete system is a separate question from whether each addresses a useful requirement.

## Portability, then interoperability

DDS 1.0 defined no common wire representation. Two products could expose substantially the same application interface while using mutually unintelligible discovery and data protocols. Replacing one might still mean replacing an entire DDS island or inserting a gateway. Source portability had opened one boundary; communicating between independent implementations required opening another.

The OMG requested proposals for an interoperability protocol in 2005. A joint RTI–Thales submission was recommended for adoption in 2006, and the first formal DDS Interoperability Wire Protocol, DDSI-RTPS 2.0, appeared in April 2008. Its numbering reflected an older protocol: RTI had developed RTPS around 2001, and a public Internet-Draft described it in 2002.

DDSI had to specify how participants found one another, how endpoints advertised their topics, types, locations and relevant policies, and how compatible readers and writers communicated. Globally unique identities gave protocol state a stable basis. Sequence numbers, announcements of available history and acknowledgement of received or missing samples provided reliable delivery and repair over an unreliable transport. Large samples could be fragmented and repaired without imposing a separate byte-stream connection for every reader.

This was considerable machinery, but it still left room for different implementations. Products could trade memory against retransmission work provided their externally visible behaviour conformed. Fitting it to established internals was sometimes awkward—OpenSplice’s internal identifiers and the protocol identities were decidedly different—but the consequential result was a common network realisation of the application model.

Public interoperability demonstrations began in 2009. These mattered because independent implementations expose ambiguities that review within one code base can leave undisturbed. They showed that the common wire was more than a paper agreement, but they did not immediately make heterogeneous systems an ordinary engineering choice.

In my experience it is only in recent years that deliberate integration of several DDS vendors’ implementations has become common. From the first standard, or even the adoption of DDSI, this took the better part of two decades. A technical possibility, a demonstrated capability and an ordinary deployment practice are different stages. The interval does not negate the achievement: customers eventually obtained both an application model and a working wire independent of one supplier.

## Reconstructing an application world

The original standard also contained an optional second layer, the Data Local Reconstruction Layer, DLRL. Where DCPS exposed samples and keyed instances, DLRL attempted to reconstruct a local network of application objects, with identity, attributes, relations, inheritance, collections and selections. An object could map onto several topics and generated language objects and a cache were to shield applications from that representation.

This came closer to making the reconstructed world itself the interface. Its affinity with SPLICE’s database tradition is apparent, though DLRL was neither a transcription of SPLICE nor a small addition to it. Another model, mapping language and body of generated machinery stood above DCPS. Making the layer optional allowed applications requiring direct access to samples to avoid it.

For many years OpenSplice appears to have been the only product with a substantial shipped implementation. Even there DLRL was hardly used and was eventually removed. OpenSplice 6.3, released in 2013, put the direction of change into unusually clear relief: its DDSI2 interoperability services became generally available while DLRL was deprecated in preparation for removal.

This does not establish that applications had ceased to need identities and relations. Instead it establishes that DLRL was not an attractive way to provide them. It required an additional modelling language, generated interface, cache and mapping while leaving difficult details dependent on the application. A relation between two radar tracks may have different rules from one between a component and its sensor or a vehicle and its route. Almost all applications used DCPS directly and constructed precisely the local model they needed, repeatedly solving those problems outside DDS.

DLRL’s presence in DDS 1.0 documents an ambition extending beyond sample delivery. Its disappearance records the failure of that particular answer, but that failure doesn’t demonstrate that applications no longer need a coherent notion of shared state.

## What grew in its place

DDS did not become less capable. It spread beyond the defence systems that helped bring it into existence. Its protocol matured, its type system became extensible and discoverable, and security acquired a serious specification. DCPS retained a recognisably data-centric core: keys represent populations of objects, reader histories are more than network queues, durability can preserve state for later readers, and queries select by value. DDS never became merely a message API.

Its most vigorous development nevertheless concerned communication and representation. XTypes added structural type descriptions and assignability rules, dynamic inspection and serialisations supporting controlled evolution. DDS Security added authentication, access control, key exchange and protection for discovery, metadata and application data. Language mappings, constrained-device protocols, shared-memory paths and profiles extended the family further.

These are substantial achievements. Long-lived systems cannot reasonably stop every component when a field is added to a structure. Open discovery without authentication is inadequate in hostile environments. Wire interoperability cannot rest on vendors making similar guesses about retransmission. The work also admits objective tests: encodings can be compared, protocol traces examined, cryptographic exchanges attacked and independently developed products connected.

There are limits to what those achievements establish. Type machinery can determine whether one version of `Track` is structurally assignable to another. It cannot say whether altitude is relative to mean sea level, whether a position is measured or extrapolated, or when an old observation ceases to be useful evidence. Authentication and protected delivery cannot establish that an observation is true.

No generic middleware can supply all the meaning of an application. My concern is a different one: the architecture within which applications assign those meanings. SPLICE had attempted to make replication, retention, reconfiguration and recovery parts of one system argument. DDS increasingly provided precise mechanisms whose interpretation and composition became the application’s responsibility. The distinction is especially visible in what happened to context data.

## The memory that never became interoperable

DDS defines four durability levels. `VOLATILE` imposes no obligation to retain data for readers arriving later. `TRANSIENT_LOCAL` retains history at a writer for late readers, but the history disappears with the writer. `TRANSIENT` places it in service-maintained storage independent of the writer’s lifetime; `PERSISTENT` permits it to survive a restart as well.

The distinction between transient-local and transient is fundamental. One retains an endpoint’s history, the other admits a value into storage on behalf of the data space where it may remain after its producing process has terminated. SPLICE called the latter *context data*, a name that much better conveys its purpose.

Consider a restarted process rebuilding the portion of the world it needs. With SPLICE, or a DDS implementation using an independent durability service, it can obtain state from an already populated store. Original writers need not retain and replay their histories, and recovery leaves their ordinary CPU load and network traffic essentially unaffected. In SPLICE’s node-wide shared memory, retention did not even require another copy of a value.

Transient-local durability assigns responsibility differently. History belongs to the writer, and satisfying a late or restarted reader involves that writer and the network. The mechanisms may look similar from the receiving application’s point of view, but they have different consequences for normal operation, failure and recovery.

DDSI made transient-local behaviour interoperable. It has never standardised the service protocol required for interoperable transient or persistent data. The question has repeatedly been deferred and remains open. Implementations and proposals exist, but in my recollection agreement was never within reach.

Sending an old sample is easy but defining a distributed service’s history is not. Stores can diverge during disconnection, writers can disappear, and a reader can arrive while replicas are being aligned. Retention limits, lifespans, ordering, coherent sets and instance lifecycle all constrain what the service may subsequently supply. A common protocol must make these behaviours agree, not simply specify how to transfer stored bytes. The difficulty is real, and helps explain why a facility present at the application boundary never acquired an equivalent common wire contract.

Several vendors provide useful transient or persistent services within their product families. Proprietary agreement between writer, store and reader is not cross-vendor transient data. The contrast within SPLICE’s own lineage is instructive.

OpenSplice always supported transient data through its durability service. State independent of an application writer was native to its architecture but DDS’ notion of transient-local fitted less well. Its documentation at one time described approximating transient-local through transient storage with specific disposal and cleanup settings. Proper interoperable transient-local behaviour required a separate writer history in its DDSI2 service.

Eclipse Cyclone DDS presents the converse. As of 2026 the open-source implementation supports `TRANSIENT_LOCAL` but no `TRANSIENT` data. The commercial derivative Zetta DDS does support `TRANSIENT`, necessarily using a proprietary protocol. One feature has become common while the stronger shared-space property remains a product choice.

After almost two decades of work on wire interoperability, independent implementations can discover one another, match their advertised policies, exchange assignable versions of types and protect their samples, but they still have no standard means to make a value outlive its writer in a shared transient store. For a standard descended partly from SPLICE, this is a particularly telling boundary to have left closed.

## Success on the narrower question

There were good reasons for concentrating on protocols, types and security. These problems recur across industries and can be standardised without deciding what a radar track, robot pose or battery cell means. A generic object layer risks being either empty or oppressive and DLRL illustrated the difficulty. Vendors, users and standards committees could reasonably favour work with clear boundaries, measurable results and immediate demand. The result remains one of the few middleware families combining typed data, keys, discovery, multicast, reliability, real-time controls and implementation independence in a reasonably coherent whole.

That does not compensate for what was lost. SPLICE proposed an architectural perspective from which timeliness, consistency, availability, bounded resources, reconfiguration and recovery were considered together. DDS limits itself to shuffling data around, and even *architecture* has largely changed its referent in this discourse. It commonly describes a middleware’s internal organisation, a deployment topology or a selection of components. It less often denotes a principle by which one might argue that the resulting system satisfies several non-functional requirements together. Vendors cannot be blamed for this in isolation. Users have shown little more appetite for the question: latency is easy to rank, while architectural coherence is harder to specify or benchmark.

The result is a technically richer middleware family with a poorer public account of how complex systems ought to be built. That is what I mean by the space becoming a bus. The architectural question remains, even when the means of communication have become quite decent.

# Sources

A list of [sources](sources.html) is available. Much of the early history rests on recollections and a few documents my father had kept.
 
 *Copyright © 2026 Erik Boasson. Dated 13 September 2026. Licensed under the [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).*
