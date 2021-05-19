# Understanding Python Escape Sequences

## Introduction

One of the things that often confuses new Python programmers is the difference between what strings contains, and how they are represented on the screen.  I'm going to try to go over some example that demonstrate how important this is, and how easy it is to get wrapped around the axle while thinking about it.

I'm going to focus on Python 3 here.  The situation in Python 2 is quite similar, but the terms are a bit different.

## Examples

Python strings can contain non-printable characters.  You're probably familiar with newline and tab, which we write in Python with backslashes, as \n and \t.  That sentence contains the first example of the difference between contents and representation.  The tab character, which has ASCII value 9, is represented in Python strings by the two-character sequence \t.  Consider the following terminal session:

```Python
>>> s = "ab\tcd"
>>> s
'ab\tcd'
>>> len(s)
5
>>> print(s)
ab      cd
>>> print(repr(s))
'ab\tcd'
>>>
```

We create a string variable on the first line, and bind it to a string constant.  Despite the fact that it took us 8 keystrokes to type that string, the string contains exactly 5 characters.  It is a common mistake to believe the string contains 6 characters, but it's not true.  That string does not contain the letter "t", nor does it contain a backslash.  We _write_ it that way, because otherwise it's hard to enter non-printable characters.  When we use "print", that prints the _contents_ of the string.  In this case, it prints "ab" and then a tab character (which my console handles by inserting spaces out to column 8), and then "cd".  If we just type the variable name, "s", Python shows us the *representation* of the string, just as we typed it.

The "\__repr__" function returns the string *representation* of an object.  In this case, we can see that it's the same thing we got by typing just the variable name itself.

Let's look at another example, one that came up recently on StackOverflow.

```Python
>>> x = bytes( random.randint(0,96) for _ in range(30) )
>>> x
b'\x124-*\x1e+\x0110Q\x0bM@\x1f\r\x10F\x102\n\x03)\nJG-\x1d3%\x1c'
>>> print(x.decode())
F2+10QM@
)
JG-3%
>>> print(len(x))
30
>>> print(len(repr(x)))
66
>>>
```

Here, I generate 30 random integers from 0 to 96, and store them as a **byte** string.  The Python representation of this value is a way we could type this into a command line to produce the same result.  Note that everything here is a printable character.  Even though the string is 30 characters long, the representation is 66 characters long.  When we print this value (after converting to Unicode), you can see three lines of mostly garbage.

It's instructive to look at the string representation.  Python tries to display real characters, when the character is printable.  Otherwise, it has to use escape codes.  The first character here is ASCII value 18 (ctrl-R), which is represented as ""\x12".  Again, it's important to note that this 30-character string does not contain any backslashes, nor does it contain the letter "x".  Python just shows it that way for our convenience.

The second character happens to be printable digit "4".  Why don't we see that in the console?  Well, the 15th character of our string happens to be a carriage return (ASCII 13, represented by "\r"), which my console handles by moving the cursor to the start of the line.  Thus, the next characters overwrite the "4".  The characters immediate after the carriage return are ctrl-P, "F", ctrl-P, "2", and a newline (\n), and that's what we see at the start of the first line.

