# JavaNow #

## Introduction ##
JavaNow is a framework to develop parallel and high performance applications using Linda based tuple space model, which runs on network of workstations. It is a rewrite of original system I wrote back in '97-99. Though, for client APIs the system behave same as original JavaNow and Linda, i.e., it provides simple APIs like
  * get - removes a matching tuple and blocks if it does not exist.
  * getIfExists - removes a matching tuple in non-blocking fashion.
  * read - returns a copy of matching tuple and blocks if it does not exist.
  * readIfExists - returns a copy of a matching tuple in non-blocking fashion.
  * put - stores a tuple to the tuplespace.
  * eval - executes a thread that turns into a tuple, which is added to the tuple space.
  * subscribe - registers a listener that is notified when a tuple is added or removed.
  * size - returns size of tuplespace

In addition to basic Linda operations it provides following bulk and data flow operations:
  * getAllIfExists - removes all tuples in non-blocking fashion.
  * readAllIfExists - returns copy of all tuples in non-blocking fashion.
  * clear - removes all tuples.
  * putDelayed - It takes two tuple spaces and performs blocking get operation on first tuple and stores it to the second tuple space.
  * evalDelayed - It takes two tuple spaces as arguments and performs blocking get operation on first tuple space and then evaluates a thread passing that tuple and stores it in second tuple space.

## Master/Slave ##
JavaNow provides Master/Slave framework that can be used to implement parallel applications similar to map/reduce, where master distributes the work to workers (map phase) and then collects the results (reduce phase).

## What's New ##
Though, the interface and functionality of this system is very similar to the original system. The big difference is in implementation as the original system was written in Java  and used proprietary RPC protocol. The new system uses REST over http for communication between client and server and a message middleware (ActiveMQ) for communication between server to server. Also, the new system uses Tomcat as a container that uses JGroup for data replication and failover. In addition, the data is stored in the user sessions, which is automatically cleaned up after a period of inactivity. Thus it provides a far better availability, failover and resource management. The cluster of tomcat servers are behind a reverse proxy server (Apache), which actually masquerades all connections to Tomcat from client. Thus, clients don't connect to the Tomcat server directly. The reverse proxy server provides load balancing and failover using a proprietary algorithm. The load balance algorithm ensures that each tuple is stored on two servers: primary and secondary. Thus in the event of server crash, the data is not completely lost.
All interactions between client and server use REST protocol. For example,
  * put/eval operations uses HTTP PUT protocol to add tuple to the tuplespace
  * get/read operations use GET protocol, though additional parameters identify blocking nature and read/get operation.
  * size operation uses HTTP HEAD protocol.
  * clear operation uses HTTP DELETE protocol.
  * putDelayed and evalDelayed operations also use PUT protocol.
The URLs to the tuple space will follow REST convention such as
  * http://host/application/tuplespace
  * http://host/application/tuplespace/tuple

## Dynamic Resource Provisioning and Computing Grid ##
In addition to Master/Slave and TupleSpace features, the new system allows users to lend their workstations for computing. A user will download an applet that will allow the server to send a computing task to the user's workstation where it will run and on completion will store the result in the tuple space. Thus it will add dynamic provisioning of the computing resources by adding more computing resources at runtime.

## Development ##
A parallel application will use Master/Slave or Map/Reduce based architecture to divide an application into smaller tasks. The application will use ActiveTuple to create tasks that will be automatically run on a set of Tomcat servers or additional resources available. The application will use TupleSpace as a shared memory to store intermediate and final results.
## Deployment ##
The application will be deployed as a war on a cluster of Tomcat servers and will be launched by an administrative console provided by the JavaNow framework. The users will be available to monitor the progress of the application using the monitoring facilities of the framework.