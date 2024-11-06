---
title: Merklized Storage
description: Properties of Storage using Merkle Tries
duration: 60 min
instructors: ["Shawn Tabrizi"]
---

# Merklized Storage

---

### Why is this topic important?

To understand the underlying structure which allows for "verifiable proofs" in Polkadot.

Merkle Tries.

---

## Since Bitcoin...

<div class="grid grid-cols-2">

<div>

<img style="width: 600px;" src="./assets/bitcoin-merkle-tree.png">

</div>
<div>

<img style="width: 600px;" src="./assets/bitcoin-merkle-proof.png">

</div>
</div>

---

### Merkle Tree

<div class="grid grid-cols-2">

<div>

<img style="width: 600px; filter: invert(1);" src="./assets/merkle-tree.svg">

</div>
<div class="text-small">

1. Collect all the data you want to place in the merkle tree structure.
1. Find the hash of each piece of data.
1. Take two hash nodes and hash them into a new hash node.
1. Repeat this process for the new nodes until there is a single hash.
1. The final hash is the merkle root hash.

</div>
</div>

---

### Merkle Proof

<div class="grid grid-cols-2">

<div>

<img style="width: 600px; filter: invert(1);" src="./assets/merkle-proof.svg">

</div>
<div class="text-small">

- Blue: Data you need in the proof.
- White: Data you calculate via hashes.
	- Not needed in the proof!
- Black: Data implied by the Blue nodes.
	- Not needed in the proof!
- Pink: Merkle root hash
	- Not needed in the proof!
	- Must be recalculated by you.
	- Must be provided by a verifiable decentralized source.
	- Must match.

</div>
</div>

---

### Patricia Trie

<div class="flex-container">
<div class="left-small">

<img src="../../assets/img/4-Substrate/patricia-trie.svg" />

</div>

<div class="right">

- Position in the tree defines the associated key.
- Space optimized for elements which share a prefix.

</div>
</div>

---

### Merkle Trie

<img style="width: 1200px;" src="./assets/merkle-trie-fusion.png" />

---

### Base 16

<div class="flex-container">
<div class="left">

<img src="../../assets/img/4-Substrate/base-16-labeled.svg" />

</div>

<div class="right">

- We will mostly show binary trees for simplicity.
- But everything scales up as you add more nodes.
- 16 is a nice choice because it is 1/2 of a byte (two hex characters)
  - one hex character is a "nibble"

</div>
</div>

---

### Merkle Trie Complexity

<div class="text-center">

Reading, Writing, Proofs

</div>

---

### Merkle Read

<div class="image-container">

<img src="../../assets/img/4-Substrate/merkle-read.svg" />

<div class="top-right">

- $O(\log{n})$ reads
- Not so great.

</div>
</div>

---

### Merkle Write

<div class="image-container">

<img src="../../assets/img/4-Substrate/merkle-write.svg" />

<div class="top-right text-small">

- Very expensive for a database
- $O(\log{n})$ reads, hashes and writes

</div>

<div class="bottom-left black-box text-small">

1. Follow the trie path to the value: $O(\log{n})$ reads
2. Write the new value: 1 write
3. Calculate new hash: 1 hash
4. Repeat (2) + (3) up the trie path: $O(\log{n})$ times

</div>
</div>

---

### Merkle Proof

<div class="image-container">

<img src="../../assets/img/4-Substrate/merkle-proof.svg" />

<div class="top-right text-small">

- $O(\log{n})$
- Great for light clients!
- Low bandwidth, low computation!

</div>

<div class="bottom-left black-box text-small">

1. Full Node: Follow the trie path to the value: $O(\log{n})$ reads.
1. Full Node: Upload data of trie nodes read.
1. Light Client: Download trie node data.
1. Light Client: Verify by hashing: $O(\log{n})$ hashes.

</div>
</div>

Notes:

- Message is that proof is just enough trie content (can be a bag of node or some ordered node that needs to be complete with hashing as in compact proof TODO should we make a slide for compact proof and generally proof serialization?) to build a subset/subpart of the full state trie.

This incomplete trie will then be accessed and used identically as the full state trie, but if access is not part of the proof, then the action is not finishing: Proof Incomplete case.

Invalid proof are proof where the hashing don't match (can be see as multiple trie).

TODO Could have some schema with the full state, then the proof and then two query on the proof: one that access data available and one that fail because incomplete : Probably already exists in storage deep dive

---

### Compact Proof

- Simple encoding to remove redundant information
- Trie node codec already strives for compact encoding
- Still hashes info is redundant
- Nodes are ordered to reflect the trie structure

Notes:

- trie node codec strives to make things compact

So merkle hash is calculated over most compact number of bytes.

- Still hashes info is redundant.

again in a three node V1 and V2 only tree, if proof is for V1 only, then the proof contains two nodes: root (a branch) and V1 (a leaf).
Then the encoding of root will contains two hashes V1 leaf hashes and V2 leaf hashes.
Obviously V1 leaf hashes can be calculated by hashing V1 leaf, so we can just remove it from the root node and gain 32 non compressable bytes.

- Nodes are ordered to reflect the trie structure

by ordering in a given way we can deduce the child parent relationship of nodes.
This can be done in multiple way, for instance encode in the trie node iteration: root -> V1 then when decoding stack root and when unstack complete with V1 hash.
or the other way V1 -> root (here the building need to stack root), then when decoding V1 then root.
Most/all trie algo are about keeping a stack of node (when more memory is used there is something wrong (~ 1 or two nodes)).

---

### Proof Recorder

- Tool used to create proofs for transactions
- Simple footprint of all trie nodes accessed
- Then re-encoded (compact proof)

Notes:

Another message to convey is that producing proof is really only recording all access made during some actions (key access, value insert, value change, trie iteration...).
Any kind of changes work.
-> write is a bit tricky in the sense it only read access and in memory changes. eg three node trie with V1 and V2 and a parent node, inserting V3 can just be adding a sibling to V1 and V2, but V3 will not be in proof, just the parent node.

This could be extended by the idea that key value caching should be disable for the first action otherwise trie node would not be access and we would not register proof correctly.
-> can extend to Basti pr where there is two kind of cache: trie node level cache that is safe to use and key value cache that
Not sure it is worth going to far on cache strategy, but may be relevant to mention that by its structure trie node cache is shared between block.

---

### Storage and proof size.

- Base 16 trie good for disk storage and requires less hashing.
- Binary trie proof footprint is smaller and uses less bandwidth.

<div class="flex-container">
<div class="left" style="margin: 20px;">

<img src="../../assets/img/4-Substrate/base-16.svg" />

</div>
<div class="right" style="margin: 20px;">

<img src="../../assets/img/4-Substrate/base-2.svg" />

</div>
</div>

Notes:

The trie structure (hexary) is mostly related to the storage model and do not produce the more compact proofs. One direction would be to decorelate storage from merklization. eg hexary node in storage but merklization over binary node. But the model get more complex.

A final message to it should be that (eth see it), the storage model is still not the most efficient: we use merkle trie index to access node that are stored under a btree index (rocksdb), a true state db would have it's inner indexing directly using the merkle structure.
Paritydb in this sense in a good middle ground as it implement a hash map access directly so the merkle trie index is over a hash map rather than a btree map: that is a huge gain.

What works in memory as simple data structure, also work as a db over disk and also extend to being merklized. Usually things can be mapped or referred to rather naturally. For instance an optimization of radix trie is not storing the full merkle path in each node and get the key with the value: this work in memory (not a huge gain), this work on disk (huge gain as you can have fix len node which is big gain for disk access), can work with merkle proof (but tricky if codec still store the full partial key).

---

### Pruning

<div class="image-container">

<img src="../../assets/img/4-Substrate/pruning-1.svg" />

<div class="top-right" style="width: 40%">

- For holding older block states, and then cleaning up.
- Let’s update two values in this trie.

</div>
</div>

---

### Pruning

<div class="image-container">

<img src="../../assets/img/4-Substrate/pruning-2.svg" />

<div class="top-right" style="width: 40%">

- We create new database entries, but keep the old ones too!

</div>
</div>

---

### Pruning

<img src="../../assets/img/4-Substrate/pruning-3.svg" />

---

### Pruning

<div class="image-container">

<img src="../../assets/img/4-Substrate/pruning-4.svg" />

<div class="top-left" style="width: 30%">

- Eventually, we prune the old data.

</div>
</div>

---

### Real World Implementation

A merkle trie library is applied on top of a regular key/value database.

To understand how to parse the data in the database, you need to understand two different keys:

<br>
<div class="text-center">

1. Merkle Trie Key Path
2. Key Value Database Key Hash

</div>

---

### What You Will See

<img src="../../assets/img/4-Substrate/navigate-storage-toc.svg" />

---

### Navigating Substrate Storage

<img src="../../assets/img/4-Substrate/navigate-storage-1.svg" />

---

### Navigating Substrate Storage

<img src="../../assets/img/4-Substrate/navigate-storage-2.svg" />

---

### Navigating Substrate Storage

<img src="../../assets/img/4-Substrate/navigate-storage-3.svg" />

---

### Navigating Substrate Storage

<img src="../../assets/img/4-Substrate/navigate-storage-4.svg" />

---

### Navigating Substrate Storage

<img src="../../assets/img/4-Substrate/navigate-storage-5.svg" />

---

### Navigating Substrate Storage

<img src="../../assets/img/4-Substrate/navigate-storage-6.svg" />

---

### Navigating Substrate Storage

<img src="../../assets/img/4-Substrate/navigate-storage-7.svg" />

---

### What You Just Saw

<div class="flex-container">
<div class="left">

<div>

<img src="../../assets/img/4-Substrate/patricia-trie-path.svg" style="width: 400px" />

</div>

<br>

Patricia provides the **trie path**.

</div>
<div class="right">

<div>

<img src="../../assets/img/4-Substrate/merkle-path.svg" style="width: 400px" />

</div>

<br>

Merkle provides the recursive **hashing** of children nodes into the parent.

</div>
</div>

---

<!-- .slide: data-background-color="#4A2439" -->

# Questions
