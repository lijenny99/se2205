## Hash Tables

### Introduction

- A **table** is an ADT that consists of table entries each containing a **unique** key *K* and associated information *I*
- Each table entry is then a **key pair (K,I)**
- How can tables be represented?
  - Arrays
  - Binary search trees
  - AVL trees
- **Hashing** is an alternative that is very efficient under some circumstances
- **Hash functions** *h(K)* map a key *K* to an address in a table
- If the function *h* provide a **unique one-to-one mapping** of *K* to an address, then table entries can be accessed directly
- If not, it is necessary to evoke **collision resolution policies**

### Collision Resolution Policies

- Collisions occur when *h(K) = h(K’)* where *K* and *K’* are different keys (ie. not a one-to-one mapping)
- Can be resolved via collision resolution polices: open addressing and separate chaining
- Hashing with **buckets**: Each bucket will represent a sub-table

#### Open Addressing

- When a collision occurs, a probe for other available locations is performed
- First, the location at which (K,I) is to be stored is determined through the hash function h(K)
- One assumption with open addressing is that there is at least one empty spot available for the storage of *(K,I)* (ie. table is not full)
- If there is a collision, then a sequence of probing is performed to search for the next available spot
- **Linear probing:** Search decrement is constant for all keys
  - Each consecutive table entry is examined until an available spot is found
  - **Probe sequence** for every colliding key can overlap
- **Double hashing:** Search decrement is not the same for all keys
  - Probe sequence has a higher chance of being different for colliding keys
  - Can find an empty spot faster

```java
// Hash function
public int h(Key<K> k) {
  return k.getValue()% M;
}

// Probe decrement
public int p(Key<K> k, int M) {
  int quo = k.getValue/M;
  if (quo != 0)
    return quo;
  else
    return 1;
}

// Key insertion
public void insertKey(Key<K> k, Info<I> info, Table<T> t) {
  int index = h(K);
  int probeDecr = p(K);
  while (t.get(index)!= null) {
    index = index-probeDecr;
    if (index<0)
      index = index + M;
  }
  t.set(index,k,info);
}

// Key searching
public boolean searchKey(Key<K> k, Table<T> t) {
  int index = h(K);
  int probeDecr = p(K);
  while (t.get(index) != null) {
    index = index - probeDecr;
    if(index<0)
      index = index+M;
  }
  t.set(index,k,info);
  if (t.get(index)!=null)
    return false;
  else
    return true;
}
```

#### Separate Chaining

- All colliding keys are stored in a linked list in ascending order of the key value
- If the amount of table entries is large, then the table is divided into buckets where each bucket represents a subtable
- Each subtable can be assigned a linked list in which key pairs are stored in ascending order
- Allows for conversation of space 

### Attributes

- A good hash function is one that maps keys in a **uniform or random manner**
- What this means is that the chance of mapping a key into any spot in the table is equally likely
- What do you think about how often a collision may take place in the table?
  - More frequent than what you think
  - Example: von Mises paradox
- Some options for selecting hash functions
  - **Dividing**
    - When double hashing is used, *h(K) = K%M*, *p(K) = max(1, K/M)* and M is a prime number
    - *h(K)* is the remainder and *p(K)* is the quotient
  - **Folding**
    - Key is divided into sections and these sections are added together (ie. K = 982 347 812, *h(K)* = 982 + 347 + 812 + 2141)
  - **Middle-squaring**
    - Key is divided into sections and the middle section (ie. K = 982 347 812, *h(K)* = 347<sup>2</sup>)
  - **Truncation**
    - Key is divided into three sections and the last section is retained (ie. K = 982 347 812, *h(K)* = 812)
- Table attributes
  - When open addressing is used, the number of slots required to store M entries are reserved beforehand
  - Available table locations for storage range from 0 to M - 1
  - **Load factor α** (0 <= α < 1): suppose that the table has N occupied entries
    - α = N/M
  - **Clustering:** Sequence of adjacent occupied entries in the hash table (puddles)
  - **Primary Clustering:** Encountered in *linear probing* where colliding element expands the cluster
    - Once formed, puddles grow very fast (puddle formation, growth mergers) with linear probing but this is not the case for double hashing
- Ensuring that probing sequence covers the entire table
  - *Linear* probe sequence will cover the entire table!
  - How about *double hashing*?
    - If you examine all the values p(K) may take for table of size 7, 11, 13, …, the probe sequences do cover the entire table
    - These are all *prime numbers*
  - It can be shown that as long as p(K) and M are **relatively prime** (ie. no common divisor except 1) then probe sequences via double hashing will cover the entire table
  - Could also set *M = 2<sup>k</sup>* and forcing p(K) to be odd numbers
    - p(K) = {1, 3, 5, 7} and M = 7

### Performance

- Successful search refers to the key already existing in the table and unsuccessful search refers to the insertion of a new key into the table

![image-20190420182221820](/Users/jennyli/Library/Application Support/typora-user-images/image-20190420182221820.png)

- **Small load factor:**
  - Successful search: All three techniques have the same performance
  - Unsuccessful search: Separate chaining has better performance
- **Medium load factor:**
  - Successful search: Separate chaining is slightly better than the other two techniques
  - Unsuccessful search: Separate chaining is the best and then double hashing
- **Large load factor:**
  - Successful search: Separate chaining is the best by a large gap
  - Unsuccessful search: Separate chaining remains effective while the other two techniques degrade
- If the load factor is less than 0.5, then the average number of comparisons range from 1.25 to 2
- Since the performance is **not dependent** on the size of the table n, the performance of an insertion or a search is essentially ==O(1)==
- If the table is guaranteed to be **half full** at the time, then the performance will remain at ==O(1)==

