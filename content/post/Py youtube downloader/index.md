+++
author = "Lulaide"
title = "Py youtube downloader"
date = "2024-08-29"
description = "一个可以用来下载YouTube视频的Python库"
tags = [
    "Python",
    "Linux"
]
categories = ["分享"]
image = "image.png"
+++

>此文章简单分享一个功能齐全的 YouTube 下载器，以及如何在终端使用它
## `yt-dlp`项目

- 项目地址：
```
https://github.com/yt-dlp/yt-dlp
```
- pip 安装
```bash
pip install yt-dlp
```

## 终端下载视频的简易方法

- 使用-F参数获取视频信息
```bash
yt-dlp -F https://www.youtube.com/shorts/j9ank-HGDoI
```
```
ID      EXT   RESOLUTION FPS CH │    FILESIZE   TBR PROTO │ VCODEC          VBR ACODEC      ABR ASR MORE INFO
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
sb1     mhtml 25x45        1    │                   mhtml │ images                                  storyboard
sb2     mhtml 48x27        2    │                   mhtml │ images                                  storyboard
sb0     mhtml 50x90        1    │                   mhtml │ images                                  storyboard
233     mp4   audio only        │                   m3u8  │ audio only          unknown             Default
234     mp4   audio only        │                   m3u8  │ audio only          unknown             Default
139-drc m4a   audio only      2 │   293.58KiB   49k https │ audio only          mp4a.40.5   49k 22k low, DRC, m4a_dash
139     m4a   audio only      2 │   293.66KiB   49k https │ audio only          mp4a.40.5   49k 22k low, m4a_dash
140-drc m4a   audio only      2 │   776.26KiB  130k https │ audio only          mp4a.40.2  130k 44k medium, DRC, m4a_dash
140     m4a   audio only      2 │   776.14KiB  130k https │ audio only          mp4a.40.2  130k 44k medium, m4a_dash
602     mp4   144x256     15    │ ~ 696.95KiB  117k m3u8  │ vp09.00.10.08  117k video only
269     mp4   144x256     30    │ ~   1.09MiB  187k m3u8  │ avc1.4D400C    187k video only
160     mp4   144x256     30    │   723.15KiB  121k https │ avc1.4D400C    121k video only          144p, mp4_dash
603     mp4   144x256     30    │ ~1019.44KiB  170k m3u8  │ vp09.00.11.08  170k video only
229     mp4   240x426     30    │ ~   2.02MiB  346k m3u8  │ avc1.4D4015    346k video only
133     mp4   240x426     30    │     1.55MiB  266k https │ avc1.4D4015    266k video only          240p, mp4_dash
604     mp4   240x426     30    │ ~   1.92MiB  328k m3u8  │ vp09.00.20.08  328k video only
230     mp4   360x640     30    │ ~   4.79MiB  821k m3u8  │ avc1.4D401E    821k video only
134     mp4   360x640     30    │     3.59MiB  615k https │ avc1.4D401E    615k video only          360p, mp4_dash
605     mp4   360x640     30    │ ~   3.57MiB  612k m3u8  │ vp09.00.21.08  612k video only
231     mp4   480x854     30    │ ~   8.21MiB 1405k m3u8  │ avc1.4D401F   1405k video only
135     mp4   480x854     30    │     6.61MiB 1133k https │ avc1.4D401F   1133k video only          480p, mp4_dash
606     mp4   480x854     30    │ ~   6.25MiB 1070k m3u8  │ vp09.00.30.08 1070k video only
311     mp4   720x1280    60    │ ~  24.57MiB 4206k m3u8  │ avc1.640020   4206k video only
298     mp4   720x1280    60    │    21.87MiB 3745k https │ avc1.640020   3745k video only          720p60, mp4_dash
612     mp4   720x1280    60    │ ~  21.69MiB 3713k m3u8  │ vp09.00.40.08 3713k video only
312     mp4   1080x1920   60    │ ~  38.20MiB 6540k m3u8  │ avc1.64002A   6540k video only
299     mp4   1080x1920   60    │    35.09MiB 6009k https │ avc1.64002A   6009k video only          1080p60, mp4_dash
617     mp4   1080x1920   60    │ ~  29.63MiB 5073k m3u8  │ vp09.00.41.08 5073k video only
```

- 查看视频信息，选择下载的视频，记住对应的`ID`
比如我要下载
```
ID      EXT   RESOLUTION FPS CH │    FILESIZE   TBR PROTO │ VCODEC          VBR ACODEC      ABR ASR MORE INFO
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
617     mp4   1080x1920   60    │ ~  29.63MiB 5073k m3u8  │ vp09.00.41.08 5073k video only
```
```bash
yt-dlp -f 617 https://www.youtube.com/shorts/j9ank-HGDoI
```
一段时间后视频会下载完成
```
[youtube] j9ank-HGDoI: Downloading m3u8 information
[info] j9ank-HGDoI: Downloading 1 format(s): 617
[hlsnative] Downloading m3u8 manifest
[hlsnative] Total fragments: 10
[download] Destination: Minecraft Parkour： NOOB vs PRO vs GOD vs TECHNO #shorts [j9ank-HGDoI].mp4
[download] 100% of   26.27MiB in 00:00:07 at 3.65MiB/s
```
>欢迎指出错误，以便更正!
>Welcome to comment!