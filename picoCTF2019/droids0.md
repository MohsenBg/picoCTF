# 🚀 Droids 0

- **📛 Challenge Name:** Droids0  
- **🎯 Difficulty:** Hard  
- **🔗 Challenge:** [View on PicoCTF](https://play.picoctf.org/practice/challenge/11?category=3&originalEvent=1&page=2)  
- **🤖 File Type:** APK  

## Introduction  
In this write-up, we will walk through solving the `Droids0` challenge using `adb` and `jadx` to reverse-engineer the APK and extract the flag.  

## My Experience  

### Challenge Question:  
---
> Where do droid logs go. Check out this file. 
---

## Reverse Engineering Steps  

### 1️⃣ Installing the APK with ADB  
![Installing APK on Android VM with ADB](./image/ADB_InstallApk_Droids0.png)  

### 2️⃣ Opening the App in Android x86 VM  

#### App Menu  
![](./image/Android_MenuApp_Droids0.png)  

#### App View  
![](./image/Android_OpenApp_Droids0.png)  

### 3️⃣ Analyzing the APK with JADX  

#### MainActivity  
![](./image/Jadx_MainActivity_Droids0.png)  

#### FlagstaffHill Class  
![](./image/Jadx_FlagstaffHill_Droids0.png)  

### 4️⃣ Extracting Logs with ADB Logcat  
![](./image/ADB_Logcat_Droids0.png)  

## 🎉 The Flag  

```
picoCTF{a.moose.once.bit.my.sister}
```
