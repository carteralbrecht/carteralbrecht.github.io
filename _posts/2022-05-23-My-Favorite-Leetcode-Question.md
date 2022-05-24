---
layout: single
title:  "My Favorite LeetCode Question"
type: posts
permalink: /posts/median-of-data-stream
excerpt: "Finding the median of a data stream"
tags: interview leetcode
classes: wide
---
Data structures and Algorithm interviews can always be hit-or-miss. Sometimes the questions are very intuitive, and sometimes they seem impossible unless you have seen the problem before. To me, a good DS&A interview question is one that strikes a perfect balance between knowledge of data structures and clever thinking.

This is why my favorite question on LeetCode is [#295 Find Median from Data Stream](https://leetcode.com/problems/find-median-from-data-stream/)

## Problem statement
Implement a class that supports the following operations:
- `void addNum(int num)` ingest an integer from the data stream
- `int findMedian()` return the median of all the integers seen so far

Note that the stream of integers is unordered.

## Initial Thoughts
When I approach a new problem, I first like to run through a few examples.

If we have processed `[4,2,5]`, the median of this is `4` because these integers in sorted order is `[2,4,5]` and the middle element is `4`.

If we have processed `[8,2]`, the median of this is `5` because these integers in sorted order is `[2,8]`, and because this list is even, we take `(2 + 8) / 2 = 5` 

After looking at these examples, it is clear that we need to evaluate the integers seen so far in sorted order, and then find the middle element.

## Approach 1

One idea would be to maintain a list of integers we have seen so far, and then sort it whenever we need to find the median. 
- When `addNum(int num)` is called, we can append `num` to the end of our list.
- When `findMedian()` is called, we can sort the list and then return the middle.

Using this approach, `addNum(int num)` can be done in `O(1)` time because we are just appending to the end of the list. `findMedian()` will take `O(n log n)` because we have to sort the list.

Time complexity
- `addNum(int num)` is `O(1)`
- `findMedian()` is `O(n log n)`

## Approach 2

Sorting the list each time `findMedian()` is called is killing our runtime. What if we kept our list sorted and each time `addNum(int num)` was called, we inserted it into its sorted position? Then we could easily implement `findMedian()` in `O(1)` time.

Inserting into the sorted list will take us `O(n)` time.
- `O(log n)` to find insertion position
- `O(n)` to shift subsequent elements

Time complexity
- `addNum(int num)` is `O(n)`
- `findMedian()` is `O(1)`

## Approach 3 (Optimal)

In approach 2, we maintain the sorted order of the entire list. However, we don't really need to know the order of the whole list.

Looking at an example, `[1, 2, 3, 4, 5]`, we can see that the answer is `3`, because it is in the middle. We can look at this as `[1, 2]` `[3]` `[4,5]`. We have everything to the left of the middle in one list, and everything to the right of the middle in another list.

The solution here is to maintain two heaps. A max heap that stores everything to the left of the middle. And a min heap that stores everything to the right of the middle. We will keep the size of each equal, and alternate adding to them. We can find the median by looking at the root of the max heap and the root of the min heap.

{% highlight java %}
// our max heap holds everything to the left of the middle
private PriorityQueue<Integer> leftSide = new PriorityQueue<>(Collections.reverseOrder());
// our min heap holds everything to the right of the middle
private PriorityQueue<Integer> rightSide = new PriorityQueue<>();
// we will use a toggle to keep track of which heap to add to next
private boolean sizeEven = true;

public double findMedian() {
    // if we have as many things to the left as we do to the right
    // then we need to average the two inner elements
    if (sizeEven)
        return (leftSide.peek() + rightSide.peek()) / 2.0;
    // This means that the left side must have one more element
    // than the right side, so we should return the root of the maxheap.
    else
        return leftSide.peek();
}

public void addNum(int num) {
    // When the size of the two heaps are the same, we will first add num to the right side. 
    // Then we can take the smallest element from the right and add it to the left.
    if (sizeEven) {
        rightSide.offer(num);
        leftSide.offer(rightSide.poll());
    // When the size of the two heaps are not the same, we will first add num to the left side. 
    // Then we can take the largest element from the left and add it to the right.
    } else {
        leftSide.offer(num);
        rightSide.offer(leftSide.poll());
    }
    // toggle
    sizeEven = !sizeEven;
}
{% endhighlight %}

Time complexity
- `addNum(int num)` is `O(log n)`
- `findMedian()` is `O(1)`