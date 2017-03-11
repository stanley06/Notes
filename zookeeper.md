## ZOOKEEPER
- distributed co-ordination service -> manage large set of hosts
- can be used for leader election, group membership, and configuration maintenance,  event notification, locking, and as a priority queue mechanism. 
- highly concurrent, very fast, mainly for read-heavy access patterns
	- read: any node of a zookeeper ensemble
	- write: quorum-based (majority of the nodes should write success), utilize an atomic broadcast protocol
	- best to main odd number of nodes in zookeeper ensemble ?????
- active connection with all its clients using a heartbeat mechanism
- keeps a session of each active client connected to it. if disconnected for more than a specific timeout, session expire. 


