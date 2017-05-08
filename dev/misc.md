---
layout: page
title: Misc
---

# Misc Topics

See https://www3.cs.stonybrook.edu/~algorith/# for more information on many of these topics.

## Topological Sort

See http://courses.cs.washington.edu/courses/cse326/03wi/lectures/RaoLect20.pdf.

This is sorting a DAG such that each directed edge goes from left to right.
The algorithm can find cycles.
It is useful for building a compile order from a dependency list.
It can be used to find the shortest path by adding step after the sort.
See next section

Build the graph using an adjacency list plus and auxiliary int array counting incoming edges for each vertex.
Then for each vertex in the auxiliary array with incoming edge count of 0, put the vertex into a processing queue.
Process each item in the queue by moving it to the output list and decrementing the incoming queue for all its adjacent edges.
Each time you decrement an incoming count to zero, add that vertex to the processing queue.
If the processing queue is empty and there are still nodes with incoming counts, there is a cycle.

See `~/src/dependency-graph` for the code.

### Shortest Path using Topological Sort

Start with a topological sort.
Create a new vector of distances equal to the number of vertexes.
Initialize it with infinity.
Set the distance of the starting node to 0.
For each vertex in the output list, for each of its edges, set the value of the distance array for the destination vertex to the min of that destination vertex distance and the current vertex distance plus the weight of the edge between the two.

The shortest path to node n is the value of the distance array at index n.

See `~/src/shortest-path-topological-sort`.

## Sieve of Eratosthenes

Used to find all prime numbers up to a value n.

Make an array of bool of size n to represent the list of primes, initialized to true.
Set 0 and 1 to false and 2 to true.
For each non-crossed-off item (true), cross off its multiples by setting them to false and advance to the next non-crossed-off values.
When you get to the end of your vector, the non-crossed-off values are the primes.

See `~/src/sieve-of-eratosthenes`.

## Max Subarray

### Brute Force

Just move two indexes in a nested loop, resetting the sum each time you move the left index forward.

See `~/src/max-subarray`.

### Recursive

This is better than brute force: O(n log n).

    Divide the given array in two halves
    Return the maximum of following three
        Maximum subarray sum in left half (Make a recursive call)
        Maximum subarray sum in right half (Make a recursive call)
        Maximum subarray sum such that the subarray crosses the midpoint

The base case is when there is only one element.

See `~/src/max-subarray`.

### Kadane Algorithm

This is a dynamic programming solution, O(n).
It is not obvious to me how it works--I have to think about it every time.

    Initialize:
        max_so_far = 0
        max_ending_here = 0
    Loop for each element of the array a
        max_ending_here = max_ending_here + a[i]
        if (max_ending_here < 0)
                max_ending_here = 0
        if (max_so_far < max_ending_here)
                max_so_far = max_ending_here
    return max_so_far

See also the Python code at <https://en.wikipedia.org/wiki/Maximum_subarray_problem>.

See `~/src/max-subarray`.

## Knapsack Problem

Given a knapsack that can hold weight W and n items with weight w and value v, choose items to fit that will maximize the value.
The weights are integers and an item can be fully included or excluded (0/1.)

Note that for partial values, sort by value:weight ratio and use a greedy solution.

### Brute Force

The brute force approach is to take all combinations of items.
This is like counting from 0 to 2 to the n.
Each bit holds the include/exclude decision for an item.
So exponential order runtime.

### Recursive

The next approach is recursive.
The function takes the list of values, how much capacity is left, the current total value, and the item number under consideration.
The total value is returned.
Terminate if there is no more capacity or no more items.
If there is not enough capacity for this item, skip it can call recursive with item+1.
Otherwise, call recursively with this item and without and return the max.
Note that if we have more than one of each item, call recursively with n instead of n+1 to include this item again.

    KNAP-IND-REC(n,c,w,W)
    if n <= 0
        return 0
    if W < wn
        withLastItem = -1 // undefined
    else
        withLastItem =cn+KNAP-IND-REC(n-1,c,w,W-wn)
    withoutLastItem = KNAP-IND-REC(n-1,c,w,W)
    return max{withLastItem,withoutLastItem}

See `~/src/knapsack-recursive`.

### Dynamic Programming

And finally there is a dynamic programming approach.
Need to build up a 2D table.

* This is the best description of the DP algorithm implementation: https://www.youtube.com/watch?v=EH6h7WA7sDw.
* This is pretty good too, and has an alternative way of finding the kept values from the table: https://www.youtube.com/watch?v=8LusJS5-AGo.

See `~/src/knapsack`.

#### Make a two dimensional solution array.

* The number in the cells is the solution of the subproblem--the highest value.
* The array represents a set of smaller knapsack problems.
* One dimension is knapsack capacity.
Lets make this columns.
* The other dimension is the index into the item array.
Lets make this rows.
* A problem with 0 items has a solution of 0.
So the first row is all 0's because the capacity doesn't matter if we have no items.

#### Fill it out

* Start with subproblem with 1 item and capacity of 1.
* Does the item fit?
** Fill in 0 if it doesn't fit.
** If it does fit, determine how much space is left over.
Can I include anything in that leftover space?
To figure that out, look in the previous row in the column for that amount of leftover space.
We get to add that amount to the value of the current row and put that into the cell.
** Compare that combined weight with the value of the cell in the row above.
Take the higher value and put it into the cell.
If the took the value from the cell above, that means we aren't using the item for this row.

The answer is in the bottom right cell.

## Making Change

There are several problems here:

* How many ways are there to make change with a given set of denominations?
* Can you create a exact change with the given set of denominations?
* Make change using the fewest number of coins.

### How many ways are there to make change?

#### Recursive

The function takes the array of values, the index into the array under consideration, and the sum we are trying to find.
We want to keep subtracking values from the sum until it is exactly 0.
So the terminating conditions are:

* the sum is 0. We return 1 in this case, indicating that this is 1 way where the sum can be obtained.
* the sum < 0. Return 0 in this case because the step can't reach the sum exactly given its value.
* n < 0. Return 0 because we have reached the start of the array without a solution.

The body of the recusive function returns the count considering this item and trying to use n again plus the count not considering this item and trying the previous n.

See `~/src/ways-to-make-change-recursive`.

#### Dynamic Programming

Create a 2D array S with coins down the side and amounts up to amount needed across the top.
Note the existance of a 0th row and column, which makes the array 1 wider than the amount needed and 1 taller than the number of coins.
Initialize the 0th column with 1's.
Initialize the 0th row with 0's.
Array v is the value of the coins, 0-based.
Watch for off-by-1 errors.

    for each row starting with 1
        for each col starting with 1
            if coin value for current row (v[row-1]) <= col
                S[row][col] = S[row-1][col] + S[row][col - v[i-1]]
            else
                S[row][col] = S[row-1][col]
    return value in botton right cell

See `~/src/ways-to-make-change-dp`.

### Can you create change?

You can answer this question in terms of the first question: if the number of ways to make change is greater than 0, the answer is yes.
In thinking about this, I solved it in a different way.
The first time I saw the question, I considered this an instance of the general knapsack problem because I had just solved that.
The value of each item is the same as the weight.
You create multiple items for each denomination, up to the number of coins you are allowed that add up to the sum you are looking for, or the number you have on hand.

### Make change using the fewest number of coins

It happens that the US money system uses values such that a simple greedy algorithm can be used to make change.
Otherwise you can solve it as described in the previous section _Can you create change?_ by sorting the coins array from largest to smallest first.

## Boggle

To find all the words:
Built a trie of dictionary words.
It needs isPrefix() and isWord() methods.

Build an adjacency list out of the word grid.
This is tedious because you have to check the edge of the grid.
Then create a vector of bool to represent the visited nodes.
For each node in the grid, traverse recursively with a clean found word string and visited vector.
You need to bring along:

* the grid
* the output list, by reference so you can add to it
* the node to start from/visit next
* the adjacency list, by const ref since it won't be modified
* the visited vector, by value since we need it going forward but not back
* the word so far, by value
* the trie, by const ref.

Terminate the recursion if this node was visited.
Mark the node as visited.
Grow the string so far by adding the char at the current node.
Terminate if the string is not a prefix (and therefore won't be a word if we continue).
This prunes that branch of the tree.
If the string is a word in the trie, add to the output list, but keep going.
For all adjacent neighbors, call recursively.

See `~/src/boggle`.

## Lowest Common Ancestor

To find the lowest common ancestor, find the node where the values are in different children or you found one of the nodes.
That means if one node is the child of another, the node that is the parent of the other is returned, not the parent of the parent.
You can do this iteratively from the root.

See `~/src/lowest-common-ancestor`.

## Edit Distance

See https://web.stanford.edu/class/cs124/lec/med.pdf.

Three kinds of edits:

* substitute a new char for an existing char
* remove a char
* insert a char.

Usually, each has cost 1, but substitution sometimes costs 2, in which case it can be replaced with a remove/insert pair.

Create a 2D table.
The first word across the top and the second word down the side.
Initialize first row and column with 0-n.
Build up the table.
For each cell starting with 1, we want the minimum distance of
a) the min of the distances in adjacent row plus 1 and column plus 1, which would be an insert, and
b) the distance in the diagonal distance + 1, which would be a substitution (if allowed).

The minimum edit distance is in the last cell.

See `~/src/edit-distance`.

## NP/P/NP Complete/NP Hard

An NP problem "yes" solution can be verified in polynomial time.
A P problem can be solved in polynomial time.
The "P=NP?" question asks, is there a polynomial solution for all NP problems?
NP Complete is a set of decision problems such that if any of them can be solved in polynomial time, all of them can be, and the world would be a fundamentally different place.
No polynomial time solution is known for these problems.
NP Complete problems are NP and NP Hard.
NP Hard includes problems that can be reduced to an NP problem in polynomial time, so are at least as hard as NP problems.

Pedantically, the traveling salesman problem, where the problem is to find the lowest cost path, is not NP complete.
It is NP hard, but it is not NP because you can't verify the that solution is optimal in polynomial time.
Since NP complete requires NP, the TSP optimization problem is not NP complete.
However, the decision problem, "can a given path of less than x cost be found" can be verified in polynomial time so this question is NP complete.
It is usually easy to convert an optimization problem to a decision problem that is no harder.

The following problems are NP-complete:

* Determining if a graph contains a simple path with at least a given number of edges.
* Determining if a directed or undirected graph has a Hamiltonian cycle.
* Determining if a clique of a given size exists in a graph.
* The subset-sum problem.
* Can a graph be colored in 3 colors.
* The vertex cover problem.
* And thousands more.

## Bellman-Ford Algorithm

Solves the single-source shortest-path problem on weighted directed graphs in the general case where weights may be negative.
Slower than Dijkstra's, O(VE), but more versatile because of the allowance for negative edges.
It can detect negative cycles, the presences of which makes the problem unsolvable.
It is also suited to distributed implementations.
It is used in Internet routing.

This is a Dynamic Programming solution.
The 2D table has been condensed down to a single array, dist.
It's builds up the table starting with paths with at most 1 edge and increasing from there.

    There are V vertices.
    Array dist of size V.
    Init array dist to ∞.
    For starting node s, dist[s] = 0.
    Repeat V-1 times (because there are at most V-1 edges in any simple path)
        For each edge u->v (you can iterate a list of edges or the edges leaving each node)
            dist[u] = min(dist[u], dist[v] + l(v,u)) (do nothing if dist[u] is ∞.)
    If you do this one more time and any dist changes, there is a negative cycle.

See `~/src/bellman-ford` for the code.

## Dijkstra's Algorithm

Finds the shortest path in weighted, directed graph.
Weights cannot be negative.

    Each node is labeled with the (tentative) distance from the start to the target.
    Initialize source node label with 0 and initialize every other node to infinity.
    Keep a fibonacci heap unvisited nodes where the node label is the priority (or just search a list).
    Add all nodes to it.
    Consider all unvisited neighbors of the current node.
    If the current node is reachable (dist[current] != infinite) find the sum of the current node's label and the weight of the edge to the neighbor.
    If it's less than the neighbor's label, change the label to this value.
    When all neighbors are visited, mark the current node as visited and remove it from the unvisited nodes set.
    If the target node is marked visited, then the algorithm is done.
    Otherwise, find the unvisited node with the lowest label and visit it.

O(E + V log V) because we loop through every edge and we keep the vertices in a heap.

See `~/src/dijkstra` for the code.

## Prim's Algorithm

Greedy algorithm to find the minimum spanning tree in a connected, weighted, undirected graph.
(A spanning tree is a subgraph that connects all nodes.
A minimum spanning tree is a subgraph with minimum weight.)
It could handle forests by running it on every disconnected component of the graph.
O(E + V lg V).

    Create an empty set for visited nodes (MST).
    Pick an arbitrary node to start with.
    Add current node to the visited set.
    Find smallest edge from all visited to unvisited nodes and add to the MST.
    Use a fibonacci heap by edge weight as the key and emanating vertex as the value (or just search for it).
    Repeat until all vertices are in the MST.

The implementation is very similar to Dijkstra's algorithm.
Start from an arbitrary node, like the first in your list.
Instead of updating the dist vector with the distance so far plus the distance to the adjacent, you only update it with the distance to the child.
You also need to keep a vector of parents so that each time you update a dist for a child, you add the current to the parent list as the value at the child index.
With this and your adjacency list/matrix you can build the MST: for each node v > 0, the edges are between v and parent[v].

## Kruskal's Algorithm

A greedy algorithm used to find the minimum spanning tree in a connected, weighted undirected graph.
If not connected, finds the MST forest.
It uses Disjoint Sets.
O(E lg V).

At a high level:

* Sort all edges from low to high.
* Pick smallest edge.
Check if it forms a cycle with the MST forest.
If not, add it.
Else, discard it.
* Repeat until there are v-1 edges in the MST.
* The results will be a list of edges.

To implement:

    For this problem, the graph may be represented as a list of edges (source, dest, weight.)

    Initialize a priority queue with edges (source, dest, weight) by minimum weight.
    Create a disjoint set for the vertices.
    Create a result list of edges.
    While edgeCount < number of vertices-1 and priority queue is not empty
        Remove edge from priority queue
        If the two ends of the edges not in same set in disjoint set,
            Union the ends
            edgeCount++
            Add the edge to the result list.
    Return the result list, which is the list of edges in the MST.

See `~/src/kruskal` for the code.

## Breadth First Search for Shortest Path

Find a shortest path in a connected directed or undirected graph which may have cycles.
Can be used to find the distances from a start node to every other, or the path from the start to any other.

    Init a visited array to false.
    Init a dist array to infinity.
    Init a prev array to -1.

    dist[start] = 0
    visited[start] = true
    Add start to queue.
    While queue not empty
        dequeue v
        for each w adjacent to v
            if not visited[w]
                visited[w] = true
                prev[w] = v
                dist[w] = dist[v] + 1
                enqueue(w)

Upon completion, you can walk back form prev[n] to the start to find the shortest path.
You can also look at dist[n] to find the distance to n, or max if no path.

See `~/src/breadth-first-search-shortest-path` for the code.

## Permutations

### std::next_permutation()

    In a while loop:
        Start with a sorted vector, lowest to highest.
        Work from right to left and find the place where two adjacent numbers are in order: the left is less than the right.
        Now go back to the right end and find the first number greater than the left one you just found.
        Swap those two.
        Reverse the array from the right position you just swapped to the end.

See `~/src/permutations` for the code.

### Steinhaus–Johnson–Trotter algorithm

This a recursive algorithm that generates a list of permutations by only swapping adjacent pairs.
The sequence of permutations for a given number n can be formed from the sequence of permutations for n − 1 by placing the number n into each possible position in each of the shorter permutations.
When the permutation on n − 1 items is an even permutation then the number n is placed in all possible positions in descending order (left to right), from n down to 1; when the permutation on n − 1 items is odd, the number n is placed in all the possible positions in ascending order.

This differs from std::next_permutation() in that if the input has repeated values, there will be duplicates in the output.

See `~/src/permutations` for the code.

## Floyd-Warshall Algorithm

This is used to find the shortest path between all pairs of nodes in a directed graph.
Instead, you could run one of the single-source shortest-path algorithms instead, but this is faster.
Negative weights are allowed, but not negative cycles.

    Initialize a solution matrix as the source adjacency matrix with non-edges set to ∞ and distance to self set to 0.
    For k = 0 to V
        For i = 0 to V
            For j = 0 to V
                dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])

This is O(V³)

## Trie

Also known as a prefix tree.
One can use a trie to build a suffix tree.

### Implementation

Use a nested Node class that maps char to SP to Node and keeps a bool flag to tag the end of a word.
The outer class holds a Node member variable named "children'.

found(string) is iterative:

    Set current node pointer to the root.
    For each char in the string, find a node value in children map for the key char and sets current to the found node.
    If no found node, return false.
    If all chars found, return true.

Can keep a bool in the Node to indicate if the node terminates a word.
This allows you to implement isPrefix(string) as found(string) above and isWord(string) by returning the bool "word" flag of the last node found.

insert(string) is iterative:

    Set current node pointer to root.
    For each character in the string:
        if the char does not exist in current->children, create a new node and add it to children and set current to it.
        else set current to the existing node in current->children.
    If tracking prefix vs. word, set the bool "word" flag on the last child.

See `~/src/trie` for the code.

## Suffix Tree

This is useful for determining if a string is a substring of another or the longest common substring of two strings.
A suffix tree can be implemented by inserting all n suffixes of a string of length n.
A generalized suffix tree is one made up of different words, each with a different termination symbol.
A generalized suffix tree can be used to find the longest palindrome in a string.

See http://algo2006.csie.dyu.edu.tw/paper/2/A24.pdf.

## Red-Black Tree

A balanced binary search tree.
Each node stores its color.

### Properties

* Every node is red or black.
* The root is black.
* the leaf nodes (NIL) are black.
* If a node is red, then both its children are black.
* For each node, all simple paths from the node to descendant leaves contain the same number of black nodes.

Rebalancing is done via local rotations.

## AVL Tree

A balanced binary search tree.
Each subtree of a node differs in height by at most one.
Each node stores its height.

## Splay Tree

A self-balancing tree that moves recently accessed to the root by rotations.
Commonly accessed nodes are near the root.

## Rod Cutting Problem

You can solve this using DP.
Build a solution table from the bottom up.
The first solution is 0.
Iterate through all lengths to fill up the table.
For each iteration, evaluate cutting the rod into one piece of various lengths and add the price of each length plus the solution of the remainder from the table.
You want the max of those.
Put that into the solution table for this iteration and continue.

See `~/src/cut-rod` for an implementation.

## Longest Common Subsequence

A subsequence is a string in which all the letters are in common to two other strings, in the same order but not necessarily consecutive.

x: ABCBDAB +
y: BDCABA

Some Longest Common Subsequences (LCS) include DBAB, BCAB, BCBA, etc.

### Brute Force

For every subsequence of x, is it a subsequence of y?
Use the increasing bit vector technique to find subsequences.
You are counting from 0 to 2^m, where m is the length of y.
And you are scanning y of length n every time.
This algorithm is O(2^m · n), or O(2^m).
Exponential.

### Dynamic Programming

Need a 2x2 table, with strings across side and top, like edit distance.
Initialize 0 row and column with 0's.
This is similar to edit distance, except that the table tracks the distances of matches, not edits, so it sort of has the inverse meaning.
When s[j] == s[j], the length of the substring increases, so d[j][i] =  d[j-1][i-] + 1.
When they don't match, d[j][i] = max(d[j-1][i], d[j][i-1]).

Once built, you can reconstruct the LCS by tracing backwards through the table.
Start at bottom right.
If same as adjacent, move to adjacent.
If not, select the char in the row and column (it should be the same) and move diagonal.
If you choose a different adjacent cell because they both match, you will select a different LCS.

You can also consider this problem an edit distance where only insert and delete are allowed.

See `~/src/longest-common-subsequence` for an implementation.

## B-Trees

Balanced search trees designed to work well on disk.
Have a high branch factor, often tied to the characteristics of the disk.
Each node stores keys which serve as dividing points for each child.

## Fibonacci Heaps

This type of heap is mergeable.
It also supports "decrease-key" which assigns an element a new key value, which is no greater than the current key.
This makes it useful in Dijkstra's and Prim's algorithms.
It also supports "delete".

## Disjoint Sets

This data structure maintains a collection of disjoint dynamic sets.
Each set is represented by a representative.
We get these operations:

* make-set
* union
* find-set

We can use this data structure to determine if two nodes are connected.
We do this by unioning both nodes of all edges to the disjoint set and checking if find-set returns the same representative for both nodes.

Kruskal's algorithm makes use of this data structure.

See `~/src/kruskal` for an implementation.

## Graph Coloring Problem

Given a graph G and k colors, assign a different color to each node so that adjacent nodes get colors.
The chromatic number is the minimum coloring.
A graphic with maximum degree d has chromatic number d+1.
Sudoku is a graph coloring problem where there is an edge between cells if they are in the same row, column, or block.
BTW, any _map_ can be colored in 4 or fewer colors.

A cycle with an odd number of vertices takes 3 three colors and one with an even number takes 2.

### Brute Force

Test all possible color combinations.
This is like counting in base k for k colors from 0 to V.
O(V^k).

### Greedy Approach

Visit each node in some order and assign it the lowest legal color for that node give the adjacent nodes.
For a graph of maximum degree d, this results in at most d+1 colors.
Different orderings may result in better or worse colorings.

### Backtracking

Given a graph and k colors, can the graph be properly colored?
The solution is like the greedy approach, except that when we get to a point where we can't chose a legal color because we used them all up, we backtrack and use the next legal color.
It is recursive.

    Initialize an array of colors of size V to -1 in indicate uncolored.
    The boolean recursive function takes the adjacency matrix, the color array, the current node, and the number of allowable colors.
    The base case returns true if no more nodes.
    For all colors
        If color is legal
            Assign to colors[current]
            If calling recursive with current+1 returns true
                return true
            else try the next color.
    If all colors tried, return false.

See `~/src/graph-coloring` for an implementation.

### Bipartite Testing

Use BFS, coloring the current node in-order and then coloring the neighbors with 2 alternating colors.
If a neighbor is already assigned a different color, the test fails and the graph is not bipartite.

## Node Cover Problem

Here is an algorithm for an approximation.
The result will be no worse than 2x the optimal solution:

    The result set will be a set of nodes.
    For each edge (u,v) in E
        Add u and v to result.
        Remove all edges from E which are incident on u or v.

## Traveling Salesman Problem

This is NP hard.

### Brute Force

The brute force solution is O(n!).
Just list all permutations and calculate the distance.

### Approximation Using MST

The cost will be no worse than 2x that of the optimal solution as long as the cost function satisfies triangle inequality..

    Create an MST using Prim or Kruskal.
    Visit it in preorder.

### Dynamic Programming

There is a dynamic programming algorithm that is exponential is time and space.
I don't know how it works.

## Heaps

If your nodes start at index 0, then:

* left child = 2n+1
* right child = 2n+2
* parent = (n-1)/2

If your nodes start at index 1, then;

* left child = 2n
* right child = 2n+1
* parent = n/2

To heapify, start at the lowest parent and run `shiftDown()` on all nodes in reverse.

To `shiftDown()`, iteratively find largest of current and children.
If a child is largest, swap it with the parent and iterate down with the swapped node as the new parent.
Recursion not required here.

`popHeap()` swaps the first and last and pushes the new root down using `shiftDown()`.

`pushHeap()` add the new item to the end, and swaps it with the parent if out of order.
Then keep pushing up if still higher than new parent.

## Producer/Consumer Problem

You have a buffer into which a producer deposits data and a consumer retrieves data.
A producer cannot add data to a full buffer and a consumer cannot remove data from an empty buffer.

### Semaphores

You need two semaphores and a mutex.
The first semaphore puts the producer to sleep if the buffer is full when it has a new item to add and tries to decrement it.
It is initialized to the size of the buffer.
It is incremented when the consumer removes and item.
The second semaphore puts the consumer to sleep when the buffer is empty and the tried to decrement it.
It is initialized to zero.
It is incremented by the producer after it has added an item.
The mutex protects the buffer from multiple threads.

See `~/src/producer-consumer-semaphore` for an implementation of semaphores using condition variables used in a solution to the problem.

### Condition Variables

A condition variable works in conjunction with a mutex.
It it used to communicate between threads, like a Windows "event".
To block until a condition it true, you lock the mutex, then call wait() on the condition variable, specifying the condition to wait for.
The condition variable will release the mutex and block until another thread call notify_one() or notify_all() on the condition variable.
At that point it takes the mutex again.

To use a condition variable to solve the producer/consumer problem, the producer waits until the queue is not full, adds to the queue, releases the mutex, then calls notify().
The consumer waits until the queue is not empty, pulls from the queue, releases the mutex, then calls notify().
The work of producing and consuming does not need to be protected by the mutex: only accessing the queue.

See `~/src/producer-consumer-condition-variable` for an implementation.

## Reader/Writer Problem

In this problem, threads share data that some threads "read" and other threads "write".
Want to allow multiple concurrent readers but only a single writer at a time, and if a writer is active, readers wait for it to finish.

If you use a single mutex for readers and writers to share, you are blocking simultaneous reads.

A variation on this theme is to keep a reader counter.
The first reader grabs the resource mutex and the last reader releases it.
To implement this, you need a another mutex, the readcount mutex, to guard the read counter and resource mutex.
While a reader thread is blocked waiting on the resource mutex while the writer writes, any other reader threads will block on the readcount mutex until the first reader thread is released.
This can starve writers.

To give writers preference, add another mutex.
The writer takes this before trying the resource mutex.
Each reader is required to take this mutex before checking the readcount mutex on the way in.
This lets existing readers finish, but blocks new ones.
This can starve readers.

There is a solution that is fair to both.

## Quickselect

The purpose of this is an algorithm is to find the nth smallest element in an unordered array.
It is similar to quicksort in that it shares the partition algorithm.
Iteratively partition the array and examine the pivot point p returned.
If p == k, then return the value at k.
If p < k, repeat the partition starting at p+1.
If p > k, repeat the partition ending at p-1.

Worst case is O(n²).
Average is O(n).

See also std::nth\_element() for the same functionality.
This typically uses introselect, which is similar, but has better worst-case runtime by switching algorithms if one is taking too long.

See `~/src/find-mean` for an implementation.
