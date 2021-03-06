
# Sorting
> Given an array $A$ of numbers, find a permutation of it so that the array's elements are monotonic in its indices.

## Discussion
Sorting is a classic algorithms problem that has been extensively explored by various authors some of whose approach to the problem is truly novel and path breaking.

Without digressing too much from the primary purpose of this essay (more of a personal reminder about concepts for interview prep)

There are various [sorting algorithms](https://en.wikipedia.org/wiki/Sorting_algorithm) each with its own niche use case. However, to keep the discussion short and on point we'll be discussing the following sorting algorithms:

1. Insertion sort
2. Merge sort
3. Quicksort
4. Heap sort

Each of these has their own special property that interests me leading to inclusion here. By default we will be talking about sorting in ascending order since deriving the opposite is usually a simple change of ordering comparisons.

### Small aside on selection sort
There are other simple algorithms like selection sort whose primary focus is that after the $i^{th}$ iteration $A[:i]$ will never change. i.e. it already contains sorted elements in the correct order.

This makes it that selection sort is what is called a *partial sorting* algorithm. i.e. a sorting algorithm that can be stopped midway to get a part of the correctly sorted answer which can then be extracted and processed outside. A simple use case for partial sorting algorithms would be sorting extremely large lists using a buffer and merge procedure. Or getting the k-th order statistics of an array. 

Selection sort is obviously extremely inefficient, so I've only written this because I tend to forget the difference between selection and insertion sort after years of studying CS, because they're both algorithms that rely heavily on swap and the initial subarray $A[:i]$ is sorted for both. Just that insertion sort is stricter in specification saying that $A[:i]$ must be sorted and never change after the $i^{th}$ iteration.

## Algorithms --- Insertion Sort
### Properties
With insertion sort the key idea is to remember how it operates:

1. It starts at the initial index
2. Each iteration expands its scope of processing by 1
3. At the start of $i^{th}$ iteration $A[0:i-1]$ are in sorted order
4. After the $i^{th}$ iteration $A[i]$ is **inserted** at some $k$ in $[0:i-1]$, causing elements $[k:i-1]$ to be shifted rightwards

Thus the name insertion sort. Each iteration it is trying to *process a single element by trying to insert it into aan already sorted array*.

Why is Insertion sort special? Read the [wiki page](https://en.wikipedia.org/wiki/Insertion_sort) to see it has some interesting properties like being stable, online, in-place. One more important property is that it will make fewer modifications if the list is almost sorted or sorted.

It is also more efficient than most other $\Theta(n^2)$ algorithms so its useful to have it as a bottom layer sort for small arrays in a hybrid approach where large arrays are processed by efficient algorithms and smaller segments are sorted by insertion sort which is better in this regime of array sized.

>Another useful example to remember about this process is how a deck of cards may be sorted by hand. You start with the top card and keep picking up single cards from the top of deck and inserted them into their correct places in the "sorted deck". The sorted deck has elements in the right relative order but not all of them are in their final places. 

### Code --- Insertion Sort
```cpp
class Solution 
{
    public:
    void insertionSort(vector<int>& nums, int p, int r)
    {
        for(int i = p + 1; i <= r; ++i)
        {
            // Current processed element
            int x = nums[i];
            int j = i - 1;
            
            // Move elements left of x
            // rightwards to make space for x
            for(; nums[j] > x && j >= 0; --j)
            {
                std::swap(nums[j], nums[j + 1]);
            }
            
            // j now points left of insertion index
            nums[j + 1] = x;
        }
    }
 }
```
## Algorithms --- Merge Sort
Merge sort is a divide and conquer algorithm for sorting a large array. Well, how is it divide and conquer? 

The key objective is to sort smaller and smaller segments of the array called *runs* and *merge* these together into a larger sorted array. Since it is possible to merge two sorted arrays into one in linear time if variable extra space is allowed we have a sorting algorithm performing at $\Theta(n \log n)$ time complexity.

### Properties
Merge sort is also stable, although it is not in-place (since we require an array of size $\Theta(n)$ for temporary storage). It generally outperforms quicksort, performing fewer comparisons but requires more memory. Read more [here](https://en.wikipedia.org/wiki/Merge_sort#Analysis).

### Code --- Merge Sort

```cpp
#include <climits>

using namespace std;

class Solution 
{
    public:
    void mergeSort(vector<int>& nums, int p, int r)
    {
        if(p < r)
        {
            // Pivot index
            int q = (p + r) / 2;
            // Left half
            mergeSort(nums, p, q);
            // Right half
            mergeSort(nums, q + 1, r);
            // merge sorted arrays
            merge(nums, p, q, r);
        }
    }
    
    // Merges two sorted arrays
    private:
    void merge(vector<int>& nums, int p, int q, int r)
    {
        int lsize = q - p + 1;
        int rsize = r - q;
        int lt = 0;
        int rt = 0;
        
        // Allocate temp storage for merging
        vector<int> ltemp(lsize + 1);
        vector<int> rtemp(rsize + 1);
        
        for(int i = 0; i < lsize; ++i)
        {
            ltemp[i] = nums[i + p];
        }
        
        for(int i = 0; i < rsize; ++i)
        {
            rtemp[i] = nums[i + q + 1];
        }
        
        // Poor man's infinity
        // Sentinel value so that 
        // bounds checking is simplified for us
        ltemp[lsize] = INT_MAX;
        rtemp[rsize] = INT_MAX;
        
        for(int i = p; i <= r; ++i)
        {
            if(ltemp[lt] <= rtemp[rt])
            {
                // Ties broken towards left
                nums[i] = ltemp[lt];
                lt += 1;
            }
            else
            {
                nums[i] = rtemp[rt];
                rt += 1;
            }
        }
    }
};
```

## Algorithms --- QuickSort
The key to understanding how something such as quicksort came about relies on understanding how the partitioning routine works in the grand scheme of sorting.

Like merge sort this is also a divide and conquer algorithm, but unlike merge sort majority of the work is done at the divide step. The combine step is essentially *free* here. Quicksort repeatedly partitions an array in a special manner and combines these recursively partitioned segments into a sorted array.

### Properties
To get droll facts out of the way, quicksort is in-place, it is not stable. Performs better than merge sort practically due to having [spatial locality of reference](https://en.wikipedia.org/wiki/Locality_of_reference), utilizing caches better.

The partition routine's sole job is to select a pivot element $x$, any element and arrange the array so that all elements smaller than $x$ are to its left and all larger are to its right. Here's an image illustrating the process of pivoting an partitioning:

<img src="http://slideplayer.com/slide/4423219/14/images/11/Quick+Sort+Partitions.jpg" alt="Quicksort overview">

Some questions to ask yourself about the partitioning routine:

**Is partitioning $\equiv$ sorting?**
No, partitioning only ensures that after its done, the selected element is in its correct place (where it would be after we're done sorting the array)

**How are these two ideas related?**
Think of a sorted array $[1, 2, 3, 4, 5, 6]$. The usually definition of a sorted array is one that was mentioned at the beginning of this essay, here's another equivalent definition, which may help you connect the dots to the idea behind quick-sort, but first a definition (note that this can be defined the other way around too, but then we'd be dealing with descending sorts)

An element $a_k$ in a sequence A is said to be **ordered** if the following holds (for non repeating elements):
$$(a_i \lt a_k \implies i \lt k) \:\wedge(a_i \gt a_k \implies i \gt k)$$

Now a sequence is said to be sorted if *every element of the sequence is ordered*.

This is just what the partitioning function does. It picks an element and ensures that it is ordered. This is then done recursively over the array with a divide and conquer strategy because linearly partitioning from left to right would be $\Theta(n^2)$ and end up in a lot of repeated data moves. divide and conquer ensures that elements are not shuffled repeatedly, resulting in a performance gain.

Quicksort on average performs $\Theta(n \log n)$ with a worst case scenario of $\Theta(n^2)$. There are ways change partitioning (median of medians, randomized pivot selection) that ensure worst case of $\Theta(n \log n)$, but we won't be doing that here.

### Code --- QuickSort
```cpp
#include <climits>

using namespace std;

class Solution 
{
    public:
    void quickSort(vector<int>& nums, int p, int r)
    {
        if(p < r)
        {
            // Pivot index
            int q = partition(nums, p , r);
            // Left half, q - 1 because the element nums[q] 
            // is already ordered and sorted
            quickSort(nums, p, q);
            // Right half
            quickSort(nums, q + 1, r);
        }
    }
    
    // Partition a subarray of an array
    private:
    int partition(vector<int>& nums, int p, int r)
    {
        // Arbitarily select pivot to be middle element
        // This is the step that is changed to 
        // get randomized / median-of-medians quicksort
		int i = p - 1;
		int j = r + 1;
		int x = nums[p];
		
		// Once pivot element is selected, order the array
		// around pivot
		while(true)
		{
			// Search for first element larger than x to its left
			do
			{
			    i++;
			} while(nums[i] < x);
			
			// Search for first element smaller than x to its right
			do
			{
			    j--;
			} while(nums[j] > x);
			
			// Swap these two elements if the condition still holds
			// Else return the pivot index 
			// no violations implies the array was already partitioned
			if(i >= j)
			{
			    return j;
			}
			
			std::swap(nums[i], nums[j]);
		}
    }
};
```

## Algorithms --- Heap Sort

### Properties
Heap sort is a special sorting algorithm that has a lot of desirable properties including being our first efficient partial sorting algorithm so far. As a reminder, being able to partially sort means we do not need to run the whole algorithm to find out say the $5^{th}$ largest element. This is useful for performing order statistics , priority queue implementations on dynamic sequences. There are better techniques to do the same for static sequences (OS-Trees!), but those are for a later discussion.

Heap sort is similar to selection sort, in that we find the extreme (min) element first but move to the tail of the array. Another key difference is that we are smart about selection here, the input array is *heapified*, i.e. arranged in a special way so that finding the extrema is a $\Theta(1)$ operation.

A heap is an array that can be viewed as a nearly complete binary tree with each tree node having a corresponding array element. This below is the relationship of array indices wrt their relative positioning in the tree ($0$ indexing, with $0$ as root node)

$$parent(i) = \left\lfloor \frac{i - 1}{2} \right\rfloor$$

$$left(i) = 2i + 1$$

$$right(i) = 2i + 2$$

The kind of heap we are dealing with for creating a sorting algorithm is a max-heap (min-heap for descending sort). In addition to the above three, a max-heap also obeys:

$$A[parent(i)] \geq A[i]$$

So in order to sort using something like this, we need to do the following

1. Create a max heap out of an unordered array $\Theta(n)$
2. Extract maximum from the max heap $\Theta(1)$
3. Maintain the max-heap property $O(\log n)$

The total cost of this would be $cost_{1} + \#elements (cost_2 + cost_3)$. It turns out that this is $O(n \log n)$

`buildMaxHeap`
 We start at the middle of the array because of how the heap indexing works(roots to the left will have children on the right half), starting at n - 1 would end up processing elements twice and lead to incorrect heap creation.

`maxHeapify`: 
This function assumes that the parent nodes follow the heap property and trickles down the validation to child nodes. Each iteration assumes that the largest element is at the root(maxheap property) and checks against either child to see if this is violated. If a violation occurs, the offender is exchanged and we heapify that subtree only. One thing to note is that is assumes that there is **at most a single violation per iteration**. Multiple violations per iteration would mean that the $O(\log n)$ bound is no longer valid.

Handy table to remember for partial sorting vs complete sorting using heaps:

| Partial sort  	| Type of Heap  	| Full sort	type	|
|---					|---						|---	
| Ascending  	| Min heap 			| Descending 	|
| Descending  	| Max heap		 	| Ascending 		|


### Code --- Heap Sort
```cpp
#include <climits>

using namespace std;

class Solution 
{
    public:
    void heapSort(vector<int>& nums, int k)
    {
        int n = nums.size();
        
        buildMaxHeap(nums);
        
        // Emit largest values one by one.
        // Repeat n times with heapification
        for(int i = n - 1; i >= n - k; --i)
        {
            std::swap(nums[0], nums[i]);
            maxHeapify(nums, 0, i);
        }
    }
    
    private:
    void buildMaxHeap(vector<int>& nums)
    {
        int n = nums.size();
        
        // Create a max heap
        for(int i = n/2 - 1; i >= 0; --i)
        {
            maxHeapify(nums, i, n);
        }
    }
    
    private:
    void maxHeapify(vector<int>& nums, int i, int size)
    {
        int l = 2 * i + 1;
        int r = 2 * i + 2;
        int largest = i;
        
        if(l < size && nums[l] > nums[largest])
        {
            largest = l;
        }
        
        if(r < size && nums[r] > nums[largest])
        {
            largest = r;
        }
        
        if(largest != i)
        {
            std::swap(nums[i], nums[largest]);
            maxHeapify(nums, largest, size);
        }
    }
};
```

## References
1. CLRS chapter 6 - Heapsort
2. [GeeksforGeeks - Insertion Sort](http://www.geeksforgeeks.org/insertion-sort/)
3. [GeeksforGeeks - Merge Sort](http://www.geeksforgeeks.org/merge-sort/)
4. [GeeksforGeeks - Quick Sort](http://www.geeksforgeeks.org/quick-sort/)
5. [GeeksforGeeks - Quick Sort using Hoare and Lomuto Parition](http://www.geeksforgeeks.org/hoares-vs-lomuto-partition-scheme-quicksort/)
6. [GeeksforGeeks - Heap Sort](http://www.geeksforgeeks.org/heap-sort/)