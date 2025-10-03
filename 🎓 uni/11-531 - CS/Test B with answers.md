Name: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_
## Part 1: choose correct answers
1. What is the result of `[1, 2] * 2 + [3]`?
	a. `[2, 4, 3]`
	**b**. `[1, 2, 1, 2, 3]`
	c. `[2, 4, 2, 4, 3]`
	d. Error
2. What will this print?
***
```python
x = 10
def func():
	x = 9
	def other_func():
		nonlocal x
		x = 5
	other_func()
	print(x)

func()
```
***
	a. 9
	**b**. 5
	c. SyntaxError
	d. 10
3. Which expressions will result to `True`?
	**a**. `float("5") == 5`
	b. `0.1 + 0.95 == 1`
	**c**. `[] == []`
	d. `bool("") == True`
4. How many iterations will `range(9, 4, -1)` give?
	**a**. 5
	b. 6
	c. Error
	d. 0
5. What will be printed after running this code?
***
```python
a = {"a": 1, "b": 2}
a |= {"b": 4}
a.update({"c": 3})
a.pop("a")
print(a)
```
***
	a. Error
	b. `{'b': 2, 'c': 3}`
	c. `{'a': 1, 'b': 2, 'c': 3}`
	**d**. `{'b': 4, 'c': 3}`
## Part 2: write down your answer
6. How to skip an iteration in loop? When does loop's `else` block will work out?


7. What is the difference between `list` and `tuple`? Write a few methods that differ between them (at least two for each).


8. Are the type annotations are necessary in function for a signature? What will be signature without it?


9. How to accept many keyword arguments into a function? Give a short example.


10. When does `else` block work out in `try / except`? Give a short example.
## Part 3: write a code
11. Write a function called `group_by_month` that groups dates by month.
    Dates are being passed in format `DAY.MONTH.YEAR` one-by-one through console until `STOP` word passed.
    If passed dates format incorrect your program have to raise `ValueError`.
12. Write a function that changes diagonals in a square matrix vertically.
    For example:
```python
[
	[1, 2, 3],  # swap 1 and 7
	[4, 5, 6],
	[7, 8, 9]   # swap 3 and 9
] =>
[
	[7, 2, 9],
	[4, 5, 6],
	[1, 8, 3]
]
```
13. Write a program that finds files with the same name inside of file system.
    File names are passed as a list with possible list inside it. As a result return not unique names across file system.
    For example:
```python
["file.txt", "other.txt", ["new.txt", ["file.txt"]]]
# result has to be ["file.txt"] (as a list or set)
```