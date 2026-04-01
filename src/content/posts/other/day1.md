---
title: day1
published: 2025-10-28
description: "写了两道题目"
tags: ["ctfwp", "web", "python"]
category: ctf
draft: false
---

## CTF+ 题目1 [Sign_in](https://www.ctfplus.cn/problem-detail/1975492169263222784/description "Sign_in")
#### 标签: Misc 0xgame2025

由题目发现信息: MGhRa3dve0dvdm0wd29fZDBfMGhRNHczXzJ5MjVfQHhuX3JAbXVfUHliX3BlWH0=
显然为Base64编码 解码后发现信息 0hQkwo{Govm0wo_d0_0hQ4w3_2y25_@xn_r@mu_Pyb_peX}
但是仍然不符合flag的格式，通过观察发现疑似凯撒密码 通过 "2y25" 猜测 应为 "2o25" 位移量为10
解得: 0xGame{Welc0me_t0_0xG4m3_2o25_@nd_h@ck_For_fuN} 获得flag

## CTF+ 题目2 [Vigenere](https://www.ctfplus.cn/problem-detail/1975492206248595456/description)
#### 标签 CRYPTO 0xgame2025

附件提供python 代码
```python
from string import digits, ascii_letters, punctuation
from secret import flag

key = "Welcome-2025-0xGame"
alphabet = digits + ascii_letters + punctuation


def vigenere_encrypt(plaintext, key):
    ciphertext = ""
    key_index = 0
    for char in plaintext:
        bias = alphabet.index(key[key_index])
        char_index = alphabet.index(char)
        new_index = (char_index + bias) % len(alphabet)
        ciphertext += alphabet[new_index]
        key_index = (key_index + 1) % len(key)
    return ciphertext


print(vigenere_encrypt(flag, key))

# WL"mKAaequ{q_aY$oz8`wBqLAF_{cku|eYAczt!pmoqAh+

```


发现为一段维吉尼亚密码的加密程序 其中flag库不存在
但是该维吉尼亚密码提供了密钥，且发现该维吉尼亚字母表与常规不一样存在数字和符号
故完整的字母表为
```txt
0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ!"#$%&'()*+,-./:;<=>?@[\]^_`{|}~
```

所以根据维吉尼亚密码的公式:


$$ C_i = P_i + K_i (\mod L) $$

<br>

$$ P_i = C_i - K_i (\mod L) $$

<br>
其中 L 为 字母表的长度

得到解密代码
``` python
from string import digits, ascii_letters, punctuation

key = "Welcome-2025-0xGame"
alphabet = digits + ascii_letters + punctuation

def vigenere_decrypt(ciphertext, key):
    plaintext = ""
    key_index = 0
    for char in ciphertext:
        if char in alphabet:
            bias = alphabet.index(key[key_index])
            char_index = alphabet.index(char)
            new_index = (char_index - bias) % len(alphabet)
            plaintext += alphabet[new_index]
            key_index = (key_index + 1) % len(key)
        else:
            plaintext += char  # 保留不在字母表中的字符
    return plaintext

ciphertext = 'WL"mKAaequ{q_aY$oz8`wBqLAF_{cku|eYAczt!pmoqAh+'
result = vigenere_decrypt(ciphertext, key)
print(result)
```
得到flag: 0xGame{you_learned_vigenere_cipher_2df4b1c2e3}



