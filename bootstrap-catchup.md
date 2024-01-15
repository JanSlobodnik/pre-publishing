
## Bootstrap and catchup

Before it can start producing blocks, a node needs to reconstruct the transition frontier by querying peers for blocks of transactions. This begins with the _root_ block and the _best tip_ (the latest block of the chain with the highest score), followed by the contents of the blocks between the root and the best tip.

This is the initial process for new nodes that connect to the Mina network, and served as our starting point. All components involved in this process have been rewritten into Rust code.

![image](https://github.com/JanSlobodnik/pre-publishing/assets/60480123/4eb825c5-1bf9-4c8f-8393-c865b83a8997)


After a node joins the network, discovers its peers and establishes secure connections with them, it can then query other nodes for the root block and the best tip. 

 

For this purpose, the node utilizes remote procedure calls (RPCs) made across the P2P layer (`P2P/RPCS`). RPCs use strongly typed interfaces, which means that the types of requests and responses are well-defined and enforced, which can help ensure that queries are structured correctly and that the expected data is returned.

Before a node can fully participate in Mina (i.e. produce blocks that extend the blockchain), she needs to sync with other nodes in the network. 

This process begins with her node bootstrapping. 

To do that, a Mina node needs to: 



1. Discover her peers
2. Query for a _root_ (earliest known) block
3. Query for the _best tip_ (latest block with the highest score)
4. Catch up with the rest of the network by querying the contents of the blocks between the root and the best tip.

For nodes to request and receive this type of information, we utilize RPC (remote procedure call) communication that is performed across the Mina P2P network. Note that in other networks, RPCs usually refer to the functionality exposed via HTTPS endpoints outside the P2P. However, in Mina, these endpoints are in GraphQL, and RPC communication is part of the Mina P2P network.

**Querying for the root block.**

The node queries other blocks in the network to provide her with the current _root block_ (earliest block) of the _transition frontier _(tree-like data store containing the last _k_ blocks). Subsequent blocks that are obtained during the catchup process are applied from this initial state determined by the root block. 

**Querying for the best tip**

The node then makes the `get_best_tip` RPC which queries for the _best tip_, the Mina blockchain's latest block with the highest known _chain strength_ (a [scoring method](https://docs.minaprotocol.com/glossary#chain-strength) Mina utilizes to determine the canonical block)

**Catchup - filling in blocks between root and best tip**

The node now has the root block as well as the best tip. All that remains for her to sync is to _catchup_ the blocks between the root and the tip.


![image](https://github.com/JanSlobodnik/pre-publishing/assets/60480123/8dee5db4-6859-45c9-9b37-c2b2cbff3028)


The node uses the RPC `get_transition_chain` to call peers (other nodes in the P2P network).


![image](https://github.com/JanSlobodnik/pre-publishing/assets/60480123/4edb3858-148d-432d-a76e-a8b3ddc5ac80)


This returns a bulk set of blocks associated with a provided set of state hashes. With this information, The node can apply blocks to their staged ledger and scan state, which 

reconstructs the transition frontier.


![image](https://github.com/JanSlobodnik/pre-publishing/assets/60480123/2fae107a-4498-45cb-8b66-686695910a6b)


Once this process is complete, the node has all of the blocks required, can now sync with the network and potentially produce blocks that extend the Mina blockchain.
