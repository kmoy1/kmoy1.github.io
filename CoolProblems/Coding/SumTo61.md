# Sum To 61(A)

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
    if _______________________________:
        return True
    elif _____________________________:
        return False
    with_sum = _________________________ and helper(________________, __________________)
    with_mul = _________________________ and helper(________________, __________________)
    return with_sum or with_mul
return _____________________________
```

TEST

{::options parse_block_html="true" /}

<details><summary markdown="span">View Solution</summary>
```python
print('Hello World!')
```
Of course, it has to be Hello World, right?
</details>
<br/>

{::options parse_block_html="false" /}

## Challenge: Combine_To_61_Unleashed

Now that you've handled that, can you do the same thing, except now consider *all* valid subsets of a list, without the contiguous restriction? 
