## Graphs

### Introduction

- Graph consists of a set of nodes that may have a relationship with one another
- They are a generalization of trees with many useful and interesting applications
- **Transportation Network**: Vertices are cities and links are roads connecting cities
  - Links can represent the distance (shortest path problem) or the cost (route with minimal cost problem)
- **Oil Pipeline Network:** Vertices are pumping stations and links are pipelines
  - Each link or pipeline can have various diameters (max flow rate)
- **Graph Exploration:** Each vertex is a city and the links are road
  - Traversing a maze or the traveller’s salesman problem

### Graph Definition

- If the order of the relationship between nodes is important, then (v<sub>i</sub>, v<sub>j</sub>) != (v<sub>j</sub>, v<sub>i</sub>) and these graphs are referred to as **digraph**
  - Origin and terminus of the edge
- If the order of relationship between nodes does not need to be preserved, then *(v<sub>i</sub>, v<sub>j</sub>) = (v<sub>j</sub>, v<sub>i</sub>)* and these graphs are referred to as **undirected graphs**
- **Adjacency:** If two vertices are connected by an edge, then the two vertices are adjacent to each other
- **Path:** A collection of edges where at least one node in each edge is adjacent to a node in the consecutive edge
- **Cycle**: A special type of graph where the first vertex is equal to the last
- **Simple Cycle:** All nodes (with the exception of the first and last nodes) occur only once in a path forming a simple cycle (n > 3)
- Undirected graph
  - **Degree** of a vertex v<sub>x</sub>: Number of edges in which v<sub>x</sub> is one of the endpoints
  - **Connected:** There exists a path between the vertices
  - **Connected Component**: A set containing vertices that are all connected to one another
  - **Maximal Connected Component**: Largest subset of nodes that are all connected to another
- Directed graph
  - **In-degree** of v<sub>x</sub>: Number of nodes in Pred(v<sub>x</sub>) 
  - **Out-degree** of v<sub>x</sub>: Number of nodes in Succ(v<sub>x</sub>) 
  - **Strongly Connected**: All vertices are connected to one another

### Graph Representations

#### Adjacency Matrix

- Representation of a graph that is composed of n rows and n columns (ie. there are n vertices/nodes in a graph)
- Undirected graph
  - If there is an edge, then both entries A[i] [j] and A[j] [i] are set to 1
  - Thus A is a **symmetric** matrix
- Directed graph
  - Rows are originating nodes and columns are terminus nodes
  - If there is an edge, then A[i] [j] is set to 1
  - When there is no edge, A[i] [j] is set to 0
  - A is not always symmetric 
- If edges have weight, then weights are listed explicitly in the corresponding entry instead of 1
- Store: ==O(|V|<sup>2</sup>)==
- Accessing: ==O(1)==

#### Adjacency List

- Linked lists are engaged to form an adjacency list
- Information about every combination of edges possible in the graph is not stored in adjacency lists
- Array of n references is created where each element in the array represents a node in the graph
- Linked list residing in the i<sup>th</sup> index of the array represents all edges where v<sub>i</sub> is the originating node (all non-zero columns for the i<sup>th</sup> entry in the adjacency matrix)
- The j<sup>th</sup> node in the i<sup>th</sup> linked list represents a terminating node forming the edge *(v<sub>i</sub>, v<sub>j</sub>)*
- Store: ==O(|E|)==
- Accessing: ==O(|E|)==

<u>Graph ADT</u>

- `numVertices()`: Total number of vertices in the graph
- `vertices()`: Iteration of all the vertices in the graph
- `numEdges()`: Total number of edges in the graph
- `edges()`: Iteration of all the edge in the graph
- `getEdge(u,v)`: Returns the edge connected by vertices u and v
- `endVertices(e)`: Returns the vertices that form the edge e
- `opposite(v,e)`: Returns the other vertex of the edge incident to v
- `outDegree(v)`: Returns the number of outgoing edges from v
- `inDegree(v)`: Returns the number of incoming edges to v
- `outgoingEdges(v)`: Iteration of all the outgoing edges from v
- `incomingEdges(v)`: Iteration of all the incoming edges to v
- `insertVertex(v)`: Creates a new vertex v
- `insertEdge(u,v,x)`: Creates a new edge from u to v and store x
- `removeVertex(v)`: Deletes v and all the associated edges
- `removeEdge(e)`: Deletes edge e from the graph

### Graph Applications

- **Breadth-first** traversal is similar to level-order traversal as all siblings are visited first before visiting the descendants
  - Can use *queues* for implementing BFS

```java
public Iterable<Vertex<V>> breadthFirstTrav(Vertex<V> u) {
  List<Vertex<V>> snapshot = new ArrayList<Vertex<V>>();
  LinkedListQueue<Vertex<V>> LLQ = new LinkedListQueue();
  int [] visitedList = new int[numVertices()];
  for (int i = 0; i < numVertices(); i++)
    visitedList[i] = 0;
  LLQ.enqueue(u);
  Vertex<V> v;
  while (!isEmpty(LLQ)) {
    v = LLQ.dequeue();
    snapshot.add(v);
    for (Edge<E> e: outgoingEdges(v))
      if (visitedList[opposite(v,e).getLabel()] == 0) {
        LLQ.enqueue(opposite(v,e));
        visitedList[opposite(v,e).getLabel()] = 1;
      }
  }
  return snapshot;
}
```

- **Depth-first** traversal visits descendants first and then visits siblings
  - Can use *stacks* for implementing DFS

```java
public Iterable<Vertex<V>> depthFirstTrav(Vertex<V> s) {
  List<Vertex<V>> snapshot = new ArrayList<Vertex<V>>();
  LinkedListStack<Vertex<V>> LLS = new LinkedListStack();
  LinkedListStack<Vertex<V>> tempS = new LinkedListStack();
  int [] visitedList = new int[numVertices()];
  for (int i = 0; i < numVertices(); i++)
    visitedList[i] = 0;
  Vertex<V> v;
  LLS.push(s);
  while(!isEmpty(LLS)) {
    v = LLS.pop();
    snapshot.add(v);
    for (Edge<E> e: outgoingEdges(v)) {
      if (visitedList[opposite(v,e).getLabel()]!=1)
        tempS.push(opposite(v,e));
        visitedList[opposite(v,e).getLabel()] = 1;
    }
    while(!isEmpty(tempS))
      LLS.push(tempS.pop());
  }
  return snapshot;
}
```

#### Graph Searching

#### Topological Ordering in a Graph

- Suppose there are a set of events that take place in a process
- Some events may be **contingent** on the completion of other events
  - One example that you may be dealing with currently is course pre-requisites 
  - Courses can be nodes/vertices and directed edges indicate that one course it the pre-requisite of another
  - May be interested in order in which you can take these required courses throughout your degree
  - This notion of *order* in the graph is referred to as **topological ordering**
- Will you be able to find the topological ordering of any graph?
  - No, the graph must be **directed** and **acrylic**
- Algorithm
  - The **in-degree** of a node indicates the number of pre-requisites required before being able to take that course
  - After taking the first pre-requisite of a course v<sub>x</sub>, the number of prerequisites left before v<sub>x</sub> can be taken is *inDegree(v<sub>x</sub>) - 1*
  - Suppose that the number of pre-requisites left for v<sub>x</sub> is recorded in a variable p<sub>x</sub>
  - Initializing *p<sub>x</sub> = inDegree(v<sub>x</sub>)* and every time a course v<sub>y</sub> is taken, decrement p<sub>x</sub> variable of all v<sub>x</sub> in Succ(v<sub>y</sub>) with p<sub>x</sub> = 0

```java
public Iterable<Vertex<V>> topologicalOrdering() {
  List<Vertex<V>> snapshot = new ArrayList<Vertex<V>>();
  LinkedListQueue<Vertex<V>> llq = new LinkedListQueue();
  int [] visitedList = new int[numVertices()];
  int [] inDegArr = new int[numVertices()];
  
  for (Vertex<V> v : vertices()) {
    visitedList[v.getLabel()] = 0;
    inDegArr[v.getLabel()] = inDegree(v);
    if (inDegree(v)==0)
      llq.enqueue(v);
  }
  
  Vertex<V> u;
  while (!isEmpty(llq)) {
    u = dequeue(llq);
    snapshot.add(u);
    visitedList[u.getLabel()] = 1;
    for (Edge<E> e: outgoingEdges(u)) {
      inDegArr[opposite(u,e).getLabel()]--;
    }
    for (Vertex<V> v: vertices()) {
      if (inDegArr[v.getLabel()]==0 && visitedList[v.getLabel()]!=1)
        llq.enqueue(v);
    }
  }
  return snapshot;
}
```

#### Minimal Spanning Trees

- Consider an undirected, connected graph with weighted edges
- A **spanning tree** is a tree constructed using some edges in E that connects all vertices in the graph
- The sum of all weights on the edges used to create the spanning tree is the **weight** W of the spanning tree
- Applications of minimal spanning trees
  - Circuits (need to connect separate electric terminals but cost of wiring is important)
  - Communications: broadcasting signals
- **Prim’s algorithm** is used widely to construct the minimal spanning tree of a graph 
- Also called the **greedy** algorithm as decisions are made without taking the full picture into account
- Summary — sets T (vertex set) and U (edge set) are maintained throughout this algorithm
  - First any node in the graph is selected and added to T
  - At every iteration of this algorithm, an edge with minimal weight that connects a node not in T to a node in T is added to the edge set U
  - The node in the newly added edge that is not in T is added to T
  - This is repeated until all vertices in G are added to T