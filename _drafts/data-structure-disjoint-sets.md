---
layout: post
title: Data Structure: Disjoint-Sets
---

The most common use I have found for the *Disjoint Sets* data structure is to determine if adding a specific edge to a graph would form a cycle.
In Kruskal's algorithm, it is used in this way to build a minimal spanning tree.
Building a maze is similar to applying Kruskal's algorithm except we use random edges instead of shortest edges.
I'll discuss how to build a maze this way in the future.
This article will discuss the data structure.

In mathematics, sets are disjoint if they have no members in common.
Think of a Disjoint Sets data structure as a collection of such sets.
Initially, each set contains only one element.
Once we initialize the data structure with the number of sets, there are only two supported operations: we can ask the data structure which set an element belongs to, and you can merge two of the sets.
The data structure is sometimes called "Union-Find".

To use the `findSet()` operation, we must understand the notion of a "representative" element.
The data structure chooses one element as the representative for a set.
So if element *a* and element *b* are in the same set, and the data structure has chosen *a* as the representative, then both `findSet(a)` and `findSet(b)` will return *a*.

## Usage

In C++, the class interface for such a data structure might look like this:

```cpp
class DisjointSets {
    public:
        DisjointSets(std::size_t numElements);
        std::size_t findSet(std::size_t element);
        void unionSets(std::size_t rep1, std::size_t rep2);

    private:
        std::vector<std::size_t> nodes;
};
```

Note that the elements are of type `size_t` as this type can represent the maximum number of elements we can keep in a vector.
If there is satellite data it can be stored elsewhere in a vector indexed by these elements.
This keeps the implementation simple.

Upon creation there will be `numElements` sets, each containing one element, the representative for that set.
That means that initially `findSet(n)` returns n for all n < numElements.

Now if we call `unionSets(0,1)`, then `findSet(0)` and `findSet(1)` will both return *0* or both return *1*, depending on the implementation.
This return value is the representative for the new set containing *{0,1}*.
If we then call `unionSets(findSet(0), 2)` there will be one set that contains *{0,1,2}* and single-element sets for each of the remaining elements greater than 2.
Note that `unionSets()` takes representatives as arguments.
So we can't simply call `unionSets(0,2)` after we called `unionSets(0,1)` because the representative for that first set may be *0* or *1*.
We could find the representative on behalf of the caller in the implementation of `unionSets()`, but doing so would impose a performance penalty on those callers who don't need it.
The proper mechanism here is an assertion that the parameters are in fact representatives.

If we continue to call `unionSets(findSet(0), n)` repeatedly for the rest of the values of *n < numElements-1*, `findSet(n)` will return the same value for every *n* because there will be only one set.

## Implementation

The way I have chosen to implement this is to use a `vector<size_t>` where each element contains either its own index position, indicating that it is a representative, or the index to a parent position in the vector which should be a parent in the chain used to find the representative.
Links are followed this way until an index is found that "points" to itself, indicating that it is the representative.

Initially, each element contains its own index.
For example, the vector v is initialized to *{0,1,2,3,...,numElements-1}*.
This means each of numElements set contains one element and the representative for set *n* is *n* because *v[n] == n*.
Once we call `unionSets(0,1)` the vector might look like this: *{0,0,2,3,...,numElements-1}*.
`v[1]` "points" to `v[0]` and `v[0]` points to itself, so *0* is the representative of the set *{0,1}*.
`findSet(0)` and `findSet(1)` both return *0*.

To continue the example, if we then call `unionSets(findSet(0),2)`, the vector might look like this: *{0,0,2,3,...,numElements-1}*.
`v[2]` points to `v[1]` and `v[1]` points to `v[0]` and `v[0]` points to itself.
So *0* is the representative for the set containing *{0,1,2}*.
Note that the vector could also be represented as *{0,0,0,3,...,numElements-1}* and the result would be the same: `v[1]` and `v[2]` both point directly to their representative.
This is an implementation detail and an optimization opportunity.
`findSet()` could perform better if each element in a set pointed directly to its representative rather than having to traverse a tree to the root each time.
This is referred to as "path compression."
When I measured the effect of this optimization on a large test problem, the execution time dropped from over 100 seconds to 1 second.

The implementation of `unionSets()` can be as simple as:

```cpp
void DisjointSets::unionSets(size_t rep1, size_t rep2)
{
    nodes[rep2] = rep1;
}
```

After this call, `nodes[rep2]` points to `rep1`, and `rep1` is either the representative or points to the next parent in its tree.

A recursive implementation of `findSet()` can be as simple as:

```cpp
size_t DisjointSets::findSet(size_t element)
{
    if (nodes[element] == element) {
        return element;
    }

    return findSet(nodes[element]);
}
```

This implementation uses tail recursion, which enables the compiler to turn the recursion into a loop.
However, it does not have the path compression optimization described above and it was the slowest implementation I measured by far.

Below is the same recursive implementation with path compression added in the last line.

```cpp
size_t DisjointSets::findSet(size_t element)
{
    if (nodes[element] == element) {
        return element;
    }

    return nodes[element] = findSet(nodes[element]);
}
```

In this version, we lose the tail recursion optimization but gain a huge performance win by improving the runtime complexity.
This version runs in one one-hundredth of the time as the first version.
But now we have to worry about overflowing the stack.

Since the compiler can't eliminate the recursion any more, we can rewrite the function ourselves as a loop.

```cpp
size_t DisjointSets::findSet(size_t element)
{
    size_t orig = element;

    while (nodes[element] != element) {
        element = nodes[element];
    }

    nodes[orig] = element;

    return element;
}
```

This is slightly slower than the recursive version because the path compression is less aggressive, but is safer because we no longer have to worry about the stack.
We could improve the path compression in this version at the expense of making it a little harder to read, but the gain would be small and so I'll skip it for now.

For completeness, our constructor looks like this:

```cpp
DisjointSets::DisjointSets(size_t numberElements)
    : nodes(numberElements)
{
    iota(begin(nodes), end(nodes), 0);
}
```

The call to `std::iota()` in the constructor initializes the vector to {0,1,2,3...,numberElements-1}.

## Applications

In the introduction, I mentioned Kruskal's algorithm and generating a maze.
Here are a couple other applications of the data structure.

### Disconnected Subgraphs

We can use Disjoint Sets to answer the question, "Given a graph, how many disconnected subgraphs does it contain?"

To answer this question, simply iterate each edge, add both connected vertices to the data structure, then count the number of sets.
To do this, our implementation could track the number of sets, decrementing it each time a union is performed, and provide a member function to retrieve the count.
Or we could use it as-is and call `findSet()` iteratively after adding the vertices, and then count the number of unique representatives it returns.

### Detecting a Cycle

We can use Disjoint Sets to answer the general question, "Given a graph, are there any cycles?"
This is similar to how Kruskal's algorithm uses it.

To answer this question, iterate each edge in the graph.
We call `findSet()` for each of the two vertices of the edges.
If the two calls return the same representative, then there is already a path between the vertices and adding the vertex in question would create a cycle and we have answered the question and can stop.
If not, we call `unionSets()` for the two vertices and continue with the next edge.
Once we have visited each edge and have not stopped then there are no cycles.

## Conclusion

There aren't many uses for the Disjoint Sets data structure, but it is really useful in solving certain graph problems.
It can be implemented in a few lines of code and made efficient by adding path compression.
File this away somewhere so it will be handy when you need it.
