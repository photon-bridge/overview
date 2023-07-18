When someone questions me about what is blockchain, I typically respond by describing it as a tool that allows distribute trust. Indeed, the essence of blockchain allows for powerful decentralization of trust. Let's examine the nature of blockchain and understand the problem Mina seeks to solve, and the improvements it requires.

**Blockchains comprise several components:**

- A peer-to-peer network that connects users
- Transactions that let users communicate their intentions
- Consensus to ensure agreement on validity rules and decisions
- State machine that processes transactions and creates "state transitions"
- Chain that maintains the integrity of state transitions through cryptographic links, a history of state transitions
- Incentives to fend off the Sybil attack
- Clients to make blockchain accessible

The continual growth of state transitions, tracked in each block, causes an increase in data needed for running a full node. This poses a serious problem for Ethereum and other existing blockchains, undermining decentralization - state growth. To secure and make blockchain data available, full nodes need to ensure data availability, but as every node has to store all the data from the beginning, state size grows exponentially. This is the problem Mina aims to resolve.
![0_t-7KCCJyhojgu_KI](https://github.com/photon-bridge/overview/assets/71966179/c8d14850-cf73-4f77-a10f-fac9c2656c75)

Mina employs recursive zero knowledge proofs, limiting the state to only 22kb, addressing the issue of state growth. While intriguing, Mina does encounter a challenge: onchain data storage is limited to 8 fields. Hence, if you're developing a complex algorithm requiring more than 8 fields of storage, you must store the data offchain. But offchain storage is vulnerable to liveness and censor attacks as most existing offchain storage solutions for Mina lack cryptoeconomic security and rely on centralized servers.
![1_mNL-dPAg8il71_SoNouQYg](https://github.com/photon-bridge/overview/assets/71966179/a3f007d8-8260-4548-ae67-291facab9ae3)


Should the offchain storage become unreachable, it renders the zkapp unusable, as its liveness guarantees depend on data storage. This becomes crucial if the zkapp is a rollup or a financial protocol. Moreover, centralized servers can censor users' transactions.

In summary, it's crucial to distinguish between data storage and data availability. Data storage solutions like Arweave, File etc. are useful for non-financial applications. For financial applications, strong liveness guarantees are required. To provide liveness guarantees and robust cryptoeconomic security, we propose Celestia as a DA layer of zkapps, ensuring liveness for zkapps. We've modeled our bridge on the design of the Quantum Gravity Bridge (QGB).

To clarify, we are not using Celestia for transaction ordering or other security properties, nor changing  anything on the Mina side. Our focus is solely to address Mina's offchain problem.

**Several challenges arise:**

- It's virtually impossible to prove Celestia’s consensus inside Mina’s VM.
- Attempting to use Celestia's hashes (sha256) in Mina's VM proves troublesome as Mina only supports the poseidon hash algorithm. Proof equivalence is an engineering challenge.
- Attempting a permissionless approach can allow a malevolent actor to disrupt the system by updating Mina's merkle tree without sending any data to Celestia.
  **Proposed Architecture**

![işşlödsşlfö](https://github.com/photon-bridge/overview/assets/71966179/0ae6cefe-ed88-4c75-b7c4-d108c56089dd)

1. **Users transmit data to Celestia:**

To secure strong cryptoeconomic assurances for Data Availability, we utilize Celestia, which offers Data Availability via erasure coding. Users submit their data to the Celestia network as the merkle root within Mina's zkapp, indicating offchain storage, can only be updated if the data is accessible on Celestia.

2. **Users request a signature from Signers that confirm the data is accessible and has been disseminated to the network:**

In our design, Signers act as a third-party committee, but they could be integrated natively into Celestia with minimal coding. If native support is absent, we can employ Signers as third-party providers, which is exactly what we've done.

Incentives for Signers can be readily established, as Signers possess onchain addresses.

Since Celestia's block time is shorter than Mina's, users won't encounter additional confirmation delays.

3. **The Signer committee verifies whether the data is accessible and has been distributed across the network. They then sign a message using the Poseidon hash algorithm.**

Mina employs the Poseidon hash algorithm, which is snark-friendly, while Celestia utilizes Sha256. This discrepancy, among other reasons, prevents us from verifying Celestia’s state in a trust-minimized manner, hence the need for the Signers.

4. **The Signer committee provides signatures to users:**

We initially considered employing Multi-Party Computation (MPC) for this signing mechanism instead of multi-signature, to make it more cost-effective and efficient. However, this proved too complex for a hackathon.

These signatures validate the Data Availability (DA) in a trusted manner, akin to Quantum Gravity Bridge (QGB). If 60% of the signers sign an identical message, the Merkle root at Mina can be updated.

5. **Users send the transaction with signatures.**
