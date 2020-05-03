# Combine To 61(A)

I developed this problem back in Fall 2019 as a part of Berkeley CSM for CS61A, and it's one of my proudest creations. 

Let's take a simplified version of the maximum subset sum problem. 

**Mission: Implement combine_to_61, where we want to return true a contiguous subset of a given list can be combined to 61.** This means that for a given subset, some combination of multiplication and addition operations to those numbers results in 61.

For those who aren't familiar with the term, a subset of some list X is defined as a list whose elements are all in X. *Contiguous* just means that the numbers are next to each other in the original list (X). So if I have `X = [1,2,3,4,5]`, `[1,2,3]` is a valid contiguous subset of X because all 3 of its elements are in X, and 1, 2, and 3 were all next to each other. For another example, while `[1,3,5]` is a valid subset, it is *not* contiguous and thus not a valid subset to be considered in this problem. Remember that X itself, and the empty list `[]`, are both subsets of X!

```python
def combine_to_61(lst):
    """
    >>> combine_to_61([3, 4, 5])
    False # no combination will produce 61
    >>> combine_to_61([2, 6, 10, 1, 3])
    True # 61 = 6 * 10 + 1
    >>> combine_to_61([2, 6, 3, 10, 1])
    False # elements must be contiguous
    """
    def helper(lst, num_so_far):
        if ________:
            return True
        elif ________:
            return False
        with_sum = _____________ and helper(________, ________)
        with_mul = _____________ and helper(________, ________)
        return with_sum or with_mul
    return _____________________________
```

TEST

{::options parse_block_html="true" /}

<details><summary markdown="span">View Solution</summary>
```python
def combine_to_61(lst):
    def helper(lst, num_so_far):
        if num_so_far == 61:
            return True
        elif not lst:
            return False
        with_sum = num_so_far + lst[0] <= 61 and helper(lst[1:], num_so_far + lst[0])
        with_mul = num_so_far * lst[0] <= 61 and helper(lst[1:], num_so_far * lst[0])
        return with_sum or with_mul
    return helper(lst, 0)
```
</details>

<details><summary markdown="span">View Explanation</summary>
The first thing we notice from our skeleton code is that we take a helper function that takes an extra parameter `num_so_far`. In general, added parameters such as this are meant as a tracker of progress somehow. This should immediately give us a clue that `num_so_far` will track our total sum-product recursively.
    
Given that, our first base case becomes easy: If our tracker has a value of 61, we know we've found some product/sum of elements in `lst` that results in value 61. 
 
Our second base case will probably consider the other parameter list: in general, try to consider base cases for each parameter. So we ask ourselves, "When do we know immediately whether or not our list can combine to 61 or not?" Well, on the simplest possible input, an empty list, of course! There's no possible way you can do anything sum or product with an empty list. 
 
With our base cases done, now comes the fun part of the problem: Implementing the sum/product algorithm. We see in our skeleton code that our helper function returns `with_sum or with_mul`. This indicates that `with_sum` and `with_mul` should be one-liner booleans! With that knowledge, what do we do? Well, in recursive problems such as this, it's incredibly to common perform an action with one element and recurse on the rest. So to test summing one element from lst into our sum-product cumulative total, we can call `helper(lst[1:], num_so_far + lst[0])`, and to test multiplying that element, we call `helper(lst[1:], num_so_far * lst[0])`. 

We can prune our recursive tree a bit with our contiguousness requirement: if the operation of summing OR multiplying by the first element of `lst` results in a sum-product greater than 61, we know we'll have overshot it and that subset won't work.

</details>

{::options parse_block_html="false" /}


## Challenge: Combine_To_61_Unleashed

Now that you've handled that, can you do the same thing, except now consider *all* valid subsets of a list, without the contiguous restriction? 
