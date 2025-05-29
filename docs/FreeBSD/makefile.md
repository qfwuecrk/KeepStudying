# makefile

- 大写 Makefile；小写 makefile。当两个文件同时存在时，优先引用的是小写的makefile。一般发布的源码中是大写的Makefile。

- makefile 会根据该文件和所依赖的文件的时间戳来判断文件是否需要更新。

- 源代码结构

```c
/*
 └─root
    ├─Includes
    │	├─tool1.h
    │	├─tool2.h
    ├─Sources
    │	├─tool1.c
    │	├─tool2.c
    ├─main.c
*/

/* ---------- main.c ---------- */
#define _CRT_SECURE_NO_WARNINGS

#include "tool1.h"
#include "tool2.h"

int main(void)
{
    mytool1();
    mytool2();

    return 0;
}

/* ---------- tool1.h ---------- */
#ifndef TOOL1__
#define TOOL1__

#include <stdio.h>

void mytool1(void);

#endif // !TOOL1__

/* ---------- tool1.c ---------- */
#include "tool1.h"

void mytool1(void)
{
    printf("RUN IN FUNCTION: %s\n", __FUNCTION__);
    return;
}

/* ---------- tool12.h ---------- */
#ifndef TOOL2__
#define TOOL2__

#include <stdio.h>

void mytool2(void);

#endif // !TOOL2__

/* ---------- tool2.c ---------- */
#include "tool2.h"

void mytool2(void)
{
    printf("RUN IN FUNCTION: %s\n", __FUNCTION__);
    return;
}
```

- makefile示例
```shell
mytool:main.o tool1.o tool2.o
    gcc main.o tool1.o tool2.o -o mytool

main.o:main.c
    gcc main.c -c -Wall -g -o main.o
tool1.o:tool1.c
    gcc tool1.c -c -Wall -g -o main.o
tool2.o:tool1.c
    gcc tool2.c -c -Wall -g -o main.o

clean:
    rm *.o mytool -rf
```

- （上述优化）makefile 变量示例；$(RM) 是 rm -f
```shell
OBJS=main.o tool1.o tool2.o
CC=gcc
CFLAGS+=-c -Wall -g

mytool:$(OBJS)
    $(CC) $(OBJS) -o mytool

main.o:main.c
    $(CC) main.c $(CFLAGS) -o main.o
tool1.o:tool1.c
    $(CC) tool1.c $(CFLAGS) -o main.o
tool2.o:tool1.c
    $(CC) tool2.c $(CFLAGS) -o main.o

clean:
    $(RM) *.o mytool -r
```

- （上述优化）%.o:%.c 某某的.c生成某某的.o；`$^` 上述所依赖的文件；`$@`生成对应的目标文件
```makefile
OBJS=main.o tool1.o tool2.o
CC=gcc
CFLAGS+=-c -Wall -g

mytool:$(OBJS)
    $(CC) $^ -o mytool

%.o:%.c
    $(CC) $^ $(CFLAGS) -o $@

clean:
    $(RM) *.o mytool -r
```

- 上述makefile可执行的命令 make；make clean；