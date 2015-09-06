# EasyDarwin支持HLS #
## 正常运行easydarwin ##
或在调试模式下运行也可以，重点注意配置文件中的
MOVIE_FOLDER设置；
## 配置nginx ##
WEB目录为easydarwin.xml中movie_folder同一个目录、
nginx配置文件相关部分，如下
![](http://i.imgur.com/Tk66PsZ.png)

Easydarwin配置文件相关部分
![](http://i.imgur.com/q9R2lfN.png)

## 运行nginx ##
- [nginx下载地址](http://nginx.org/download/nginx-1.8.0.zip)
- 双击程序，运行
- 重新载入配置文件 `nginx -s reload `
- 停止 `nginx -s stop`

## 获取HLS串 ##
- 原rtsp流串：`rtsp://admin:12345@14.23.115.10/mpeg4/ch1/sub/av_stream`
- url请求接口：`http://127.0.0.1:8080/api/easyhlsmodule?name=live&url=rtsp://admin:12345@14.23.115.10/mpeg4/ch1/sub/av_stream`
- 返回，如图 
- ![](http://i.imgur.com/JQoBD0X.png)
- 可以直接在vlc中播放 http://127.0.0.1/live/live.m3u8
