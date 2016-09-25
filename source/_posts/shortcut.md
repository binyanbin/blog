---
title: 我的mac精华浓缩版
date: 2016-9-4
desc: mac 快捷键 效率工具
---

## mac下的效率神器

1.Alfred  效率神器，脚本之王，谁用谁知道。要破解版的直接找我:)。
2.iTerm  terminal分屏神器，当然不仅仅是分屏
3.CatchMouse  多屏显示器必备，最多可以支持四台显示器随意切换屏幕和光标。

## 跨平台工具

1.Eclipse  
2.Workbench  mysql客户端必备
3.Sublime Text 
4.leanote 笔记本+blog,支持markdown,思维导图，最重要的是免费   
5.Evernode 印像笔记本
6.Chrome  
7.Postman  webapi测试专用
8.AxaTools  小工具集合，各种转换计算和正则测试。
9.atash UML作图专用
10.Jmeter  性能测试
11.Xccello 个人/团队任务管理

## 开启mac 移动硬盘ntfs格式的读写功能
虽说exFAT 可以跨操作系统使用，但总感觉不如ntfs
如何开启:
在创建一个文件  sudo vim /etc/fstab 
输入 LABEL=valume none ntfs rw,auto,nobrowse
第一列:LABEL=valume 表示移动硬盘的卷名
第二列:加入文件系统的位置
第三列:文件格式
第四列:文件系统状态 rw表示可读写
第五列:好像是是否备份的意思。
第六列:是否在finder中显示，nobrowse不显示。
如何使用:
保存fstab文件之后重启mac,在finder下 cmd+shift+g，输入/Volumes/valume进入(valume为你的移动硬盘卷名)

## 快捷键 

1.mac基础:
ctrl+上下左右   切屏
cmd+上下左右  定位 
cmd+shift+上下左右  定位+选择
fn+上下左右  滚动
ctrl+cmd+f  全屏
cmd+w 关闭当前窗口
cmd+q 退出应用程序

2.chrome(你不可能不用的)  
cmd+l 地址栏
cmd+option+左右 切页面
ctrl+tab 切页面
cmd+t 新页面
cmd+w 关闭页面
cmd+shift+w 关闭窗口
cmd+option+i 开发者工具

3.vim(用linux服务器你无法绕过它)
:set number 显示行号
:syntax on 语法显示
:%d 删除全部内容
w下一个单词
b上一个单词
gg文件首
Ｇ文件尾
0行首
^行首文字
$行尾
dd删除当前行
3x 删除后3个字符 3X 删除前3个个字符
11G 调到第11行
Ctrl+f Ctrl+b 翻页
/word 查找 n 查找下一个
u  ctrl+r   撤销 重做


4.bash
ctrl+a 开头
ctrl+e 结毛
ctrl+k 清除光标后至行尾
ctrl+u清除光标至之前
tab 补全
ctrl+c 终止

5.sublime
ctrl+k+b 目录显示/隐藏
cmd+option+number 分屏
ctrl+shift+number 移屏
cmd+option+左右 切文件
ctrl+tab 切文件
cmd+n 创建新文件
cmd+p 查找文件
cmd+shift+p  插件命令
cmd+alt+g 快找
cmd+f 查找 alt+cmd+r 正则
cmd+shift+f 替换
cmd+k+u 大写 cmd+k+l 小写
cmd+k+num 折到第几层
cmd+k+j 全部展开
cmd+k+k 删除到行末尾
cmd+w 关闭文件 
cmd+shift+w 关闭窗口
ctrl+shift+k 删除行
F6 拼写检查

6.eclispe (java开发必备)
cmd+e 已打开文件
cmd+o 文件导航
cmd+alt+s 生成
cmd+f7 切换窗体
f3 转定义
cmd+shift+g 转引用
cmd+shift+r 打开文件
cmd+shift+h 打开类型
ctrl+h 综合查找
cmd+alt+r 重命名
cmd+1 快速修复
ctrl+q 回到最后一次编辑的地方
alt+/  内容提示
cmd+/ 注释
cmd+t 查看继承关系
cmd+shift+f 格式化
cmd+alt+j 添加完整注释
ctrl+m 隐藏package
cmd+shift+[ 分坚屏

## 附赠hexo blog发布脚本

``` bash
#!/bin/bash
cd ~/Documents/git/blog
hexo clean
hexo g
cp -r ~/Documents/git/blog/public/ ~/Documents/git/binyanbin.github.io/
cd ~/Documents/git/binaynbin.github.io/
git commit -a -m 'submit'
git push
```

持续补充中......