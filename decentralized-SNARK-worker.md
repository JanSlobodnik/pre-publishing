


# The Decentralized SNARK Worker

A blockchain typically grows in size with every new block added, as every node participating in the network needs to store the complete transaction history. This can present storage and computational challenges.

In Mina, it is possible to create a compressed (or _succinct_) representation of the blockchain by generating SNARK proofs for blocks of transactions, as well as creating SNARK proofs for a pair of existing SNARK proofs (a technique known as _recursive composition_). If you ‘fold’ the blocks of transactions enough times through recursive SNARKs, you can create a representation of the entire blockchain as a fixed-size SNARK proof that is only a few kilobytes in size. 

Below is a very simplified model of this concept:

![image3](https://github.com/JanSlobodnik/pre-publishing/assets/60480123/2e26f117-5feb-4d93-bc17-83a04a172e9c)


The Mina blockchain’s use of SNARKs and recursive composition allows its nodes to operate without needing to process or store the complete transaction history. However, it comes at the cost of performing SNARK work (the generation of SNARK proofs).

SNARK work is performed by SNARK workers – participants in the Mina blockchain who lend their computational resources to produce SNARK proofs for pending transactions. In return for their efforts, SNARK workers set a fee, which they earn when their produced SNARKs are ‘purchased’ by block-producing nodes and used to validate transactions.

SNARK work is quite expensive in terms of time and computation, so whenever a SNARK worker performs the same job as another SNARK worker, it wastes time and computation that could have instead been spent on a different SNARK job.

While SNARK workers could avoid working on jobs that were already complete (if they see them, since each node has its own local view of the SNARK pool, which contains completed SNARK jobs), they could not avoid working on jobs already in progress because they had no way to communicate this information. 

Additionally, SNARK workers must perform jobs in a specific sequential order (the same order as they have been added), or else there will be gaps in between SNARK jobs and block producers will have to wait until these gaps are filled before being able to process these SNARKs into blocks of transactions.

Fixing the cause of inefficiency is a very effective way of improving the Mina network’s performance, allowing for faster block times and increased throughput. If we can improve the coordination of SNARK work, then they can use resources more efficiently.


![image4](https://github.com/JanSlobodnik/pre-publishing/assets/60480123/0451a3f4-b460-4d70-aa0b-54a3a3ffc502)




## The life-cycle of a SNARK proof in Mina

Our goal is to optimize the process of SNARK generation and application by reducing the amount of duplicate work being done, as well as prioritizing the completion of SNARK jobs that are needed first by block producers.

To gain a better understanding of what we’ve achieved through the decentralized SNARK worker, we will now explain the life-cycle of a SNARK in Mina along with the two areas for optimization we’ve identified, as well as our solution.

_Please note that the concepts and diagrams below show a very simplified scheme of a SNARK’s life-cycle. We have also removed parts of the diagram that are not relevant to certain steps (such as the Staged Ledger in steps 2 and 3)._


### Concepts:

**SNARK Proof** - SNARK (Succinct Non-interactive Argument of Knowledge) proofs are a cryptographic tool that lets you prove knowledge of information without revealing it. It's _succinct_ for being small and fast to verify, and _non-interactive_ as it doesn't require ongoing  communication between the prover and verifier. It can demonstrate a complex calculation's correctness without exposing the solution. _SNARK proofs_ are used in Mina to create a compressed representation of the blockchain. 

Please note that in this article, we use the following terms: 

_SNARK work_ - a general term for the generation and application of SNARK proofs

_SNARK job_ - a pending request for the generation of a SNARK proof

_SNARK proof_ - a completed SNARK job, either not yet added to a block, or already included in block.

**Staged Ledger** - The staged ledger is the current account state. It contains a pending accounts ledger of unSNARKed transactions. Whenever a transaction is created, it first goes to the Staged Ledger and is applied to the Mina blockchain without an associated SNARK proof.

![image11](https://github.com/JanSlobodnik/pre-publishing/assets/60480123/c24082ef-6986-4a24-adbf-7f660e7c6cfc)


_unSNARKed transaction - A transaction that has been added to the Staged Ledger and applied to the Mina blockchain without an associated SNARK job._

The Staged Ledger also contains:

**Scan State** - A queue of transactions for which there are pending SNARK jobs _and_ transactions that already have SNARK proofs included in a produced block. The order in which pending SNARK jobs are added to the Scan State must be adhered to by block producers when selecting SNARK 


![image12](https://github.com/JanSlobodnik/pre-publishing/assets/60480123/17ad0945-f4db-47e2-b9cd-a3db5870f864)


_Pending SNARK job -_ A transaction that hasn’t yet been associated with a SNARK job. Think of it as a request for a SNARK job.

_Completed SNARK job -_ A SNARK job of a transaction that has already been included in a produced block.

**SNARKed Ledger** - This ledger only contains transactions that have an associated SNARK proof. The snarked ledger is updated once a SNARK proof has been emitted from the _scan state_ that attests to all of the transactions included in a tree of transactions added via prior block producers.

![image](https://github.com/JanSlobodnik/pre-publishing/assets/60480123/6ebaa26e-4dd0-48d3-a3c2-28c3c44e14ff)



_SNARKed transaction -_ A Transaction with SNARK is a transaction that has an associated SNARK proof.

**Snark Pool -** The SNARK pool is a pool of completed SNARK jobs that have yet to be included in a block. SNARK Workers aim to set high prices yet competitive fees. 


_completed SNARK -_ a job in the SNARK pool that has been produced by a SNARK worker, but has yet to be purchased by a block producer and included in a block.

![image](https://github.com/JanSlobodnik/pre-publishing/assets/60480123/d3f43718-d11c-4de6-8c03-f03397055e2d)


**Alice -** a representation of a Mina user initiating a transaction or other operation that requires a SNARK proof.

**SNARK workers** - nodes that create SNARK proofs, which are used to compress transactions pending SNARK jobs, as well as two or more already SNARKed transactions and other operations in Mina.

**Coordinator nodes ** - Nodes that coordinate SNARK work within a cluster of associated SNARK workers. A SNARK worker may connect to a coordinator node, after which it will receive SNARK jobs via the coordinator. Additionally, it will send and receive information about completed SNARK jobs via the coordinator. 

**Block producers**, who buy SNARK jobs using Mina tokens for block inclusion, always opt for the most affordable job. While multiple SNARK jobs can exist for a single transaction, the cheapest one is automatically chosen by the block producer. 


### 1) Adding a transaction to the Staged Ledger and its pending SNARK job to the Scan State

Alice initiates a transaction that requires a SNARK proof. This could be a simple funds transfer, a smart contract interaction, or other actions supported by Mina.

When a new transaction is added to the Mina blockchain, it's initially without a SNARK proof. As Alice’s transaction is applied to the blockchain, a statement of her transaction is created and added to the Scan State. The statement includes the information that the transaction is pending a SNARK job.


![image](https://github.com/JanSlobodnik/pre-publishing/assets/60480123/042dc23b-8da1-43f3-884f-959d3f584077)



Transactions pending SNARK jobs (marked as orange in the diagram above) are not yet added to the SNARKed ledger - they are waiting until a SNARK worker produces a SNARK proof for it.


### 2) Processing pending SNARK jobs: 

SNARK Workers choose pending SNARK jobs to be processed inside the SNARK pool. In order for SNARK jobs to be used in blocks of transactions, they need to be included in the blocks in a specific order (i.e. the same order as they were added to the Scan State).

In the Scan State, SNARK Workers can be set to two different modes: 


#### A) Sequential

SNARK Workers always choose the first (oldest) pending SNARK job from the Scan State. 


![image](https://github.com/JanSlobodnik/pre-publishing/assets/60480123/30c41eb6-01b2-48c5-b05d-406bc7111731)



Here, it is possible to use a [coordinator](https://docs.minaexplorer.com/minaexplorer/guide-to-snark-work#snark-coordinator) node that informs SNARK workers of which SNARK jobs have been completed, and thus they can avoid working on already completed SNARK jobs.


![image](https://github.com/JanSlobodnik/pre-publishing/assets/60480123/1a14d7da-beab-47da-90e6-e365245d83d6)



However, the problem is that multiple coordinator nodes cannot communicate information about ongoing SNARK jobs between each other. This problem cannot be solved by having just one coordinator node for the entire network because this creates a central point of failure and poses a security risk.

Additionally, while the coordinator does receive information regarding which SNARK job is complete, the coordinator does not inform workers of whether a SNARK job is currently being worked on. This is inefficient because there is a significant chance of multiple SNARK workers picking up the same job and unnecessarily performing redundant work. In the worst-case scenario, it would be equivalent to having a single SNARK worker.


#### B) Random

In this mode, SNARK workers choose SNARK jobs randomly. While SNARK workers know which jobs have been completed, they do not know which ones are currently being worked on by other workers.


![image](https://github.com/JanSlobodnik/pre-publishing/assets/60480123/078a0c77-befa-47f4-8a8b-2a213f93cbcf)



The problem with this approach is that completed SNARK jobs must be processed in a specific order (as they were added as pending jobs to the Scan State), otherwise, Block Producers cannot process them.

Say there are 3 jobs in the Scan State: 1,2,3. One Worker selects 1, another selects 3. While they can finish the SNARK jobs, the Block Producer who chooses SNARK job no. 3 will not be able to use it until SNARK job number 2 is completed.


### 3) An extra layer for SNARK worker communication


![image](https://github.com/JanSlobodnik/pre-publishing/assets/60480123/f5571901-326d-4082-b6dd-bb764b5b9102)



We want to have SNARK workers perform SNARK jobs in the correct order, and we also want to  prevent them from working to the same job as another SNARK worker is currently working on.

To solve this issue, we have implemented an extra P2P layer through which nodes can communicate job commitment:


![image](https://github.com/JanSlobodnik/pre-publishing/assets/60480123/b5748519-7eaa-4c78-99e1-584866f01ad7)


Once a SNARK worker starts performing a SNARK job, it will use this layer to:

1) Sequentially perform SNARK jobs, i.e. first complete the SNARK job for the first transaction in the Scan State.

2) Communicate SNARK job commitment,  which will help us reduce situations in which 2 or more SNARK workers are working on the same job.

**Implementation details**

The purpose of this commitment layer is to synchronize the _work pool_ (a combined pool of SNARKs and SNARK commitments) between each peer That way, each snarker knows what other snarkers are working on, in real time:


![image](https://github.com/JanSlobodnik/pre-publishing/assets/60480123/689bf03d-a567-40ee-b069-658f6615c7d2)



This prevents snark workers from working on the same job. However, it is still possible that two snarkers create a commitment at the same time - it can occur because there is a delay before a commitment reaches another peer. Snark Worker B, or Bob, might make a commitment to the same job as Snark Worker A (Alice) before he receives information that Alice has committed to said SNARK job.

 

If a SNARK worker sees a commitment from another SNARK worker that is ‘better’ (i.e. cheaper), then it will utilize a mechanism that cancels its own commitment - the winning snark work commitment is broadcasted to its peers (other SNARK workers in the network), and the losing commitment is removed. If it has already begun the SNARK job, then it will cancel the SNARK job. Whether a commitment is better or worse depends on the SNARK worker’s fee for the SNARK job - a lower fee equals a better commitment. 

However, we want to avoid malicious nodes from broadcasting low prices that they do not intend to honor and thus stopping legitimate nodes from performing SNARK jobs at market price. For this purpose, we plan on developing a scoring system for SNARK workers that automatically rejects commitments from workers that are scored low. 

If the fee is the same for two or more snarkers, then a tie-breaking mechanism is implemented in which we use SNARK information that is outside of the snark worker’s control, so that the snark worker can’t manipulate the parameters to make its commitment superior.

 

This information consists of the snark job ID and the hash of the Snark worker’s public key (Account address). These two pieces of information are hashed together, and compared with a hash computed by using the same two values from another commitment, with the bigger one winning the right to commit to that SNARK job. Please note that selecting the bigger hash as the winner is an arbitrary choice, it can be set to the smaller one, the purpose is only to have a tie-breaking mechanism for SNARK work.

While in the original solution, coordinator nodes improved synergy on the level of clusters of nodes (they had to belong to the same cluster as the coordinator node), now we are coordinating between all nodes on a network level. Communication is performed directly between nodes via the P2P network. The only remaining bottleneck is the network’s performance - how fast the message can reach the peer, and how fast it can broadcast messages to other peers.

Commitment logic is an extra P2P layer used by the node itself. Rust nodes use the same WebRTC-based P2P layer between themselves, and as such the new solution is only for Rust nodes (OCaml nodes use a libp2p-based network). \



### 3) Block producers selecting SNARK jobs for blocks

Block producer selects Completed SNARK jobs from the SNARK pool to include in block.


![image](https://github.com/JanSlobodnik/pre-publishing/assets/60480123/dfe37c36-f6fe-4c7e-b492-595d31a983ff)


Think of the SNARK pool as a marketplace for SNARKs jobs – SNARK workers will always aim to set the highest fee possible while remaining competitive within the market. 

Block producers pay Mina tokens for SNARK jobs that they want to include in blocks, therefore they always look to purchase the cheapest SNARK job. Its possible for multiple SNARK jobs to be made for the same transaction, but the Block producer will automatically pick the cheapest SNARK job.

If the price is the same for multiple SNARK jobs belonging to one transaction, then they select the oldest SNARK job.

Please note that, unlike the Staged Ledger, the SNARK pool is local to each block producer. Therefore each block producer may have a different view of the SNARK pool, since there may be SNARK jobs that haven’t yet reached some block producers.

