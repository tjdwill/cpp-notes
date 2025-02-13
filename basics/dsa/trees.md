# Trees

One of the most fascinating data structures (imo), *tree*s are data structure that naturally allow
for the storage of hierarchical information. A *tree* is type of graph, a series of objects called *nodes* (or
more formally as *vertices*) that are connected via links called *edges*. Unlike most graphs,
however, trees are special in that for any node in the tree, there is only **one** path to said node
from the *root*, the designated beginning of the tree. Trees are a complex topic with an involved
nomenclature, so here are some important terms to keep in mind:

## Glossary

- **Tree**:
    a collection of vertices and edges such that there is only one path between the desginated root
    and any other vertex (node).
- **Vertex**:
    More commonly known as "nodes", vertices are simple objects that can have a name and other associated information. I consider 
    them to be informational "payloads".
- **Edge**: Connector between two nodes.
- **Path**: List of successive linked nodes.
- **Root**: A node that is used as the "beginning" of the tree. Any node can serve as the root of a
  tree.
- **Parent**: the node immediately above and linked to the considered node.
- **Sibling**: Nodes that are connected to the same parent.
- **Children**: Nodes that are beneath (and connected to) the considered node
- **Leaf**: A node with no children. Also known as a *terminating* or *external* node.
- **Internal node**: A non-terminating node.
- **Forest**: A set of trees. It is possible to consider a single tree as a forest.
- **Ordered Tree**: A tree whose children have a specific order.
- **Level**: The number of nodes on the path from the given node (non-inclusive) to the root. The
  root is at level 0.
- **Height**: The largest level in the tree.
- **Path Length**: Sum of levels for every node.
- **Multiway Tree**: (a.k.a m-way tree) An ordered tree whose internal nodes have a specific number of children
- **Binary Tree**: A multiway tree where internal nodes have two children.
- **Full Binary Tree**: A binary tree where internal nodes completely fill every level except the
  last.
- **Complete Binary Tree**: A full binary tree in which internal nodes on the bottom (deepest) level
  all appear to the left of the external nodes on that level.
- **Least Common Ancestor**: Given two nodes, X and Y, the least common ancestor W is the node that
  lies on the path of both nodes, but the children of W do *not* lie on both paths.

## Properties of Trees 

Trees are mathematical objects with many properties, but Sedgewick focuses on five:

1. *There is exactly one path connecting any two nodes ni a tree*.
    - An implication of this is that any node can serve as the tree's root.
2. *A tree with `N` nodes has `N-1` edges*.
3. *A binary tree with `N` terminal nodes has `N+1` external nodes*.
4. *The external path length of any binary tree with `N` internal nodes is `2N` greater than the
   internal path length*.
5. *The height of a full binary tree with `N` internal nodes is about log<sub>2</sub>N*.

More about these properties can be found in the relevant section (*Algorithms in C* (1990) pg. 38).

## Binary Trees

Binary trees seem to have special importance in CS. Part of this special consideration is that they
have enough pedagogical value while not being too complex in implementation of various algorithms.
They are also useful for representing arithmetic expressions, yes/no decision trees, sorting, etc. 

One really cool property about trees is that we can represent any forest as a binary tree by the
following:

For a given internal node, the left link points to the first child, and the right link points to the
internal node's immediate sibling. 

However, I'm not sure how this works for forests with multiple, disparate trees. Maybe we connect
them all to a dummy root node?

## Applications

Trees are commonly found in domains such as file systems (the directory structure is modeled as a
tree), the DOM in Javascript, parsing (abstract syntax trees), networking, and, generally, areas where some
hierarchy needs to be modeled.

## Representation

While you *can* represent some trees using an array representation, the typical method is to link
records of data (structs). This can be especially effective when, for example, you lack *a priori*
information on the number of children for a given node (ex. a general, non-m-way tree).

```cpp
/// Basic structure for a binary tree
template<typename T>
class BTreeNode
{
    T data;
    std::shared_ptr<TreeNode> left_child;
    std::shared_ptr<TreeNode> right_child;
    // possibly store parent as well
}
```

I'm inclined to leave terminating nodes as null, but I've seen some implementations use a dummy node
or even have the node point to itself. I don't really know what the tradeoffs are; maybe it's a way
to prevent undefined behavior from unintended `nullptr` dereferencing? 

## Traversal

Traversal, the act of systematically visiting every node in a tree, is one of the most important
operations to perform on the tree. There are four types of traversal (assume a binary tree):

1. *Preorder*: Visit the root, then the left subtree, and then the right subtree.
    - Inclined toward recursion; each subtree node can be considered its own root.
    - Can use this traversal to write an arithmetic expression in prefix form from a parse tree.
2. *Inorder*: Visit the left subtree, then the root, and then the right subtree.
    - Infix notation from parse tree.
    - A.k.a *symmetric order*
3. *Postorder*: Visit the left subtree, the right, and, finally, the root.
    - Postfix notation
4. *Level order*: Visit nodes from top to bottom and left-to-right.
    - Non-recursive 
    - A queue is useful for this representation.

For the first three, a stack implementation can be used in place of recursion.
Finally, preorder, postorder, and level order are well-defined for forests, inorder assumes binary
tree. I'm not really sure just how important that is because we can represent any forest as a binary
tree. It's possible that we take that into consideration if order matters (ex. wanting to touch all
of the nodes of a given tree in the set before the others.)