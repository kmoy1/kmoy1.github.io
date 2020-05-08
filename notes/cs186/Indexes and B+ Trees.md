# B+ Trees and Indexes

**An index is a data structure that makes reads ON A SPECIFIC KEY a lot faster.** Let's say we have ten thousand queries on looking up various usernames in a massive username table (perhaps they're all registered sex offenders in a state or something). If you make an index on the username and search on it, you'll be able to finish all these queries far, *far* quicker than searching each row of the table for each query. In this note, we'll examine the B+ tree, a data structure which is a specific kind of index. 

### B+ Tree Properties

We have to establish some properties first. 

- A B+ tree has an **order** *d*. Each tree node *must* have [d,2d] entries, with the exception of leaves, which can have less than d entries if data is deleted, since **only leaf pages contain the important data themselves (or pointers to them).** We'll cover this in a bit.

- Node entries must be sorted. 
- In between entries *of inner nodes*, pointers to child nodes exist. These pointers *do not* count as entries of a node. We know there are at most 2d entries in a node, so **each node can have at most 2d+1 pointers**. We call the pointers of an inner node the node's **fanout**.
- The possible entries (or *keys*) of child nodes to the left of an entry must be < entry, while child keys to the right must be >= entry.

Because of this sorting, the tree actually is quite similar in concept to a BST, and searching + insertion can be done in a similar fashion, in similar runtime too!

The **height** of a tree is the length of a root-to-tree path, which is uniform for all leaves.

### B+ Tree Insertion

