# Sum To 61(A)

I developed this problem back in Fall 2019 as a part of Berkeley CSM, and it's one of my proudest creations. 

Let's take a simplified version of the maximum subset sum problem. 

**Mission: Implement combine_to_61, where we want to return true a subset of a given list sums to 61.** 

For those who aren't familiar with the term, a subset of some list X is defined as a list whose elements are all in X. So if I have `X = [1,2,3,4,5]`, `[1,3,5]` is a valid subset of X because all 3 of its elements are in X. Remember that X itself, and the empty list `[]`, are both subsets of X!

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
