
## Committing and producing new SNARKs

SNARK proofs are the backbone of the Mina blockchain and are used for verifying the validity of transactions, blocks and other SNARKs. We want to optimize the production of SNARKs so that the Mina blockchain can continue operating and expanding.


**This is an overview of SNARK workflows. Click on the picture for a higher resolution:**

[![image7](https://github.com/JanSlobodnik/pre-publishing/assets/60480123/e98bec61-fe17-46cf-85d1-1c049dbc367d)](https://raw.githubusercontent.com/JanSlobodnik/pre-publishing/main/OpenMina%20%2B%20ZK%20Diagrams1.png)



### Receiving a Block to Update Available Jobs

Since blocks contain both transactions and SNARKs, each new block updates not only the staged ledger, but also the scan state (which contains SNARK proofs).


### 

<p id="gdcalert2" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image2.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert3">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image2.png "image_tooltip")


Via the GossipSub (P2P), a node receives a new block that contains transactions and SNARK proofs.



<p id="gdcalert3" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image3.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert4">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image3.png "image_tooltip")


The Transition Frontier, which contains the staged ledger and the scan state, is updated. The staged ledger includes the new blocks. The scan state is updated with the new jobs.



<p id="gdcalert4" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image4.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert5">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image4.png "image_tooltip")



### Receiving a Commitment from Rust Node

We want to avoid wasting time and resources in SNARK generation, specifically, we want to prevent Snarkers from working on the same pending snark job. For that purpose, we have introduced the notion of a _commitment_, in which SNARK workers commit to generating a proof for a pending SNARK job. 

This is a message made by SNARK workers that informs other peers in the network that they are committing to generating a proof for a pending SNARK job, so that other SNARK workers do not perform the same task and can instead make commitments to other SNARK jobs.

Commitments are made through an extra P2P layer that was created for this purpose.



<p id="gdcalert5" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image5.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert6">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image5.png "image_tooltip")


Commitments are sent across WebRTC, which enables direct communication between peers via the P2P network.



<p id="gdcalert6" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image6.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert7">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image6.png "image_tooltip")


Valid commitments are added to the _commitment pool_.

For a commitment to be added here, it has to: 



1. Be made for a SNARK job that is still marked as not yet completed in the scan state
2. Have no other prior commitment to that job. Alternatively, if there are other commitments, then only the one with the cheapest fee will be added.



<p id="gdcalert7" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image7.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert8">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image7.png "image_tooltip")


The work pool, which is a part of the modified SNARK pool, is updated with a commitment (including its fee) for a specific pending SNARK job.



<p id="gdcalert8" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image8.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert9">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image8.png "image_tooltip")


The commitments, once added to the commitment pool, are then broadcasted by the node other peers in the network through direct WebRTC P2P communication.



<p id="gdcalert9" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image9.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert10">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image9.png "image_tooltip")



### Receiving a SNARK from an OCaml node

The Rust node receives a SNARK proof from an OCaml node (an OCaml SNARK worker.



<p id="gdcalert10" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image10.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert11">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image10.png "image_tooltip")


The SNARK is verified in Rust.



<p id="gdcalert11" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image11.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert12">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image11.png "image_tooltip")


If it is the lowest fee SNARK for a specific pending SNARK job, then it is added to the SNARK pool, from where block producers can take SNARKs and add them into blocks.



<p id="gdcalert12" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image12.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert13">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image12.png "image_tooltip")


If it is the lowest fee SNARK for that job, then it is added to the SNARK pool



<p id="gdcalert13" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image13.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert14">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image13.png "image_tooltip")


After this, the updated SNARK pool with the completed (but not yet included in a block) SNARK is broadcast across the PubSub P2P network via the topic `mina/snark-work/1.0.0` (SNARK pool diff) _and_ directly to other nodes via WebRTC.



<p id="gdcalert14" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image14.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert15">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image14.png "image_tooltip")




<p id="gdcalert15" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image15.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert16">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image15.png "image_tooltip")
 \



### Receiving SNARK from Rust node



<p id="gdcalert16" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image16.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert17">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image16.png "image_tooltip")


Node receives new available jobs and updates its SNARK pool.


### Committing and producing a SNARK

Once committed to a pending SNARK job, a SNARK worker will then produce a SNARK. 

If a commitment is for a SNARK job that is marked as not yet completed in the scan state and there are no prior commitments to that job (Alternatively, if there are other commitments, then it is the commitment with the cheapest fee for the SNARK work), it is added to the SNARK pool.



<p id="gdcalert17" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image17.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert18">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image17.png "image_tooltip")


From the SNARK pool, it can be committed to one of the following:



1. An available job that hasn’t been completed or included in a block
2. A job that has been already performed, but the new commitment has a lower fee

 \
If the commitment is for the lowest fee available, then the SNARK worker begins working on the SNARK proof, which is performed in OCaml. After it is done, the generated SNARK is sent back to the SNARK worker (Rust).



<p id="gdcalert18" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image18.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert19">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image18.png "image_tooltip")


A SNARK worker starts working on the committed job. The SNARK proof that is generated is then checked by a prover in OCaml, after which it is sent back to the SNARK worker.

The SNARK proof is then sent to the SNARK pool.



<p id="gdcalert19" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image19.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert20">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image19.png "image_tooltip")


From here, it is broadcast to Rust nodes directly via WebRTC P2P, and to OCaml nodes indirectly via the `mina/snark-work/1.0.0` (SNARK pool diff) topic of the PubSub P2P network.
