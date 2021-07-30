- [简介](#简介)
- [下载php源码](#下载php源码)
- [生成扩展代码](#生成扩展代码)
- [修改文件](#修改文件)
- [编译扩展](#编译扩展)
  - [根据config.m4文件生成configure文件](#根据configm4文件生成configure文件)
  - [编译](#编译)
- [测试扩展](#测试扩展)
  - [修改php.ini文件](#修改phpini文件)
  - [拷贝example扩展](#拷贝example扩展)
  - [验证扩展是否加载成功](#验证扩展是否加载成功)
- [实现自己的扩展函数](#实现自己的扩展函数)
  - [函数声明](#函数声明)
  - [函数实现和注册](#函数实现和注册)
  - [编译](#编译-1)
  - [测试](#测试)
- [参考文档](#参考文档)

# 简介
这是一个php扩展的示例，主要介绍如何用C/C++快速开发一个php扩展。

# 下载php源码
基于php-7.0.33来编写扩展
```
# 下载php-7.0.33
$ wget https://www.php.net/distributions/php-7.0.33.tar.gz
# 解压
$ tar xzf php-7.0.33.tar.gz
```

# 生成扩展代码
```
$ cd php-7.0.33/ext
$ sh ./ext_skel --extname=example
```
如果输出如下内容代表生成php7 example扩展代码成功了
```
$ sh ./ext_skel --extname=example
Creating directory example
Creating basic files: config.m4 config.w32 .gitignore example.c php_example.h CREDITS EXPERIMENTAL tests/001.phpt example.php [done].

To use your new extension, you will have to execute the following steps:

1.  $ cd ..
2.  $ vi ext/example/config.m4
3.  $ ./buildconf
4.  $ ./configure --[with|enable]-example
5.  $ make
6.  $ ./sapi/cli/php -f ext/example/example.php
7.  $ vi ext/example/example.c
8.  $ make

Repeat steps 3-6 until you are satisfied with ext/example/config.m4 and
step 6 confirms that your module is compiled into PHP. Then, start writing
code and repeat the last two steps as often as necessary.
```
ext_skel是一个专门用来生成扩展代码的shell脚本，执行完之后会在ext目录下生成一个example目录，example目录下的文件如下：
```
drwxr-xr-x  3 work work 4096 Jul 30 21:32 .
-rw-r--r--  1 work work  505 Jul 30 21:32 example.php
drwxr-xr-x  2 work work 4096 Jul 30 21:32 tests
-rw-r--r--  1 work work    7 Jul 30 21:32 CREDITS
-rw-r--r--  1 work work    0 Jul 30 21:32 EXPERIMENTAL
-rw-r--r--  1 work work 2332 Jul 30 21:32 php_example.h
-rw-r--r--  1 work work 5195 Jul 30 21:32 example.c
-rw-r--r--  1 work work  362 Jul 30 21:32 config.w32
-rw-r--r--  1 work work  398 Jul 30 21:32 .gitignore
-rw-r--r--  1 work work 2120 Jul 30 21:32 config.m4
drwxrwxr-x 76 work work 4096 Jul 30 21:32 ..
```
用ext_skel工具生成的扩展默认是C文件；这里如果有面向对象的需求，可以将example.c重命名为example.cpp(mv example.c example.cpp)

# 修改文件
修改ext/example/config.m4，打开--enable-example开关，config.m4文件中这--enable-example是被注释掉的。
注意：config.m4文件的注释符是dnl
```
dnl PHP_ARG_ENABLE(example, whether to enable example support,
dnl Make sure that the comment is aligned:
dnl [  --enable-example           Enable example support])
```
改为：
```
PHP_ARG_ENABLE(example, whether to enable example support,
dnl Make sure that the comment is aligned:
[  --enable-example           Enable example support])
```
# 编译扩展

## 根据config.m4文件生成configure文件
用phpize工具根据config.m4文件生成configure文件
```
$ export PHP_HOME=you_php_home
$ $PHP_HOME/php/bin/phpize
```
如果输入类似下面的内容说明执行成功了
```
Configuring for:
PHP Api Version:         20151012
Zend Module Api No:      20151012
Zend Extension Api No:   320151012
```
这时候当前目录下就会多出一些文件：
```
$ ls 
acinclude.m4  autom4te.cache  config.guess  config.m4   configure     config.w32  example.c    EXPERIMENTAL  ltmain.sh        missing        php_example.h  tests
aclocal.m4    build           config.h.in   config.sub  configure.in  CREDITS     example.php  install-sh    Makefile.global  mkinstalldirs  run-tests.php
```
其中比较重要的文件是configure，相信大家对这个文件应该不陌生，很多开源软件中都有这个文件。

## 编译
执行configure生成example扩展so，先不着急我们看看configure文件都支持哪些参数：
```
$ ./configure --help
`configure' configures this package to adapt to many kinds of systems.

Usage: ./configure [OPTION]... [VAR=VALUE]...

To assign environment variables (e.g., CC, CFLAGS...), specify them as
VAR=VALUE.  See below for descriptions of some of the useful variables.

Defaults for the options are specified in brackets.

Configuration:
  -h, --help              display this help and exit
      --help=short        display options specific to this package
      --help=recursive    display the short help of all the included packages
  -V, --version           display version information and exit
  -q, --quiet, --silent   do not print `checking ...' messages
      --cache-file=FILE   cache test results in FILE [disabled]
  -C, --config-cache      alias for `--cache-file=config.cache'
  -n, --no-create         do not create output files
      --srcdir=DIR        find the sources in DIR [configure dir or `..']

Installation directories:
  --prefix=PREFIX         install architecture-independent files in PREFIX
                          [/usr/local]
  --exec-prefix=EPREFIX   install architecture-dependent files in EPREFIX
                          [PREFIX]

By default, `make install' will install all the files in
`/usr/local/bin', `/usr/local/lib' etc.  You can specify
an installation prefix other than `/usr/local' using `--prefix',
for instance `--prefix=$HOME'.

For better control, use the options below.

Fine tuning of the installation directories:
  --bindir=DIR            user executables [EPREFIX/bin]
  --sbindir=DIR           system admin executables [EPREFIX/sbin]
  --libexecdir=DIR        program executables [EPREFIX/libexec]
  --sysconfdir=DIR        read-only single-machine data [PREFIX/etc]
  --sharedstatedir=DIR    modifiable architecture-independent data [PREFIX/com]
  --localstatedir=DIR     modifiable single-machine data [PREFIX/var]
  --libdir=DIR            object code libraries [EPREFIX/lib]
  --includedir=DIR        C header files [PREFIX/include]
  --oldincludedir=DIR     C header files for non-gcc [/usr/include]
  --datarootdir=DIR       read-only arch.-independent data root [PREFIX/share]
  --datadir=DIR           read-only architecture-independent data [DATAROOTDIR]
  --infodir=DIR           info documentation [DATAROOTDIR/info]
  --localedir=DIR         locale-dependent data [DATAROOTDIR/locale]
  --mandir=DIR            man documentation [DATAROOTDIR/man]
  --docdir=DIR            documentation root [DATAROOTDIR/doc/PACKAGE]
  --htmldir=DIR           html documentation [DOCDIR]
  --dvidir=DIR            dvi documentation [DOCDIR]
  --pdfdir=DIR            pdf documentation [DOCDIR]
  --psdir=DIR             ps documentation [DOCDIR]

System types:
  --build=BUILD     configure for building on BUILD [guessed]
  --host=HOST       cross-compile to build programs to run on HOST [BUILD]
  --target=TARGET   configure for building compilers for TARGET [HOST]

Optional Features and Packages:
  --disable-option-checking  ignore unrecognized --enable/--with options
  --disable-FEATURE       do not include FEATURE (same as --enable-FEATURE=no)
  --enable-FEATURE[=ARG]  include FEATURE [ARG=yes]
  --with-PACKAGE[=ARG]    use PACKAGE [ARG=yes]
  --without-PACKAGE       do not use PACKAGE (same as --with-PACKAGE=no)
  --with-libdir=NAME      Look for libraries in .../NAME rather than .../lib
  --with-php-config=PATH  Path to php-config php-config
  --enable-example           Enable example support
  --enable-shared=PKGS    Build shared libraries default=yes
  --enable-static=PKGS    Build static libraries default=yes
  --enable-fast-install=PKGS
                          Optimize for fast installation default=yes
  --with-gnu-ld           Assume the C compiler uses GNU ld default=no
  --disable-libtool-lock  Avoid locking (might break parallel builds)
  --with-pic              Try to use only PIC/non-PIC objects default=use both
  --with-tags=TAGS        Include additional configurations automatic


Some influential environment variables:
  CC          C compiler command
  CFLAGS      C compiler flags
  LDFLAGS     linker flags, e.g. -L<lib dir> if you have libraries in a
              nonstandard directory <lib dir>
  LIBS        libraries to pass to the linker, e.g. -l<library>
  CPPFLAGS    (Objective) C/C++ preprocessor flags, e.g. -I<include dir> if
              you have headers in a nonstandard directory <include dir>
  CPP         C preprocessor

Use these variables to override the choices made by `configure' or to help
it to find libraries and programs with nonstandard names/locations.

Report bugs to the package provider.
```
比较重要的参数是下面两个：
```
--with-php-config=PATH  Path to php-config php-config
--enable-example           Enable example support
```

执行下面几条命令生成example.so
```
$ ./configure --with-php-config=$PHP_HOME/php/bin/php-config --enable-example
$ make -j4
```
如果看到如下的输出就说明你example扩展已经编译成功了
```
Libraries have been installed in:
   /home/liushuai06/work/php_source_code/php-7.0.33/ext/example/modules

If you ever happen to want to link against installed libraries
in a given directory, LIBDIR, you must either use libtool, and
specify the full pathname of the library, or use the `-LLIBDIR'
flag during linking and do at least one of the following:
   - add LIBDIR to the `LD_LIBRARY_PATH' environment variable
     during execution
   - add LIBDIR to the `LD_RUN_PATH' environment variable
     during linking
   - use the `-Wl,--rpath -Wl,LIBDIR' linker flag
   - have your system administrator add LIBDIR to `/etc/ld.so.conf'

See any operating system documentation about shared libraries for
more information, such as the ld(1) and ld.so(8) manual pages.
----------------------------------------------------------------------

Build complete.
Don't forget to run 'make test'.
```
执行完了之后，example.so位于modules目录下
```
$ ls modules
example.so
```
# 测试扩展

## 修改php.ini文件
在php.ini文件末尾增加如下内容：
```
[example]
extension="example.so"
```

## 拷贝example扩展
将编译好的example.so拷贝到php.ini文件中extension_dir配置的位置。 比如我的php.ini中extension_dir配置如下：
```
extension_dir="/home/work/php/ext"
```
则只需要将编译好的example.so拷贝到/home/work/php/ext目录
```
cp modules/example.so /home/work/php/ext
```
## 验证扩展是否加载成功
```
cd $PHP_HOME
$ php/bin/php --ri example

example

example support => enabled
```
如果执行php/bin/php --ri example，那么说明example扩展已经安装成功了。
当然我们还可以用源码中的example.php文件测试一下
```
$ $PHP_HOME/php/bin/php example.php
Functions available in the test extension:
confirm_example_compiled

Congratulations! You have successfully modified ext/example/config.m4. Module example is now compiled into PHP.
```

# 实现自己的扩展函数
到现在为止，我们已经把扩展的架子搭好了，下来我们实现几个自定义的函数，然后在php中调用这些函数来给大家演示下如何在扩展中实现自己的函数。

## 函数声明
在php_example.h文件 #endif	/* PHP_EXAMPLE_H */ 行之前增加如下内容：
```
// 声明全局函数 求和
PHP_FUNCTION(example_sum);
// 声明全局函数 echo
PHP_FUNCTION(example_echo);
```
这里我们增加两个函数example_sum(求两个数之和)和example_echo(输出一个字符串)。

## 函数实现和注册
在example.c文件中增加如下内容：
```C
/**
 * @params double num1
 * @params double num2
 * @return double sum
 * @bref
 * 实现全局函数 example_sum
 * */
PHP_FUNCTION(example_sum)
{
    double num1,num2,sum;
    if (zend_parse_parameters(ZEND_NUM_ARGS() TSRMLS_CC, "dd", &num1, &num2) == FAILURE) {
        RETURN_FALSE;
    }
    sum = num1 + num2;
    // RETURN_* 的定义在zend_API.h中
    RETURN_DOUBLE(sum);
}
 
/**
 * @params string 
 * @return string
 * @bref
 * 实现全局函数 example_echo
 * */
PHP_FUNCTION(example_echo)
{
    char *arg = NULL;
    size_t arg_len, len;
    zend_string *strg;
    if (zend_parse_parameters(ZEND_NUM_ARGS(), "s", &arg, &arg_len) == FAILURE) {
        return;
    }

    strg = strpprintf(0, "%.78s", arg);
    RETURN_STR(strg);
}
```
注册函数，同样函数修改example.c文件，找到example_functions定义的地方，然后注册example_sum和example_echo方法
```C
/* {{{ example_functions[]
 *
 * Every user visible function must have an entry in example_functions[].
 */
const zend_function_entry example_functions[] = {
    PHP_FE(confirm_example_compiled,    NULL)       /* For testing, remove later. */
    PHP_FE(example_sum,                 NULL)       /* register global example_sum function */
    PHP_FE(example_echo,                NULL)       /* register global example_echo function */
    PHP_FE_END  /* Must be the last line in example_functions[] */
};
/* }}} */
```

## 编译
```
$ $PHP_HOME/php/bin/phpize
$ ./configure --with-php-config=$PHP_HOME/php/bin/php-config --enable-example
$ make -j4
```
生成的example.so在modules目录下，将其拷贝到对应的目录
```
cp -rf modules/example.so /home/work/php/ext
```
## 测试
验证example扩展是否加载成功
```
$ $PHP_HOME/php/bin/php --ri example
```

编写php文件测试扩展函数功能，新建test_example.php文件内容如下：
```
<?php

$num1 = 1.1;
$num2 = 2.2;
var_dump(example_sum($num1, $num2));

$str = "hello world";
var_dump(example_echo($str));
```
执行test_example.php
```
$ $PHP_HOME/php/bin/php test_example.php
float(3.3)
string(11) "hello world"
```
如果输出如上内容，那么恭喜你，说明你已经测试通过啦，晚上给自己加个鸡腿吧！

# 参考文档

* [用C/C++扩展你的PHP](https://www.laruence.com/2009/04/28/719.html)
