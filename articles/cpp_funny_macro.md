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
