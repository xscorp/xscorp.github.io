# Funny C++ puzzle

Hi, this is xscorp and in this article, we will be seeing a funny C++ puzzle on macros.

Have a look at the code below

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

To choose maximum among two numbers, the following macro is used
```c++
#define max(a,b) ((a>b) ? a : b)
```
This macro simply returns the greater number among the passed arguments ```a``` and ```b```.

So let me ask you a question, **what is the minimum value of the variable ```number``` which is printed on the screen?**

