2026-09-17 17:59

Tags: 

The tree starts as one leaf that is the root node. Internal nodes contain separator keys and child references. The leaf nodes will contain the actual data as key/value entries, the internal nodes will just be pathways to reach that data. The data being in leaves is great for range scans because we can make them a doubly-linked list.

If a leaf overflows with values we can split the leaf, at the worst case scenario this propagates all the way up and splits the root node. If a leaf splits, a separator for the new right leaf is copied into its parent.

A B+ tree order must at least be 3 because every node (except the root) contains at least m/2 entries, where m is the order. If we had an order of 2 that would be 2/2 = 1 which means it is just a binary search tree which is not implied by B+ trees.

For the max amount of keys we have to know the order. The order of a B+ Tree tells us that and internal node can have a max of m children and a max of m - 1 keys.

![[B+ Tree.png]]

So here we have a B+ tree where we want to insert the data key 57 in the leaf node, but since we can only have 3 keys max for each child we need to split the right leaf node. So we'll take the data keys of `[49, 50, 56, 57]` and split it in two, the middle value being promoted and copied to an internal node.

![[Overflow.png]]

![[B+ Tree Split.png]]

So this is what the B+ tree looks like now, as you can see we have 4 children which each child having 3 or less keys so this is valid. 56 stays in the root node because unlike a regular B tree, a B+ tree only stores data in the leaf nodes so we can use the key 56 as a pathway but the main data point attached to it still needs to stay in the leaf node.

There is a difference between a root leaf node split and a non-root leaf node split. This is because if we have a root leaf node split, we have to create a new internal node because there is no parent already whereas with a non-root leaf node we just add it to the existing parent. Also we won't have to worry about data being there in the next node because the root leaf is the tree's only leaf before it splits. In a non-root leaf node we may have to check something like

```
if leaf.next_leaf is not None:
    leaf.next_leaf.previous_leaf = right_leaf
```

If an internal node splits, the middle key of the internal node is promoted and moved up. It does not need to be copied over because it does not have data like the leaves. Lets take our tree from earlier.

If we add two keys 58 and 59, 58 is accepted into the leaf node without any modification needed because 3 keys are allowed, but when we add 59 the leaf node overflows because we'd have 56, 57, 58, and 59 which is 4 keys when we can only have 3. So we need to split the leaf node, taking the median value and promoting it (and copying it because it's a leaf node). But the internal node now overflows as well which means we need another split. The median value we promote here is 56 so 56 is now our new root node. Technically the 36, 45, and 56 was our root node before hand but this works for an internal split as well of course.

![[Internal Node Overflow.png]]

![[Internal Node New Root.png]]
