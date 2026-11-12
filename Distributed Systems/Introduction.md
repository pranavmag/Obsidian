2026-11-10 21:50

A distributed system is where multiple computers communicate over a network to get a task done like peer-to-peer file sharing. If a problem can be solved on a single computer then it's best to do it that way because distributed systems are harder to build and come with their own problems so it should be built if the system needs it.

#### Why Build One?

1. Performance (parallelism) - Many CPUs can be used
2. Fault Tolerance - Many machines are doing the work so if one fails just cut over to another
3. Many systems me be at different locations so if an system needs coordination with many physical locations it falls under distributed systems
4. Security / isolation - split untrusted or buggy code onto separate machines that interact only through a narrow, well-defined network protocol.

Distributed systems are hard to build because of the first two problems as well as performance bottlenecks. Many parts executing at the same time on multiple machines and machines could fail at any time. Sometimes the machines are working fine but the network that they all communicate across could be slow or down.

#### Implementation Tools

1. Remote Procedure Call (RPC) - makes a call to another machine look like a local function call
2. Threads - let a program use multi-core machines and have a structured way for concurrency
3. Concurrency Control (locks) - needed because threads share state and the locks coordinate concurrent access so no data is corrupted

#### Performance and Scalability

Ideally 2x the computers gives 2x the performance or throughput. For example if you're scaling a website we start with one browser, one web server, and one database but as traffic grows we can add more web servers and at that point the database becomes a bottleneck and you need more databases.

Scaling up can introduce issues like machines failing. Fault tolerance is extremely important and has to be built into the design itself. The two types of fault tolerance are availability and recoverability. Availability means that a system can keep providing a reliable service and recoverability means that even though a system may stop it will resume correctly with no loss of data once it is restored. A good available system is also recoverable.

The main tools are non-volatile storage and replication (multiple copies of state).

Consistency is also extremely important. If we have like a key-value store where replica A puts data in but the client crashes before the data reaches replica B then replica B is potentially reading stale data if it does a GET operation on that data.


