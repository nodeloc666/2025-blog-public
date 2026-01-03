# 暂停Windows更新1万年
1. `win+R`-->输入`regedit`,确定
2. 依次点击`HKEY_LOCAL_MACHINE`-->`SOFTWARE`-->`Microsoft`-->`WindowsUpdate`-->`UX`-->`Settings`-->`DWORD(32位)值`
3. 重命名为：`FlightSettingsMaxPauseDays`


![image.png](https://img.uutv.dpdns.org/file/1741407249188_image.png)
4.双`FlightSettingsMaxPauseDays`，选择十进制，输入`35000`(大约95年左右)
**注意：这个数字别太大，会导致你电脑刷出来的**


![image.png](https://img.uutv.dpdns.org/file/1741407306018_image.png)


5. 回到系统设置-->Windows更新,就可以选择你设置的暂停最大天数了


![333.png](https://img.uutv.dpdns.org/file/1741407699577_333.png)


![image.png](https://img.uutv.dpdns.org/file/1741407597253_image.png)


## 如何恢复更新呢？
**点击继续更新就行了**


## 视频教程（也是此文章内容来源）
[怎么让Win11暂停更新一万年?](https://www.bilibili.com/video/BV1zR9uYiET5/?share_source=copy_web&vd_source=ac00d7148458adb021ff968c1fdd8bac)


