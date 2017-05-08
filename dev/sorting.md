---
layout: page
title: Sorting
---

# Sorting

## Quicksort

Average O(n log n).
Worst case O(n²) if already sorted.
Recursive.

I like Lomuto partition.
Could also use Hoare or std::partition().

Choose a value pivotValue to use as a partition.
Choose the last value.
Partition around this value.

Initialize swapPoint to the start.
Move forward to value just before last.
If current <= pivotValue, swap it with the swapPoint and move swapPoint forward.
Swap the last value with swapPoint to get it to the pivot position.

Call recursively, excluding the pivot point.

## Radix Sort

This is an interesting sort.
It looks at the value in digit positions.
It can work left-to-right (MSD) or right-to-left (LSD).
Use LSD if you want number semantics and MSD if you want string semantics.
The complexity is O(wn) for n keys with w digits.
It has low complexity when the range of values to sort is small.
But it's still generally slower than comparison sorts in practice.

### LSD Radix Sort

This sort is a stable sort.

Each number is put into a bucket based on the value of the current digit, preserving original order.
Then empty each bucket in order into an empty original queue and repeat for the next digit.

See `~/src/radix-sort-lsd`.

## Heapsort

To sort, heapify, then `popHeap()` while walking the end towards the front.

Can use `std::make_heap()` and `std::pop_heap()` to do the work in a few lines of code.

## Counting Sort

This is good for when you have a limited number of values.
It is a stable sort.
O(k + n) where k is the number of keys and n is the length of the array.

Create a count array with one element per value that is used to count the number of values.
If there is no satellite data, you can just iterate through the array and output the index i count[i] times.
If there is satellite data, you need two more arrays: one to store the key in the original incoming order, and one to store the values in incoming order.

Then you need a step to adjust the count array so that instead of holding the exact count, it holds the position of the values of this key in the output array.
So count[0] is now 0 because we want to output these items to output[0].
Then count[1] becomes the old value of count[0].
And count[2] becomes the value of count[0] plus the old value of count[1].
So keep a running total.

Once the count array is in order, walk the original key array.
Look up count[key[i]]--that is the place in the output array for value[i].
Then increment count[key[i]] because then next time the same key is enountered, it goes in the next spot in the output array.

## Merge Sort

This is divide and conquer.
It is implemented recursively.
Split the array, sort each half recursively, then merge the two sorted halves.
Merge sort requires extra memory for the merge step.
It is average and worst case O(n lg n).
It is useful for sorting when the entire data set can't fit into memory.

