# Optimizing Python Code Performance: A Deep Dive into Python Profilers 
![](https://www.kdnuggets.com/wp-content/uploads/mehreen_optimizing_python_code_performance_deep_dive_python_profilers_1.png)
  
Image by Author

Although Python is one of the most widely used programming languages, when it comes to large data sets it often suffers from poor execution times. Profiling is one of the methods to dynamically monitor the performance of your code and identify the pitfalls. These pitfalls may indicate the presence of bugs or poorly written code that is consuming a lot of system resources. Using Profilers will provide detailed statistics of your program that you can use to optimize your code for better performance. Let's take a look at some of the Python Profilers along with their examples.

cProfile is a built-in profiler in Python that traces every function call in your program. It provides detailed information about how frequently a function was called, and its average execution times. As it comes with the standard Python library so we do not need to install it explicitly. However, it is not suitable for profiling live data as it traps every single function call and generates a lot of statistics by default.

Example
-------

```
import cProfile def sum_(): total_sum =  0  # sum of numbers till 10000  for i in range(0,10001): total_sum += i return total_sum

cProfile.run('sum_()')
```

**Output**  
 

```
4  function calls in  0.002 seconds Ordered  by: standard name
```

| ncalls | tottime | percall | cumtime | percall | percall filename:lineno(function) |
| 1 | 0.000 | 0.000 | 0.002 | 0.002 | <string>:1(<module>) |
| 1 | 0.002 | 0.002 | 0.002 | 0.002 | cprofile.py:3(sum_) |
| 1 | 0.000 | 0.000 | 0.002 | 0.002 | {built-in method builtins.exec} |
| 1 | 0.000 | 0.000 | 0.000 | 0.000 | {method 'disable' of '_lsprof.Profiler' objects} |

As you can see from the output, the cProfile module provides a lot of information about the function's performance. 

*   ncalls =  Number of times the function was called 
*   tottime =  Total time spent in the function 
*   percall = Total time spent per call
*   cumtime =  Cumulative time spent in this and all sub-functions
*   percall = Cumulative time spent per call.

Line Profiler is a powerful python module that performs line-by-line profiling of your code. Sometimes, the hotspot in your code may be a single line and it is not easy to locate it from the source code directly.  Line Profiler is valuable in identifying how much time is taken by each line to execute and which sections need the most attention for optimization. However, it does not come with the standard python library and needs to be installed using the following command:

```
pip install line_profiler
```

Example
-------

```
from line_profiler import  LineProfiler  def sum_arrays():  # creating large arrays arr1 =  [3]  *  (5  **  10) arr2 =  [4]  *  (3  **  11)  return arr1 + arr2

lp =  LineProfiler() lp.add_function(sum_arrays) lp.run('sum_arrays()') lp.print_stats()
```

**Output**  
 

```
Timer unit:  1e-07 s Total time:  0.0562143 s File: e:\KDnuggets\Python_Profilers\lineprofiler.py Function: sum_arrays at line 2
```

| Line # | Hits | Time | Per Hit | % Time | Line Contents |
| 2 |  |  |  |  | def sum_arrays(): |
| 3 |  |  |  |  | \# creating large arrays   |
| 4 | 1 | 168563.0  | 168563.0  | 30.0 | arr1 = \[1\] * (10 ** 6)  |
| 5 | 1 | 3583.0 | 3583.0 | 0.6 | arr2 = \[2\] * (2 * 10 ** 7) |
| 6 | 1 | 389997.0  | 389997.0  | 69.4 | return arr1 + arr2 |

*   Line #  = Line number in your code file
*   Hits  = No of times it was executed
*   Time  = Total time spent to execute the line
*   Per Hit = Average time spent per hit
*   % Time = Percentage of time spent on the line relative to the total time of function
*   Line Contents = Actual Source Code

Memory profiler is a python profiler that tracks the memory allocation of your code. It can also generate flame graphs to help analyze memory usage and identify the memory leaks in your code. It is also useful to identify the hotspot regions that are causing a lot of allocations because python applications are often prone to memory management issues. Memory profilers profile the line-by-line statistics about the memory consumption and it needs to be installed using the following command:

```
pip install memory_profiler
```

Example
-------

```
import memory_profiler import random def avg_marks():  # Genrating Random marks for 50 students for each section sec_a = random.sample(range(0,  100),  50) sec_b = random.sample(range(0,  100),  50)  # combined average marks of two sections avg_a = sum(sec_a)  / len(sec_a) avg_b = sum(sec_b)  / len(sec_b) combined_avg =  (avg_a + avg_b)/2  return combined_avg
    
memory_profiler.profile(avg_marks)()
```

**Output**  
 

```
Filename: e:\KDnuggets\Python_Profilers\memoryprofiler.py
```

| Line # | Mem usage | Increment | Occurrences | Line Contents |
| 4 | 21.7 MiB  | 21.7 MiB  | 1 | def avg_marks(): |
| 5 |  |  |  | \# Genrating Random marks for 50 students for each section |
| 6 | 21.8 MiB | 0.0 MiB  | 1 | sec_a = random.sample(range(0, 100), 50) |
| 7 | 21.8 MiB | 0.0 MiB  | 1 | sec_b = random.sample(range(0, 100), 50) |
| 8 |  |  |  |  |
| 9 |  |  |  | \# combined average marks of two sections |
| 10 | 21.8 MiB | 0.0 MiB  | 1 | avg\_a = sum(sec\_a) / len(sec_a) |
| 11 | 21.8 MiB | 0.0 MiB  | 1 | avg\_b =  sum(sec\_b) / len(sec_b) |
| 12 | 21.8 MiB | 0.0 MiB  | 1 | combined\_avg = (avg\_a + avg_b)/2 |
| 13 | 21.8 MiB | 0.0 MiB  | 1 | return combined_avg |

*   Line #  = Line number in your code file
*   Mem usage = Memory usage of Python Interpreter
*   Increment  =  Difference in memory consumed of the current line to previous one
*   Occurences = No of times the code line was executed
*   Line Contents = Actual Source Code

Timeit is a built-in Python library that is specifically designed for evaluating the performance of small code snippets. It is a powerful tool that can help you identify and optimize the performance bottlenecks in your code, allowing you to write faster and more efficient code. Different implementations of an algorithm can also be compared using the timeit module but the downside is that only the individual lines of blocks of code can be analyzed using it.

Example
-------

```
import timeit
code_to_test =  """
# creating large arrays
arr1 = [3] * (5 ** 10)
arr2 = [4] * (3 ** 11)
arr1 + arr2
""" elapsed_time = timeit.timeit(code_to_test, number=10)  print(f'Elapsed time: {elapsed_time}')
```

**Output**  
 

```
Elapsed time:  1.3809973997995257
```

Its usage is limited to only evaluating the smaller code snippets. One important thing to note is that it displays different times each time the code snippet is run. This is because you may have other processes running on your computer and the allocation of the resources may vary from one run to the other, making it difficult to control all the variables and achieve the same processing time for each run.

Yappi is a python profiler that allows you to easily identify performance bottlenecks. It is written in C, making it one of the most efficient profilers available. It has a customizable API that lets you profile only the specific parts of your code that you need to focus on, giving you more control over the profiling process. Its ability to profile concurrent coroutines provides an in-depth understanding of how your code is functioning.  

Example
-------

```
import yappi def sum_arrays():  # creating large arrays arr1 =  [3]  *  (5  **  10) arr2 =  [4]  *  (3  **  11)  return arr1 + arr2 with yappi.run(builtins=True): final_arr = sum_arrays()  print("\n--------- Function Stats -----------") yappi.get_func_stats().print_all()  print("\n--------- Thread Stats -----------") yappi.get_thread_stats().print_all()  print("\nYappi Backend Types: ",yappi.BACKEND_TYPES)  print("Yappi Clock Types: ", yappi.CLOCK_TYPES)
```

> **Note:** Install yappi using this command: `pip install yappi`

   
**Output**  
 

```
---------  Function  Stats  -----------  Clock type: CPU Ordered  by: totaltime, desc
```

| name | ncall | tsub | ttot | tavg |
| ..lers\\yappiProfiler.py:4 sum_arrays | 1 | 0.109375 | 0.109375 | 0.109375 |
| builtins. next | 1 | 0.000000 | 0.000000 | 0.000000 |
| .. \_GeneratorContextManager.\_\_exit__ | 1 | 0.000000 | 0.000000 | 0.000000 |

  
\-\-\-\-\-\-\-\-\- Thread Stats -----------  

  
 
| name | id | tid | ttot | scnt |
| _MainThread | 0 | 15148 | 0.187500  | 1 |

```
Yappi  Backend  Types:  {'NATIVE_THREAD':  0,  'GREENLET':  1}  Yappi  Clock  Types:  {'WALL':  0,  'CPU':  1}
```

Remember to name your modules differently for the built-in modules. Otherwise, the import will import your module(i.e your python file) instead of the real built-in modules.

By using these profilers, developers can identify bottlenecks in their code and decide which implementation is best. With the right tools and a little bit of know-how, anyone can take their Python code to the next level of performance. So, get ready to optimize your Python performance optimization and watch it soar to new heights!

I am glad that you decided to read this article and I hope it has been a valuable experience for you.

    **[Kanwal Mehreen](https://www.linkedin.com/in/kanwal-mehreen1)** is an aspiring software developer with a keen interest in data science and applications of AI in medicine. Kanwal was selected as the Google Generation Scholar 2022 for the APAC region. Kanwal loves to share technical knowledge by writing articles on trending topics, and is passionate about improving the representation of women in tech industry.