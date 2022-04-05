Locales : [中文版](#chinese)
This document is for redirecting multiple screencasts between desktops.
It requires Linux and Docker skill.

# English
## Server (Linux Desktop/Server/Virtual Machine)
### nginx-rtmp
Run the following command :
```bash
docker run --rm -d -p 1935:1935 --name nginx-rtmp tiangolo/nginx-rtmp
```

## Client Sender
### OBS
- In the "URL" enter the `rtmp://<ip_of_host>/live` replacing `<ip_of_host>` with the IP of the host in which the container is running. For example: `rtmp://192.168.0.1/live`
- In the "Stream key" use a "key" that will be used later in the client URL to display that specific stream. For example: `test`

- Controls > Settings , Stream Tabs (Server : `rtmp://192.168.0.1/live` , Stream Key : `test` )
![](streaming_obs_sender.png)

## Client Receiver 
### OBS
 - Sources > Add VLC Video Source , Playlist > Add Path/URL (example : `rtmp://192.168.0.1/live/test`)
![](streaming_obs_receiver.png)
# Chinese
本文件是給有多台主機桌面需要同時直播使用
需要有一定的 Linux 跟 Docker 的操作知識

## 伺服器 (Linux 主機)
### nginx-rtmp
執行以下指令 :
```bash
docker run --rm -d -p 1935:1935 --name nginx-rtmp tiangolo/nginx-rtmp
```

## 直播輸出主機
### OBS
- 主機地方輸入 `rtmp://<ip_of_host>/live` , `<ip_of_host>` 是伺服器的IP 位置. 例如 `rtmp://192.168.0.1/live`
- 直播金鑰部份作為串流區分, 可以任意輸入. 例如 `test`

- 控制項 > 設定 , 直播分頁下 (主機 : `rtmp://192.168.0.1/live` , 直播金鑰 : `test` )
![](streaming_obs_sender.png)

## 播放主機
### OBS
- 來源 > 加入 VLC 視訊來源 , 增加 播放清單 (例如 `rtmp://localhost/live/test`)
![](streaming_obs_receiver_cht.png)