---
title: day2
published: 2025-10-18
description: "写了两道题目"
tags: ["ctfwp", "web", "python"]
category: ctf
draft: false
---

## MeoCtf 题目 [吃豆人](https://ctf.xidian.edu.cn/training/18?challenge=820)
### 标签 Mini_L-CTF_2025 Misc
在进入靶机地址后得到如下画面
![图片1](/images/day2/1.png)
然后通过F12 打开开发者工具的时候分析**game.js** 文件分析胜利检查代码
```javascript

 // 胜利检测
    if (score >= 5000 && !hasGotFlag) {
        fetch('/submit_score', {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ score: score })
        })
            .then(response => response.json())
            .then(data => {
                if (data.flag) {
                    alert("🎉 恭喜！你的flag是：" + data.flag);
                } else {
                    alert("未达到指定分数！");
                }
            });
        hasGotFlag = true;
    }
```

得知获得flag的条件是 全局变量score >= 5000, 同时分析发现函数
```javascript
function initGame() {
    people = [{ x: 200, y: 200 }];
    dx = box;
    dy = 0;
    score = 0;
    isGameOver = false;
    hasGotFlag = false;

    redBallCount = 1000;
    blueBallCount = 1;
    balls = [];

    spawnBalls();

    if (game) clearInterval(game);
    game = setInterval(draw, 150);
}
```
会在游戏开始时初始化score变量， 所以修改score的初始化值为5000
就能得到flag:miniLCTF{tH1S_g4Me_so-eA5Y-rIghT?6083a665}


## MeoCtf 题目 [麦霸评分](https://ctf.xidian.edu.cn/training/18?challenge=836)
### 标签 Mini_L_-CTF_2025 Misc
在进入靶机地址后得到如下画面
![图片2](/images/day2/2.png)
将网页保存到本地
分析发现 整个代码中有4个API

##### 主要API接口：
 + prepare-recording          // 准备录音
 + compare-recording          // 上传并比较录音（POST）
 + get-original-audio         // 获取原始音频URL
 + clean-recording/{filename} // 清理特定录音文件（DELETE）
 + clean-all-recordings       // 清理所有录音文件（DELETE）

从 stopRecording 中分析可以看出在停止录音后将处理好的音频数据 传入处理 uploadRecording
其中 uploadRecording 作用是 调用API compare-recording 将音频上传至服务并返回 分数和flag
其中传回来的数据如下
```txt
flag: null
message: "很不错诶，继续加油，你就是明日之星"
originalAudioUrl: "/original.wav"
similarity: "11.46"
success: false
```
所以只需要利用这个API 就可以获得本题的 flag
故将 实例文件下载到本地 并在文件末尾添加如下代码:

```javascript
// 在页面中添加文件选择按钮
const fileInput = document.createElement('input');
fileInput.type = 'file';
fileInput.accept = 'audio/wav';
fileInput.style.display = 'none';
document.body.appendChild(fileInput);

// 添加上传按钮
const uploadBtn = document.createElement('button');
uploadBtn.textContent = '上传本地WAV文件';
uploadBtn.className = 'button';
uploadBtn.onclick = () => fileInput.click();
document.querySelector('.controls').appendChild(uploadBtn);

// 文件选择处理
fileInput.addEventListener('change', function(e) {
    const file = e.target.files[0];
    if (file && file.type === 'audio/wav') {
        // 显示预览
        recordedAudio.src = URL.createObjectURL(file);
        recordingContainer.style.display = 'block';
        
        // 直接上传
        uploadRecording(file);
    }
});
```
![图片3](/images/day2/3.png)
得到flag: miniLCTF{Y0u-@rE_4-t41ENt3D-slngEr376fddc2}
