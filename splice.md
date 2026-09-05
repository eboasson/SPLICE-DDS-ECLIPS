This is part one of three. The other parts are the [route to standardization](dds.html), and a [plea for aiming higher](eclips.html).

# SPLICE: forty years of an architectural idea

In the spring of 1986, my father, Maarten Boasson, presented SPLICE at New York University, Philips Research U.S.A. and General Electric Research. These were the earliest presentations outside Hollandse Signaalapparaten, his employer.

SPLICE was his software architecture for constructing a system whose functions can be developed independently, moved between machines and recovered after failure, and without making every function understand the arrangement of all the others. It made applications describe the information they needed and produced. A supporting infrastructure would take responsibility for finding it, distributing it and keeping it available. That sounds familiar now. Its consequences are still less familiar than the communication mechanism through which it is usually described. To see them, it helps to go back two years, to a pair of memoranda in Hengelo.

## The industrial problem

Hollandse Signaalapparaten (Signaal) constructed complex real-time command-and-control systems for naval use. By the early 1980s, adding functions in software had become relatively easy, but integrating the resulting collection of functions had not. A memorandum written on 4 January 1984 by Max Ceuleers, head of the department, describes the consequences clearly.

Logically concurrent functions had been divided into small tasks and scheduled sequentially on a central computer. Their behaviour consequently became entangled through timing, interrupts and shared memory. Modifying one function could perturb the carefully debugged timing of others. Functions carried over from one delivery were modified and tested again for the next. Once the central computer ran out of capacity, functions were simplified or transferred to additional microprocessors. The result was “wildgroei”: uncontrolled and unmanageable growth. (In those days, documents like these were still written in Dutch in The Netherlands!)

Ceuleers proposed assembling systems from recognisable functional modules with well-defined responsibilities. Proven functions should be reusable, and individual deliveries should be configurable without becoming further variants of a monolithic program. The hard question was how independently usable modules were to communicate. Giving each module a clean point-to-point interface to its expected collaborators would merely replace one form of coupling by another: the software would still encode the configuration in which it happened to be deployed.

On 29 February 1984, Maarten circulated a memorandum entitled *Systeem Architectuur*—“System Architecture”. Its subject was communication between subsystems and dynamic reconfiguration. An application requiring information would request data of a particular type; one producing such information would supply data carrying the corresponding type indication. An intermediate layer would locate a source, register a continuing interest, distribute subsequent updates and retain received values locally. Applications neither addressed one another nor represented the current assignment of functions to processors.

Much of the proposed machinery belongs unmistakably to its time: small processors with local ROM, shared buses, Ethernet, duplicated cables and dedicated processors managing distributed databases. The essential abstraction does not. The aim was to meet functional independence, timeliness, resource bounds, fault recovery and continued evolution together. Communication machinery existed to support that architecture.

Ceuleers supplied the industrial diagnosis and organisational backing; Maarten supplied the data-oriented architecture. It would be misleading, though, to imagine the answer being invented from a blank sheet in the eight weeks between the memoranda. My father’s recollection placed his first thoughts in the late 1970s or very early 1980s. The work appears to have remained unofficial for some time, until practical difficulties and a sufficiently developed proposal justified a formal study. In his later inaugural lecture, he thanked Ceuleers for allowing the investigation to continue despite opposition within Signaal.

## Removing the other component

Suppose a component A obtains track information from a component B. (This example and terminology are modern, but the distinction is the memorandum’s.) If A’s interface contains B’s identity, address and protocol, it represents the current decomposition of the system. Interposing a directory changes only the resolution mechanism. Replacing B, moving its function or introducing another source still changes something about which A has made an assumption. The February design removed B’s identity from A’s interface.

Each functional subsystem would be associated with a Distributed Data Base Manager, or DDBM. A subsystem requiring information not produced locally requested the appropriate type from its DDBM; the request was relayed to the other DDBMs. If a source existed, the data were returned. Otherwise the requester received an indication that the information was unavailable, leaving diagnostic functions to determine the cause.

Producers need not enumerate consumers, consumers need not identify producers, and equivalent information could come from several sources. Physical routes and distribution state still existed, of course. They belonged to the infrastructure and system-management functions. An ordinary application represented its function and the information it supplied or required, allowing sources to be duplicated, replaced, moved or divided without changing the consumer’s statement of need.

The design also went beyond a remote lookup. On satisfying a request, a DDBM registered the requesting DDBM “as subscriber” to the relevant type. Subsequent values could then be distributed automatically to the registered destinations. A subscription arose from reading data: the distribution layer recorded the continuing interest and maintained a local copy. Applications read from local stores that network traffic updated selectively.

Values carried validity periods, so stale data could be recognised. Production time and quality were explicit properties, and where several sources supplied equivalent information, the DDBM could select among them according to a criterion established during initialisation or reconfiguration. A DDBM also recorded the last local use of a type and could send a special message when distribution was no longer required.

Reconfiguration followed the same division of responsibility. Diagnostic functions would identify failed processors; reconfiguration functions could move work to reserve equipment or stop a less important function to preserve a more important one. The memorandum observes explicitly that other processors need not be notified when a reserve processor assumes a failed processor’s task. Once the replacement requested and supplied the appropriate types, the DDBMs could establish the flows.

This did not solve failure detection, capacity allocation or the ranking of functions. It confined configuration-dependent reasoning to the facilities responsible for configuration. Applications did not each require their own recovery wiring. The same abstraction addressed integration during development and physical reorganisation during operation.

## A database assembled from replicas

Back then, it did not yet use the name SPLICE. Nor did it contain the complete later model: sorts, worlds and context data were still absent, and DDBMs appeared principally as dedicated hardware. Reading the mature architecture back into 1984 would make a tidier history, but a less accurate one.

By the 1986 presentations, the proposal had acquired its name. It began with the ordinary verb *to splice*: to join, with the useful suggestion that streams may also be split and joined. The expansion, **Subscription Paradigm for the Logical Interconnection of Concurrent Engines**, came afterwards, according to family recollection. The dedicated processors eventually gave way to software accompanying ordinary applications, while the distinction between functional data and the supporting machinery survived.

In SPLICE, a subscription caused the relevant data to be maintained in a store local to the application. Distribution updated the store asynchronously and the application queried it when its computation required data. The shared database was an abstraction constructed from partial replicas with no single machine containing the authoritative whole and no synchronous transaction making all copies identical at every instant. Each application obtained the portion selected by its subscriptions and observed updates after finite, variable delays. Within those limits, a system could be designed against one information space instead of a changing diagram of processes and connections.

The unit of information became the *sort*: a name, a structured type and key fields. Several objects of one sort could coexist, distinguished by their keys, while a later value with the same key would replace an object’s previous state. A track sort, for example, could contain many tracks distinguished by track number. *Worlds* provided logical scopes: the same sorts could be used for live operation, training, test or replay, with publications and subscriptions meeting only where sort and world agreed.

There remained a difficulty that ordinary subscriptions did not solve. A process might require information published before it had started (or restarted). If distribution took place only in response to current subscriptions, the process might receive nothing until every producer happened to write again. Some state had to be retained in anticipation of future subscribers. SPLICE called it *context data* and implemented retention using replicated daemons that automatically subscribed to all context data and made it available to ordinary applications.

That placed responsibility outside the ordinary writer. A restarted process obtained current state from an already populated store and the original producers did not have to replay it. Under normal operational conditions this imposed essentially no additional network traffic or producer CPU load. Because SPLICE was usually implemented with one common shared-memory store on each machine, retaining a value as context did not require a second in-memory copy either.

Hans van ’t Hag, who joined Signaal in 1983 and later worked on the middleware beneath TACTICOS, remembers recognising the need to retain data even when no current application required it. My guess is that Hans identified the practical problem and my father generalised the answer and drew its architectural consequence. After four decades, I would not turn that guess into a firm division of credit. The consequence itself is clear: retention became independent of current application subscriptions, giving the shared data space a life beyond its present users.

## Choosing it for TACTICOS

Around 1989, Signaal’s technical director Herman Driessen chose SPLICE as the foundation of TACTICOS rather than the “modular architecture” then being developed. In his 1996 inaugural lecture, Maarten thanked Driessen for choosing his architecture for a new generation of Signaal products and maintaining the choice in the face of considerable resistance. He added that Signaal had done well by it.

By the early ’90s, the architecture was being tested in a full-scale naval command-and-control system. The 1984 memorandum had settled the central abstraction but the engineering still had to settle representation and identity, subscription administration, reconstruction after restart and operation within severe bounds on processor time, memory and network capacity.

My father’s 1990 chapter “Architecture of Real-Time Systems” gives the argument in concentrated form. Conventional communication makes a module’s interface encode another module and assumptions about the timing of their interaction. A process should instead state which data are to be communicated and leave their production, location and delivery to the supporting system. The chapter appeared in *Beauty Is Our Business*, the volume for Edsger W. Dijkstra’s sixtieth birthday. That connection was personal too: a 1979 summer school in Santa Cruz had led to my father’s membership of Dijkstra’s Tuesday Afternoon Club in Eindhoven and to a lifelong friendship.

## Ten megabits, used carefully

TACTICOS had to combine sensor observations, ship state, track processing, tactical functions and operator displays, and manage something of the order of a thousand tracks. Around 1990, the available network was 10 Mbit/s Ethernet, duplicated using two cables for redundancy. Duplication protected against a cable or physical-network failure; it did not double the usable capacity.

This was shared, half-duplex Ethernet. Rising load brought more collisions, retries and variability. I remember a design rule that limited utilisation to a fairly low percentage, on the argument that collisions were then practically negligible. I do not recall the exact figure. Something of the order of two megabits per second was all the design could safely treat as available.

Messages were kept compact and, where possible, were combined to fill network packets instead of paying the framing overhead for every small update. Ethernet was a broadcast medium, but the subscription administration allowed data to be efficiently ignored wherever it was not needed. Local reads did not generate network requests and replies.

Low utilisation could not eliminate delay or jitter. For a moving target, the position in a newly arrived packet was already a position in the past. Special Ethernet boards inserted timestamps as packets were streamed onto the wire, allowing accurate clock synchronisation. This allowed a subscriber to use position and velocity at a known instant to extrapolate a track to the time required by a display or another function.

## The local store

Almost all SPLICE implementations reserved one central place on each machine: a block of shared memory containing both the local administration and the local data. (I believe there was *one* exception: an investigation into implementing it on transputers.) All processes operated on it directly. The conceptual database was distributed across machines, but its local realisation was shared rather than dispersed among private queues.

On the processors available for TACTICOS, repeated copying, general-purpose inter-process communication and additional context switches would have made the complete system infeasible. The common store allowed local publications, subscriptions and queries with very little movement of data. It was a practical requirement for running TACTICOS on the hardware of the day.

The cost was reduced isolation within a machine. The block was a shared point of failure, and a process capable of corrupting it could damage more than its own state. Later DDS vendors were right to raise that objection. Assessing it requires looking at protection, detection and recovery as well as process boundaries.

There is a useful distinction here between the information model and its implementation. Many DDS implementations later acquired shared-memory transports to avoid the network stack and, where possible, serialisation and copying. Those transports need not reproduce the single SPLICE block containing administration and data. Eclipse Cyclone DDS, in some sense a descendant of SPLICE, eventually abandoned the inherited node-wide design, at least for a time. A common local store had a compelling historical rationale; it was not a requirement for the shared-data-space abstraction.

## Recovering information as well as processes

Separating information from process lifetime matters as much as the freedom to place processes on different machines. Starting a replacement executable is only part of recovery. It must also acquire the state needed to do useful work, while the surviving system must recognise the information it produces. Context data addressed the first problem; stable data identity and anonymous flows addressed the second. Diagnostic and reconfiguration functions still had their own work to do, but ordinary applications did not have to reconstruct the system’s connections themselves.

Some refinements did not survive experience. SPLICE allowed equivalent values to be selected through a quality measure that could diminish with age. Signaal found that the general mechanism did not work usefully in practice. Timestamps, validity periods and application-specific physical models remained useful. The unsuccessful part was the attempt to reduce the merits of a datum to one general quality figure.

Over the following decade, experience accumulated in fielded naval combat-management systems. Performance, extensibility and fault tolerance became properties exercised across delivered systems and successive generations of equipment. The architecture also returned to the laboratory, where I became directly involved in its implementation.

## Rebuilding it

In 1995, Hollandse Signaalapparaten contracted me to write a new SPLICE implementation. I joined the company in 1996 and continued as its lead developer. The result, *SPLICE-lite*, served as a research instrument in which we could explore the consequences and limits of the architecture.

Most of it was my work. Typed and keyed sorts, worlds, publications and subscriptions, receiver-side stores, queries, and periodic, context and persistent data came from SPLICE. Category subscriptions, resources and pluggable databases were mine, as were the built-in topics it added. Multisorts were probably an older idea, but I believe mine was the first implementation usable in practice.

The API made the database interpretation literal. A subscription created or attached to local storage; incoming data altered it; a read selected values by a predicate. Depending on the choice of key, a sort could represent a current value, a set of objects or a history. A multisort joined sorts on corresponding keys, bringing together independently produced parts of one conceptual object without requiring them to share one structure, update rate or retention policy. Position, classification and threat evaluation could remain separate because their production was separate, while being queried together as attributes of one track.

The built-in topics applied the same model to SPLICE’s own administration. Participants and relevant data-space entities were represented within the data space, making ordinary distribution, retention and observation available for administration too. A diagnostic application could use the information architecture to examine the infrastructure providing it. This idea would matter later in DDS and ECLIPS.

The other additions explored how far the model could remove unnecessary dependencies. Category subscriptions let a recorder or diagnostic program select classes of present and future sort-world combinations, without enumerating all those known when it was written. Resources dealt with cases where a client really did need a provider to perform a service or control scarce equipment: claiming a named resource established private data flows without putting the provider’s location into the client.

Pluggable databases made the receiving store itself replaceable, with its own insertion, query and waiting operations. That freedom came with real risks because plug-ins participated in the common memory and locking discipline; a serious error could damage the node-wide service. Parts remained experimental. SPLICE-lite was useful partly because it let us investigate such boundaries in working software.

## Pulling plugs, crashing cars

My colleague Jan-Willem Lokin built an air-defence demonstration on SPLICE-lite. Norm Howes of the Institute for Defense Analyses subsequently used it for presentations to American military audiences. Processes could be introduced, duplicated, stopped or restarted while the tactical picture remained expressed by the same data objects. It made the architecture much easier to explain.

At one presentation on a Sun Microsystems site, we showed the demonstration to engineers from Raytheon and invited them to choose a machine and pull its plug whenever they liked. Before they did so, an X server froze. This surprised us as much as it did them. I decided the machine with the frozen display was as good a candidate as any, and pulled its plug.

The processes that had run there were relocated automatically. Everything else continued, without an operator reconfiguring the applications and—especially to the visiting engineers’ surprise—without the track identifiers changing. That was not cosmetic: preserving a display while replacing every track with a newly identified one would have destroyed continuity for computations and operators using those identities. An accidental display failure had given us a more persuasive demonstration than the planned one.

Around 1998, we discussed another application with Rijkswaterstaat: monitoring road traffic and distributing current traffic information to the matrix signs above and beside Dutch roads. I wrote a demonstrator in which each car was an independent process, proceeding autonomously along a Möbius-strip multi-lane motorway. Merging relied on the ugly hack of letting cars drive along the shoulder until they “dared” to join the traffic. The occasional collision furnished useful work for the monitoring functions.

The demonstrator also used SPLICE-lite’s generalisation of context data and resources to explore application-controlled retention. This took the independence of retained information from its producer a step further: an application could participate in deciding what to keep, instead of leaving that decision entirely to a generic context daemon. The distinction would later become important to ECLIPS, where the location and policy of retained state became programmable parts of the data-space model.

## The architecture behind the machinery

In his inaugural lecture at the University of Amsterdam on 18 October 1996, *Het onmogelijke duurt iets langer*—[“The impossible takes a little longer”](The impossible takes a little longer.pdf)—my father stated the proposition in more general terms. Subordinating structure to functional decomposition forces the relation between each pair of functional components to be specified and implemented separately, introducing avoidable complexity. One should first design a global architecture into which the functional components fit.

Reuse consequently depends at least as much on the architecture as on the components. A component must be specifiable independently; properties of the whole then follow from those of the components together with those of the “supporting and connecting architecture”. SPLICE gave the argument concrete expression.

SPLICE-lite also made the design available for formal treatment. Work with Dutch universities and CWI reduced it to variants of *Basic SPLICE*, comparing an ideal global data space with implementations using asynchronously updated local sets. The papers established equivalence and expressiveness results for carefully restricted cores, rather than correctness of complete implementations. They helped explain how a small set of operations could sustain a shared-data-space abstraction without a central server or a general agreement protocol.

By the end of the 1990s, SPLICE had supported operational combat-management systems, generated a substantial research implementation and admitted a useful formal account. Signaal, and later Thales, nevertheless had no ambition to establish a general middleware business around it. Customers began to object that TACTICOS depended on an interface and development path controlled by one supplier. SPLICE had succeeded well enough to become infrastructure, and that success created pressure to open its boundary.

Meanwhile, RTI’s NDDS had reached neighbouring ideas from the world of robotics. Those two routes would meet in DDS.

# Sources

A list of [sources](sources.html) is available. Much of the early history rests on recollections and a few documents my father had kept.
 
*Copyright © 2026 Erik Boasson. Dated 13 September 2026. Licensed under the [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).*
