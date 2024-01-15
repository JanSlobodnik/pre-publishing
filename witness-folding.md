

# Witness folding

Every expanding system needs to scale up to meet the demands of its growing user base. Scalability is vital for blockchains to handle increased user demand, reduce costs, and accommodate a broader range of applications, ultimately contributing to their mainstream adoption. 

However, blockchains are notoriously difficult because blocks of transactions have to be validated through strong cryptography, which is computationally intensive.

The Mina blockchain approaches the scalability problem through a unique and innovative method called "zk-SNARKs" (Zero-Knowledge Succinct Non-Interactive Argument of Knowledge) combined with a succinct blockchain design. ZK SNARK proofs can be used to create a very small representation of the blockchain’s ledger, which can be used to  

First, let's explain what are the key concepts:


### Key concepts

**Witnesses** - Witnesses serve as the input that we insert into a circuit, with the resulting output being the generated ZK SNARK proof. 

Witnesses are processed locally and consist of public and private inputs. A witness is the result of a simple computation that updates the blockchain state, for example, in a transaction, this updates the account balance of two accounts.

**Circuits -** a series of mathematical operations that are applied to a witness. Think of a circuit as a virtual machine or computer program. This virtual machine is designed to do a specific task, like adding numbers together, checking if something is true, or performing complex calculations. 

A circuit is typically represented as a collection of _polynomials_. Each gate or operation within the circuit is associated with a polynomial equation. These polynomial equations capture the relationships and computations performed within the circuit.

**Polynomials -** a polynomial as a mathematical expression made up of variables (like x, y, z), coefficients (numbers), and mathematical operations (addition and multiplication). For example, a simple polynomial could be: 3x^2 + 2y - z

**Output / Proof -** This final output, along with some extra information, becomes the proof. This proof convinces others that you've correctly used the virtual machine (the circuit) to perform the computation without revealing the inner workings or your initial data.


### Current approach

The current approach to creating a ZK SNARK representation of the Mina blockchain involves the use of _witnesses_ that are created from transactions, for which we generate ZK SNARK proofs. These ZK SNARK proofs are then merged into proofs of proofs,  a technique known as _recursive ZK proofs _or _recursion_. This process ends with the generation of a top proof, a SNARK proof that represents the entire binary tree.


<img width="691" alt="image1" src="https://github.com/JanSlobodnik/pre-publishing/assets/60480123/7328c08f-2aa7-437f-8ee0-86c4b7f28a88">


However, this process involves the generation of a large amount of intermediary SNARK proofs (SNARKs that have to be generated _before_ we can get the top proof). This is costly in terms of computing power and time. Since blocks require ZK SNARKs, this also increases block time. 


### New approach - witness folding

We have come up with an alternative method in which the witnesses themselves are ‘folded’ into one top proof:


<img width="465" alt="image2" src="https://github.com/JanSlobodnik/pre-publishing/assets/60480123/5a4ef043-dc13-45fd-bb5c-946160e61752">



The idea of witness folding is that when we have to prove many operations, we combine the fast part of them all, and then perform the slow part for all of them combined only once

So instead of ~100x20s time, we do 100x0.5s + 20s once (so that is ~70s instead of ~2000s)

other than it being much faster, it also simplifies things:



* we no longer need the scan state and all the complexity it adds
* there will be no more latency if we have the block prover include all the proofs in the block
* no need for snark workers (and therefore no fee transactions taking up space)
* If we get rid of the scan state, blocks will probably become smaller, because they don't need to include all the completed jobs.

If the same technique is applied to ZKapp transactions it would be possible to easily support a higher amount of account updates per ZKapp transaction (the current limit is 8 account updates)
