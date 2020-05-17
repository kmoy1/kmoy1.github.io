# Sorting

Most traditional sorting algorithms (i.e. quick sort, insertion sort, etc.) rely having all elements in memory. However, for databases this is *nearly always* NOT the case: data we want sorted is now *massive*, far larger than even the largest memory space allows. 

## IOs: A Review

Recall an IO is our notion of "cost" when writing a page from memory to disk or reading a page from disk into memory. Because disk accesses are generally very expensive, they are usually dominant in algorithms, so we only measure algorithm cost by IOs. Therefore, we want our sorting algorithm to minimize incurred IOs. Additionally, assume buffer manager caching is ignored. Once we unpin a page (stating we finished using it) we always flush to disk: it will cost 1 IO to access that page again. 

## 2-Way External Merge Sort

Let's start with a *strawman algorithm*: one that isn't good but demonstrates the general idea. 

Because our memory is too small to fit all data, we will **sort different pieces of it separately and then merge it together.** (You might know this as MergeSort). 

### Conquer Phase

The merging phase here can only merge sorted (sub)lists. This tells us that our first step in sorting is sorting each page in memory. This is the **conquer phase**: we "conquer" pages by sorting their records. 

Then we merging the pages together, same way as in MergeSort. We are creating a bunch of **sorted runs,** which are just *sequences* of sorted pages. So if a page A holds 2 records [1,3] and page B holds 2 records [2,4], merging them would create the 2-page sorted run [1,2] [3,4].

Then we simply continue merging these sorted runs until one final sorted run is made from all pages in memory. This is our fully sorted data. 

### Analysis

How many IOs does this take? Each *pass* over the data will take $2N$ IOs, where $N$ is the number of data pages. A pass consists of reading all the necessary pages to memory AND writing every page to disk (after sorting it). We'll have to do this for every merge step as well as the single conquer step, to account for our memory limitations.

How many passes do we need to sort a table with $N$ pages fully, then? 1 comes from the conquer pass. For merging, each pass cuts the number of sorted runs in half. You can think of having $N$ sorted runs after the conquer pass, and the goal is to reduce it to 1 sorted run. So we need $\text{ceil}(\log_2(N))$ merge passes after conquer. So this requires $1+ \text{ceil}(\log_2(N))$ passes overall.

How many **buffer pages** do we need? Remember that memory consists of page-sized slot for data pages to be stored. The conquer pass sorts each page individually, so by just loading, sorting, and flushing on a single page, the conquer pass need only 1 buffer page.

What about the merge passes?  Merging only compares the first value for the two merging lists to find the minimum. So **only the first page of each sorted run will contain this minimum**, so we only need 2 buffer pages for each sorted run. When we merged all the records of a page, we just load the next page in the associated sorted run, and we know order will be retained (since its a sorted run). 

The **input buffer** is the buffer page that stores the front of each sorted run- we need 2. We'll naturally also need an **output buffer** to store our output: the page that stores the result of merging. Once the output buffer fills we flush it to disk and clear it for more.

So in all, this sorting algorithm only utilizes 3 pages. So if we have 100 buffer pages, we'll have 97 pages sitting on their ass for the entirety of the process. We can do better than that. Can we edit our algorithm to utilize *all* the buffer pages? 

## Full External Sort

We have $B$ buffer pages, and we want to use most, if not all, of them. 

First, let's optimize the conquer pass. Rather than just sorting individual pages, let’s load B pages and sort them all at once- **creating a sorted run of $B$ pages**. Now we are producing fewer and longer sorted runs in the conquer pass- which leads to fewer merge passes, and thus fewer IOs incurred!

Now let's optimize the merge passes. What if we could merge more than 2 sorted runs for each pass? If we had $B-1$ input buffers and one output buffer, **we could actually merge $B-1$ sorted runs each pass.** 

### Analysis

The conquering pass produces only $\text{ceil}(N/B)$ sorted runs now to merge. During merging, we divide the number of sorted runs by $B−1$ each pass (merge $B-1$ runs each pass).

Thus overall we take $1 + \lceil log_{B-1}(\lceil N/B \rceil) \rceil$ passes and $2N* (1 + \lceil log_{B-1}(\lceil N/B \rceil) \rceil)$ IOs. 

