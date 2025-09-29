Name: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_
## Part 1: choose correct answers
1. What is the result of `[1, 2] + [3] * 2`?
	**a**. `[1, 2, 3, 3]`
	b. `[1, 2, 6]`
	c. `[1, 2, 3, 6]`
	d. Error
2. What will this print?
***
```python
x = 10
def func():
	global x
	x = 5

func()
print(x)
```
***
	a. 10
	**b**. 5
	c. SyntaxError
	d. Nothing
3. Which expressions will result to `True`?
	a. `"5" == 5`
	**b**. `0.1 + 0.9 == 1`
	c. `[] != []`
	**d**. `bool("False") == True`
4. How many iterations will `range(2, 10, 2)` give?
	a. 5
	b. 8
	**c**. 4
	d. 0
5. What will be printed after running this code?
***
```python
a = [1, 2, 3, 1, 2, 4]
s = set(a)
s |= {4, 5}
s.discard(5)
print(s & {3, 7, 8})
```
***
	a. `{1, 2, 3, 4, 5, 7, 8}`
	b. `{1, 2, 3, 4}`
	**c**. `{3}`
	d. `{1, 2, 3, 4, 7, 8}`
## Part 2: write down your answer
6. How to stop the loop with keyword? How to skip an iteration?


7. What is the difference between `list` and `set`? Write a few methods that differ between them (at least two for each).


8. What is included into functions signature?


9. How to accept many non-keyword arguments into a function? Give a short example.


10. When does `finally` block work out in `try / except`? Give a short example.
## Part 3: write a code
11. Write a function called `group_by_year` that groups dates by year.
    Dates are being passed in format `DAY.MONTH.YEAR` one-by-one through console until `STOP` word passed.
    If passed dates format incorrect your program have to raise `ValueError`.
12. Write a function that changes diagonals in a square matrix horizontally.
    For example:
```python
[
	[1, 2, 3],  # swap 1 and 3
	[4, 5, 6],
	[7, 8, 9]   # swap 7 and 9
] =>
[
	[3, 2, 1],
	[6, 5, 4],
	[9, 8, 7]
]
```
13. Write a program that finds files with the same name inside of file system.
    File names are passed as a list with possible list inside it. As a result return not unique names across file system.
    For example:
```python
["file.txt", "other.txt", ["new.txt", ["file.txt"]]]
# result has to be ["file.txt"] (as a list or set)
```