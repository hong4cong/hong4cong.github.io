---
layout: post
title:  "Xcode调试之LLDB"
date:   2017-02-16 10:23:32
categories: iOS

---

# Xcode调试之LLDB

### WHAT’S LLDB?

1. LLVM的组件包括clang(前端编译器)，libc++(llvm的C++标准库)，lldb(调试器)等。
2. 作为LLVM提供的软件调试器，lldb(LLDB debugger)虽处于早期开发当中，依旧提供了对C,C++,Objective-C,Swift语言程序的调试支持。
3. lldb继承了GDB的优点，弥补了GDB的不足，如GUI，性能改善，插件式支持(提供C++/Python的API)和扩展。
4. LLDB可工作于OSX，Linux，FreeBSD，Windows，支持i386，x86-64,ARM架构。从Xcode4.3开始，lldb即为默认调试器。

### LLDB语法

```
<command> [<subcommand> [<subcommand>...]] <action> [-options [option-value]] [argument [argument...]]
```

1. command和subcommand：LLDB调试命令的名称。命令和子命令按层级结构来排列：一个命令对象为跟随其的子命令对象创建一个上下文，子命令又为其子命令创建一个上下文，依此类推。
2. action：执行命令的操作
3. options：命令选项
4. arguement：命令的参数
5. `[]`：表示命令是可选的，可以有也可以没有

例子：

```
breakpoint set -n main
```

1. `command`: `breakpoint` 表示断点命令
2. `action`: `set` 表示设置断点
3. `option`: `-n` 表示根据方法name设置断点
4. `arguement`: `mian` 表示方法名为mian

#### ~/.lldbinit

LLDB有了一个启动时加载的文件`~/.lldbinit`，每次启动都会加载。所以一些初始化的事儿，我们可以写入`~/.lldbinit`中，比如给命令定义别名等。但是由于这时候程序还没有真正运行，也有部分操作无法在里面玩，比如设置断点。



### LLDB命令

#### expression

expression命令的作用是执行一个表达式，并将表达式返回的结果输出。expression的完整语法是这样的：

```
expression <cmd-options> -- <expr>
```

1. cmd-options：命令选项，一般情况下使用默认的即可，不需要特别标明。
2. `--`: 命令选项结束符，表示所有的命令选项已经设置完毕，如果没有命令选项，`--`可以省略
3. expr: 要执行的表达式

说`expression`是LLDB里面最重要的命令都不为过。因为他能实现2个功能:

* 执行某个表达式

  我们在代码运行过程中，可以通过执行某个表达式来动态改变程序运行的轨迹。假如我们在运行过程中，突然想把self.view颜色改成红色，看看效果。我们不必写下代码，重新run，只需暂停程序，用`expression`改变颜色，再刷新一下界面，就能看到效果

  ```
  // 改变颜色
  (lldb) expression -- self.view.backgroundColor = [UIColor redColor]
  // 刷新界面
  (lldb) expression -- (void)[CATransaction flush]
  ```

* 将返回值输出

  也就是说我们可以通过`expression`来打印东西。
  假如我们想打印self.view：

  ```
  (lldb) expression -- self.view
  (UIView *) $1 = 0x00007fe322c18a10
  ```

#### p & print & call

一般情况下，我们直接用expression还是用得比较少的，更多时候我们用的是`p`、`print`、`call`。这三个命令其实都是`expression --`的别名

1. `print`: 打印某个东西，可以是变量和表达式

2. `p`: 可以看做是`print`的简写

   ```
   (lldb) p 16
   (int) $5 = 16
   (lldb) p/x 16
   (int) $6 = 0x00000010
   (lldb) p/t 16
   (int) $7 = 0b00000000000000000000000000010000
   (lldb) p/c 17
   (int) $8 = \x11\0\0\0
   (lldb) p/s "hello"
   (const char [6]) $10 = "hello"
   ```

3. `call`: 调用某个方法。

e.g: 下面代码效果相同：

```
(lldb) expression -- self.view
(UIView *) $5 = 0x00007fb2a40344a0
(lldb) p self.view
(UIView *) $6 = 0x00007fb2a40344a0
(lldb) print self.view
(UIView *) $7 = 0x00007fb2a40344a0
(lldb) call self.view
(UIView *) $8 = 0x00007fb2a40344a0
(lldb) e self.view
(UIView *) $9 = 0x00007fb2a40344a0
```

4. LLDB为`expression -O --`定义了一个别名：`po`,一般打印的时候，打印出来的是对象的指针，而不是对象本身。如果我们想打印对象用po

#### breakpoint

1. breakpoint set命令用于设置断点

   e.g: 我们想给所有类中的`viewWillAppear:`设置一个断点:

   **使用-n根据方法名设置断点**

   ```
   (lldb) breakpoint set -n viewWillAppear:
   Breakpoint 13: 33 locations.
   ```

   **使用-f指定文件**, e.g: 我们只需要给`ViewController.m`文件中的`viewDidLoad`设置断点：

   ```
   (lldb) breakpoint set -f ViewController.m -n viewDidLoad
   Breakpoint 22: where = TLLDB`-[ViewController viewDidLoad] + 20 at ViewController.m:22, address = 0x000000010272a6f4
   ```

   这里需要注意，如果方法未写在文件中（比如写在category文件中，或者父类文件中），指定文件之后，将无法给这个方法设置断点。

   **使用-l指定文件某一行设置断点**

   e.g: 我们想给`ViewController.m`第38行设置断点

   ```
   (lldb) breakpoint set -f ViewController.m -l 38
   Breakpoint 23: where = TLLDB`-[ViewController text:] + 37 at ViewController.m:38, address = 0x000000010272a7d5
   ```

   **使用-c设置条件断点**

   e.g: `text:`方法接受一个`ret`的参数，我们想让`ret == YES`的时候程序中断：

   ```
   (lldb) breakpoint set -n text: -c ret == YES
   Breakpoint 7: where = TLLDB`-[ViewController text:] + 30 at ViewController.m:37, address = 0x0000000105ef37ce
   ```

   **使用-o设置单次断点**

   e.g: 如果刚刚那个断点我们只想让他中断一次：

   ```
   (lldb) breakpoint set -n text: -o
   'breakpoint 3': where = TLLDB`-[ViewController text:] + 30 at ViewController.m:37, address = 0x000000010b6f97ce
   ```

   **breakpoint command**

   有的时候我们可能需要给断点添加一些命令，比如每次走到这个断点的时候，我们都需要打印`self`对象。我们只需要给断点添加一个`po self`命令，就不用每次执行断点再自己输入`po self`了

   ##### breakpoint command add

   `breakpoint command add`命令就是给断点添加命令的命令。

   e.g: 假设我们需要在`ViewController`的`viewDidLoad`中查看`self.view`的值
   我们首先给`-[ViewController viewDidLoad]`添加一个断点

   ```
   (lldb) breakpoint set -n "-[ViewController viewDidLoad]"
   'breakpoint 3': where = TLLDB`-[ViewController viewDidLoad] + 20 at ViewController.m:23, address = 0x00000001055e6004
   ```

   可以看到添加成功之后，这个`breakpoint`的id为3，然后我们给他增加一个命令：`po self.view`

   ```
   (lldb) breakpoint command add -o "po self.view" 3
   ```

   `-o`完整写法是`--one-liner`，表示增加一条命令。`3`表示对id为`3`的`breakpoint`增加命令。
   添加完命令之后，每次程序执行到这个断点就可以自动打印出`self.view`的值了

   如果我们一下子想增加多条命令，比如我想在`viewDidLoad`中打印当前frame的所有变量，但是我们不想让他中断，也就是在打印完成之后，需要继续执行。我们可以这样玩：

   ```
   (lldb) breakpoint command add 3
   Enter your debugger command(s).  Type 'DONE' to end.
   > frame variable
   > continue
   > DONE
   ```

   输入`breakpoint command add 3`对断点3增加命令。他会让你输入增加哪些命令，输入'DONE'表示结束。这时候你就可以输入多条命令了

   注意：多次对同一个断点添加命令，后面命令会将前面命令覆盖

   ##### breakpoint command list

   ```
   (lldb) breakpoint command list 3
   'breakpoint 3':
       Breakpoint commands:
         frame variable
         continue
   ```

   ##### breakpoint command delete

   ```
   (lldb) breakpoint command delete 3
   (lldb) breakpoint command list 3
   Breakpoint 3 does not have an associated command.
   ```

   ##### breakpoint list

   如果我们想查看已经设置了哪些断点，可以使用`breakpoint list`
   e.g:

   ```
   (lldb) breakpoint list
   Current breakpoints:
   4: name = '-[ViewController viewDidLoad]', locations = 1, resolved = 1, hit count = 0
     4.1: where = TLLDB`-[ViewController viewDidLoad] + 20 at ViewController.m:23, address = 0x00000001055e6004, resolved, hit count = 0
   ```

   ##### breakpoint disable/enable/delete

#### watchpoint

`breakpoint`有一个孪生兄弟`watchpoint`。如果说`breakpoint`是对方法生效的断点，`watchpoint`就是对地址生效的断点

如果我们想要知道某个属性什么时候被篡改了，我们该怎么办呢？有人可能会说对setter方法打个断点不就行了么？但是如果更改的时候没调用setter方法呢？
这时候最好的办法就是用`watchpoint`。我们可以用他观察这个属性的地址。如果地址里面的东西改变了，就让程序中断

##### watchpoint set

`watchpoint set`命令用于添加一个`watchpoint`。只要这个地址中的内容变化了，程序就会中断。

##### watchpoint set variable

一般情况下，要观察变量或者属性，使用`watchpoint set variable`命令即可
e.g: 观察`self->_string`

```
(lldb) watchpoint set variable self->_string
Watchpoint created: Watchpoint 1: addr = 0x7fcf3959c418 size = 8 state = enabled type = w
    watchpoint spec = 'self->_string'
    new value: 0x0000000000000000
```

想监视vMain变量什么时候被重写了，监视这个地址什么时候被写入

```
(lldb) p (ptrdiff_t)ivar_getOffset((struct Ivar *)class_getInstanceVariable([MyView class], "vMain"))
(ptrdiff_t) $0 = 8
(lldb) watchpoint set expression -- (int *)$myView + 8
Watchpoint created: Watchpoint 3: addr = 0x7fa554231340 size = 8 state = enabled type = w
new value: 0x0000000000000000
```



#### thread & thread backtrace & bt

有时候我们想要了解线程堆栈信息，可以使用`thread backtrace`
`thread backtrace`作用是将线程的堆栈打印出来。我们来看看他的语法

```
thread backtrace [-c <count>] [-s <frame-index>] [-e <boolean>]
```

`thread backtrace`后面跟的都是命令选项：

`-c`：设置打印堆栈的帧数(frame)
`-s`：设置从哪个帧(frame)开始打印
`-e`：是否显示额外的回溯
实际上这些命令选项我们一般不需要使用。

e.g: 当发生crash的时候，我们可以使用`thread backtrace`查看堆栈调用

```
(lldb) thread backtrace
* thread #1: tid = 0xdd42, 0x000000010afb380b libobjc.A.dylib`objc_msgSend + 11, queue = 'com.apple.main-thread', stop reason = EXC_BAD_ACCESS (code=EXC_I386_GPFLT)
    frame #0: 0x000000010afb380b libobjc.A.dylib`objc_msgSend + 11
  * frame #1: 0x000000010aa9f75e TLLDB`-[ViewController viewDidLoad](self=0x00007fa270e1f440, _cmd="viewDidLoad") + 174 at ViewController.m:23
    frame #2: 0x000000010ba67f98 UIKit`-[UIViewController loadViewIfRequired] + 1198
    frame #3: 0x000000010ba682e7 UIKit`-[UIViewController view] + 27
    frame #4: 0x000000010b93eab0 UIKit`-[UIWindow addRootViewControllerViewIfPossible] + 61
    frame #5: 0x000000010b93f199 UIKit`-[UIWindow _setHidden:forced:] + 282
    frame #6: 0x000000010b950c2e UIKit`-[UIWindow makeKeyAndVisible] + 42
```

我们可以看到crash发生在`-[ViewController viewDidLoad]`中的第23行，只需检查这行代码是不是干了什么非法的事儿就可以了。

LLDB还为backtrace专门定义了一个别名：`bt`，他的效果与`thread backtrace`相同，如果你不想写那么长一串字母，直接写下`bt`即可.

##### thread return

Debug的时候，也许会因为各种原因，我们不想让代码执行某个方法，或者要直接返回一个想要的值。这时候就该`thread return`上场了。

```
thread return [<expr>]
```

`thread return`可以接受一个表达式，调用命令之后直接从当前的frame返回表达式的值。

e.g: 我们有一个`someMethod`方法，默认情况下是返回YES。我们想要让他返回NO,我们只需在方法的开始位置加一个断点，当程序中断的时候，输入命令即可:

```
(lldb) thread return NO
```

效果相当于在断点位置直接调用`return NO;`，不会执行断点后面的代码

##### thread其他不常用的命令

thread 相关的还有其他一些不常用的命令，这里就简单介绍一下即可，如果需要了解更多，可以使用命令`help thread`查阅

1. `thread jump`: 直接让程序跳到某一行。由于ARC下编译器实际插入了不少retain，release命令。跳过一些代码不执行很可能会造成对象内存混乱发生crash。
2. `thread list`: 列出所有的线程
3. `thread select`: 选择某个线程
4. `thread until`: 传入一个line的参数，让程序执行到这行的时候暂停
5. `thread info`: 输出当前线程的信息

#### frame

我们在控制台上输入命令`bt`，可以打印出来所有的frame

##### frame variable

平时Debug的时候我们经常做的事就是查看变量的值，通过`frame variable`命令，可以打印出当前frame的所有变量

如果我们要需要打印指定变量，也可以给`frame variable`传入参数:

```
(lldb) frame variable self->_string
(NSString *) self->_string = nil
```

不过`frame variable`只接受变量作为参数，不接受表达式，也就是说我们无法使用`frame variable self.string`，因为`self.string`是调用`string`的`getter`方法。所以一般打印指定变量，我更喜欢用`p`或者`po`。

##### frame info: 查看当前frame的信息

##### frame select: 选择某个frame









其他略

详情参考

[小笨狼与LLDB的故事](http://www.jianshu.com/p/e89af3e9a8d7)

[DEBUGGING WITH LLDB](http://kangwang1988.github.io/tech/2016/03/27/Debugging-with-lldb.html)

