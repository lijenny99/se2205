# Trees #

### Introduction ###

- **Graph**: Express *relationships* amongst entities (eg. connections in social media, coupling amongst systems, constraints amongst components, etc.)
  - Edges can be ordered (**directed**) or unordered (**undirected**)
  - A **path** from v<sub>i</sub>, v<sub>j</sub> of length _n_ consists of a sequences of edges _P = e<sub>1</sub>, …, e<sub>n</sub>_ that connect these vertices (for directed graphs, these edges must be in the same direction)
  - **Cycle:** Path that starts and ends at the same vertex while visiting remaining vertices in the path only once
  - **Connected:** No vertices are disjoint
  - **Unconnected:** Vertices are disjoint
- Trees
  - A special type of graph composed of nodes/vertices and links/edges
  - All nodes must be connected and not form cycles
  - Must consist of a single path from the root to each descendant in the tree
  - Many types
    - **Static:** Shape is fixed
    - **Dynamic:** Shape can change with insertion/deletion of nodes or shape modifying changes
    - **Search Trees:** Efficient organization of data for searching
    - Stores sub-solutions of problems (game tree)
    - Efficient implementation of ADT/algorithmic operations (heaps/priority queues)
  - **Family** relationship (parents/children, ancestors/decendants)
  - **Geometric** relationship (left/right node)
  - **Biological** (roots/leaves)
  - Metrics
    - **Depth:** Number of edges from root to node
    - **Height:** Edges between node to its deepest leaf descendant
    - **Degree:** Number of edges incident on a node
    - **Maximum Children per Node M:** Trees can be classified based on this
    - **Full:** Each node has exactly M children or zero children
    - **Complete:** Completely filled tree except for the bottom level (fill from left to right)
    - Other types include left linear, zig-zag, and right linear trees
- Operations on Trees
  - Possible operations are:
    - Insertion
    - Traversal
    - Deletion
    - Rotation
    - Merge
  - Manner in which these operations are performed depends on the purpose of the tree
  - The asymptotic performance of these operations are dictated by the structure of the tree (tree height)
  - In the worst case, number of node traversals is proportional to the longest path from the root to one of the leaves in the tree
  - **Left/Right Linear, Zig-Zag Tree**
    - Longest path length: ==n-1==
    - Asymptotic complexity of node traversal: ==O(n)==
  - **Complete Trees**
    - Number each node from left to right
    - Path length of each node i from the root is ==log<sub>2</sub>i== 
    - Largest path length is ==log<sub>2</sub>n==
    - Asymptotic complexity node traversal: ==O(logn)==
  - Need to iterate through the nodes in a tree — implement an interface that extends “Iterable”

<u>Tree ADT Interface</u>

```java
public interface Position<E> {
  E getElement() throws IllegalStateException;
}

public interface Tree<E> extends Iterable<E> {
  // returns the reference root of the node
  Position <E> root(); 
  
  // returns the parent of node p
  Position<E> parent(Position<E> p) throws IllegalArgumentException; 
  
  // returns the collection of nodes that are immediate descendants of node p
  Iterable<Position<E>> children(Position<E> p) throws IllegalArgumentException;
  
  // return the count of the total number of children of node p
  int numChildren(Position<E> p) throws IllegalArgumentException;
  
  // check if it is not a leaf
  boolean isInternal(Position<E> p) throws IllegalArgumentException;
  
  // check if no children
  boolean isExternal(Position<E> p) throws IllegalArgumentException;
  
  // check if it has no parent
  boolean isRoot(Position<E> p) throws IllegalArgumentException;
  
  int size();
  boolean isEmpty();
  Iterator<E> iterator();
  Iterable<Position<E>> positions();
}
```

<u>Abstract Tree</u>

```java
public abstract class AbstractTree<E> implements Tree<E> {
  public boolean isInternal(Position<E> p) {
    return numChildren(p)>0;
  }
  public boolean isExternal(Position<E> p) {
    return numChildren(p)==0;
  }
  public boolean isRoot(Position<E> p) {
    return p == root();
  }
  public boolean isEmpty() {
    return size()==0;
  }
}
```

<u>Auxiliary Tree Methods</u>

- **Depth of a node** `p: depth(p)`

  - Recursive implementation where at each recursive call, the parent of the current node is passed as the argument until the root node is reached
  - Counter of each recursive call is maintained
  - `depth(p) = depth(parent(p)) + 1` with base-case `isRoot(p) == True` where `depth(p) = 0`

- **Height of a subtree rooted at node** `p: height(p)`

  - Recursively compute the maximum height of the children and account for the edge connect the node p to the children

  - `height(p) = max[height(children(p))] + 1` where the base-case is `isExternal(p) == True` where `height(p) = 0`

  - Alternatively,

    ```java
    public int height(Position<E> p) {
      int h = 0;
      for (Position<E> c: children(p))
        h = max(h, 1+height(c));
      return h;
    }
    ```

### Binary Trees

- **Simple** definition: Each node has a maximum of two children
- **Formal** definition: Each node has exactly two children where these children can be _empty_. Nodes with two empty children are leafs
- **Recursive** definition: _Empty tree_ or a node with _left_ and _right_ subtrees that are binary trees
- **Extended** binary trees: Explicitly show empty children with a square

<u>Binary Tree ADT</u>

```java
public interface BinaryTree<E> extends Tree<E> {
  Position<E> left(Position<E> p) throws IllegalArgumentException;
  Position<E> right(Position<E> p) throws IllegalArgumentException;
  Position<E> sibling(Position<E> p) throws IllegalArgumentException;
}
```

<u>Abstract Binary Tree</u>

```java
public abstract class AbstractBinaryTree<E> extends AbstractTree<E> implements BinaryTree<E> {
  public Position<E> sibling (Position<E> p) {
    if (isRoot(p))
      return null;
    Position<E> parent = parent(p);
    if (p == left(parent))
      return right(parent);
    else
      return left(parent);
  }
  public int numChildren (Position<E> p) {
    int count = 0;
    if (left(p) != null)
      count++;
    if (right(p) != null)
      count++;
    return count;
  }
  public Iterable<E> children(Position<E> p) {
    List<Position<E>> clist = new ArrayList(2);
    if (left(p) != null)
      clist.add(left(p));
    if (right(p) != null)
      clist.add(right(p));
    return clist;
  }
}
```

#### Linked

- Each node in a tree will have two pointers (to the children). Need to traverse through all nodes in a path from root to the destination node
- Need to check the validity of arguments and states

```java
public class LinkedBinaryTree<E> extends AbstractBinaryTree<E> {
  protected static class Node<E> implements Position<E> {
    private E element;
    private Node<E> parent;
    private Node<E> left;
    private Node<E> right;
    
    public Node(E e, Node<E>, p, Node<E> l, Node<E> r) {
      element = e;
      parent = p;
      left = l;
      right = r;
    }
    
    public E getElement() {return element;}
    public Node<E> getParent() {return parent;}
    public Node<E> getLeft() {return left;}
    public Node<E> getRight() {return right;}
    
    public void setElement(E e) {element = e;}
    public void setParent(Node<E> p) {parent = p;}
    public void setLeft(Node<E> l) {left = l;}
    public void setRight(Node<E> r) {right = r;}
    
  }
  
  // node validity 
  protected Node<E> validate(Position<E> p) throws IllegalArgumentException {
    if (!(p instanceof Node)) // check if p is instance of node
      throw new IllegalArgumentException("Not valid position type");
    Node<E> node = (Node<E>)p; // cast p to node type
    if (node.getParent() == node) // check if node is active in tree
      throw new IllegalArgumentException("p is no longer in the tree");
    return node;
  } 
  
  // factory design pattern that creates new instances of node
  protected Node<E> createNode(E e, Node<E> p, Node<E> l, Node<E> r) {
    return new Node<E>(e,p,l,r);
  }
  
  // internal fields associated with tree
  protected Node<E> root = null;
  private int size = 0;
  
  // constructor
  public LinkedBinaryTree(){}
  
  // accessor methods that throw exception if p is invalid
  public int size() {return size;}
  public Position<E> root() {return root;}
  
  public Position<E> parent(Position<E> p) throws IllegalArgumentException {
    Node<E> node = validate(p);
    return node.getParent();
  }
  public Position<E> left(Position<E> p) throws IllegalArgumentException {
    Node<E> node = validate(p);
    return node.getLeft();
  }
  public Position<E> right(Position<E> p) throws IllegalArgumentException {
    Node<E> node = validate(p);
    return node.getRight();
  }
  
  // creates a root node if the tree is empty
  public Position<E> addRoot(E e) throws IllegalStateException {
    if(!isEmpty())
      throw new IllegalStateException("Tree is not empty");
    root = createNode(e,null,null,null,null);
    size = 1;
    return root;
  } 
  
  // add left child if p has no left child already
  public Position<E> addLeft(Position<E> p, E e) throws IllegalArgumentException {
    Node<E> node = validate(p);
    if(node.getLeft()!= null)
      throw new IllegalArgumentException("p already has a left child");
    Node<E> leftChild = createNode(e,node,null,null);
    node.setLeft(leftChild);
    size++;
    return leftChild;
  } 
  
  // add right child if p has no right child already
  public Position<E> addRight(Position<E> p, E e) throws IllegalArgumentException {
    Node<E> node = validate(p);
        if(node.getRight()!= null)
      throw new IllegalArgumentException("p already has a right child");
    Node<E> rightChild = createNode(e,node,null,null);
    node.setRight(rightChild);
    size++;
    return rightChild;
  } 
  
  // replace the element stored in node p if it is not null
  public E set(Position<E> p, E e) throws IllegalArgumentException {
    Node<E> node = validate(p);
    E tempE = node.getElement();
    node.setElement(e);
    return tempE;
  } 
  
  // attaching two subtrees to node p if it is a leaf node
  public void attach(Position<E> p, LinkedBinaryTree<E> t1, LinkedBinaryTree<E> t2) throws IllegalArgumentException {
    Node<E> node = validate(p);
    if (isInternal(p))
      throw new IllegalArgumentException("p is not a leaf node");
    size = size + t1.size() + t2.size();
    if (!t1.isEmpty()) {
      t1.root.setParent(node);
      node.setLeft(t1.root);
      t1.root=null;
      t1.size=0;
    }
    if (!t2.isEmpty()) {
      t2.root.setParent(node);
      node.setRight(t2.root);
      t2.root = null;
      t2.size = 0;
    }
  } 
  
  // remove node at position p and attach children of p to parent of p
  // only possible if parent has one child
  public E remove(Position<E> p) throws IllegalArgumentException {
    Node<E> node = validate(p);
    if(numChildren(p) == 2)
      throw new IllegalArgumentException("p has two children");
    Node<E> child = (node.getLeft() != null ? node.getLeft():node.getRight());
    if (child!=null)
      child.setParent(node.getParent());
    if (node == root)
      root = child;
    else {
      Node<E> parent = validate(node.getParent());
      if (parent.getRight() != null)
        parent.setLeft(child);
      else
        parent.setRight(child);
    }
    size--;
    E temp = node.getElement();
    node.setElement(null);
    node.setLeft(null);
    node.setRight(null);
    node.setParent(node); // sets parent of node to itself
    return temp;
  } 
}
```

#### Sequential

- All nodes are stored in a linear array, can directly access the child or parent node via indices obtained through simple computations
- Label each node from left to right level by level
- Arrange these nodes in the array at index mapped to the label
- Start at index 1
- Can directly compute indices at which the parent, left child, or right child of a node resides
  - Left child of node i: ==2i==
  - Right child of node i: ==2i+1==
  - Parent of node i: ==i/2== (round down)
- Can also determine whether nodes are roots or indices
  - Root is at ==index 1==
  - All leaf nodes reside at indices of ==index*2 > n==

#### Tree Traversal

- **Pre-order:** Visit root, then visit left subtree and then right subtree
  - General tree: visit root then children
- **In-order:** Visit left subtree, then root and then right subtree
  - Only applicable for binary tree
- **Post-order:** Visit left subtree, then right subtree, then root
  - General tree: visit children from left to right and then root
- **Level-order:** Visit nodes at each level from left to right

```java
public abstract class AbstractTree<E> implements Tree<E> {
	// helper functions for tree traversal
  private class ElementIterator implements Iterator<E> {
    // private inner class implements iterator
    // internal field posIterator converts iterable collection to iterator
    Iterator<Position<E>> posIterator = positions.iterator();
    
    // supports two accessor methods
    public boolean hasNext() {return posIterator.hasNext();}
    public E next() {return posIterator.next().getElement();}
    
    // eliminates current element in iterator
    public void remove() {posIterator.remove();}
  }
  
  //overrides the iterator constructor to return a new iterator instance defined by the Element Iterator class
  public Iterator<E> iterator(){return new ElementIterator();}
  public Iterable<Position<E>> positions() {
    // returns a collection composed of references to various elements in the tree traversed using one of the tree traversal methods
  }
}
```

<u>Pre-order</u>

```java
private void preorderSubtree(Position<E> p, List<Position<E>> snapshot) {
  // adds root of current subtree to the collection
  snapshot.add(p);
  //recursively call the function on each child
  for (Position<E> c: children(p))
    preorderSubtree(c,snapshot);
}

// creates a collection of references that can be iterated
public Iterable<Position<E>> preorder() {
  List<Position<E>> snapshot = new ArrayList<Position<E>>();
  if(!isEmpty()){
    preorderSubtree(root(),snapshot);
  }
  return snapshot;
}
```

<u>Post-order</u>

```java
private void postorderSubtree(Position<E> p, List<Position<E>> snapshot) {
  // recursively call the function on each child
  for (Position<E> c: children(p))
    postorderSubtree(c,snapshot);
  // add root to collection
  snapshot.add(p);
}
```

<u>In-order</u>

```java
private void inorderSubtree(Position<E> p, List<Position<E>> snapshot) {
  Iterator<Position<E>> cList = children(p).iterator();
  // call function on left child
  if(cList.hasNext())
    inorderSubtree(cList.next(),snapshot);
  // add root to collection
  snapshot.add(p);
  // call function on right child
  if(cList.hasNext())
    inorderSubtree(cList.next(),snapshot);
}
```

<u>Level-Order</u>

```java
public Iterable<Position<E>> inorder() {
  List<Position<E>> snapshot = new ArrayList<Position<E>>();
  Position<E> p;
  if(!isEmpty()) {
    LinkedListQueue<Position<E>> llQ = new LinkedListQueue();
    // if tree is not empty, root is enqueued first
    llQ.enqueue(root);
    while(!llQ.isEmpty()){
      // then queue is dequeued, and the nood is added to the snapshot
      p=llQ.dequeue();
      // children of node are enqueued
      for(Position<E> c : children(p))
        llQ.enqueue(c);
      snapshot.add(p);
    }
  }
  return snapshot;
}
```

<u>Pre-Order (no recursion)</u>

```java
public Iterable<Position<E>> preOrderIt() {
  List<Position<E>> snapshot = new ArrayList<Position<E>>();
  Position<E> p;
  if (!isEmpty()) {
    LinkedListStack<Position<E>> lls = new LinkedListStack();
    lls.push(root);
    while(!lls.isEmpty()) {
      p = lls.pop();
      LinkedListStack<Position<E>> tempS = new LinkedListStack();
      for (Position<E> c : children(p))
        tempS.push(c);
      while(!tempS.isEmpty())
        lls.push(tempS.pop());
      snapshot.add(p);
    }
  }
  return snapshot;
}
```

#### Modifying Operations on Heaps

- Special type of binary tree
- Items are inserted and deleted from the tree so that a *particular property* is always heeded by key elements in each node of the tree
- Key element in parent node is always *smaller* than that of the descendants (eg. int, char)
- Delete operation on the heap removes the node containing the smallest element (ie. the root)
- Items are inserted/deleted into a heap so that it is always *complete*
- Compare (key1, key2)
  - Returns 0 if key1 = key2
  - Returns negative if key 1 < key 2
  - Returns positive if key 1 > key 2



- Insertion
  - New node to be inserted is placed at the next available spot on the left in the last level of the heap (end of the array)
  - This node is *bubbled up* and exchanged with parent nodes until no more exchanges are possible
    - Compare the parent (j-1)/2 and see if this node meets the heap property
    - If it doesn’t hold, swap parent and child
    - Repeat up the tree until the root is reached
  - Since a node is always inserted into a complete tree, complexity is ==O(logn)==

```java
public interface Entry<K,V> {
  K getKey();
  V getValue();
}

public interface PriorityQueue<K,V> {
  int size();
  boolean isEmpty();
  void insert(K key, V value);
  Entry<K,V> removeMin();
  Entry<K,V> min();
}

public class DefaultComparator<E> implements Comparator<E> {
  public int compare(E a, E b) throws ClassCastException {
    return ((Comparable<E>) a).compareTo(b);
  }
}

// In the heap class:

private Comparator<K> comp = new DefaultComparator<K>();
protected int compare(Entry<K,V> a, Entry<K,V> b) {
  return comp.compare(a.getKey(), b.getKey());
}
protected boolean checkKey(K key) throws IllegalArgumentException {
  try {
    return (comp.compare(key,key)==0);
  } catch (ClassCastException e) {
    throw new IllegalArgumentException("Incompatible Key.");
  }
}

protected void unheap (int j) {
  while (j>0) {
    int p = parent(j);
    if (compare(heap.get(j), heap.get(p))>=0)
      break;
    swap(j,p);
    j = p;
  }
}
```

- Deletion — RemoveMin
  - Node at the root is removed and replaced with the right-most leaf at the bottom-most level of the heap (last element)
  - This node is the *bubbled down* and exchanged with child nodes until no more exchanges are possible 
  - Complexity is the number of levels in the heap (bubbling down operation): ==O(log~2~n)==

```java
protected void downheap (int j) {
  while(hasLeft(j)) {
    int leftIndex = left(j);
    int smallChildIndex = leftIndex;
    if (hasRight(j)) {
      int rightIndex = right(j);
      if(compare(heap.get(leftIndex),heap.get(rightIndex))>0)
        smallChildIndex = rightIndex;
    }
    if (compare(heap.get(smallChildIndex),heap.get(j))>=0)
      break;
    swap(j,smallChildIndex);
    j = smallChildIndex;
  }
}
```

- Heapifying
  - Find out which nodes are not leaves and start from the leftmost node
  - Apply “downheap” on that node and repeat until root is reached
- Comparison with other Sorting Algorithms 
  - Bubble sort: ==O(n^2^)==
  - Heapifying a tree: ==O(n)==
  - Sorting a heap: ==O(nlogn)==
  - Retrieving/removing the largest value from array sorted by bubble sort: ==O(1)==
  - Retrieving/removing a key from a heapified tree: ==O(logn)==
  - Inserting an element into an array sorted by bubble sort: ==O(n)==
  - Inserting a key into a heapified tree: ==O(logn)==

#### Binary Search Tree

- Special type of tree that has several *ordering* constraints on all nodes in the tree
- **Left subtree **of a node in a BST will have data values that are *strictly less* than the node’s key
- **Right subtree** of a node in a BST will have data values that are *strictly greater* than the node’s key
- No duplicate nodes present
- No restriction on the structure
- Have a linked implementation
- Purpose is to search for data and the typical operations on it are `search` or `insert`

<u>Search</u>

```java
public Position<Entry<K,V>> treeSearch(Position<Entry<K,V>> p, K key) {
  if(isExternal(p))
    return p;
  int comp = compare(key, p.getKey());
  if (comp==0)
    return p;
  else if(comp<0)
    return treeSearch(left(p),key);
  else
    return treeSearch(right(p),key)
}
```

```java
public Position<Entry<K,V>> treeSearch(K key) {
  int comp;
  Position<Entry<K,V>> c = root();
  while(c!=null) {
    comp = compare(key,c.getKey());
    if (comp == 0)
      return c;
    else if (comp<0)
      c = left(c);
    else
      c = right(c);
  }
  return null;
}
```

<u>Insertion</u>

```java
public V put(K key, V value) {
  Entry<K,V> newEntry = new Entry(key,value);
  int comp;
  Position<Entry<K,V>> c = root();
  while(!isExternal(c)) {
    comp = compare(key,c.getKey());
    if (comp == 0) {
      int tempVal = c.getValue();
      tree.set(c,newEntry);
      return tempVal;
    }
    else if (comp<0)
      c = left(c);
    else
      c = right(c);
  }
  if (comp<0)
    tree.addLeft(c,newEntry);
  else
    tree.addRight(c,newEntry);
  return null;
}
```

<u>Deletion</u>

```java
public V remove(K key) throws IllegalArgumentException {
  checkKey(key);
  Position<Entry<K,V>> p = treeSearch(root(),key);
  if(isExternal(p))
    return null;
  else {
    V old = p.getValue();
    if(isInternal(left(p)) && isInternal(right(p))) {
      Position<Entry<K,V>> replacement = treeMax(left(p));
      set(p,replacement.getElement());
      p = replacement;
    }
    Position<Entry<K,V>> leaf = (isExternal(left(p))?left(p):right(p));
    remove(leaf);
    remove(p);
  }
}
```

- Operations on a BST
  - Tree structure influences the performance of a BST
  - Best case: Tree is *balanced* (height of subtrees of all nodes differ by only 1) and performance is ==O(logn)==
  - Worst case: Tree is *left linear* or *right linear* (performance degrades to O(n))
  - Unlike heaps, there is no restriction on maintaining  a balance in the structure of the BST

#### AVL Trees

- **A**delson-**V**elskii **L**andis
- Class of binary search trees that are *almost balanced*
- A tree satisfies this AVL property if the difference between heights of the left sub-tree and right sub-tree of all nodes is at most 1 (shortcut: compare the longest path of both subtrees)
- In an AVL tree, every insertion or deletion operation ensures that this AVL property is maintained
- It can be shown that the operations on an AVL tree is ==O(logn)==
- Insertion or deletion operations may cause an AVL tree to lose its AVL property
- Operations to restore the AVL property include
  - Right rotation, left rotation
- Four different cases that deviate the AVL property
  - Left Left, Left Right, Right Right, Right Left
- **Balance factor BF** (height of left subtree - height of right subtree) can be used to determine whether a tree heeds the AVL property or not (+/- 1)

<u>Right Rotation</u>

```java
public Position<Entry<V,K>> rightRotate(Position<Entry<V,K>> n) {
  Position<Entry<V,K>> t1 = left(n);
  Position<Entry<V,K>> t2 = right(t1);
  t1.setRight(n);
  n.setLeft(t2);
  return t1;
}
```

<u>Left Rotation</u>

```java
public Position<Entry<V,K>> leftRotate(Position<Entry<V,K>> n) {
  Position<Entry<V,K>> t1 = right(n);
  Position<Entry<V,K>> t2 = left(t1);
  t1.setLeft(n);
  n.setRight(t2);
  return t1;
}
```

```java
struct Node * leftRotate(struct Node * n) {
  struct Node * t1 = n->rChild;
  struct Node * t2 = t1->lChild;
  t1->lChild = n;
  n->rChild = t2;
  n->height = max(height(n->lChild), height(n->rChild))+1;
  t1->height = max(height(t1->lChild), height(t1->rChild))+1;
  return t1;
}
```

- Identifying type of imbalance introduced
  - After **insertion**, check every node to see if it is balanced or not (with BF)
  - If the BF is *greater* than 1, then the left subtree of the node is deeper than the right subtree
    - If the node inserted is *lesser* than the key in the left child of the unbalanced node, then this is a **left left** tree 
      - Perform right rotation around unbalanced node
    - If the node inserted is *greater* than the key in the left child of the unbalanced node, then this is a **left right** tree
      - Perform left rotation around left child, then right rotation at unbalanced node
  - If the BF is *lesser* than -1, then the right subtree of the node is deeper than the left subtree
    - If the node inserted is *lesser* than the key in the left child of the unbalanced node, then this is a **right left** tree 
      - Perform right rotation around right child, then left rotation at unbalanced node
    - If the node inserted is *greater* than the key in the left child of the unbalanced node, then this is a **right right** tree
      - Perform left rotation around unbalanced node
- Efficiency
  - Modifying or search operations on an AVL tree is O(h) where *h* is the height of the AVL tree
  - Nodes in an AVL tree are typically placed on **two adjacent levels**
  - The minimum number of nodes on an AVL tree can be computed with the following **recursive** definition:
  - An AVL tree consists of:
    - Root node
    - One subtree with height being at least `h-1`
    - Another subtree with height being at least `h-2`
  - Assuming that the AVL tree under consideration is an extended binary tree (empty children are also accounted for) then the above is a recurrence relation
    - N<sub>1</sub> = 1
    - N<sub>2</sub> = 2
    - N<sub>h</sub> = N<sub>h-1</sub> + N<sub>h-2</sub> + 1
  - Height is **bounded** by ==log(n)== and therefore that is the AVL tree’s complexity
  - During an insertion, the tree may need to be **rebalanced**
  - Since balancing operations require constant time, complexity of insertion and deletion is also ==O(logn)==