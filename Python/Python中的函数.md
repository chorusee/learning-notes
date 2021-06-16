## Python中的函数

### 把函数作为对象

Python中的一切皆对象，下面的代码说明了Python函数是对象，factorial是function类的实例，在内存中为其分配了一块空间。

```python
>>> def factorial(n):
...     return 1 if n < 2 else n * factorial(n-1)
...
>>> type(factorial)
<class 'function'>
>>> id(factorial)
1736201726992
```

在Python中，函数是一等对象。“一等对象”是满足下述条件的实体：

+ 在运行时创建
+ 能赋值给变量或数据结构中的元素
+ 能作为参数传给函数
+ 能作为函数的返回结果

下面的代码展示了函数对象的“一等”本性。可以将factorial函数赋值给fact变量，然后通过变量名调用。还能把它作为参数传递给map函数。

```python
fact = factorial
>>> fact
<function factorial at 0x000001943DAFEC10>
>>> fact(5)
120
>>> map(factorial, range(11))
<map object at 0x000001943DA821C0>
>>> list(map(factorial, range(11)))
[1, 1, 2, 6, 24, 120, 720, 5040, 40320, 362880, 3628800]
```

### 函数的参数

+ 位置参数

  ```python
  >>> def fun(arg1, arg2):
  ...     print(f'arg1: {arg1}, arg2: {arg2}')
  ...
  >>> fun(1, 2)
  arg1: 1, arg2: 2
  ```

+ 默认值参数

  ```python
  >>> def fun(arg1, arg2=2):
  ...     print(f'arg1: {arg1}, arg2: {arg2}')
  ...
  >>> fun(1)
  arg1: 1, arg2: 2
  ```

  调用时如果arg2不传入实参，就赋“=”后的值。

+ 关键字参数

  关键字参数形式为 kwargs=value 。

  ```python
  >>> def fun(arg1, arg2):
  ...     print(f'arg1: {arg1}, arg2: {arg2}')
  ...
  >>> fun(arg1=1, arg2=2)
  arg1: 1, arg2: 2
  ```

+ 特殊参数

  函数定义如下：

  ```python
  def fun(pos1, pos2, /, pos_or_kwd, *, kwd1, kwd2):
  ```

  正斜杠（/）前的仅为位置参数，正斜杠到星号（*）之间的可为位置参数也可为关键字参数，星号后的仅为关键字参数。正斜杠与星号是可选的。

  1. 位置或关键字参数

     函数定义未使用 / 和 * 时，参数可以按位置或关键字传递。

  2. 仅位置参数

     仅限位置时，形参的顺序很重要，且这些形参不能使用关键字传递。仅限位置形参应放在 / 前。/ 用于在逻辑上分割仅限位置形参与其他形参。如果函数定义中没有 / ，则表示没有仅限位置形参。

  3. 仅限关键字参数

     把形参标记为 *仅限关键字*，表明必须以关键字参数形式传递该形参，应在参数列表中第一个 *仅限关键字* 形参前添加 `*`。

+ 可变参数（不定长参数）

  ```python
  >>> def print_multiple_items(separator, *args):
  ...     print(separator.join(args))
  ...
  >>> print_multiple_items(',', 'hello', 'world')
  hello,world
  ```

  *args 表示可接收不定长的参数，调用时 ',' 传递给了separator，后面两个参数传递给了 *args，\*args 形参后的任何形式参数只能是仅限关键字参数，即只能用作关键字参数，不能用作位置参数。

  当有一个列表（元组），要把这个列表（元组）作为参数传递时，要用 * 拆包：

  ```python
  >>> a = [1, 2, 3]
  >>> print_multiple_items(*a)
  (1, 2, 3)
  ```

  若直接将a传入，会将这整个列表作为一个参数：

  ```python
  >>> print_multiple_items(a)
  ([1, 2, 3],)
  ```

  

+ 不定长关键字参数

  ```python
  >>> def print_kwargs(**kwargs):
  ...     print(kwargs)
  ...
  >>> print_kwargs(one=1, two=2)
  {'one': 1, 'two': 2}
  ```

  将一个字典作为参数传入需要用 ** 拆包：

  ```python
  >>> d = {'one': 1, 'two': 2}
  >>> print_kwargs(**d)
  {'one': 1, 'two': 2}
  ```

  直接将字典对象传入会产生异常：

  ```python
  >>> print_kwargs(d)
  Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
  TypeError: print_kwargs() takes 0 positional arguments but 1 was given
  ```

+ 多种参数类型组合要以位置参数、

拆包

### 高阶函数



