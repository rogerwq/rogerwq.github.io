---
layout: post
title: A SMILES Writer Implemented in Rust
date: 2021-10-16 21:22:20 +0900
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
# img: smiles.jpg # Add image post (optional)
fig-caption: SMILES is widely used in Cheminformatics # Add figcaption (optional)
tags: [Cheminformatics, SMILES, molecule]
---

The algorithm of building a SMILES string for a molecular structure was mentioned by D. Weininger et al [1], "the structure is treated as a tree and a SMILES string is generated that corresponds to a depth-first search (DFS) of that tree ... and a second
DFS search is performed for polycyclic structures to to ensure proper ordering of ring closure labels." However the details of implementations cannot be found in this paper. 

A. Varnek [2] explains in details this algorithm and the Python code is provided.

Following the instruction from [2], I have implemented this algorithm in Rust available from [here](https://github.com/chiral-data/rust-chem/blob/main/src/smiles_writer.rs) and a demonstration by example can be found [here](https://vj8hk.csb.app/smiles).

Two new findings are worth to be mentioned:
1. The python code from [2] writes bond symbols for ring closures twices, which is incorrect.  
2. The order of closure digits by the same closing atom is not defined in SMILES specification [3]. For example, _Oc1cccc2ccccc12_ and _Oc1cccc2ccccc21_ are both valid SMILES. However, for canonical SMILES, this order has to be definitive.


### Reference 
1. David Weininger et al, (1989), [SMILES. 2. Algorithm for generation of unique SMILES notation](https://pubs.acs.org/doi/pdf/10.1021/ci00062a008) 
2. Alexandre Varnek, (2017), [Tutorials in Chemoinformatics](https://www.google.co.jp/books/edition/Tutorials_in_Chemoinformatics/L8toswEACAAJ?hl=en), Page 422 - 425.
3. [OpenSMILES specification](http://opensmiles.org/opensmiles.html)