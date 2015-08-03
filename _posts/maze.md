---
layout: post
title: Choosing a Data Structure
---

A few years ago my friend Arnold told me that whenever he learns a new programming language, he writes code to solve one specific problem.
That problem can be stated as follows:
> Write a program to generate and print a maze, where each room is connected to every other without cycles.
This is a nice size problem for what Arnold does with it.
As it turns out, there are a few simple ways to extend it to explore some interesting programming topics.
To start with, in this article I am going to discuss choosing a data structure to represent a maze as needed for the basic problem as stated.

## Requirements and Vocabulary

Mazes can be made in a variety of shapes and styles.
For this exercise, we will only be discussing two dimensional, rectangular, simply-connected mazes of arbitrary size.
There will be no closed-off areas and no passage cycles.
We will call the rooms of the maze "cells".
We will label the cells starting from 0 and increasing from left to right across the first row, then continuing with the next row.
Two adjacent cells may or may not have a wall between them.
If there is no wall, then we say that there is a passage between them.
Start and end points are not relevant to the creation of the maze layout and can be any two cells chosen later.
We use "north" to refer a cell "above" another cell, "east" to refer to a cell to the right, "south" for down, and "west" for left.

## Uses for the Data Structure

When choosing a data structure it can be useful to list the things you expect it to do.
For now we want to:
* Generate a valid maze.
* Print the layout of the maze to the console.

Later we will want to:
* Solve the maze.

## A Graph

The type of maze we are discussing may be represented as a graph.
The cells are nodes and the edges between cells are the edges.
Any cell may be chosen as the root and the nodes can be labeled with the cell number.

TODO: insert a diagram of maze to graph.

Does this help us?
Let's see.
Graphs are typically represented using an adjacency list or an adjacency matrix.

### Adjacency List

In an adjacency list, each node carries a list of those nodes it is connected to.
So the main part of our data structure would be a vector of nodes, where each node contains a vector of neighbors.
The vector or neighbors will be small: a node can be connected to between one and three others.
Determining if two nodes are connected is indexing the vector of nodes to find the first node, and then searching its small vector of neighbors for the second.
Finding a node's neighbors is indexing the vector of nodes to find the node then walking the small vector of neighbors.
This is not too complicated.

If two nodes are connected, each will point to the other.
While redundant in that way, this is still a relatively efficient data structure.
It could work, but let's keep exploring.

### Adjacency Matrix

For a maze with n number of nodes, picture a matrix labeled 0-n across the top and 0-n down the left size.
For each position x,y in the matrix, the table will contain a 1 if x and y are connected or a 0 if x and y are not connected.

TODO: insert a diagram.

Determining if two nodes are connected is looking in at a single position in the matrix to see if it is a 1.
Finding a node's neighbors is walking that node's row looking for ones.
Again, this is not too complicated.

The matrix holds zeros and ones, so we only need one bit for each position.
However, we need n² bits.
This will be a very sparse matrix because each node can only have four neighbors.
This is very inefficient use of space.
