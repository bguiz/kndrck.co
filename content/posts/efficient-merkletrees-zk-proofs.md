---
title: Visualizing Efficient Merkle Trees for Zero-Knowledge Proofs
date: 2020-01-03
author: Kendrick Tan
tags:
  - cryptography
  - merkle trees
  - append-only merkle trees
  - visualization 
  - animation
disqus: yes
---

# Prelude
I've been given a grant by the Ethereum Foundation to work on an interesting project ([MACI](https://github.com/barryWhiteHat/maci)) that ultilize zero-knowledge proofs a few months ago. The work was technically challenging and I enjoy working on it. My main issue with it was that the implementation, and technicalities behind some of the chosen data structures were pass around through word-of-mouth and even the terminology varied between people. 

The goal of this blog post is to discuss and visualize the implementation of a data structure that I've been using extensively while building the MACI project - merkle trees, and specifically a simplified version of the audited [semaphore merkle tree](https://www.npmjs.com/package/semaphore-merkle-tree).

# Introduction
The data structures ultilized when building out zero-knowledge proof applications on the blockchain has to be time and space efficient due to the limitations of both the zero knowledge circuit compiler(s) and blockchain VM(s).

Merkle trees are great for this usecase as they can be used to verify if some data exists in the tree very efficiently, as even at worse case, time and space complexity is linear[^1]. Their strong data verification properties is also perfect for our use case of proving _something_ but without revealing that particular something.

![merkle-tree](https://upload.wikimedia.org/wikipedia/commons/thumb/9/95/Hash_Tree.svg/1920px-Hash_Tree.svg.png)
##### Figure 1: Merkle tree visualization, source: wikimedia

Figure 1 shows a merkle tree with a depth of 3, giving us 4 (2^(N-1), where N is the depth) data blocks to store our data (a merkle tree can have an arbitrarily depth of N). As shown in the figure, each pair of data block is hashed with some arbitrary hash function recursively until we reach the final root value. This root value is significant as it represents the hashes of all previous values, and with that root value, we can use it to proof if some data exists within the tree.

# Merkle Tree Overview
I personally understand the rationale behind technical implementations quicker once I've used them, and so I would like to start off by showing some code examples on how one might:
1. Create a merkle tree
2. Insert a leaf into the merkle tree
3. Update a leaf in the merkle tree
4. Verify if some leaf exists in the merkle tree

Note: Get familiar with the required input parameters to insert, update, and verify data in a merkle tree, as we will be discussing them later on.

## 1. Creating A Merkle Tree
```javascript
const { createMerkleTree } = require('./src/utils/merkletree.js')

const depth = 4
const zeroValue = 0

const merkleTree = createMerkleTree(depth, zeroValue)
```

## 2. Inserting A Leaf
```javascript
const data = [1, 2, 3, 4]
const leaf = hash(data)

merkleTree.insert(leaf)

console.log(`Newly inserted leaf index: ${merkleTree.nextIndex - 1}`)
// > Newly inserted leaf index: 0
```

## 3. Updating An Existing Leaf
Note: Don't worry too much about the `pathElements` just yet, we will cover it later on.

```javascript
const leafIndex = 0

const newData = [1, 2, 3]
const newLeaf = hash(newData)

const [pathElements, _] = merkleTree.getPathUpdate(leafIndex)

merkleTree.update(leafIndex, newLeaf, pathElements)
```

## 4. Verifying Arbitrary Data Exists In Merkle Tree
Note: Again, don't worry too much about the `pathElements`, we will cover it later on.

```javascript
const [pathElements, _] = merkleTree.getPathUpdate(leafIndex)

const leafIndex = 0

const dataToVerify = [1, 2, 3]
const leafToVerify = hash(dataToVerify)

const [pathElements, _] = merkleTree.getPathUpdate(leafIndex)

const leafExists = merkleTree.verifyLeafExists(
  leafIndex,
  leafToVerify,
  pathElements
)

console.log(`Leaf exists in tree: ${leafExists}`)
// > Leaf exists in tree: true
```

# Merkle Tree Technicalities

[^1]: https://brilliant.org/wiki/merkle-tree/
