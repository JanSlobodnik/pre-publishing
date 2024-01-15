
## Using the Web Node to participate in Mina

It’s always beneficial for a blockchain network to decentralize as much as possible. Distributing the network across multiple nodes improves the blockchain’s ability to resist attackers. Additional nodes provide critical redundancy for the network, ensuring it runs even if other nodes go offline.

To help decentralize the Mina network, We’ve developed the Mina Web Node, a blockchain node that can be set up in a matter of seconds and configured through a standard internet browser such as Chrome or Firefox. 

<a href=https://raw.githubusercontent.com/JanSlobodnik/pre-publishing/main/WebNode.png><img width="800" alt="WebNode-small" src="https://github.com/JanSlobodnik/pre-publishing/assets/60480123/ba3cba1f-2e37-484e-9b27-550650e97412"></a>


Mina users can simply launch their own Web Node through a [link in their browser](https://openmina.com/web-node), which will initialize the building of the Web Node.

**Setting up the browser for the Mina Web Node**

The browser requires a specific environment for the Web Node to run, which is achieved by loading a Web Assembly (wasm) file. Web Assembly is a binary instruction format through which we can run code in a browser even if it’s written in languages other than javascript.

**Getting ready to verify blocks**

For verifying block SNARKs, we need to fetch a verifier index, which contains constants necessary for verifying a block for a given chain.

<img width="900" alt="WebNode1" src="https://github.com/JanSlobodnik/pre-publishing/assets/60480123/5c999f01-8901-4829-b4e7-a746f5ec5723">

**Connecting directly to Mina network**

The Web Node has to connect directly to the Mina network via WebRTC, without using intermediaries. Thanks to WebRTC, in-browser Web Nodes don't put extra weight on the network. Instead, they serve other browser-based nodes or even native nodes themselves, hence contributing their share to the network.

<img width="900" alt="WebNode2" src="https://github.com/JanSlobodnik/pre-publishing/assets/60480123/e336ab9e-74fe-44c6-9485-08ba7fbfb8fd">

**Catching up with the network**

The Web Node downloads the latest block and verifies it. In the demo version, this step also fetches the latest state of the demo account along with its Merkle proof from the peer’s ledger. 

Using the Merkle proof, we can verify that this account state is indeed part of the ledger with the hash that is part of a block we verified.

Through the Web Node, users and developers can access the Mina blockchain directly, without any intermediaries, which preserves privacy and increases security. 

With the Web Node, users can:



* Transfer MINA funds directly, without any intermediaries
* Verify blocks of transactions

However, unlike block-producing Mina nodes and nodes running SNARK workers, the Web Node cannot produce blocks and SNARKs - it can only verify them. 

On the other hand, developers are able to: 



* Deploy ZKApps
* Observe ZKApp state
* Interact with ZKApps

Developers can also embed Webnode into consumer products (apps) to simplify onboarding and enhance security.
