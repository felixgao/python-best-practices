!!! Summary

    :white_check_mark: (Almost) Always use dictionaries
    
    :white_check_mark: Use a unique index
 
    :x: Avoid code heavily oriented around pandas (especially indexing)


# Lookup Tables

lookup tables or cache is what computer scientist calls a space-time trade off.  What does that mean is you are using additional memory (RAM) to get faster access to data.  

## Why dictionary is faster than list or tuple?

When lookup for items, dictionaries have constant time complexity, O(1) while lists have linear time complexity, O(n).

However, dictionaries have much bigger space complexity compare to lists eventhough both have O(n) space complexity in terms of big-O. 


### I really want to use list

before you do that, you should be aware of the cost of the methods.

| Method                  |  List    |  Dict      |
| ----------------------- | -------- | ---------- |
| `INSERT` (Head or Tail) | O(1)     |  O(1)/O(N) |
| `INSERT` (Random)       | O(N)     |  O(1)/O(N) |
| `DELETE` (Head or Tail) | O(1)     |  O(1)/O(N) |
| `DELETE` (Random)       | O(N)     |  O(1)/O(N) |
| `GET`                   | O(1)     |  O(1)/O(N) |
| `Search` (Sorted)       | O(log N) |  O(N)/O(N) |
| `Search` (Random)       | O(N)     |  O(N)/O(N) |
| `COPY`                  | O(N)     |  O(N)/O(N) |

the amortized worst case scenario for dict is O(N). So becareful just because you have a dict, doesn't mean your get O(1) runtime behavior. 

when you do 
```python

    # This is `Search` O(N)
    if x in lst: 
        print(x)

    # To get O(log N) in search for lst you can do.
    # assume lst is sorted
    index = bisect_left(lst,x)    
    if index != len(l) and x == lst[index]:
        print(x)
```

## My key is not hashable

If you just want check for existence of object you can do one of the two things here

```python
    list_of_names = [...] # this is a really large list

    # if you only want to check for existince a few times
    names_existence_lookup = set(list_of_names)
    for candidate in list_of_candidates:
        if candidate in names_existence_lookup:
            print(f"{candidate} on the short-list")

    # can also create a dict with fixed value with the sentient that is special in python
    names_existence_lookup = { name:None for name in list_of_names}
    for candidate in list_of_candidates:
        if candidate in names_existence_lookup:
            print(f"{candidate} on the short-list")
```

In python `list`, `set`, and `dict` are not hashable objects.  The reason these can't be hashed is because the content can change therefore, it is important to convert these into an immutable datatype before use for hashable objects. 
