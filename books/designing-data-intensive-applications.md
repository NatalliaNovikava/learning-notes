#Foundations of Data Systems
## Reliable, Scalable and Maintainable applications

A data-intensive application are typically built up from blocks.
- store date (database)
- remember the result of expensive computation (caches)
- allow users to search by key words (indexes)
- send a message to another process (stream processing)
- periodically crunch a large amount of data (batch processing)
- Reliability. To work correctly even in the face of adversity
- Scalability. Reasonable ways to deal with growth
- Maintability. People should be able to work on the system productively.

###Reliability
Typical expectations:
- The application performs the function that the user expected
- It can tolerate the user mistakes and using software in unexpected ways
- It's performance is good enough for the required use case, under the expected load and data volume
- The system prevents any unauthorized access and abuse


The things that can go wrong are called **faults**. 
And system that anticipate faults and can cope with them is called **fault-tolerant** or **resilient**.
**Fault** is usually defined as one component of systems deviating from its spec, whereas **failure** is when the system stops providing the service.

You should generally prefer tolerating faults over preventing faults.

* **Hardware Faults**
The first response is to add redundancy to individual components in order to reduce the failure rate of the system.
There is a move toward systems that can tolerate the loss of entire machines, by using software fault-tolerance techniques in preference or in addition to hardware redundancy.

* **Software Errors**
   - software bug that causes every instance of an application server to crash
   - a runaway process that uses up some shared resources : CPU time, memory, disk space etc
   - service that the system depends on slows down or becoming unresponsive
   - cascading failures

* **Human Errors**
   - minimize opportunities for errors
   - provide fully featured non-production sandbox environment
   - unit tests, integration test, manual and automated tests
   - allow quick and easy recovery from human errors
   - set up monitorings such as performance metrics and error rates

###Scalability


 