# Storage Module

The Jackal Protocol incorporates two crucial algorithms for decentralized storage: Jackal Proof-of-Persistence (JPOP) and Internal Detection Of Loss (IDOL) protocols. This document provides an overview of these algorithms, their functionalities, and their interaction with users and Storage Providers.

### Jackal Proof-of-Persistence (JPOP)


JPOP is a Proof-of-Storage algorithm that governs the relationship between the storage provider and the user. It operates through a series of contracts containing the Merkle Tree root hash of the file and information required to prove ownership. Storage Providers are responsible for posting Merkle Proofs within a challenge window determined by the blockchain.

The challenge windows require miners to post the raw data chunk and the required Merkle Hashes to prove the data belongs to the Merkle Root stored on the contract. The challenge indexes are chosen at random by the blockchain using a block-hash-based random number generator.

### Internal Detection Of Loss (IDOL) Protocol

The IDOL protocol ensures that data remains available and accessible. When a Storage Provider successfully posts a Merkle Proof within the challenge window, and Validators verify the data, the Storage Provider is rewarded. The rewards are proportional to the file size associated with the contract relative to other active contracts on the network.

If a Storage Provider fails to provide a valid proof within the allotted timeframe, the contract is marked with a missed proof. After a certain number of missed proofs, the contract is burned, and the user is alerted the next time they query the contract. Storage Providers receive penalties for every contract burned due to missed proofs, which remain on their record for an adjustable period.

The IDOL protocol comes into play when contracts with missed proofs are moved to a new list where they can be claimed by other providers. The new provider downloads the file from one of the two online providers storing the same file, resumes the contract's proof action, and restores redundancy.

### Interaction Outline

1. A user processes the file they wish to upload to get its size & merkle tree root.
2. The user posts the proposed contract to the network with the merkle root and file size.
3. The user sends the file to any available storage providers on the network.
4. The providers process the file, similarly to the user, and make sure the merkle root and size match.
5. If all matches, the provider will generate an initial proof using the 0 index of the file and post that to chain.
6. Upon the proof being verified, the provider is added to the files metadata on chain, and starts to take up a proof-slot.

