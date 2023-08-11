---
CIP: \?
Title: Merit based transaction prioritization
Category: Ledger
Status: Proposed
Authors:
    - Trym Måke Bruset <trymyrt@gmail.com>
Implementors: []
Discussions:
    - https://github.com/cardano-foundation/cips/pulls/?
Created: 2023-08-11
License: CC-BY-SA-2.0
---

<!-- Existing categories:

- Meta     | For meta-CIPs which typically serves another category or group of categories.
- Wallets  | For standardisation across wallets (hardware, full-node or light).
- Tokens   | About tokens (fungible or non-fungible) and minting policies in general.
- Metadata | For proposals around metadata (on-chain or off-chain).
- Tools    | A broad category for ecosystem tools not falling into any other category.
- Plutus   | Changes or additions to Plutus
- Ledger   | For proposals regarding the Cardano ledger (including Reward Sharing Schemes)
- Catalyst | For proposals affecting Project Catalyst / the Jörmungandr project

-->

## Abstract
A transaction prioritization mechanism is critical to allow time sensitive applications to operate safely even during times of congestion. 
The canonical proposals for handling this is through fee markets or recently a tiered fee system, however there’s no fundamental reason this mechanism needs to, or even should be tied directly to the fee calculation of a transaction. 
The following proposes a system based on a community governed set of principles, and the merits of the individual transactions, with no impact on the fair, deterministic fee calculation mechanism.

## Motivation: why is this CIP necessary?
### Why is it needed
With the expansion of the expressivity of the EUTxO model and the applications being built and offered in the ecosystem, new requirements are being asked of the underlying chain. 
Many applications, and transactions are time sensitive in nature, meaning that their value changes possibly drastically depending on delays to their settlement. 
Some examples of these are:
- Oracle feeds with validity intervals after which they are unusable 
- Certain L2/sidechain/Hydra settlements
- Liquidations in collateralized debt protocols, where the value of the collateral is collapsing and an initiated liquidation might be very costly for the liquidator if the delay is too long regardless of protocol parmaterization
### Other proposals
Practically all other proposals involve some change to the fee structure. Two notable examples are
#### Fee market
By introducing a competitive marketplace for the user submitted fees, the block producers pick the highest paying transactions first and work their way down the list. The rationale being that if your transaction is valuable enough you should be willing and able to pay a competitive rate for it.
#### Tiered fees
An alternative to an open fee market is a tiered fee system, where users are essentially able to trade lower fees for longer delays. The rationale being that users with low priority transactions are happy to pay less for their transaction to go through and don’t mind the delay, and the more time sensitive transactions won’t mind paying the higher tiers.
### Why this proposal is preferable
The premise for why this proposal is preferable is quite simple: the priority of a transaction is not intrinsically tied to how much the user issuing it is willing or able to pay for it.
To use an analogy, if there’s a queue on the highway that passes the hospital, a patient’s merit to be rushed past this queue to the hospital is not reliant on their ability to pay for a private helicopter ride there.

In the current system there is no enforcement of a “first in first out” principle, so changes to the transaction prioritization does not fundamentally invalidate the fairness of transaction processing. However linking it exclusively to the financial capabilities of the related users, even if it doesn’t impact the determinism of these fee calculations, is -although a natural consideration- arguably not the ideal approach.


## Specification

### Components
The main components of the system are, as an effect of tying into the CIP1694 structure, reasonably simple: 
- a list of principles
- a script application process
- an application review committee
- a script registry

#### List of principles
This is a public list of principles against which the suitability of script applications will be evaluated. The list can be amended through a voting process, ideally leveraging the CIP 1694 framework once implemented.
#### Script application process
Scripts that are assumed worthy by an applicant can be submitted along with an application detailing how the script works and how it adheres to the list of principles. The application will in turn be evaluated by the application reviewers, and approved or denied. The decision making process will be based on the contents of the community determined list of principles whatever they may be. 
#### Script registry
Once a script application has been approved, the script is included in a script registry. When stake pools and nodes receive transactions, the transaction is checked against the registry and if there is a script involved that is on the current registry, the node will arrange it according to its priority in the node mempool. If the node happens to be a stake pool about to mint a new block, the block contents will be based on the possibly reorganized mempool contents.
Crucially the community governance is not carried out on an individual script basis directly in the registry, rather on the principles against which the scripts are evaluated.
### Participants
The primary participants in this system are the block producing stake pool operators (SPOs), the voting community, script applicants and a set of application reviewers.
Although serving separate functions, depending on the implementation there might possibly be overlap between who actually serves in these roles.
#### Block producing SPOs
The SPOs, as the block producers, will be responsible for actually operating with the transaction prioritization mechanism through adjusting their selection of transactions based on the priority.
#### The voting community
The whole community will be able, and responsible, to participate in maintaining and modifying the set of principles on which the application reviewers will base their judgements, through community voting.
#### Script applicant
The script applicant is a concerned party that submits an application to have a script evaluated against the list of principles, with the goal of having the script included in the script registry. They might be project founders, developers or just interested ecosystem participants.
#### Application reviewer
The technical specification should describe the proposed improvement in sufficient technical detail. In particular, it should provide enough information that an implementation can be performed solely based on the design outlined in the CIP. A complete and unambiguous design is necessary to facilitate multiple interoperable implementations.


## Rationale: how does this CIP achieve its goals?
The purpose of this proposal is twofold. 
1. To propose a possible solution to an important problem facing the Cardano ecosystem (transaction prioritization).
2. To highlight a new way of approaching possible solutions to this problem, and hopefully -should the concrete proposal outlined herein not be approved- spark a conversation and expand the domain of the conversation outside the fee market and tiered fees debate.

The intended functionality of this system is as a mechanism that enables the voting community to weigh in on the principles that merit a transaction's worthiness of prioritization. That way we can collectively decide on what is most important to the community, be that the requirements of scaling solutions like sidechains, L2s and Hydra; native DeFi applications; or something else entirely.
A system like this has the potential to provide a transparent, inclusive and fair way to handle transaction prioritization, without impacting most regular users’ experience or fundamental values like deterministic transaction fees.

### Analogy
Let’s first go through a useful analogy, and then how this solution is intended to complement other ongoing developments in the ecosystem.

Transaction prioritization only matters when there is congestion on-chain, so let’s consider highway traffic as a useful real world analogy.
When going from A to B when there’s a bunch of traffic, in the real world you have a couple of options:
- You could sit in traffic and await your turn
- You could get on a bus with a bunch of other people to save the number of vehicles on the road (but still be stuck in traffic)
- You could take the next exit, and do smaller roads to where you need to go
- You could pay a bunch extra to take a helicopter ride
or
- You could (assuming you’re in an emergency response vehicle) turn on your siren and have people move voluntarily to make way for you to pass.

Now the last one is the most appropriate analogy to the proposed system here, since there are only certain classes of vehicles that are allowed to drive with sirens, and there is a social consensus that it makes sense for them to be allowed to get prioritized when they need to get somewhere fast. Furthermore the social consensus is based on the merit of their services in principle, since if it was your house that was burning, or you had a medical emergency, you would not want to wait longer than necessary for the emergency response vehicle to arrive because they were stuck in traffic and were subject to the same queueing rules as everyone else.
There are a number of applications that have similar use cases, in particular protocols reliant on collateralized debt positions, such as synthetic assets, lending platforms and algorithmic stablecoin protocols, and the associated actions of increasing margin, decreasing leverage and liquidations. Although there are other mitigating strategies that can be taken by these protocols, sidechains and Hydra being promising directions, a proper transaction prioritization mechanism is an important complement to those efforts in order to make time-sensitive applications more reliable and secure to build on Cardano.

### Context
#### CIP 1694
The governance structure proposed by CIP 1694 is a perfect backdrop to the current proposal. Once the community is able to participate in governance actions, it’s only natural to have ecosystem solutions that factor community governance into their designs.
This proposal is flexible enough to use CIP 1694 as the mechanism to perform amendments to the list of principles (ultimately depending on the final design of CIP 1694, but likely through information actions), and could in principle leverage more of the roles and contributors from CIP 1694, such as the constitutional committee and the d-reps to fill/assist the roles of the proposed system.
#### Other scaling solutions
Sidechains, L2s, Hydra, there are a multitude of scaling solutions being developed and proposed that use the main chain as the settlement layer and fundamental value storage solution. These solutions are good compliments to a transaction prioritization mechanism, but are not in, and of themselves sufficient replacements.
On the other hand, settlement transactions, such as during the closing event of a Hydra head, are good candidates for prioritized transactions.
#### Fee based solutions
There are other more conventional ways of solving the transaction prioritization problem, which involve working with the parameters at hand, i.e. changing the fee model.
A full EIP1559 style fee market has a few benefits for the projects requiring their transactions going through first, but it has a few important drawbacks, principally that it comes at the expense of fee determinism, although there is no shortage of other concerns.
A tiered fee system has been put forth as a more nuanced alternative, that allows users to effectively trade transaction delay for lower fees, and allocates a dynamic percentage of block space to different tiers. This approach has the benefit of preserving deterministic fee calculations, but still tie the priority of transactions intrinsically to the cost of submitting those transactions. This is unfortunate in the case of transactions that don’t themselves have direct value add/extract inherent within them, such as finite validity interval oracle feeds. 


## Path to Active

### Acceptance Criteria
<!-- Describes what are the acceptance criteria whereby a proposal becomes 'Active' -->

### Implementation Plan
<!-- A plan to meet those criteria. Or `N/A` if not applicable. -->

## Copyright
CC-BY-SA-2.0

[CC-BY-4.0]: https://creativecommons.org/licenses/by/4.0/legalcode
[Apache-2.0]: http://www.apache.org/licenses/LICENSE-2.0
