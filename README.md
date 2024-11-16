---
icon: hand-wave
cover: .gitbook/assets/Screenshot 2024-11-17 at 2.02.56 AM.png
coverY: 0
layout:
  cover:
    visible: true
    size: full
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Pint TS SDK

Essential is the first declarative blockchain protocol. The protocol implements the declarative principles described in the above sections in a decentralized, permissionless system, architected around the idea of blocks as pure state mutations.

State mutations originate as solutions, provided by _solvers_. A solver may be a third-party entity which competes to find optimal solutions. It may also be simply a centralized program (e.g. a server, or a front-end app or wallet) which serves solutions for a specific application. Note than unlike solutions in an imperative system (which take the form of a series of ordered transactions), solutions in a declarative system are simply a list of state mutations.

Solutions become state through the process of _validation._ **A solution is considered valid when it satisfies all the contracts whose state it mutates.**

Valid solutions then become candidates for inclusion in the next block. This occurs through an out-of-protocol mechanism, whereby solvers bid for inclusion to a solution aggregator (analagous to a block builder). Bids for inclusion are themselves expressed as constraints on state. Note that the protocol itself is concerned only with the _validity_ of solutions. _Optimality_ is achieved through auction dynamics, both at the application and solution aggregation level.

In the next section we shall see how contracts are structured to express conditions over how their state can be mutated.

### Quickstart your Journey by building and deploying your first Pint smart contract

<table data-view="cards"><thead><tr><th></th><th></th><th data-hidden data-card-cover data-type="files"></th><th data-hidden></th><th data-hidden data-card-target data-type="content-ref"></th></tr></thead><tbody><tr><td><strong>Quick Start</strong></td><td>Create a boiler plate to interact with your pint smart contract from the frontend</td><td></td><td></td><td><a href="getting-started/quickstart.md">quickstart.md</a></td></tr></tbody></table>
