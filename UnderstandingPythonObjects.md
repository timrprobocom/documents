# Understanding Python Objects

## Introduction

One of the key concepts that every Python beginner needs to understand is the distinction between names and objects.  Python does things a bit differently than other languages.  The rules are very consistent, but they aren't always obvious.  This document is an attempt to describe the "mental model" that I use for keeping these concepts straight.

I assume a basic understanding of Python here.

## Object Cloud

A Python program consists of names that are bound to objects.  In my mental model, I picture my Python space has having a big cloud full of anonymous, unnamed objects.  Separate from that, my program has a set of names that are bound to those unnamed objects.  You can also use terms like "refer to" or "point to".  It's important to keep this distinction between names and objects in your brain.

Consider this code:

```
a = [1,2,3]
```

That creates an anonymous, unnamed list object containing three elements.  It also creates a name in my global namespace, and binds that name to that three-element list.  It is tempting to think "a contains a three-element list", but that "containment" concept leads to confusion.

Now, consider this code:

```
b = a
c = a
```

I now that three names in my global namespace, `a`, `b`, and `c`.  However, I still only have one three-element list in my anonymous object cloud.  All three of those names are bound to the same list object.  This can be an enormous source of confusion to Python beginners.  If I do this:

```
>>> b[1] = 7
>>> print(a,b,c)
[1, 7, 3] [1, 7, 3] [1, 7, 3]
```

Note that all three show the same modification.  That's because there is only one list here, bound to three different names.  If you truly need to have a separate copy of a mutable object, you have to do that specifically.  Each type has its own way of doing that:

```
>>> a = [1,2,3]
>>> b = a[:]
>>> b[1] = 7
>>> print(a,b)
[1, 2, 3] [1, 7, 3]
```

In the second line, we have used what is called "slice notation" to extract a section of the list into a separate list.  In this case, the section happens to be the entire list.  Thus, in this example, we have two three-element lists in our object cloud, and two names in our namespace.

The same comments apply to dictionaries:

```
>>> a = {1:2, 3:4}
>>> b = a
>>> b[3] = 7
>>> a,b
({1: 2, 3: 7}, {1: 2, 3: 7})

>>> c = b.copy()
>>> c[1] = 9
>>> b,c
({1: 2, 3: 7}, {1: 9, 3: 7})
```

In the first example, both names are bound to a single dictionary.  Changing either one changes all the names bound to that dictionary.  In the second example, we use the `copy` method of dictionaries to make a copy.  There are now two dictionaries in the object cloud.  `a` and `b` are bound to the first, `c` is bound to the second.

Even experienced programmers get fooled by this.  You probably know of the clever shortcut to create a list containing a number of identical items:

```
lst = [0] * 10
```

That creates a list containing 10 zeros.  Very convenient.  Now, consider the following reasonable attempt to create a list of lists, essentially a two-dimensional array

```
>>> twod = [[]]*10
>>> twod
[[], [], [], [], [], [], [], [], [], []]
```

What's not obvious here is that our object cloud does not contain 11 separate lists, as you might expect.  Instead, there are two lists: one empty list, and another list containing 10 items that are bound to that one empty list.  Look what happens when we try to modify one of them:

```
>>> twod[3].append(1)
>>> twod
[[1], [1], [1], [1], [1], [1], [1], [1], [1], [1]]
>>>
```

This is not an easy bug to spot.  Even more interesting, consider this code:

```
>>> lst = []
>>> lst.append(lst)
>>> lst
[[...]]
```

Our object space only contains one object here.  That object is a one-element list, whose first element is bound to itself.  This is an infinite loop, but Python is smart enough to catch it.  That's why it displays the ellipsis (`...`) instead of printing infinitely.

We can even modify one of those lists, which bizarrely modifies ALL of the lists:

```
>>> lst[0][0][0].append(3)
>>> lst
[[...], 3]
>>>
```



## Mutability

There are two very general categories of objects in Python: those that can be changed, and those that cannot.  The built-in types that can be changed are `list`, `dict`, and `set`.  These are called "mutable".  The types that cannot be changed are `int`, `float`, `string`, `bytes`, and `tuple`.  There are called "immutable".

This can be surprising to people.  After all, I can write:

```
s = "One"
s = "Two"
```

which certainly seems to be changing a string.  However, that's not what it's doing.  The anonymous string object "One" has not been changed.  Instead, the name `s` is simply being bound to a brand new string object.  Compare this, for example, to a list operation:

```
lst = [1,2,3]
lst[1] = 7
```

In this case, `lst` is bound to the same object throughout, and the second statement actually modifies that list object.  If you try the same operation on a tuple, you'll find that it fails:

```
lst = (1,2,3)
lst[1] = 7
```

"But what about replace?" you might say.  `string.replace` does not change the string object.  Instead, it returns a brand new string, created from the old with the replacements:

```
>>> s = "Two"
>>> s.replace('w','h')
Tho
```

After this code, "s" remains unchanged.  If you were instead to say:

```
>>> s = s.replace('w','h')
```

Now `s` is bound to the new string, "Tho".  The old string ("Two") no longer has any names bound to it, so Python will mark it for deletion.