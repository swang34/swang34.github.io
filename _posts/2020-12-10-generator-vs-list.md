---
layout: post
title:  "Python Generator vs List"
categories: blog
---

[*Reference：Python Tutorial: Generators - How to use them and the benefits you receive*][jekyll-docs1] 

**Note:** The contents and code snippets are referring to the tutorial from Corey Schafer above, with some modifications. 


# 什么是generator?

Python Generator是一个iterator object，可以用来很方便的loop through一个function。它不会返回一整个list，而是在用户需要的时候，返回一个一个的value。
因为不需要储存一整个list，它比loop using list更有效率, 可以节省时间和储存空间。

# 什么时候要用generator？
- 要loop的element很多，所以需要time and storage performance的时候。
- 不需要返回一整个list（比如求和这种可以accumulate的）的时候。

# 例子1: difference in output

首先，来看一下用list和generator去loop through a list，它们的output会有什么区别。

这是用list的code。我们所做的是对一个list中的每一个value求平方。
```python
def square_numbers(nums):
	res = []
	for i in nums:
		res.append(i*i)
	return res

my_nums = square_numbers([1,2,3,4,5])
print(my_nums)
```
返回的结果如下,这是一个list。
```python
[1,4,9,16,25]
```
下面换成generator的code，我们把return换成yield，来看看output长什么样子。
```python
def square_numbers(nums):
	for i in nums:
		yield(i*i)

my_nums = square_numbers([1,2,3,4,5])
print(my_nums)
```
这里可以看到print出来的是一个generator object, 而不是一个list。
```python
<generator object square_numbers at X8jXkW3k2Jg>
```
怎样access generator里的element呢？我们可以用next，每个next都会取generator里的下一个element，当然也可以iterate trough这个object。

```python
print(next(my_nums))
print(next(my_nums))
print(next(my_nums))

# We can also do [i for i in my_nums]
```
每个next会返回这个generator里的下一个值，直到这个generator结束。
```python
1
4
9
```

# 例子2: difference in performance 

上面，我们的list只有五个element，所以用list和generator不会有performance上的差别。当length很长的时候，generator的优点就会很明显了。我们用timeit来看看时间上两者的差别。

```python
import timeit 
import numpy as np


def square_numbers(nums):
	res = []
	for i in nums:
		res.append(i*i)
	return res

def square_numbers_generator(nums):
	for i in nums:
		yield(i*i)

nums = np.arange(10000)

def compare_functions():
	setup_code1 = """
from __main__ import square_numbers
from __main__ import nums
"""

	setup_code2 = """
from __main__ import square_numbers_generator
from __main__ import nums
"""

	stmt_code1 = "square_numbers(nums)"
	stmt_code2 = "square_numbers_generator(nums)"
    
	times1 = timeit.timeit(stmt=stmt_code1, setup=setup_code1, number=1000)
	times2 = timeit.timeit(stmt=stmt_code2, setup=setup_code2, number=1000)

	print("Time taken by list is {}".format(times1))
	print("Time taken by generator is {}".format(times2))

if __name__ == "__main__":
	compare_functions()

```
这里我们把nums变成从1到1000，用timeit去跑1000次。比起time.clock(), [timeit可以更加准确的测量function所需要的时间][jekyll-docs2]。output结果如下：
```python
Time taken by list is 2.5955459190299734
Time taken by generator is 0.0002614419790916145
```
可以看到generator只需要0.003s而list需要2.6s。我们也可以用memory_profiler来check一下，会发现generator所占用的空间也比list要小很多。
另外，因为genreator也是一个iterator，我们当然可以在外面加上一个list()去把它变成list，但是这样做就失去了generator在时间和空间上的优点。如果改一行上面的code，把
```python
	stmt_code2 = "square_numbers_generator(nums)"
```
改成
```python
	stmt_code2 = "list(square_numbers_generator(nums))"
```
我们会发现所需要的时间和list本身就很接近了。
```python
Time taken by list is 2.5068122040247545
Time taken by generator is 2.138504959992133
```

以上就是一个简单的generator introduction。Hope it's helpful。

[jekyll-docs1]: https://www.youtube.com/watch?v=bD05uGo_sVI
[jekyll-docs2]: https://stackoverflow.com/questions/17579357/time-time-vs-timeit-timeit
