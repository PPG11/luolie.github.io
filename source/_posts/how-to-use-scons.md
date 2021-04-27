---
title: how-to-use-scons
top: false
cover: false
toc: true
mathjax: true
date: 2021-04-27 15:10:14
password:
summary:
tags:
  - C/C++
categories:
---

sons 和 makefile 类似，可以编译多种源文件，不过它编译的脚本叫 SConstruct。

### 小试一下

创建一个名为 `SConstruct` 的文件，按照 `python` 语法，写入如下内容。

```python
print("... Hello, Scons !")
```

运行一下

```
$ scons
scons: Reading SConscript files ...
... Hello, Scons !
scons: done reading SConscript files.
scons: Building targets ...
scons: `.' is up to date.
scons: done building targets.
```

## 教程

### 简单编译

首先写一个简单的 C 文件

```C++
// hello.cpp
#include<iostream>
using namespace std;

int main(){
    cout << "Hello, World!" << endl;
    return 0;
}
```

然后只需要在 SConstruct 中编写

```python
Program('hello.cpp')
```

这个短小的配置文件给了 SCons 两条信息：

- `hello.cpp`: 你想编译什么（一个可执行程序），你编译的输入文件（hello.cpp）。
- `Program`: 一个编译器方法（builder_method），一个 Python 调用告诉 SCons，你想编译一个可执行程序。Program 编译方法是 SCons 提供的许多编译方法中一个。

调用 Program 编译方法的的时候，它编译出来的程序名字是和源文件名是一样的。

从 hello.cpp 源文件编译一个可执行程序的调用将会编译出一个名为 hello 的可执行程序，在 windows 系统里会编译出一个名为 hello.exe 的可执行程序。

如果想编译出来的程序的名字与源文件名字不一样，只需要在源文件名的左边声明一个目标文件的名字就可以了：

```python
Program('new_hello','hello.cpp')
```

SConstruct 文件实际上就是一个 Python 脚本。可以在 SConstruct 文件中使用 Python 的 `#` 注释：

重要的一点是 SConstruct 文件并不完全像一个正常的 Python 脚本那样工作，其工作方式更像一个 Makefile，那就是在 SConstruct 文件中 SCons 函数被调用的顺序并不影响 SCons 你实际想编译程序和目标文件的顺序。换句话说，当你调用 Program 方法，你并不是告诉 SCons 在调用这个方法的同时马上就编译这个程序，而是告诉 SCons 你想编译这个程序：

```python
print "Calling Program('hello.c')"
Program('hello.c')
print "Calling Program('goodbye.c')"
Program('goodbye.c')
print "Finished calling Program()"
```

并不会顺序执行，因为 scons 是并行执行的这点要特别注意，是不同的线程进行编译的。

### 多个源文件

如果编译的源文件有多个.c 文件。可以这样写:

```python
Program('hello_world', ['test.c', 'test1.c', 'test2.c'])
```

也可以使用 Glob 函数，定义一个匹配规则来指定源文件列表，比如\*,?等标准的 shell 模式。如下所示：

```pyhton
Program('program', Glob('*.cpp'))
```

为了更容易处理文件名长列表，SCons 提供了一个 Split 函数，这个 Split 函数可以将一个用引号引起来，并且以空格或其他空白字符分隔开的字符串分割成一个文件名列表，示例如下：

```python
Program('program', Split('main.cpp  file1.cpp  file2.cpp'))
```

或者

```python
src_files=Split('main.cpp  file1.cpp  file2.cpp')
Program('program', src_files)
```

SCons 也允许使用 Python 关键字参数来标识输出文件和输入文件。输出文件是 target，输入文件是 source，示例如下：

```python
src_files=Split('main.cpp  file1.cpp  file2.cpp')
Program(target='program', source=src_files)
```

多个程序之间共享源文件是很常见的代码重用方法。一种方式就是利用公共的源文件创建一个库文件，然后其他的程序可以链接这个库文件。另一个更直接，但是不够便利的方式就是在每个程序的源文件列表中包含公共的文件，示例如下：

```python
common=['common1.cpp', 'common2.cpp']
foo_files=['foo.cpp'] + common
bar_files=['bar1.cpp', 'bar2.cpp'] + common
Program('foo', foo_files)
Program('bar', bar_files)
```

### 编译和链接库

#### 静态库

可以使用 Library 方法来编译库文件：

```pyhton
Library('foo', ['f1.cpp', 'f2.cpp', 'f3.cpp'])
```

除了使用源文件外，Library 也可以使用目标文件

```python
Library('foo', ['f1.c', 'f2.o', 'f3.c', 'f4.o'])
```

甚至可以在文件 List 里混用源文件和目标文件

```python
Library('foo', ['f1.cpp', 'f2.o', 'f3.c', 'f4.o'])
```

使用 StaticLibrary 显示编译静态库

```python
StaticLibrary('foo', ['f1.cpp', 'f2.cpp', 'f3.cpp'])
```

#### 动态库

如果想编译动态库（在 POSIX 系统里）或 DLL 文件（Windows 系统），可以使用 SharedLibrary：

```python
SharedLibrary('foo', ['f1.cpp', 'f2.cpp', 'f3.cpp'])
```

#### 链接库

链接库文件的时候，使用\$LIBS 变量指定库文件，使用\$LIBPATH 指定存放库文件的目录：

```python
Library('foo', ['f1.cpp', 'f2.cpp', 'f3.cpp'])
Program('prog', LIBS=['foo', 'bar'], LIBPATH='.')
```

> 注意到，你不需要指定库文件的前缀（比如 lib）或后缀（比如.a 或.lib），SCons 会自动匹配。

默认情况下，链接器只会在系统默认的库目录中寻找库文件。SCons 也会去\$LIBPATH 指定的目录中去寻找库文件。\$LIBPATH 由一个目录列表组成，如下所示：

```python
Program('prog', LIBS='m', LIBPATH=['/usr/lib', '/usr/local/lib'])
```

### 节点对象

所有编译方法会返回一个节点对象列表，这些节点对象标识了那些将要被编译的目标文件。这些返回出来的节点可以作为参数传递给其他的编译方法。例如，假设我们想编译两个目标文件，这两个目标有不同的编译选项，并且最终组成一个完整的程序。这意味着对每一个目标文件调用 Object 编译方法，如下所示：

```python
Object('hello.cpp', CCFLAGS='-DHELLO')
Object('goodbye.cpp', CCFLAGS='-DGOODBYE')
Program(['hello.o', 'goodbye.o'])
```

这样指定字符串名字的问题就是我们的 SConstruct 文件不再是跨平台的了。因为在 Windows 里，目标文件成为了 hello.obj 和 goodbye.obj。一个更好的解决方案就是将 Object 编译方法返回的目标列表赋值给变量，这些变量然后传递给 Program 编译方法：

```python
hello_list = Object('hello.cpp', CCFLAGS='-DHELLO')
goodbye_list = Object('goodbye.c', CCFLAGS='-DGOODBYE')
Program(hello_list + goodbye_list)
```

**显示创建文件和目录节点**

在 SCons 里，表示文件的节点和表示目录的节点是有清晰区分的。SCons 的 File 和 Dir 函数分别返回一个文件和目录节点：

```python
hello_c=File('hello.cpp')
Program(hello_c)
```

通常情况下，你不需要直接调用 File 或 Dir，因为调用一个编译方法的时候，SCons 会自动将字符串作为文件或目录的名字，以及将它们转换为节点对象。只有当你需要显示构造节点类型传递给编译方法或其他函数的时候，你才需要手动调用 File 和 Dir 函数。有时候，你需要引用文件系统中一个条目，同时你又不知道它是一个文件或一个目录，你可以调用 Entry 函数，它返回一个节点可以表示一个文件或一个目录：

```python
xyzzy=Entry('xyzzy')
```

**将一个节点的文件名当作一个字符串**

如果你不是想打印文件名，而是做一些其他的事情，你可以使用内置的 Python 的 str 函数。例如，你想使用 Python 的 os.path.exists 判断一个文件是否存在：

```python
import os.path
program_list=Program('hello.cpp')
program_name=str(program_list[0])
if not os.path.exists(program_name):
    print(program_name, "does not exist!")
```

### 依赖性

#### 隐式依赖：\$CPPPATH Construction 变量

```c++
#include <iostream>
#include "hello.h"
using namespace std;

int main(){
    cout << "Hello, " << string << endl;
    return 0;
}
```

并且，hello.h 文件如下：

```c++
#define string "world"
```

在这种情况下，我们希望 SCons 能够认识到，如果 hello.h 文件的内容发生改变，那么 hello 程序必须重新编译。我们需要修改 SConstruct 文件如下：

```python
# CPPPATH 告诉 SCons 去当前目录('.') 查看那些被 C 源文件（.c或.h文件）包含的文件。
Program('hello.cpp', CPPPATH='.')
```

就像\$LIBPATH 变量，\$CPPPATH 也可能是一个目录列表，或者一个被系统特定路径分隔符分隔的字符串。

```python
Program('hello.cpp', CPPPATH=['include', '/home/project/inc'])
```

### 环境

#### 外部环境

外部环境指的是在用户运行 SCons 的时候，用户环境中的变量的集合。这些变量在 SConscript 文件中通过 Python 的`os.environ`字典可以获得。你想使用外部环境的 SConscript 文件需要增加一个`import os`语句。

#### 构造环境

一个构造环境是在一个 SConscript 文件中创建的一个唯一的对象，这个对象包含了一些值可以影响 SCons 编译一个目标的时候做什么动作，以及决定从那一个源中编译出目标文件。SCons 一个强大的功能就是可以创建多个构造环境，包括从一个存在的构造环境中克隆一个新的自定义的构造环境。

**创建一个构造环境：Environment 函数**

默认情况下，SCons 基于你系统中工具的一个变量集合来初始化每一个新的构造环境。当你初始化一个构造环境时，你可以设置环境的构造变量来控制一个是如何编译的。例如：

```python
env=Environment(CC='gcc', CCFLAGS='-O2')
env.Program('foo.c')
# or
env=Environment(CXX='/usr/local/bin/g++', CXXFLAGS='-02')
env.Program('foo.cpp')
```

**从一个构造环境中获取值**

你可以使用访问 Python 字典的方法获取单个的构造变量：

```python
env=Environment()
print("CC is:", env['CC'])
print("CXX is:", env['CXX'])
```

一个构造环境实际上是一个拥有方法的对象。如果你想直接访问构造变量的字典，你可以使用 Dictionary 方法：

```python
env=Environment(FOO='foo', BAR='bar')
dict=env.Dictionary()
for key in ['OBJSUFFIX', 'LIBSUFFIX', 'PROGSUFFIX']:
    print("key=%s, value=%s"  %  (key,dict[key]))
```

**默认的构造环境：DefaultEnvironment 函数**

可以控制默认构造环境的设置，使用 DefaultEnvironment 函数：

```python
DefaultEnvironment(CC='/usr/local/bin/gcc')
```

这样配置以后，所有 Program 或者 Object 的调用都将使用/usr/local/bin/gcc 编译目标文件。注意到 DefaultEnvironment 返回初始化了的默认构造环境对象，这个对象可以像其他构造环境一样被操作。所以如下的代码和上面的例子是等价的：

```python
env=DefaultEnvironment()
env['CC']='/usr/local/bin/gcc'
```

**多个构造环境**

构造环境的真正优势是你可以创建你所需要的许多不同的构造环境，每一个构造环境对应了一种不同的方式去编译软件的一部分或其他文件。比如，如果我们需要用-O2 编译一个程序，编译另一个用-g，我们可以如下做：

```python
opt=Environment(CCFLAGS='-O2')
dbg=Environment(CCFLAGS='-g')
opt.Program('foo','foo.cpp')
dbg.Program('bar','bar.cpp')
```

### 参考文献

https://blog.csdn.net/MOU_IT/article/details/95229790
