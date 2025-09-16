# CMPS 2200  Recitation 03

**Name (Team Member 1):** Viv Heurtevant




## Analyzing a recursive, parallel algorithm


You were recently hired by Netflix to work on their movie recommendation
algorithm. A key part of the algorithm works by comparing two users'
movie ratings to determine how similar the users are. For example, to
find users similar to Mary, we start by having Mary rank all her movies.
Then, for another user Joe, we look at Joe's rankings and count how
often his pairwise rankings disagree with Mary's:

|      | Beetlejuice | Batman | Jackie Brown | Mr. Mom | Multiplicity |
| ---- | ----------- | ------ | ------------ | ------- | ------------ |
| Mary | 1           | 2      | 3            | 4       | 5            |
| Joe  | 1           | **3**  | **4**        | **2**   | 5            |

Here, Joe (somehow) liked *Mr. Mom* more than *Batman* and *Jackie
Brown*, so the number of disagreements is 2:
(3 <->  2, 4 <-> 2). More formally, a
disagreement occurs for indices (i,j) when (j > i) and
(value[j] < value[i]).

When you arrived at Netflix, you were shocked (shocked!) to see that
they were using this O(n^2) algorithm to solve the problem:



``` python
def num_disagreements_slow(ranks):
    """
    Params:
      ranks...list of ints for a user's move rankings (e.g., Joe in the example above)
    Returns:
      number of pairwise disagreements
    """
    count = 0
    for i, vi in enumerate(ranks):
        for j, vj in enumerate(ranks[i:]):
            if vj < vi:
                count += 1
    return count
```

``` python 
>>> num_disagreements_slow([1,3,4,2,5])
2
```

Armed with your CMPS 2200 knowledge, you quickly threw together this
recursive algorithm that you claim is both more efficient and easier to
run on the giant parallel processing cluster Netflix has.

``` python
def num_disagreements_fast(ranks):
    # base cases
    if len(ranks) <= 1:
        return (0, ranks)
    elif len(ranks) == 2:
        if ranks[1] < ranks[0]:
            return (1, [ranks[1], ranks[0]])  # found a disagreement
        else:
            return (0, ranks)
    # recursion
    else:
        left_disagreements, left_ranks = num_disagreements_fast(ranks[:len(ranks)//2])
        right_disagreements, right_ranks = num_disagreements_fast(ranks[len(ranks)//2:])
        
        combined_disagreements, combined_ranks = combine(left_ranks, right_ranks)

        return (left_disagreements + right_disagreements + combined_disagreements,
                combined_ranks)

def combine(left_ranks, right_ranks):
    i = j = 0
    result = []
    n_disagreements = 0
    while i < len(left_ranks) and j < len(right_ranks):
        if right_ranks[j] < left_ranks[i]: 
            n_disagreements += len(left_ranks[i:])   # found some disagreements
            result.append(right_ranks[j])
            j += 1
        else:
            result.append(left_ranks[i])
            i += 1
    
    result.extend(left_ranks[i:])
    result.extend(right_ranks[j:])
    print('combine: input=(%s, %s) returns=(%s, %s)' % 
          (left_ranks, right_ranks, n_disagreements, result))
    return n_disagreements, result

```

```python
>>> num_disagreements_fast([1,3,4,2,5])
combine: input=([4], [2, 5]) returns=(1, [2, 4, 5])
combine: input=([1, 3], [2, 4, 5]) returns=(1, [1, 2, 3, 4, 5])
(2, [1, 2, 3, 4, 5])
```

As so often happens, your boss demands theoretical proof that this will
be faster than their existing algorithm. To do so, complete the
following:

a) Describe, in your own words, what the `combine` method is doing and
what it returns.
The combine method is similar to the merging step on merge sort. After the list has been split into its base cases, the combining step compares the left and right sides of the list indice by indice- if a value on the righthand list is larger than the value on the lefthand list, this implies a disagreement has been found. The minimum between the two is then appended to results, creating an ordered list. This combination continues until the entire list has been reconstructed in order.

b) Write the work recurrence formula for `num_disagreements_fast`. Please explain how do you have this.

The work formula will be W(n) = 2W(n/2) + O(n). Two recursive branches are made so W(n/2) is multiplied by two, and the /2 inside the operand is because the list is split in half each time. The merging cost is  O(n) as it iterates through  the minimum length of the left and right list, which are both proportional to n.

c) Solve this recurrence using any method you like. Please explain how do you have this.
Using the brick method, we can see the level of work is evenly distributed. For example, going to the first level of subproblems we see the cost for W(n/2) is O(n/2), and we have 2 W(n/2) subproblems which means the total cost per level is O(n). Therefore, we multiply the cost per level O(n) by maximum number of levels, which is log_2(n). so we have the work is O(nlogn).


d) Assuming that your recursive calls to `num_disagreements_fast` are
done in parallel, write the span recurrence for your algorithm. Please explain how do you have this.

For the span recurrence, we simply focus on the longest dependency chain so we ignore the recursive calls as these can be done in parallel. This makes our span recurrence S(n) = S(n/2) + O(n)

e) Solve this recurrence using any method you like. Please explain how do you have this.

Unlike the work recurrence, observe that the span recurrence will be leaf dominated as total cost per level decreases with each level, so we have a root dominated case. Therefore the cost will be the cost of the root, O(n).


f) If `ranks` is a list of size n, Netflix says it will give you
lg(n) processors to run your algorithm in parallel. What is the
upper bound on the runtime of this parallel implementation? (Hint: assume a Greedy
Scheduler). Please explain how do you have this.

From Brent's theorem, we know the time is bounded above by $\frac{W}{P} + s$. Let $W= O(nlogn), P = logn , S = O(n)$. This then simplifies to $O(n) + O(n)$, which is $\in O(n)$. So we know the upper bound on runtime is O(n). 

