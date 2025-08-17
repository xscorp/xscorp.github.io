---
layout: post
title: "A funny case of macro in C++"
tags: [Engineering]
categories: [Engineering]
author: xscorp
image:
    path: /assets/img/cpp-logo.png
---


Hello everyone, this is [xscorp](https://twitter.com/xscorp7) and today, we will be seeing a funny case of C++ macros.

Let's have a look at the code below

```c++
#include<iostream>
#define max(a,b) ((a>b) ? a : b)

int main()
{
	int number = 0;	
	for(int i=0; i<10; i++)
	{
		number = max(rand()%10 , 6);
		std::cout<<number<<std::endl;
	}
	return 0;
}
```

The above code simply generates a random number between 0 and 10 and prints out the greater number among the generated number and 6. It repeats the same process 10 times in loop.

To choose maximum among two numbers, the following macro is used in the code
```c++
#define max(a,b) ((a>b) ? a : b)
```
This macro simply returns the greater number among the passed arguments ```a``` and ```b```.

So let me ask you a question, **what is the minimum value of the variable ```number``` which is printed on the screen?**

As per the below line present in the code, it should be ```6```, right?: 
```c++
number = max(rand()%10 , 6);
```
Let's have a look at the output:

```bash
$ g++ code.cpp -o ./code
$ ./code
9
6
0
6
6
3
0
6
6
6
```
Uhhh, Everything else is nice but why ```0``` and ```3``` are getting printed on the screen when ```number``` can have a minum value of 6?
To know that, let's dive a bit on the macro being used.


## Macro demystified

The macro that was used to find the larger number was
```c++
#define max(a,b) ((a>b) ? a : b)
```
Let's write equivalent if-else block pseudo code for this macro
```c++
max(a , b):
	if a > b:
		return a
	else:
		return b
```

In general words, a Macro is simply some piece of code which is given some name. In the code, we are invoking the macro as ```max(rand()%10 , 6);```. Hence, every occurance of ```a``` and ```b``` in the macro will be replaced by ```rand()%10``` and ```6``` respectively.

```c++
max(a , b):
	if rand()%10 > 6:
		return rand()%10
	else:
		return 6
```

Do you see the problem now?
If the generated number is greater than 6, it will again generate a new random number between 0 and 10 and simply return it. So this is the reason why we are seeing ```0``` and ```3``` in the output.

Thanks for reading!
