# SPLICE, DDS and ECLIPS

*Copyright © 2026 Erik Boasson. Dated 5 September 2026. Licensed under the [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).*

**STATUS: draft**

Three instalments, with source notes and remaining historical questions collected at the end:

1. *SPLICE: forty years of an architectural idea*
2. *DDS: the common boundary, and the space becoming a bus*
3. *ECLIPS: returning to the architectural question*

------------------------------------------------------------------------

# 1. SPLICE: forty years of an architectural idea

In the spring of 1986, my father, Maarten Boasson, presented SPLICE at New York University, Philips Research U.S.A. and General Electric Research. These are the earliest presentations outside Hollandse Signaalapparaten that I can presently identify. They give 2026 a useful anniversary: forty years since an architectural idea developed for naval systems was taken to a wider audience.

The anniversary is a good occasion to return to the question behind SPLICE. How do you construct a system whose functions can be developed independently, moved between machines and recovered after failure, without making every function understand the arrangement of all the others?

The answer was to make applications describe the information they needed and produced. The supporting infrastructure would take responsibility for finding it, distributing it and keeping it available. That sounds familiar now. Its consequences are still less familiar than the communication mechanism through which it is usually described. To see them, it helps to go back two years, to a pair of memoranda in Hengelo.

## The industrial problem

Signaal constructed complex real-time command-and-control systems for naval use. By the early 1980s, adding functions in software had become relatively easy, but integrating the resulting collection of functions had not. A memorandum written on 4 January 1984 by Max Ceuleers, head of the department, describes the consequences clearly.

Logically concurrent functions had been divided into small tasks and scheduled sequentially on a central computer. Their behaviour consequently became entangled through timing, interrupts and shared memory. Modifying one function could perturb the carefully debugged timing of others. Functions carried over from one delivery were modified and tested again for the next. Once the central computer ran out of capacity, functions were simplified or transferred to additional microprocessors. The result was *wildgroei*: uncontrolled and unmanageable growth.

Ceuleers proposed assembling systems from recognisable functional modules with well-defined responsibilities. Proven functions should be reusable, and individual deliveries should be configurable without becoming further variants of a monolithic program. The hard question was how independently usable modules were to communicate. Giving each module a clean point-to-point interface to its expected collaborators would merely replace one form of coupling by another: the software would still encode the configuration in which it happened to be deployed.

On 29 February 1984, Maarten circulated a memorandum entitled *Systeem Architectuur*—“System Architecture”. Its subject was communication between subsystems and dynamic reconfiguration. An application requiring information would request data of a particular type; one producing such information would supply data carrying the corresponding type indication. An intermediate layer would locate a source, register a continuing interest, distribute subsequent updates and retain received values locally. Applications neither addressed one another nor represented the current assignment of functions to processors.

Much of the proposed machinery belongs unmistakably to its time: small processors with local ROM, shared buses, Ethernet, duplicated cables and dedicated processors managing distributed databases. The essential abstraction does not. The aim was to meet functional independence, timeliness, resource bounds, fault recovery and continued evolution together. Communication machinery existed to support that architecture.

Ceuleers supplied the industrial diagnosis and organisational backing; Maarten supplied the data-oriented architecture. It would be misleading, though, to imagine the answer being invented from a blank sheet in the eight weeks between the memoranda. My father's recollection placed his first thoughts in the late 1970s or very early 1980s. The work appears to have remained unofficial for some time, until practical difficulties and a sufficiently developed proposal justified a formal study. In his later inaugural lecture, he thanked Ceuleers for allowing the investigation to continue despite opposition within Signaal.

## Removing the other component

Suppose a component A obtains track information from a component B. (This example and terminology are modern, but the distinction is the memorandum's.) If A's interface contains B's identity, address and protocol, it represents the current decomposition of the system. Interposing a directory changes only the resolution mechanism. Replacing B, moving its function or introducing another source still changes something about which A has made an assumption. The February design removed B's identity from A's interface.

Each functional subsystem would be associated with a Distributed Data Base Manager, or DDBM. A subsystem requiring information not produced locally requested the appropriate type from its DDBM; the request was relayed to the other DDBMs. If a source existed, the data were returned. Otherwise the requester received an indication that the information was unavailable, leaving diagnostic functions to determine the cause.

Producers need not enumerate consumers, consumers need not identify producers, and equivalent information could come from several sources. Physical routes and distribution state still existed, of course. They belonged to the infrastructure and system-management functions. An ordinary application represented its function and the information it supplied or required, allowing sources to be duplicated, replaced, moved or divided without changing the consumer's statement of need.

The design also went beyond a remote lookup. On satisfying a request, a DDBM registered the requesting DDBM “as subscriber” to the relevant type. Subsequent values could then be distributed automatically to the registered destinations. A subscription arose from reading data: the distribution layer recorded the continuing interest and maintained a local copy. Applications read from local stores that network traffic updated selectively.

Values carried validity periods, so stale data could be recognised. Production time and quality were explicit properties, and where several sources supplied equivalent information, the DDBM could select among them according to a criterion established during initialisation or reconfiguration. A DDBM also recorded the last local use of a type and could send a special message when distribution was no longer required.

Reconfiguration followed the same division of responsibility. Diagnostic functions would identify failed processors; reconfiguration functions could move work to reserve equipment or stop a less important function to preserve a more important one. The memorandum observes explicitly that other processors need not be notified when a reserve processor assumes a failed processor's task. Once the replacement requested and supplied the appropriate types, the DDBMs could establish the flows.

This did not solve failure detection, capacity allocation or the ranking of functions. It confined configuration-dependent reasoning to the facilities responsible for configuration. Applications did not each require their own recovery wiring. The same abstraction addressed integration during development and physical reorganisation during operation.

## A database assembled from replicas

The February memorandum did not yet use the name SPLICE. Nor did it contain the complete later model: sorts, worlds and context data were still absent, and DDBMs appeared principally as dedicated hardware. Reading the mature architecture back into 1984 would make a tidier history, but a less accurate one.

By the 1986 presentations, the proposal had acquired its name. It began with the ordinary verb *to splice*: to join, with the useful suggestion that streams may also be split and joined. The expansion, **Subscription Paradigm for the Logical Interconnection of Concurrent Engines**, came afterwards, according to family recollection. The dedicated processors eventually gave way to software accompanying ordinary applications, while the distinction between functional data and the supporting machinery survived.

In SPLICE, a subscription caused the relevant data to be maintained in a store local to the application. Distribution updated the store asynchronously and the application queried it when its computation required data. The shared database was an abstraction constructed from partial replicas with no single machine containing the authoritative whole and no synchronous transaction making all copies identical at every instant. Each application obtained the portion selected by its subscriptions and observed updates after finite, variable delays. Within those limits, a system could be designed against one information space instead of a changing diagram of processes and connections.

The unit of information became the *sort*: a name, a structured type and key fields. Several objects of one sort could coexist, distinguished by their keys, while a later value with the same key could replace an object's previous state. A track sort, for example, could contain many tracks distinguished by track number. *Worlds* provided logical scopes: the same sorts could be used for live operation, training, test or replay, with publications and subscriptions meeting only where sort and world agreed.

There remained a difficulty that ordinary subscriptions did not solve. A process might require information before it had started, or after it restarted. If distribution took place only in response to current subscriptions, the process might receive nothing until every producer happened to write again. Some state had to be retained in anticipation of future subscribers. SPLICE called it *context data* and implemented retention using replicated daemons that automatically subscribed to all context data and made it available to ordinary applications.

That placed responsibility outside the ordinary writer. A restarted process obtained current state from an already populated store; original producers did not have to replay it. Under normal operational conditions this imposed essentially no additional network traffic or producer CPU load. Because SPLICE used one common shared-memory store on each machine, retaining a value as context did not require a second in-memory copy either.

Hans van 't Hag, who joined Signaal in 1983 and later worked on the middleware beneath TACTICOS, remembers recognising the need to retain data even when no current application required it. My guess is that Hans identified the practical problem and my father generalised the answer and drew its architectural consequence. After four decades, I would not turn that guess into a firm division of credit. The consequence itself is clear: retention became independent of current application subscriptions, giving the shared data space a life beyond its present users.

## Choosing it for TACTICOS

Around 1989, according to one of my notes, Signaal's technical director Herman Driessen chose SPLICE as the foundation of TACTICOS rather than the “modular architecture” then being developed. In his 1996 inaugural lecture, Maarten thanked Driessen for choosing his architecture for a new generation of Signaal products and maintaining the choice in the face of considerable resistance. He added that Signaal had done well by it.

In the early '90s, the architecture was being tested in a full-scale naval command-and-control system. The February memorandum had settled the central abstraction; the engineering still had to settle representation and identity, subscription administration, reconstruction after restart and operation within severe bounds on processor time, memory and network capacity.

My father's 1990 chapter “Architecture of Real-Time Systems” gives the argument in concentrated form. Conventional communication makes a module's interface encode another module and assumptions about the timing of their interaction. A process should instead state which data are to be communicated and leave their production, location and delivery to the supporting system. The chapter appeared in *Beauty Is Our Business*, the volume for Edsger W. Dijkstra's sixtieth birthday. That connection was personal too: the 1979 summer school had led to my father's membership of Dijkstra's Tuesday Afternoon Club in Eindhoven and to a lifelong friendship.

The argument was concise. Making it sustain an entire shipboard system required rather more work.

## Ten megabits, used carefully

TACTICOS had to combine sensor observations, ship state, track processing, tactical functions and operator displays, and manage something of the order of a thousand tracks. Around 1990, the available network was standard 10 Mbit/s Ethernet, duplicated using two cables for redundancy. Duplication protected against a cable or physical-network failure; it did not double the usable capacity.

This was shared, half-duplex Ethernet. Rising load brought more collisions, retries and variability. I remember a design rule that limited utilisation to a fairly low percentage, on the argument that collisions were then practically negligible. I do not recall the exact figure. Something of the order of two megabits per second was all the design could safely treat as available.

Messages were kept compact and, where possible, combined to fill network packets instead of paying the framing overhead for every small update. Subscriptions restricted distribution to places with a declared use for the data. Local reads did not generate network requests and replies.

Low utilisation could not eliminate delay or jitter. For a moving target, the position in a newly arrived packet was already a position in the past. Special Ethernet boards inserted timestamps as packets were streamed onto the wire, allowing accurate clock synchronisation. This allowed a subscriber to use position and velocity at a known instant to extrapolate a track to the time required by a display or another function.

## The local store

Almost all SPLICE implementations reserved one central place on each machine: a block of shared memory containing both the local administration and the local data. (I believe there was one exception: an investigation into implementing on transputers.) All processes operated on it directly. The database was distributed across machines, but its local portion was shared rather than dispersed among private queues.

On the processors available for TACTICOS, repeated copying, general-purpose inter-process communication and additional context switches would have made the complete system infeasible. The common store allowed local publications, subscriptions and queries with very little movement of data. It was a practical requirement for running TACTICOS on the hardware of the day.

The cost was reduced isolation within a machine. The block was a shared point of failure, and a process capable of corrupting it could damage more than its own state. Later DDS vendors were right to raise that objection. Assessing it requires looking at protection, detection and recovery as well as process boundaries.

There is a useful distinction here between the information model and its implementation. Many DDS implementations later acquired shared-memory transports to avoid the network stack and, where possible, serialisation and copying. Those transports need not reproduce the single SPLICE block containing administration and data. Eclipse Cyclone DDS, in some sense a descendant of SPLICE, eventually abandoned the inherited node-wide design, at least for a time. A common local store had a compelling historical rationale; it was not a requirement for the shared-data-space abstraction.

## Recovering information as well as processes

A replacement process declared the same publications and subscriptions as its predecessor and attached to retained state already present in the local store. Ordinary subscribed flows resumed independently. Existing consumers continued to select data by sort, world and key rather than by the replacement's identity. Timestamps and validity information exposed ageing data instead of silently making the last received value permanently true.

This separation of information from process lifetime matters as much as the freedom to place processes on different machines. Starting a replacement executable is only part of recovery. It must also acquire the state needed to do useful work, while the surviving system must recognise the information it produces. Context data addressed the first problem; stable data identity and anonymous flows addressed the second. Diagnostic and reconfiguration functions still had their own work to do, but ordinary applications did not have to reconstruct the system's connections themselves.

Some refinements did not survive experience. SPLICE allowed equivalent values to be selected through a quality measure that could diminish with age. Signaal found that the general mechanism did not work usefully in practice—and I removed it from SPLICE-lite. Timestamps, validity periods and application-specific physical models remained useful. The unsuccessful part was the attempt to reduce the merits of a datum to one general quality figure.

Over the following decade, experience accumulated in fielded naval combat-management systems. Performance, extensibility and fault tolerance became properties exercised across delivered systems and successive generations of equipment. The architecture also returned to the laboratory, where I became directly involved in its implementation.

## Rebuilding it

In 1995, Hollandse Signaalapparaten contracted me to write a new SPLICE implementation. I joined the company in 1996 and continued as its lead developer. The result, *SPLICE-lite*, retained the existing core but also served as a research instrument in which we could explore the consequences and limits of the architecture.

Most of it was my work. Typed and keyed sorts, worlds, publications and subscriptions, receiver-side stores, queries, and periodic, context and persistent data came from SPLICE. Category subscriptions, resources and pluggable databases were mine, as were the built-in topics it added. Multisorts were probably an older idea, but I believe mine was the first implementation usable in practice.

The API made the database interpretation literal. A subscription created or attached to local storage; incoming data altered it; a read selected values by a predicate. Depending on the choice of key, a sort could represent a current value, a set of objects or a history. A multisort joined sorts on corresponding keys, bringing together independently produced parts of one conceptual object without requiring them to share one structure, update rate or retention policy. Position, classification and threat evaluation could remain separate because their production was separate, while being queried together as attributes of one track.

The built-in topics applied the same model to SPLICE's own administration. Participants and relevant data-space entities were represented within the data space, making ordinary distribution, retention and observation available for administration too. A diagnostic application could use the information architecture to examine the infrastructure providing it. This idea would matter later in DDS and ECLIPS.

The other additions explored how far the model could remove unnecessary dependencies. Category subscriptions let a recorder or diagnostic program select classes of present and future sort-world combinations, without enumerating all those known when it was written. Resources dealt with cases where a client really did need a provider to perform a service or control scarce equipment: claiming a named resource established private data flows without putting the provider's location into the client.

Pluggable databases made the receiving store itself replaceable, with its own insertion, query and waiting operations. That freedom came with real risks because plug-ins participated in the common memory and locking discipline; a serious error could damage the node-wide service. Parts remained experimental. SPLICE-lite was useful partly because it let us investigate such boundaries in working software.

## Pulling plugs, crashing cars

My colleague Jan-Willem Lokin built an air-defence demonstration on SPLICE-lite. Norm Howes of the Institute for Defense Analyses subsequently used it for presentations to American military audiences. Processes could be introduced, duplicated, stopped or restarted while the tactical picture remained expressed by the same data objects. It made the architecture much easier to explain.

At one presentation on a Sun Microsystems site, we showed the demonstration to engineers from Raytheon and invited them to choose a machine and pull its plug whenever they liked. Before they did so, an X server froze. This surprised us as much as it did them. I decided the machine with the frozen display was as good a candidate as any, and pulled its plug.

The processes that had run there were relocated automatically. Everything else continued, without an operator reconfiguring the applications and—especially to the visiting engineers' surprise—without the track identifiers changing. That was not cosmetic. Preserving a display while replacing every track with a newly identified one would have destroyed continuity for computations and operators using those identities. An accidental display failure had given us a more persuasive demonstration than the planned one.

Around 1998, we discussed another application with Rijkswaterstaat: monitoring road traffic and distributing current information to the matrix signs above and beside Dutch roads. I wrote a demonstrator in which every car was an independent process, proceeding autonomously along a Möbius-strip motorway. Merging relied on the ugly hack of letting cars drive along the shoulder until they “dared” to join the traffic. The occasional collision, on the other hand, furnished useful work for the monitoring functions.

The demonstrator also used SPLICE-lite's generalisation of context data and resources to explore application-controlled retention. This took the independence of retained information from its producer a step further: an application could participate in deciding what to keep, instead of leaving that decision entirely to a generic context daemon. The distinction would later become important to ECLIPS, where the location and policy of retained state became programmable parts of the data-space model.

## The architecture behind the machinery

In his inaugural lecture at the University of Amsterdam on 18 October 1996, *Het onmogelijke duurt iets langer*—“The impossible takes a little longer”—my father stated the proposition in more general terms. Subordinating structure to functional decomposition forces the relation between each pair of functional components to be specified and implemented separately, introducing avoidable complexity. One should first design a global architecture into which the functional components fit.

Reuse consequently depends at least as much on the architecture as on the components. A component must be specifiable independently; properties of the whole then follow from those of the components together with those of the “supporting and connecting architecture”. SPLICE gave the argument concrete expression: keys that outlived processes, context data available after a restart, queries against local state and ordinary data describing the infrastructure itself.

SPLICE-lite also made the design available for formal treatment. Work with Dutch universities and CWI reduced it to variants of *Basic SPLICE*, comparing an ideal global data space with implementations using asynchronously updated local sets. The papers established equivalence and expressiveness results for carefully restricted cores, rather than correctness of complete implementations. They helped explain how a small set of operations could sustain a shared-data-space abstraction without a central server or a general agreement protocol.

By the end of the 1990s, SPLICE had supported operational combat-management systems, generated a substantial research implementation and admitted a useful formal account. Signaal, and later Thales, nevertheless had no ambition to establish a general middleware business around it. Customers began to object that TACTICOS depended on an interface and development path controlled by one supplier. SPLICE had succeeded well enough to become infrastructure, and that success created pressure to open its boundary.

Meanwhile, RTI's NDDS had reached neighbouring ideas from the world of robotics. Those two routes would meet in DDS.

------------------------------------------------------------------------

# 2. DDS: the common boundary, and the space becoming a bus

In a Stanford robotics laboratory in the early 1990s, a graphical simulator could stand in for a real robot. The user interface, motion planner, controllers, sensors and simulator ran on a mixture of workstations and VME-based real-time processors. The simulator and the robot could publish the same logical information; when the real machine became available, its data took precedence without requiring the other components to connect to a different server. A proposed teleoperation component could similarly override commands from the planner.

The mechanism was the Network Data Delivery Service, or NDDS. Gerardo Pardo-Castellote and Stan Schneider described it in two papers published in 1994. Pardo-Castellote was at Stanford's Aerospace Robotics Laboratory, Schneider at Real-Time Innovations, founded in 1991 by researchers from that laboratory. Stanford and RTI were developing the work jointly, with support from ARPA's Domain-Specific Software Architectures programme.

This is the other principal line of descent of DDS. SPLICE had arisen from the problem of structuring information in a large naval command-and-control system. NDDS arose among robots, sensors and controllers: components producing repetitive measurements, consuming them at different rates, and being replaced or moved between machines while the system was running. It had an independent history and a different starting question. The overlap lay in a consequential conclusion: the identities and addresses of communicating processes were the wrong things to represent in application software.

## Data ubiquity

The early NDDS papers use the terms *producer* and *consumer*. A producer registered the named data objects it could supply and sent updates at its own discretion. A consumer subscribed to objects it required, without specifying which process would supply them. Declarations were distributed dynamically and expired unless refreshed; there was no central name server through which all communication had to pass.

RTI called the intended effect *data ubiquity*. A measurement ordinarily required by a low-level controller might later also be useful to a planner or diagnostic program. New consumers should obtain it without modifications to the producer, and a replacement producer should not require every consumer to be redirected. Named information, rather than network topology, determined communication.

The physical-control setting shaped the details. Recovery of an old sensor value could be worse than its loss if a newer value was already available. Computation often had to be triggered by fresh data. Several sources could provide one logical item, while consumers required it at different rates. The complete set of participants was not fixed at design time.

A producer therefore had a *strength* and a *persistence*. Strength established precedence among sources; persistence bounded the interval for which the last value from a source retained that precedence. The real robot could have greater strength than the simulator. Once its values ceased and the persistence interval elapsed, the simulator could again supply the effective value. No permanent arbiter was required, nor did a failed producer have to announce its own failure.

This resembles a simplified version of SPLICE's quality-and-decay rule. SPLICE had allowed a quality measure attached to information to diminish with age; Signaal found the generality unhelpful in practice and SPLICE-lite dropped it. NDDS used a simpler step function: fixed strength until the persistence interval expired.

Consumers supplied two complementary time bounds. *Minimum separation* limited notification frequency, allowing a slow consumer to use a fast sensor without processing every update. A *deadline* caused a callback if no fresh update arrived within the stated interval. Notification could be immediate, handled in an NDDS task, or polled in the application's own execution context. Sending could similarly favour immediate delivery or group updates to reduce overhead. These facilities made a coherent answer to distributed control: anonymous typed communication, source arbitration, rate control, failure indication and explicit compromises between latency and cost.

## Named flows and keyed populations

An NDDS data instance had a system-wide name and a registered type. Two instances that any client had to distinguish required different names. Later manuals called this name a *publication topic* and recommended hierarchical strings such as `/sensor/location/type/id`. Pattern subscriptions could select sets of names or types, allowing tools to observe broad classes of traffic without being configured separately for every object.

This resembles a topic in later DDS only up to a point. A SPLICE sort, and subsequently a keyed DDS topic, may contain a population of objects: fields in each datum form a key, so that tracks 17 and 42 coexist within the same sort or topic, while successive values for track 17 remain associated with the same instance. Publicly surviving NDDS manuals confirm my recollection from the 1990s that NDDS then had no support for keys.

Like SPLICE, NDDS maintained administrative databases containing publications, subscriptions and matching state. Utility routines also provided a local object store indexed by type and name. The main difference is that NDDS presented named real-time flows, whereas SPLICE made selected portions of a distributed data space available in receiver-side, keyed and queryable data stores.

NDDS did have histories. A polled consumer could buffer several updates between polls, and reliable delivery required a producer to retain values pending acknowledgement. Send and receive queues, sequence numbers and acknowledgements permitted recovery in order. This was bounded, operational history: it bridged a polling interval or repaired loss in one named stream, rather than maintaining a separate history for every keyed object in a collection. Best effort remained a sensible default for periodic measurements, while commands might require recovery of every omission.

The implementation made connectionless UDP serviceable across heterogeneous workstations and real-time processors, using XDR for typed data and exposing controls for reliability, discovery, multicast and resource use. Its manuals increasingly addressed the practicalities of a commercial middleware product: installation, supported platforms, generated code, deployment and diagnosis. SPLICE-lite, by contrast, was a research implementation whose facilities could remain open to experiment. The difference says more about their purposes than their relative sophistication.

By the end of the decade NDDS had regularised its API around publishers, publications, subscribers and subscriptions. Individual flows carried their own properties, while publishers and subscribers grouped work for efficient processing. The outline would be recognisable in DDS, though neither the data model nor its semantics was yet that of the standard.

## A boundary customers could own

The two DDS lineages overlapped in many ways. Robotics and naval systems were both concerned with timeliness, bounded resources, multiple sources and components appearing or disappearing. Both technologies replaced addresses with typed descriptions and maintained distribution state dynamically. Their different starting points had produced overlapping solutions.

There is documented contact between the ideas by 1995. Pardo-Castellote's Stanford dissertation treats my father's 1993 IEEE article as related work and compares aspects of SPLICE with NDDS, including keys, source arbitration, configuration and update mechanisms. It establishes engagement with the published design by then, not when he first encountered it or influence on NDDS's original design. NDDS already had a history of its own when the article appeared.

The comparison also shows how one design could be read through the concerns of the other. Pardo-Castellote contrasted SPLICE's polling interface and lack of deadlines or multiple update rates with NDDS. From the SPLICE side, these mix architectural and implementation choices. A latest-state store already decoupled production and consumption rates; TACTICOS dealt with supervision above the data space; polling or notification was an access mechanism. The differences were real, but their significance depended on the question being asked.

By the end of the 1990s, customers had another question in common. NDDS was middleware with interfaces owned by RTI. SPLICE was infrastructure beneath TACTICOS, and Signaal/Thales had no ambition to build a general middleware business around it, but important interfaces beneath the combat system remained proprietary to one supplier. Success made both dependencies harder to accept.

The United States Navy's open-architecture work supplied important pressure. RTI's later retrospective describes a Navy testbed evaluating NDDS and SPLICE against Aegis requirements; contemporary presentations connect DDS with the Navy's Open Architecture Computing Environment. The broad institutional connection is documented more securely than every step in the later causal account.

Other open technologies occupied neighbouring ground. CORBA standardised remote invocation, its Notification Service distributed events, and the High Level Architecture supplied a federation model for simulation. None offered quite the application contract and operational combination already demonstrated by NDDS and SPLICE. The OMG became the venue for extracting a common contract from those working systems. RTI and Thales supplied the two principal technological lineages, with contributions from other participants; describing it as their joint work should not erase the others.

## The common model

By the time standardisation began I had left Signaal—and the field—and did not participate in the original negotiations; I returned in late 2010, on joining PrismTech, so this part of the account is reconstructed from documents and participants' recollections.

The proposal was adopted in 2003 and formally published as DDS 1.0 in December 2004. Adoption, finalisation and formal publication were separate stages, which explains the different dates sometimes given for the first standard. Its immediate subject was the service presented to an application. Networking and discovery would require a later specification.

At the centre of the mandatory Data-Centric Publish–Subscribe layer, DCPS, is a rather nebulous *global data space*. Within a domain, a participant contains publishers and subscribers, which in turn contain writers and readers referring to named, typed topics. Compatible readers and writers match without either application knowing the address or identity of the other. The decomposition accommodates NDDS's separation between individual flows and the entities grouping them.

A DDS topic, however, is more than the externally named flow of early NDDS. Fields in its data type can form a key. Values with the same key are successive samples of one *instance*; different keys denote different instances within the topic. One topic can therefore contain the population of tracks, vehicles or sensors that SPLICE represented with a keyed sort. A topic without a key contains a single instance and can stand in for NDDS's named flows.

The two levels of naming—topic and key—accommodate both a publish–subscribe interpretation, in which writers send successive samples, and a shared-space interpretation, in which those samples update replicated values. This works remarkably well. Some later difficulties arise when facilities serving the two interpretations have to operate together.

The standard deliberately stopped short of prescribing an implementation architecture. SPLICE implementations had placed local administration and application data in a common shared-memory database: the hardware left little room for extra copies and context switches. NDDS had evolved libraries, agents, queues and network mechanisms around a different arrangement. DDS specified observable entities and behaviour, leaving layouts, daemons and transports to the implementation.

This was essential to opening the proprietary boundary. A standard requiring one participant to rebuild its product in the image of the other would have defeated much of the purpose. Applications acquired a common vocabulary without suppliers having to agree on every internal mechanism. The first specification was nevertheless incomplete even at that boundary: it used keys without standardising how key fields were designated. Portability was a practical objective, not a promise that replacing a library would always leave every line of an application untouched.

## Quality of service as a treaty

The most conspicuous part of the negotiated design is its collection of quality-of-service policies. Reliability, history, resource limits, durability, deadlines, liveliness, ownership, ordering, lifespan and several other concerns became named settings. The intended division was declarative: applications stated the behaviour they required, while implementations determined how to provide it.

Many policies express requirements already encountered in one or both parent systems. Deadlines and time-based filtering cover periodic production and rate limitation. Reliability, histories and resource limits describe compromises between recovery, buffering and finite memory. Durability concerns availability to late subscribers. Source-timestamp ordering recognises that an observation's time may matter more than its arrival time.

For several policies, a requested/offered compatibility rule compares a reader's request with a writer's offer. An incompatible pair is not matched. That makes the contract checkable without applications naming their counterparts, although distributing and comparing these settings adds substantial machinery. I have serious objections to making this the general model, but its historical role is clear: the policies served as a treaty between products and communities with different expectations.

Assigning each policy to SPLICE or NDDS would be temptingly tidy and mostly speculative. Both had encountered many of the same operational problems. The standard gave those problems common names and negotiated semantics. It also placed an unusually large number of choices at the application boundary. Whether the choices compose into a satisfactory account of a complete system is a separate question from whether each addresses a useful requirement.

## Portability, then interoperability

DDS 1.0 defined no common wire representation. Two products could expose substantially the same application interface while using mutually unintelligible discovery and data protocols. Replacing one might still mean replacing an entire DDS island or inserting a gateway. Source portability had opened one boundary; communicating between independent implementations required opening another.

The OMG requested proposals for an interoperability protocol in 2005. A joint RTI–Thales submission was recommended for adoption in 2006, and the first formal DDS Interoperability Wire Protocol, DDSI-RTPS 2.0, appeared in April 2008. Its numbering reflected an older protocol: RTI had developed RTPS around 2001, and a public Internet-Draft described it in 2002.

DDSI had to specify how participants found one another, how endpoints advertised their topics, types, locations and relevant policies, and how compatible readers and writers communicated. Globally unique identities gave protocol state a stable basis. Sequence numbers, announcements of available history and acknowledgement of received or missing samples provided reliable delivery and repair over an unreliable transport. Large samples could be fragmented and repaired without imposing a separate byte-stream connection for every reader.

This was considerable machinery, but it still left room for different implementations. Products could trade memory against retransmission work provided their externally visible behaviour conformed. Fitting it to established internals was sometimes awkward—OpenSplice's internal identifiers and the protocol identities were not automatically the same thing—but the consequential result was a common network realisation of the application model.

Public interoperability demonstrations began in 2009. These mattered because independent implementations expose ambiguities that review within one code base can leave undisturbed. They showed that the common wire was more than a paper agreement. They did not immediately make heterogeneous systems an ordinary engineering choice.

A single supplier still provided consistent defaults, tools, diagnostics, extensions and support. Mixing products enlarged the test matrix and complicated responsibility for failures. Optional features differed, and type evolution, persistence, filtering and discovery at scale contained corners beyond a successful demonstration.

In my experience it is only in recent years that deliberate integration of several DDS vendors' implementations has become common. From the first standard, or even the adoption of DDSI, this took the better part of two decades. A technical possibility, a demonstrated capability and an ordinary deployment practice are different stages. The interval does not negate the achievement: customers eventually obtained both an application model and a working wire independent of one supplier.

## Reconstructing an application world

The original standard also contained an optional second layer, the Data Local Reconstruction Layer, DLRL. Where DCPS exposed samples and keyed instances, DLRL attempted to reconstruct a local network of application objects, with identity, attributes, relations, inheritance, collections and selections. An object could map onto several topics; generated language objects and a cache were intended to shield applications from that representation.

This came closer to making the reconstructed world itself the interface. Its affinity with SPLICE's database tradition is apparent, though DLRL was neither a transcription of SPLICE nor a small addition to it. Another model, mapping language and body of generated machinery stood above DCPS. Making the layer optional allowed applications requiring direct access to samples to avoid it.

For many years OpenSplice appears to have been the only product with a substantial shipped implementation. Even there DLRL was hardly used, suffered bit rot and was eventually removed. OpenSplice 6.3, released in 2013, put the direction of change into unusually clear relief: its DDSI2 interoperability services became generally available while DLRL was deprecated in preparation for removal. In 2015 DLRL became a separate specification.

This does not establish that applications had ceased to need identities and relations. It establishes that DLRL was not an attractive way to provide them. It required an additional modelling language, generated interface, cache and mapping while leaving difficult details dependent on the application. A relation between two radar tracks may have different rules from one between a component and its sensor or a vehicle and its route. Almost all applications used DCPS directly and constructed precisely the local model they needed, repeatedly solving those problems outside DDS.

DLRL's presence in DDS 1.0 documents an ambition extending beyond sample delivery. Its disappearance records the failure of that particular answer, but that failure doesn't demonstrate that applications no longer need a coherent notion of shared state.

## What grew in its place

DDS did not become less capable. It spread beyond the defence systems that helped bring it into existence. Its protocol matured, its type system became extensible and discoverable, and security acquired a serious specification. DCPS retained a recognisably data-centric core: keys represent populations of objects, reader histories are more than network queues, durability can preserve state for later readers, and queries select by value. DDS never became merely a message API.

Its most vigorous development nevertheless concerned communication and representation. XTypes added structural type descriptions, assignability rules, dynamic inspection and serialisations supporting controlled evolution. DDS Security added authentication, access control, key exchange and protection for discovery, metadata and application data. Language mappings, constrained-device protocols, shared-memory paths and profiles extended the family further.

These are substantial achievements. Long-lived systems cannot reasonably stop every component when a field is added to a structure. Open discovery without authentication is inadequate in hostile environments. Wire interoperability cannot rest on vendors making similar guesses about retransmission. The work also admits objective tests: encodings can be compared, protocol traces examined, cryptographic exchanges attacked and independently developed products connected.

There are limits to what those achievements establish. Type machinery can determine whether one version of `Track` is structurally assignable to another. It cannot say whether altitude is relative to mean sea level, whether a position is measured or extrapolated, or when an old observation ceases to be useful evidence. Authentication and protected delivery cannot establish that an observation is true.

No generic middleware can supply all the meaning of an application. My concern is a different one: the architecture within which applications assign those meanings. SPLICE had attempted to make replication, retention, reconfiguration and recovery parts of one system argument. DDS increasingly provided precise mechanisms whose interpretation and composition became the application's responsibility. The distinction is especially visible in what happened to context data.

## The memory that never became interoperable

DDS defines four durability levels. `VOLATILE` imposes no obligation to retain data for readers arriving later. `TRANSIENT_LOCAL` retains history at a writer for late readers, but the history disappears with the writer. `TRANSIENT` places it in service-maintained storage independent of the writer's lifetime; `PERSISTENT` permits it to survive a restart as well.

The distinction between transient-local and transient is therefore fundamental. One retains an endpoint's history. The other admits a value into storage on behalf of the data space, where it may remain after its producing process has terminated. SPLICE called the latter *context data*, a name that better conveys its purpose.

Consider a restarted process rebuilding the portion of the world it needs. With SPLICE, or a DDS implementation using an independent durability service, it can obtain state from an already populated store. Original writers need not retain and replay their histories, and recovery leaves their ordinary CPU load and network traffic essentially unaffected. In SPLICE's node-wide shared memory, retention did not even require another copy of a value.

Transient-local durability assigns responsibility differently. History belongs to the writer, and satisfying a late or restarted reader involves that writer and the network. The mechanisms may look similar from the receiving application's point of view, but they have different consequences for normal operation, failure and recovery. DDS defines the observable contract rather than this architecture, so the comparison concerns implementations with an independent service, not every conceivable implementation of `TRANSIENT`.

DDSI made transient-local behaviour interoperable. It has never standardised the service protocol required for interoperable transient or persistent data. The question has repeatedly been deferred and remains open. Implementations and proposals exist, but in my recollection agreement was never within reach.

Sending an old sample is easy; defining a distributed service's history is not. Stores can diverge during disconnection, writers can disappear, and a reader can arrive while replicas are being aligned. Retention limits, lifespans, ordering, coherent sets and instance lifecycle all constrain what the service may subsequently supply. A common protocol must make these behaviours agree, not simply specify how to transfer stored bytes. The difficulty is real, and helps explain why a facility present at the application boundary never acquired an equivalent common wire contract.

Several vendors provide useful transient or persistent services within their product families. Proprietary agreement between writer, store and reader is not cross-vendor transient data. The contrast within SPLICE's own lineage is instructive.

OpenSplice always supported transient data through its durability service. State independent of an application writer was native to its architecture but the now commonplace transient-local case initially fitted less well. Its documentation at one time described approximating transient-local through transient storage with disposal and cleanup settings. Proper interoperable transient-local behaviour required a separate writer history in its DDSI2 service.

Eclipse Cyclone DDS presents the converse. As of 2026 the open-source implementation supports `TRANSIENT_LOCAL` but no `TRANSIENT` data. The commercial derivative Zetta DDS does support `TRANSIENT`, necessarily using a proprietary protocol. One feature has become common while the stronger shared-space property remains a product choice.

After almost two decades of work on wire interoperability, independent implementations can discover one another, match their advertised policies, exchange assignable versions of types and protect their samples. They still have no standard means to make a value outlive its writer in a shared transient store. For a standard descended partly from SPLICE, this is a particularly telling boundary to have left closed.

## Success on the narrower question

There were good reasons for concentrating on protocols, types and security. These problems recur across industries and can be standardised without deciding what a radar track, robot pose or battery cell means. A generic object layer risks being either empty or oppressive and DLRL illustrated the difficulty. Vendors, users and standards committees could reasonably favour work with clear boundaries, measurable results and immediate demand.

That explanation does not remove what was lost. SPLICE did not prescribe a naval ontology. It proposed an architectural perspective from which timeliness, consistency, availability, bounded resources, reconfiguration and recovery could be considered together. Its middleware supported that perspective. The failure of DLRL does not refute it, and the absence of interoperable context data cannot be reduced to a missing packet format.

DDS remains one of the few middleware families combining typed data, keys, discovery, multicast, reliability, real-time controls and implementation independence in a reasonably coherent whole. Opening both the application interface and the wire was a substantial achievement. Yet it answered a narrower question than the one from which SPLICE began. Applications still require an account of the state they share: what survives a producer, what makes information authoritative, and what recovery means for the system as a whole.

Even *architecture* has largely changed its referent in this discourse. It commonly describes a middleware's internal organisation, a deployment topology or a selection of components. It less often denotes a principle by which one might argue that the resulting system satisfies several non-functional requirements together. Vendors cannot be blamed for this in isolation. Users have shown little more appetite for the question: latency is easy to rank, while architectural coherence is harder to specify or benchmark.

The result is a technically richer middleware family with a poorer public account of how complex systems ought to be built. That is what I mean by the space becoming a bus. The architectural question remains, even when the means of communication have become very good indeed.

------------------------------------------------------------------------

# 3. ECLIPS: returning to the architectural question

This year I have returned to ECLIPS and started trying to implement it. The work is a proof of concept, centred on the heralds that maintain the data space and the seven—or eight—operations through which an application uses it. It remains exploratory, but it is starting to look as though ECLIPS is at least implementable. Sixteen years after writing the design down, that is an interesting place to be.

It also provides the second occasion for this history. Forty years after those early SPLICE presentations, I am again working on the question from which it began: how should a distributed system be structured so that useful properties follow from its architecture? SPLICE answered that question with a selectively replicated shared data space. DDS standardised much of the machinery, accommodating another lineage and eventually providing practical wire interoperability. ECLIPS asks what the data space itself must represent if it is to carry more of the architectural argument.

The questions become concrete as soon as ordinary operation is interrupted. A producing process disappears: where does its state survive? An object is deleted: what happens to an old reference to it? Two disconnected systems continue operating and later meet: which of their decisions and identities can be reconciled? A process claims authority: who else must agree before it acts? These are related problems of system construction, although middleware commonly presents their supporting mechanisms separately.

ECLIPS attempts to bring them into one model. Whether that model is sound and economical remains to be established. The present experiment makes the attempt tangible; it does not yet supply the conclusion.

## A long gestation

The ideas began well before the later DDS developments described in the preceding instalment. In the late 1990s I was thinking about the limitations of a flat data space, the awkwardness of generic durability, the confinement of mutually distrustful subsystems, the removal of keyed objects and the joining of systems whose states had diverged. These were related doubts and possibilities, not yet a design.

Other work took precedence for much of the following decade. With some time in 2008 and 2009, I finally managed to fit the pieces together. In May 2010 I submitted the long paper, *ECLIPS, a distributed data space with induced structure and capability-based protection*, to *ACM Transactions on Computer Systems*. A much shorter version appeared later that year at the IASTED conference on Parallel and Distributed Computing and Systems in Marina del Rey.

The long paper was explicit: “ECLIPS currently is but a thought experiment.” There was no implementation, performance study or formal proof. No wonder it got rejected! It argued that formalisation should precede implementation because it might reveal a fundamental error before considerable effort had been spent, while implementation would still be required to investigate performance. In particular, testing alone could not establish the security claims.

The assumed environment was deliberately bounded: a local-area network of modest size, perhaps a few hundred nodes on Gigabit Ethernet, with a known upper bound on network latency. That was unfashionable even then, amid enthusiasm for clouds and wide-area systems, but it remained a natural setting for operational systems whose requirements concerned availability, bounded resources and correct behaviour. The design did not claim to work unchanged across millions of intermittently connected devices.

My father patiently listened while the ideas were still nebulous and scrutinised several drafts. ECLIPS was my design, subjected to unusually well-informed criticism by the inventor of SPLICE. Its ancestry is therefore personal as well as technical, but it was not simply a further SPLICE implementation. I wanted structure, retention, protection and deletion to acquire meanings within the data model itself. The resulting interface was smaller than DDS’s; the obligations beneath it were considerably more ambitious.

## Another direction: Zenoh

Zenoh provides a useful contemporary contrast. It brings live publication and subscription, stored values and queries to application services under a common model of names and matching. Key expressions select the resources of interest, and the routing system connects the relevant publishers, subscribers and queryables. The same naming model can span constrained devices, edge systems and cloud services across heterogeneous network topologies.

This includes access to state, not just the next publication. A Zenoh `get` can obtain a reply from storage or from a queryable that computes an answer. The operation reaches out to providers of information. An ECLIPS read, by comparison, examines a local store whose contents are maintained according to the rules of the data space. Both can give an application access to existing information, but the obligations behind that access are different. A shared name through which information can be requested is not quite the same as a relationship among replicas that must have received particular states.

I see the broader contrast as a choice about how much the participants must agree upon. Zenoh makes a common naming, matching and routing model useful across a wide range of participants, without saying much about the semantics. ECLIPS specifies more of the relationships among retained states, object lifecycle and authority, but in a much narrower assumed environment. A small common contract can make participation easier, a stronger contract can support more deductions about what a participating system means. Neither choice removes the questions outside its contract, and this comparison says little about the size or sophistication of the machinery a modern implementation needs.

I was involved in Zenoh's early development: Angelo Corsaro initiated the work, we refined the early designs together and I implemented an early version ("Zenoh-He" or "Zhe"). Subsequent development was done by colleagues in the French office led by Angelo, while I concentrated on Cyclone DDS and eventually withdrew from the Zenoh work. I recognise the attraction of its direction as well as that of ECLIPS. The comparison is between architectural choices, not a claim that one design descended from the other or ought to replace it.

## A graph of information flows

The familiar starting point is typed, keyed data. An instance describes the state of the object selected by its key, and a newer value may replace an older state of that object in a local store. Applications query these stores. Replication is selective and asynchronous, with eventual rather than transactional consistency.

The difference is that the structure controlling replication becomes an explicit part of the application model. A process publishes at a *nabla* and reads from a *delta*. Nablas and deltas are vertices in a dynamic directed graph, alongside neutral vertices that neither publish nor store data. Each nabla and delta belongs to one data *sort*, principally a type combined with a key. A value published at a nabla can contribute only to deltas for that sort that are reachable through the graph.

This graph describes permitted information flows, not cables, switches or hosts. Moving a process to another computer need not mean changing the architectural boundary through which its information passes. Conversely, two processes on the same computer need not share access to the same state. The graph expresses a relationship between publications and stores that is distinct from the machinery carrying updates between them.

DDS domains and partitions divide a largely flat communication space by names. ECLIPS can reproduce such divisions: a neutral vertex can represent a partition, with publications connected towards it and subscriptions away from it. It can also represent gateways, confined subsystems and selective routes. Instead of inferring these relationships from conventions for naming topics and partitions, an application can make them part of the space’s structure.

A process asks a *herald* to perform operations on its behalf. The herald maintains its local stores and the internal state required by the model, exchanging updates with other heralds. There is no single place containing the shared data space. The abstraction arises from those stores, their update rules and the graph connecting them.

Nablas and deltas resemble publications and subscriptions, but identifying them with messaging endpoints too quickly loses the point—hence why I gave them such abstract names. A delta has stored state, a nable can update the state of a keyed object, and changing the graph can change what a store is obliged to know. Delivering the next message is only one part of maintaining that model.

## Where memory belongs

Consider a restarted process that needs to recover the current state of the system. If that state survives only in the histories of its original producers, recovery depends on those producers still existing and being able to provide it. If other stores retain it independently, the process may recover even though the producers have gone. The distinction between DDS’s transient-local and transient durability, and SPLICE’s context data before them, concerns this allocation of responsibility.

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

Suppose a network partition divides an operating system. Each fragment may continue processing sensor input and issuing commands. When connectivity returns, the fragments contain the results of two histories with a common origin, rather than delayed copies of one history. A newly detected object may have acquired different identifiers on the two sides, while two different objects may have acquired the same identifier. Decisions and physical actions made in the meantime remain part of what happened.

Transport repair cannot determine what these states ought to mean together and a generic last-writer rule cannot decide whether to identify, preserve or retract an object. The same difficulty arises without a common ancestor when independently administered systems join a coalition: equal names may denote different entities, and different names equivalent ones. Merging namespaces does not reconcile their contents.

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

This year’s proof-of-concept work returns to that gap between description and execution. The work is centred on heralds and the small operation set because that is where an application-facing promise must acquire concrete behaviour. The present result is still an exploratory attempt that is starting to look implementable. I cannot yet turn that observation into a claim that the 2010 model has been validated.

One of the problems currently being worked through is deciding when an object has really disappeared. Finding no copy in the stores examined so far is insufficient: a publication may still be waiting for delivery, or retained state may still be on its way to another store. The implementation has to account for that work before drawing a system-wide conclusion from local absence. This is the kind of detail that makes a graph induced by its contents much harder to implement than to describe.

An executable model forces such questions into the open. The context invariant has to survive graph changes, while the objects describing that graph are themselves being distributed. Retention and obsolescence must interact with local removal and delayed updates. Making operational choices about these interactions may preserve the design, require a different formulation or reveal an error.

Proof and measurement remain separate tasks. A convincing demonstration cannot cover every ordering of concurrent structural changes, and passing examples cannot establish confinement. Conversely, a formal argument for the model would not show that its implementation is economical. The general `label` operation is a particularly clear reminder that a useful semantic promise and an affordable mechanism are different achievements.

It is also possible that only parts of ECLIPS deserve to survive: topology-derived retention, explicit finality, private-to-global identity mappings, weakening edges, or a single costly agreement operation among otherwise asynchronous primitives. Exploring the complete design gives those ideas a setting in which their interactions can be examined. It need not end in preserving every decision made in 2010.

That makes this a suitable place to pause the history, although the experiment continues. ECLIPS did not succeed SPLICE as an operational system or displace DDS. It returned to the architectural question my father had pursued: what structure allows timeliness, consistency, bounded resources, redundancy, recovery and evolution to be considered together? The answer may prove too costly, incomplete or mistaken. Trying to implement it is a way of subjecting that answer to another kind of criticism.

Forty years after SPLICE went public, moving bits correctly, securely and quickly remains indispensable. The larger question remains, too: what state does the system represent, where may it propagate, how long does it remain valid, and which process has authority to change it? This year's return to ECLIPS is an attempt to give those questions another concrete answer.

------------------------------------------------------------------------

# Sources and open questions

These notes cover all three instalments. Contemporary documents, later recollections and my own interpretation do different work in this account. The references below identify the principal evidence; the remaining gaps are collected at the end so that they need not repeatedly interrupt the narrative.

## SPLICE: documents and recollections

The starting documents are Max J. Ceuleers’s “Naar een andere Systeemopzet: ‘Flexibele Standaardisatie’” (Memo SEAT 335, Hollandse Signaalapparaten B.V., 4 January 1984, 6 sheets) and Maarten Boasson’s “Systeem Architectuur” (Memo SEAT 362, 29 February 1984, 21 pages). The former records the industrial problem and institutional backing; the latter describes the DDBM architecture. Maarten’s later “[Database system](https://patents.google.com/patent/EP0271945B1/en)” patent (EP 0 271 945 B1; related [U.S. Patent 5,301,339](https://patents.google.com/patent/US5301339A/en)) describes a more developed design and names him as sole inventor.

The three American venues in the spring of 1986 are recorded in my [2010 ECLIPS manuscript](eclips-20100525.pdf), which Maarten reviewed. They are the earliest presently identified presentations outside Signaal, not a precisely dated first publication of a demonstrably complete SPLICE system. The earlier chronology and the origin of the name rest partly on family recollection.

The main published accounts of the architecture are Maarten Boasson, “[Architecture of Real-Time Systems](https://doi.org/10.1007/978-1-4612-4476-9_6),” pp. 44–53 in W. H. J. Feijen, A. J. M. van Gasteren, D. Gries and J. Misra (eds.), *Beauty Is Our Business: A Birthday Salute to Edsger W. Dijkstra* (Springer-Verlag, 1990); “[Control Systems Software](https://doi.org/10.1109/9.231463),” *IEEE Transactions on Automatic Control* 38(7), July 1993, pp. 1094–1106; and J. H. van ’t Hag, “[‘Data-centric to the max’, the SPLICE architecture experience](https://doi.org/10.1109/ICDCSW.2003.1203556),” *ICDCS Workshops 2003*, pp. 207–212. The surviving SPLICE user and reference manuals from 2000 and 2001 document the later facilities.

Maarten’s inaugural lecture at the University of Amsterdam on 18 October 1996, [*Het onmogelijke duurt iets langer*](https://albumacademicum.uva.nl/id/id001933), supplies both the broader architectural argument and the acknowledgements of Max Ceuleers’s and Herman Driessen’s roles. My English translation, [*The impossible takes a little longer* (PDF)](The%20impossible%20takes%20a%20little%20longer.pdf), is included in this repository. The approximate 1989 date, identification with TACTICOS and rejected alternative architecture rest on my information; the lecture independently confirms Driessen’s decision to adopt Maarten’s architecture for a new product generation.

The formal work includes Marcello M. Bonsangue, Joost N. Kok, Maarten Boasson and Edwin D. de Jong, “[A software architecture for distributed control systems and its transition system semantics](https://doi.org/10.1145/330560.330664),” *ACM Symposium on Applied Computing 1998*, pp. 159–168; Roel Bloo, Jozef Hooman and Edwin D. de Jong, “[Semantical aspects of an architecture for distributed embedded systems](https://doi.org/10.1145/335603.335731),” *ACM Symposium on Applied Computing 2000*, vol. 1, pp. 149–155; and Jaco C. van de Pol, “[Expressiveness of Basic Splice](https://ir.cwi.nl/pub/4388),” CWI Report SEN-R0033, December 2000. The results concern restricted formal models, not proofs of the complete implementations.

The accounts of shared memory, Ethernet engineering, timestamping and extrapolation, my SPLICE-lite contributions, the abandonment of general quality-and-decay selection, and the Sun and Rijkswaterstaat demonstrations rest substantially on my recollection. Hans van ’t Hag supplied the recollection about the operational need that led to context data. My suggested division between his contribution and my father’s generalisation remains an interpretation, not established individual authorship.

## NDDS and the formation of DDS

The principal early sources are Gerardo Pardo-Castellote and Stan Schneider, “[The Network Data Delivery Service: A Real-Time Data Connectivity System](https://ntrs.nasa.gov/citations/19950005113),” *CIRFFSS ’94*, vol. II, NASA Conference Publication 3251, pp. 591–597, AIAA-94-0889-CP; and “[The Network Data Delivery Service: Real-Time Data Connectivity for Distributed Control Applications](https://doi.org/10.1109/ROBOT.1994.350903),” *IEEE International Conference on Robotics and Automation 1994*, vol. 4, pp. 2870–2876. The first bears a 1993 copyright but was published in 1994. The surviving NDDS 1.11b, 2.0b and 2.1d manuals, carrying copyrights from 1996, 1998 and 1999, document naming, source precedence, timing, buffering, reliability and the evolution of the API. [RTI’s corporate history](https://www.rti.com/products/industry-leadership) supplies the 1991 founding and 1995 commercial milestones.

Pardo-Castellote’s [*Experiments in the Integration and Control of an Intelligent Manufacturing Workcell*](https://web.stanford.edu/group/arl/cgi-bin/drupal/sites/default/files/public/publications/Pardo%2095.pdf) (Stanford PhD dissertation, June 1995; also SUDAAR 675), chapter 4, printed p. 90, compares SPLICE and NDDS. His “[OMG Data-Distribution Service: Architectural Overview](https://doi.org/10.1109/ICDCSW.2003.1203555),” *ICDCS Workshops 2003*, pp. 200–206, cites Boasson’s article in describing DDS and keyed data; van ’t Hag’s SPLICE paper immediately follows it. These establish engagement by 1995 and convergence by 2003. They neither date a first encounter nor demonstrate that SPLICE influenced the original NDDS design.

Customer pressure from both user communities is part of my recollection. Mark Swick’s “[A Sea-Worthy Standard: 20 Years of DDS for the U.S. Navy](https://www.rti.com/blog/20-years-of-dds-for-the-u.s.-navy)” (RTI, 14 March 2024) gives the clearest published account of the Navy’s role, but is a vendor retrospective. Joseph M. Schlesselman, Gerardo Pardo-Castellote and Bert Farabaugh’s “[OMG Data-Distribution Service (DDS): Architectural Update](https://doi.org/10.1109/MILCOM.2004.1494965),” *MILCOM 2004*, vol. 2, pp. 961–967, and their [HPEC presentation of 22 September 2005](https://archive.ll.mit.edu/HPEC/agendas/proc05/Day_3/Presentations/0940_Pardo-Castellote.pdf) document the joint proposal, chronology and connection with Navy open architecture. They do not independently establish every part of the retrospective causal account. I did not participate in the original DDS negotiations.

## The standards and their subsequent development

The principal specifications are OMG, [*Data Distribution Service for Real-time Systems*, version 1.0](https://www.omg.org/spec/DDS/1.0/PDF) (December 2004), and [*Data Distribution Service (DDS)*, version 1.4](https://www.omg.org/spec/DDS/1.4/PDF) (April 2015). These supply the application contract, topics and keys, QoS, durability, lifecycle and ownership. [*DDS Data Local Reconstruction Layer (DDS-DLRL)*, version 1.4](https://www.omg.org/spec/DDS-DLRL/1.4/PDF), and its [catalogue entry](https://www.omg.org/spec/DDS-DLRL/1.4/) document the later separation of the optional object layer.

The wire-protocol history uses [*DDSI-RTPS*, version 2.0](https://www.omg.org/spec/DDSI-RTPS/2.0/PDF) (April 2008), [version 2.5](https://www.omg.org/spec/DDSI-RTPS/2.5) (April 2022), and Stefaan Sonck Thiebaut *et al.*, “[Real-Time Publish Subscribe (RTPS) Wire Protocol Specification](https://datatracker.ietf.org/doc/html/draft-thiebaut-rtps-wps-00),” Internet-Draft, 22 February 2002. Vendor records document the [2009](https://community.rti.com/content/presentation/omg-dds-interoperability-demo-2009), [2010](https://community.rti.com/content/presentation/omg-dds-interoperability-demo-2010) and [2011](https://community.rti.com/content/presentation/omg-dds-interoperability-demo-2011) demonstrations. The much later arrival of routine mixed-vendor systems is my observation, not a date defined by those demonstrations.

The discussion of the wider standards family draws on [*Extensible and Dynamic Topic Types for DDS*, version 1.3](https://www.omg.org/spec/DDS-XTypes/1.3) (February 2020), and [*DDS Security*, version 1.1](https://www.omg.org/spec/DDS-SECURITY/1.1) (2018). The [OpenSplice 6 feature history](https://kbase.zettascale.tech/article/new-features-opensplice/) records DDSI2 general availability and DLRL deprecation in the same release. The assessment of DLRL's limited adoption and bit rot also draws on my own experience.

The absence of a common transient and persistent durability protocol is documented in the deferred OMG issues [DDSIRTP23-26](https://issues.omg.org/issues/DDSIRTP23-26) and [DDSIRTP25-13](https://issues.omg.org/issues/DDSIRTP25-13), and the still-open [DDSIRTP26-1](https://issues.omg.org/issues/DDSIRTP26-1). OpenSplice’s [DDSI2 deployment guide](https://github.com/ADLINK-IST/opensplice/blob/master/build/docs/DeploymentGuide/source/ddsi2-networking-service.rst) and [durability API documentation](https://download.zettascale.online/www/docs/OpenSplice/v6/apis/ospl/isocpp2/html/a01648.html) describe the architectural distinction between transient and transient-local retention. The observations about implementation costs, Cyclone DDS and Zetta DDS, and the uneven use of durability draw also on my implementation and standards experience. Product-status observations describe the situation considered in this 2026 account.

## ECLIPS

The main source is my [*ECLIPS, a distributed data space with induced structure and capability-based protection* (PDF)](eclips-20100525.pdf), a 35-page manuscript dated 25 May 2010 and marked “Submitted to ACM TOCS, 2010-05-25”. A much shorter version, “[ECLIPS, A Distributed Data Space with Induced Structure and Capability-based Protection](https://doi.org/10.2316/P.2010.724-006),” appeared at the 22nd IASTED International Conference on Parallel and Distributed Computing and Systems, Marina del Rey, 8–10 November 2010.

The long manuscript supplies the model, the explicit thought-experiment qualification, its stated limitations, the construction of SPLICE in the appendix and the acknowledgements. It cites DDS 1.2 (2007); the comparisons here also use the later specifications listed above. The chronology of my thinking in the late 1990s and 2008–09, and the account of the current implementation experiment, are my own recollections and assessments. The placement of ECLIPS after DDS in this series is an order of exposition, not a claim that its design followed all the later DDS developments discussed here.

## Zenoh as a contrasting direction

Zenoh’s [overview](https://zenoh.io/docs/overview/what-is-zenoh/) states the aim of spanning microcontrollers through cloud environments while bringing publication, storage and queryable computation under common abstractions. Its [manual of abstractions](https://zenoh.io/docs/manual/abstractions/) describes keys, key expressions, subscribers, queryables and storages; the [Rust API overview](https://docs.rs/zenoh/latest/zenoh/) describes publish/subscribe, query/reply and byte payloads. These support the description of the common access model. The contrast with ECLIPS—broader reach through fewer universal semantic commitments, compared with stronger relations among local replicas in a more restricted environment—is my interpretation of the designs, not a claim that one descends from or supersedes the other.

The account of Zenoh’s beginnings is my recollection: Angelo Corsaro initiated the DDS-XRCE process and drafted an early Zenoh design; I proposed a more compact DDSI-RTPS which went nowhere, then joined Angelo in refining Zenoh and implemented early versions, particularly Zhe. Much of the subsequent work was done by colleagues in the French office, with progressively less input from me as I concentrated on Cyclone DDS and eventually withdrew. The contemporary Zenoh design should not be attributed to me on the strength of that early involvement. Nor does the resemblance between Zenoh’s hierarchical naming and early NDDS establish descent from NDDS.

## What remains to be established

The gaps fall into four groups:

- **SPLICE’s early chronology.** Evidence of work before formal backing; the first prototype and its platform; the relationship of SigMA and the remembered Sun 3 and Atari Transputer experiments to operational TACTICOS; the transition from hardware DDBMs to software agents; and the first dated uses of sorts, keys, worlds and context data. The precise dates and contents of the 1986 presentations remain particularly valuable for the anniversary account. Contemporary notes, correspondence, diagrams, photographs or software could change this reconstruction.
- **The intermediate documentary record.** The 1990 chapter cites Maarten’s “Software design and system architecture” (NATO workshop, Brussels, 1987), “[Modeling in real-time systems](https://doi.org/10.1016/0920-5489(87)90051-1)” (*Computer Standards & Interfaces* 6(1), 1987, pp. 107–114), J. A. Droppert, H. B. M. Jonkers and H. M. H. Loomans’s *A COLD-1 Specification of SPLICE* (Philips NatLab, August 1989), and R. van der Land’s *SPLICE Routing* (Leiden, July 1989). Their recovery would narrow the gap between the early proposal and the mature descriptions. Contemporary Ethernet-loading calculations, timestamping-board documentation and records of the Sun and Rijkswaterstaat demonstrations would also help.
- **NDDS and the standardisation decisions.** Early manuals and release notes could date API changes and the introduction of keys more exactly. Contemporary Navy evaluations and programme records could establish the decisive requirements and the contributions of customer representatives. The available record is better at naming corporate participants than at assigning particular parts of DDS and DDSI to individuals.
- **The limits of the historical interpretation.** A product-by-product account of transient and persistent support, the unsuccessful durability proposals, and evidence of substantial early DLRL implementations beyond OpenSplice would strengthen or qualify the account. Examples of how applications actually handle identity, validity, authority, failover and reconciliation would test the architectural criticism more directly than another list of middleware features. ECLIPS itself still needs evidence about correctness and cost beyond the design argument and an exploratory implementation.
