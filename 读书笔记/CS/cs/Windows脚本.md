#### 1. echo

- 1.@echo off /on:  关闭/打开命令显示
- 2.echo message:  显示message

#### 2.关于图标: 

- 1.所有快捷方式可自定义图标, 可通过"属性 -> 快捷方式 -> 更改图标" 更改
- 2.所有exe文件自带图标, 可通过 "更改图标 -> 浏览" 引入新图标

#### 3.关于运行方式: 可运行文件快捷方式可以固定 "管理员权限运行"("属性 -> 高级")

#### 4.关于CMD变量: 

- 1.变量以 %var_name% 读取和调用
- 2.变量赋值以 "set var_name = value"
- 3.CMD解释器识别空格, 所以一般谨慎输入空格

#### 5.截取字符串:  %str:~n_1,n_2%

#### 6.延时:   timeout /T n, 其中n表示延迟秒数

#### 7.运行exe文件: start exe_name.exe

#### 8.条件语法:

```bash
if 条件 (pass) 
ELSE (pass)
```

#### 9.CMD脚本获取键盘输入

```cmd
@set /p input=plzInputNumber：
set sn=%input%
echo "%sn%"
pause
```

#### 10.传递参数至其他脚本文件

```cmd
@echo off
echo %1 %2 %3
start A.bat arg1 arg2 arg3
Pause
```













