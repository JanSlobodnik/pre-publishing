
# RPCs in Mina

A node needs to continuously receive and send information across the P2P network. For certain types of information, such as new transitions (blocks), the best tips or ban notifications, Mina nodes utilize remote procedure calls (RPCs).

An RPC is a query for a particular type of information that is sent to a peer over the P2P network. After an RPC is made, the node expects a response from it. 


## Implementation Details

Mina uses Janestreet's async_rpc_kernel library to implement RPCs (for both node to node and local RPCs). The network RPC functions are declared in `src/lib/mina_networking/mina_networking.ml`.

Mina uses versioned RPC that allows two peers to have different versions of the same RPC at the same time, agreeing on some version supported by both. This uses the `Both_convert` variant, which assumes that both participants should be able to use different versions.

To allow two peers to agree on a set of RPC functions (and versions) that each of them supports, there is a special RPC request, `Menu`, in which the node sends a response with a list of RPC method names along with their versions, so the caller knows which RPCs are supported by the responder.


## Wire Format

RPC messages are encoded using the dump method (prefixing with the length encoded as 8-bytes little-endian). There are three kinds of RPC messages, specified by a single tag byte: `Heartbeat `(`\x00`),` Query `(`\x01` as the tag) and Response (`\x02`), the later two followed by payload.
* For `Heartbeat`, there is no payload. 
* For the `Query`, its payload is prefixed with `Nat0`-encoded length of the payload encoding.
* For the `Response`, the data is wrapped within `Result`, and the `Ok` variant is then prepended with its length.

TODO: Handshaking message.

## Versioned RPC

The `async_rpc_kernel` defines a special RPC method, `__Versioned_rpc.Menu`, through which a peer can respond with a list of RPC methods and versions it supports. This way, its counterpart can use the same exact methods that both peers can handle.


## List of RPCs

Mina nodes use the following RPCs.



* `get_staged_ledger_aux_and_pending_coinbases_at_hash`
* `answer_sync_ledger_query`
* `get_transition_chain`
* `get_transition_chain_proof`
* `Get_transition_knowledge` (note the initial capital)
* `get_ancestry`
* `ban_notify`
* `get_best_tip`
* `get_node_status` (v1 and v2)
* `Get_epoch_ledger`


### get_staged_ledger_aux_and_pending_coinbases_at_hash

This is used to create the correct staged ledger and scan state by querying for the state hash (hash of the blockchain state).

**Query**:

**Response**: Responds with an optional tuple (an ordered set of data elements) of the following values:



* SNARK scan state
* Ledger hash
* Pending coinbase
* List of State values


### answer_sync_ledger_query

Queries a peer's sync ledger for child hashes or content by specific address, or the number of accounts.

**Query:** A pair of ledger hash and a record of the sync ledger query.

**Response:** Result containing an answer to the corresponding query, or `core.Error` describing why the query cannot be fulfilled.


### get_transition_chain

Fetches external transitions (blocks) 

**Query**: List of state hashes.

**Response**: Optional list of external transitions.


### get_transition_chain_proof

**Query**: A State hash.

**Response**: Optional pair of a state hash and a list of state body hashes.


### Get_transition_knowledge

This is used to fetch all state hashes (tip to root) from the transition frontier. Please note that unlike other RPCs, this one begins with a capital letter.

**Query**:** **

**Response**: A list of state hashes.


### get_ancestry

A network call for a client to obtain an `ancestry_proof` from its peers based on a `state_hash`.

**Query**: 

**Response**: We get the proof from a peer and the function ends.


### ban_notify

This method is used to notify a peer that they have been banned.

**Query**: Time until the ban is held.

**Response**: Empty.


### get_best_tip

Fetches the best tip.

**Query**: Empty.

**Response**: Optional record containing an external transition as data and a pair consisting of a ledger hash list and an external transition as a proof


### get_node_status (v1 and v2)

Reports the node's status.

**Query**: Empty

**Response**: Result containing the following data, or an error in case of failure. Rust doc for v1 and Rust doc for v2



* Node status data:
* Node IP address
* Node peer ID (libp2p address)
* Sync status
* Peers (a list of peer IDs)
* A list of block producer public keys
* Protocol state hash (only for v2)
* Banned peers, as a list of a pairs, each containing the ID of a banned peer and the reason the peer was banned
* List of block hashes and their timestamps
* Git commit for the Mina node program
* Uptime, in minutes,
* Optional block height (only for v2)


### get_epoch_ledger

**Query**: Ledger hash.

**Response**: Result containing sparse ledger, or a string description in case of a failure. 
