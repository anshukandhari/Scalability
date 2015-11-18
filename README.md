# Scalability Open Talk


## What is Scalability?


## Factors that defines scalability approach:
	- Platform Selection
	- Hardware
	- Application Design (You Code)
	- System Architecture: DataBase Architecture & Design
	- Deployment Architecture
	.....

## How we identify the scalability issues?
	By Inspecting current Architecture and identifying:
	- Scalability Bottlenecks
	- SPOFs and Availability Issues
	- Downtime Impact Risk Zones

## How to measure the goodness of scalable architecture
	- # of Users/sessions
	- Performance : resource utilization
	- Responsiveness: Time taken
	- Downtime/ Availability
	- Cost (includes maintainance effort)


## How we do it?
	- Paradigm 1
		Apply one of -
		* Vertical Scaling
		* Horizontal Scaling
		* Partitioning
			- DB
				- Vertical OR Horizontal
			- Services
	- Paradigm 2
		* Follow Scalability Cube
	- Caching
		* Client/ Server Side
	- HTTP Accelerator
	- Be Async
	- Other cool stuffs
	- Repeat process



## Vertical Scaling (Scaling up/ Get bigger/ Add resources)
	- It is not linear
	- Finite Limit
	- Does not scale linear
	- Increases Downtime Impact
	- Cost increases exponentially


## Horizontal Scaling (Scaling out/ Get more)
	- Increase number of nodes per app server
	- Here nodes are Identical
	- Generally Done using Load Balancer
		- Hardware LB are faster while S/W are more customizable
		- For Http servers LB is generally combined with HTTP accelerators
	- Session Management in case of scaling DB server horizontally
		- Use sticky session
			* Asymmetrical load distribution (especially during downtimes)
			* Downtime Impact – Loss of session data
		- Shared Session Store Clusture
			* Central Session Store
			* SPOF
			* Session reads and writes generate Disk + Network I/O
		- Clustered Session Management (Shared-nothing Cluster)
			* Session reads are instantaneous
			* Session writes generate Network I/O
			* No SPOF (if not implemeted via master slave) // Think why?
			* Problems:
				** Sometimes, a request may get stale session data
				** Shared nothing clusters have a finite scaling limit
					Example think about Reads to Writes – 2:1 (for 8 reads [& hence 4 writes] in case of 2/4/8 DB nodes)
	- DB scaling (Horizontally) :
		* How this is implemented?
				** Actual DB copy can be stored on a central SAN also. Nothing is shared between DB servers 
				** DB Replication: Each Db node has own complete copy of DB 
					*** Master Slave Configuration
					*** Multi Master Configuration ==> Deadlock is possible
				Note: this replication can be sync or async
					- asynch is faster but there could be lag between master & slave



## Partitioning: 
	- App and DB on separate servers i.e each node has different data/service
	- Sub-optimal resource utilization
	- Finite Scalability
	- Partitioning can be performed at various layers (Service / Data etc)

	- DB Partitioning
		* Vertical : Divide columns or tables among DB clustures. One cannot perform SQL joins. Also this this finite limit.
		* Horizontal: Division is based on rows
			** DAO knows where a given row is and directs query to that server
			** Joins should be done at code level
			** Technique:
				*** FCFS (e.g. based on ids)
				*** Least Used
				*** Round Robin
				All these require an independent lookup map
				This map itself will be stored on a separate DB (which may further need to be replicated)

    - Service Paritioning
    	* Microservices (the y axis of scalability cube)


## Scalability Cube:
	3 dimension scalability cube: http://microservices.io/articles/scalecube.html
		- X axis : duplication, scale by cloning ==> horizontal scaling
		- Y axis : functional/service decomposition, noun based and verb based ==> Single server with all services . Microservices are implementation of Y axis scalabitlity
		- Z axis : data partitioning, split data ==> Partitioning Data


## Recommendation:

	a. LB itself is an SPOF, hence multiple LB [Active-Active/Active-Passive]
	b. Session Management
		* Use Clustered Session Management if you have:
			- Smaller Number of App Servers
			- Fewer Session writes
	    * Use a Central Session Store elsewhere
		* Use sticky sessions only if you have to
	c. DB considerations:
		* Use Master-Slave Async Replication
		* Write DAO for the following:
			1. Writes are sent to a single DB
			2. Reads are load balanced
			3. Critical reads are sent to a master


## Caching & Purging (for comments example):
	- Client Side:  done by browser for non textual, static data, such as images, CSS and Javascript files. It is smart enough not to re-download them every time you hit the F5 button.  
	- Server Side: 
		1. API
			* DB
			* Object: Store complete class instance with all assembled data in a cache
			* Opcode: Saving the compiled code
		2. Page => Html is not generated again & again
		3. In Memory object/data: Redis (for all type of objects, it is persistent also, guess how? via db snapshots) OR Memcache

## Async:
	* use Message Queues ==> RedditMQ, ActiveMQ or redis itself via list 

## Other things to consider
	- HTTP Accelerators/non-blocking asynch IO/ CDNs


## Selecting DB Platform:
	- CAP Theorem: Its impossible to get all the of these
		* Consistency – all clients see the same data at the same time
 		* Availability – all clients can find all data even in presence of failure
		* Partition Tolerance – system works even when one node failed

## Some References:
	Operation  => CPU cycles
	L1			    3
	L2			    14
	RAM			    250
	Disk		    41,000,000
	Network		    240,000,000
