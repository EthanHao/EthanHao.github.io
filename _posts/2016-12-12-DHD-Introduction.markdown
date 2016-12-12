---
layout: post
title:  "Focusing on how to user Chord algorithm to realize a DHT"
date:   2016-12-12 10:45:20 -0600
categories: DistrubtedSystem Algorithm
---
## 1. Description of DHT  
DHT is a distributed hash table usually using to store extensive data with key-value pair manner. DHT is decentralized, scalable and frequently used in peer-to-peer system.
It can provide the same interface like the traditional hash table, like Put (key, value) and Get (key);

A great DHT must have these following capability:
*	Autotomy and decentralization: which means each node can work alone without the coordination of other node.
*	Fault tolerance: which means DHT is tolerant for the failing of node. There is no single point of failure in DHT system.
*	Scalable: DHT should be able to easily and efficiently add even millions of nodes.

## 2. Consistent Hashing  
The basic idea of consistent hashing is partitioning the hash value space and also mapping the hash value of node to this space. When putting a key-value pair to this DHT, it need to compare the hash (node) and hash (key) to determine the key will be locating in which node. We can demonstrate the mechanism from the below photograph.

![alt text](/img/DHT/1.png) 

Assume there are five nodes (N1, N9, N30, N60, and N90). We can use hash function to map these node to the ring. Generally we using the SHA-1(IP address: port) % 2^m to get the hash value of node.
And there are seven key-value pairs (K4, K5, K7, K10, K25, K65, and K80).As you can see from the graph, which is final state after inserting these key-value pairs into the five-node DHT.
The other important portion of DHT is the route table of each node. The basic route table of each node just save IP address of successor node. 

![alt text](/img/DHT/2.png) 

AS we see from the above graph, the basic route is very sample, just pass the request to the successor node. And it is very easy for adding or leaving a node, you just need to reassign the key-value pairs in the successor node to the new node when adding node happens.
Let us think about another scenario of looking up a key K80. We can illustrate the route path like the below graph.

![alt text](/img/DHT/3.png) 

The blue line in the above graph is the route path, obviously it is inefficient to lookup key when the node numbers are very large. So we need find a way to solve this issue.

## 3. Chord Algorithm
The primary idea of Chord algorithm is to reduce the complexity of lookup operation from O (N) to O (log (N)). How could it do that? 
Firstly let us understand the concept of Finger table. Finger table is a table containing some entries which are the IP address of other nodes, and each node maintains its own Finger table. Then how can we decide how many entries a Finger table should have?
Assume N is the total numbers of nodes and 2^m = N, then m the entries number of Finger table. 
The Finger table looks like the below table.
 
Index	          | Start	                 | successor           
---- | ----  | ---- 
Index I(0< I <=m)	| Start From (n+2^(I-1) ) mod 2^m	|  Successor (n+ n+2^(I-1)) 



The whole Finger table vision of last example like the below graph.

![alt text](/img/DHT/4.png) 

  * ###  	Lookup scenario
  Let us look at the scenario of looking up key K80 from the N1 again. According to the Finger table of N1, we easily can know K80 is in N90. The route looks like the below graph.

  ![alt text](/img/DHT/5.png) 

  That is an extremely special case for looking up a key, mostly the number of route path is at most Log (N).

  * ### 	Node joining scenario
  There are three significant things we must do when a node is going to join the DHT.

    -	##### Update the Finger table of the node which are going to join.
  
    ![alt text](/img/DHT/6.png) 

    -	##### Copy data from the successor node 
 
    ![alt text](/img/DHT/7.png) 

    -	##### All nodes in the DHT periodically update the Finger table to make sure the Finger table is the newest and accurate.  

    ![alt text](/img/DHT/8.png)
 
  * ###  	Node leaving scenario  
  If node N find the successor node has failed, then replaced it with the first live entry in the Finger table. Then notice the other node about the failing of the successor node.
