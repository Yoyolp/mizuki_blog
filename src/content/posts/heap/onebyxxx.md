---
title: Off by xxx1
published: 2026-04-19
description: "Off by one & off by null(施工中)"
tags: ["ctfwp", "heap"]
category: ctf
draft: true
---

**Off by one 和 Off by null** 都是一种发生在程序中的非预期函数功能，指非预期的溢出一个字节,或者非预期溢出 `\x00` 这个字节。利用其手法，涉及 libc malloc这个手法的前向合并和后向合并


