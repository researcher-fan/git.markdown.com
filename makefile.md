# makefile user guide
[toc]

***
## 简介
Makefile 文件描述了整个工程的编译、连接等规则。其中包括：工程中的哪些源文件需要编译以及
如何编译、需要创建那些库文件以及如何创建这些库文件、如何最后产生我们想要的可执行文件。

***
## sampe code

```
	cc = gcc
	prom = calc
	inc = $(shell find ./ -name "*.h")
	src = $(shell find ./ -name "*.c")
	obj = $(src:%.c=%.o) # 字符串替换函数，将src所有的.c字串替换成.o
	 
	$(prom): $(obj)
	    $(cc) -o $(prom) $(obj)
	 
	%.o: %.c $(inc)
	    $(cc) -c $< -o $

	clean:
		rm -rf $(obj) $(prom)
```

为解决头文件修改，make无法察觉到变化，不会重新编译,
所以头文件单独写个变量，被.o 文件所依赖

***
## 常用命令
### make 命令参数

<details>
<summary>点击展开</summary>>

参数 | 详解
--- | ---
-b, -m             |         Ignored for compatibility.
-B, --always-make  |         Unconditionally make all targets.
-C DIRECTORY, --directory=DIRECTORY | Change to DIRECTORY before doing anything.
-d                 |        Print lots of debugging information.
--debug[=FLAGS]    |        Print various types of debugging information.
-e, --environment-overrides | Environment variables override makefiles.
-f FILE, --file=FILE, --makefile=FILE | Read FILE as a makefile.
-h, --help         |        Print this message and exit.
-i, --ignore-errors |       Ignore errors from commands.
-I DIRECTORY, --include-dir=DIRECTORY | Search DIRECTORY for included makefiles.
-j [N], --jobs[=N]  |       Allow N jobs at once; infinite jobs with no arg.
-k, --keep-going    |       Keep going when some targets can't be made.
-l [N], --load-average[=N], --max-load[=N] | Don't start multiple jobs unless load is below N.
-L, --check-symlink-times |  Use the latest mtime between symlinks and target.
-n, --just-print, --dry-run, --recon |Don't actually run any commands; just print them.
-o FILE, --old-file=FILE, --assume-old=FILE | Consider FILE to be very old and don't remake it.
-p, --print-data-base    |   Print make's internal database.
-q, --question           |  Run no commands; exit status says if up to date.
-r, --no-builtin-rules   |   Disable the built-in implicit rules.
-R, --no-builtin-variables | Disable the built-in variable settings.
-s, --silent, --quiet   |    Don't echo commands.
-S, --no-keep-going, --stop | Turns off -k.
-t, --touch          |      Touch targets instead of remaking them.
-v, --version        |      Print the version number of make and exit.
-w, --print-directory   |   Print the current directory.
--no-print-directory    |   Turn off -w, even if it was turned on implicitly.
-W FILE, --what-if=FILE, --new-file=FILE, --assume-new=FILE | Consider FILE to be infinitely new.
--warn-undefined-variables |  Warn when an undefined variable is referenced.
-N OPTION, --NeXT-option=OPTION | Turn on value of NeXT OPTION

</details>>

### 伪目标

Makefile 默认会认为目标应该对应于一个文件，这会造成什么问题呢？

假设我们有一个 Makefile：

```
clean:
    ...

.PHONY: clean
```

假如当前目录下有一个名为 clean 的文件。根据上述规则，该文件存在且没有什么依赖，那么执行 make clean 的时候该文件不需要更新，clean 条目下的命令都不会被执行。
这当然不是我们想要的结果，因此我们给它指定一下 .PHONY: clean 这样 make 不会去根据文件的存在、新旧而考虑是否要跳过它。

### 变量赋值
```
	VIR_A = A
	VIR_B = $(VIR_A) B
	VIR_A = AA
```
使用”=”进行赋值，变量的值是整个makefile中最后被指定的值;
上面示例中，最后VIR_B的值是AAB；


```
	VIR_A := A
	VIR_B := $(VIR_A) B
	VIR_A := AA
```
”:=”就表示直接赋值，赋予当前位置的值；
上面示例中，最后VIR_B的值是AB；  
“？=” 表示如果该变量没有被赋值，则赋予等号后的值；  
“+=” 追加赋值；

### 函数使用

```
	$(<function> <arguments>) 
	# or
	${<function> <arguments>}
```

#### 字符串处理函数

[参考blog链接](https://blog.csdn.net/yangxuan0261/article/details/52060582#strip-string)

##### subst

```
$(subst <from>,<to>,<text>)
```

名称：字符串替换函数——subst。  
功能：把字串<text>中的<from>字符串替换成<to>。  
返回：函数返回被替换过后的字符串。  

```
	comma:= ,
	empty:=
	space:= $(empty) $(empty)
	foo:= a b c
	bar:= $(subst $(space),$(comma),$(foo))
```

##### strip
```
$(strip <string>)
```
名称：去空格函数——strip。  
功能：去掉"string"字串中开头和结尾的空字符。  
返回：返回被去掉空格的字符串值。  


***
## 常用变量
### 自动变量

自动变量 | 含义
--- | ---
$@  | 目标集合
$%  | 当目标是函数库文件时, 表示其中的目标文件名
$<  | 第一个依赖目标. 如果依赖目标是多个, 逐个表示依赖目标
$?  | 比目标新的依赖目标的集合
$^  | 所有依赖目标的集合, 会去除重复的依赖目标
$+  | 所有依赖目标的集合, 不会去除重复的依赖目标
$*  | 这个是GNU make特有的, 其它的make不一定支持

***

## 遍历编译子目录

编译该目录下除了$(exclude_dirs)的子目录，  
执行命令。make all；

```
exclude_dirs := dir1 dir2

dirs := $(shell find . -maxdepth 1 -type d)
dirs := $(basename $(patsubst ./%,%,$(dirs)))
dirs := $(filter-out $(exclude_dirs),$(dirs))

SUBDIRS :=$(dirs)

.PHONY: all $(SUBDIRS) clean distclean

$(SUBDIRS):
	$(MAKE) -C $@

all:$(SUBDIRS)
	$(info $(SUBDIRS))

clean_dirs := $(addprefix _clean_,$(SUBDIRS))
$(clean_dirs):
	$(MAKE) -C $(patsubst _clean_%,%,$@) clean

clean:$(clean_dirs)

distclean_dirs := $(addprefix _distclean_,$(SUBDIRS))
$(distclean_dirs):
	$(MAKE) -C $(patsubst _distclean_%,%,$@) distclean

distclean:$(distclean_dirs)

```






