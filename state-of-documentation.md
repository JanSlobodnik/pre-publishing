## Current state of the Rust node documentation

### By module

  - [Openmina Node](https://github.com/openmina/openmina#the-open-mina-node)
  - [The Mina Web Node](https://github.com/openmina/webnode/blob/main/README.md)
- P2P networking stack
   - [P2P services](https://github.com/openmina/openmina/blob/documentation/docs/p2p_service.md)
   - [RPCs support](https://github.com/JanSlobodnik/pre-publishing/blob/main/RPCs.md) - in progress
   -	[GossipSub](https://github.com/openmina/mina-wiki/blob/3ea9041e52fb2e606918f6c60bd3a32b8652f016/p2p/mina-gossip.md)

 - [Scan state](https://github.com/openmina/openmina/blob/main/docs/scan-state.md)
  - [SNARKs](https://github.com/openmina/openmina/blob/main/docs/snark-work.md)
- Developer tools
  - [Debugger](https://github.com/openmina/mina-network-debugger/blob/main/README.md)
  - [Front End](https://github.com/openmina/mina-frontend/blob/main/README.md)
    - [Dashboard](https://github.com/openmina/mina-frontend/blob/main/docs/MetricsTracing.md#Dashboard)
    - [Debugger](https://github.com/openmina/mina-network-debugger?tab=readme-ov-file#Preparing-for-build)



### By use-case

- [Why we are developing Open Mina](docs/why-openmina.md)
- Consensus logic - not documented yet
- Block production logic 
  - [Internal transition](https://github.com/JanSlobodnik/pre-publishing/blob/main/block-production.md)
  - External transition - not documented yet
  - [VRF function](https://github.com/openmina/openmina/blob/feat/block_producer/vrf_evaluator/vrf/README.md) - in progress

- Peer discovery/advertising
  - [Peer discovery through kademlia]() - in progress
  - [SNARK work](https://github.com/openmina/openmina/blob/main/docs/snark-work.md) - SNARK production is implemented (through OCaml). Node can complete and broadcast SNARK work.
- Compatible ledger implementation - not documented yet
- Transition frontier - not documented yet
- [Bootstrapping process](https://github.com/JanSlobodnik/pre-publishing/blob/main/bootstrap-catchup.md) - in progress
- Block application - not documented yet
- How to run
  - [Launch Openmina node](https://github.com/openmina/openmina#how-to-launch-without-docker-compose)
  - [Launch Node with UI](https://github.com/openmina/openmina#how-to-launch-with-docker-compose)
  - [Debugger](https://github.com/openmina/mina-network-debugger?tab=readme-ov-file#Preparing-for-build)
  - [Web Node](https://github.com/openmina/webnode/blob/main/README.md#try-out-the-mina-web-node)



