### oldDef.c
```
#define weak_alias(old, new) \
        extern __typeof(old) new __attribute__((weak, alias(#old)))


void DefaultHandler()
{
        puts("Default Handler");
}

weak_alias( DefaultHandler, Feature1);

```

### weak.c
```
#include<stdio.h>

/*
void Feature1()
{
        puts("Feature 1");
}
*/

int main()
{
        Feature1();
        DefaultHandler();

        return 0;
}

```

### Compilation
```
$ gcc weak.c oldDef.c -o weak
$ ./weak
```

- If you run above program as it is, it will print
```
Default Handler
Default Handler
```
- But if you uncomment `Feature1()` then it will print
```
Default Handler
Feature 1
```
