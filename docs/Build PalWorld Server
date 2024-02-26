# Palworld Server 部署

> Written by Zer0-hex

## 如果内存不够 16 G

```shell
dd if=/dev/zero of=/root/swapfilexx bs=1M count=2048	# 2G，具体大小看需求
chmod 0600 /root/swapfilexx
mkswap /root/swapfilexx
swapon /root/swapfilexx
echo /root/swapfilexx swap swap defaults 0 0 >> /etc/fstab
sysctl vm.swappiness=50
echo vm.swappiness=50 >> /etc/sysctl.conf
```



## Docker

- 安装 Docker

```shell
sudo su
apt update && apt upgrade -y
curl -fsSL https://get.docker.com | sh
tmux	# 终端复用工具
```

- 安装 Steam 镜像并运行 Bash

```
docker run -it --name=steamcmd cm2network/steamcmd bash
```

## 部署

```shell
mkdir -p ~/.steam/sdk64/
./steamcmd.sh +login anonymous +app_update 1007 +quit
cp ~/Steam/steamapps/common/Steamworks\ SDK\ Redist/linux64/steamclient.so ~/.steam/sdk64/
./steamcmd.sh +login anonymous +app_update 2394010 validate +quit
cd ~/Steam/steamapps/common/PalServer
./PalServer.sh port=8211 players=32 -useperfthreads -NoAsyncLoadingThread -UseMultithreadForDS
```



## Nginx 反向代理

- 在宿主机查看 docker 容器的 IP

```shell
docker inspect `docker ps -a | grep steam | awk '{print $1}'` | grep -oE '([0-9]{1,3}\.){3}[0-9]{1,3}' | tail -n 1
```

![image-20240125094308757](./assets/image-20240125094308757.png)

- 编辑 `/etc/nginx/nginx.conf`，在文件末尾添加以下内容，把上面得到的 IP 替换下面的 13 行位置， 17 行的端口为对外监听端口，

```shell
stream {
    # 日志格式化
    log_format proxy '$remote_addr [$time_local] '
    '$protocol $status $bytes_sent $bytes_received '
    '$session_time "$upstream_addr" '
    '"$upstream_bytes_sent" "$upstream_bytes_received"'
    '"$upstream_connect_time"';
    # 日志使用proxy格式
    access_log /var/log/nginx/udp-access.log proxy;
    open_log_file_cache off;

    upstream palworld {
        server 172.17.0.2:8211;	
    }
    upstream palworld_rcon {
        server 172.17.0.2:25575;	
    }

    server {
        listen 49494 udp reuseport ;
        proxy_connect_timeout 5s;
        proxy_timeout 20s;
        proxy_pass palworld;
    }
    server {
        listen 49499 udp reuseport ;
        proxy_connect_timeout 5s;
        proxy_timeout 20s;
        proxy_pass palworld_rcon;
    }
}
```

- 最终在帕鲁中输入 VPS 的 IP 与上方 17 行设置的端口就可以，需要在 VPS 防火墙或安全组中放行对应端口的 UDP 入向流量。



## 高级配置

### 服务端配置文件

- DefaultPalWorldSettings.ini

```shell
[/Script/Pal.PalGameWorldSettings]
OptionSettings=(Difficulty=None,DayTimeSpeedRate=0.500000,NightTimeSpeedRate=3.000000,ExpRate=3.000000,PalCaptureRate=3.000000,PalSpawnNumRate=2.000000,PalDamageRateAttack=1.000000,PalDamageRateDefense=1.000000,PlayerDamageRateAttack=1.000000,PlayerDamageRateDefense=1.000000,PlayerStomachDecreaceRate=1.000000,PlayerStaminaDecreaceRate=0.100000,PlayerAutoHPRegeneRate=1.000000,PlayerAutoHpRegeneRateInSleep=1.000000,PalStomachDecreaceRate=1.000000,PalStaminaDecreaceRate=0.100000,PalAutoHPRegeneRate=1.000000,PalAutoHpRegeneRateInSleep=3.000000,BuildObjectDamageRate=1.000000,BuildObjectDeteriorationDamageRate=0.100000,CollectionDropRate=3.000000,CollectionObjectHpRate=3.000000,CollectionObjectRespawnSpeedRate=3.000000,EnemyDropItemRate=3.000000,DeathPenalty=None,bEnablePlayerToPlayerDamage=False,bEnableFriendlyFire=False,bEnableInvaderEnemy=True,bActiveUNKO=False,bEnableAimAssistPad=True,bEnableAimAssistKeyboard=False,DropItemMaxNum=3000,DropItemMaxNum_UNKO=100,BaseCampMaxNum=8,BaseCampWorkerMaxNum=20,DropItemAliveMaxHours=1.000000,bAutoResetGuildNoOnlinePlayers=False,AutoResetGuildTimeNoOnlinePlayers=72.000000,GuildPlayerMaxNum=20,PalEggDefaultHatchingTime=0.000000,WorkSpeedRate=3.000000,bIsMultiplay=False,bIsPvP=False,bCanPickupOtherGuildDeathPenaltyDrop=False,bEnableNonLoginPenalty=True,bEnableFastTravel=True,bIsStartLocationSelectByMap=True,bExistPlayerAfterLogout=False,bEnableDefenseOtherGuildPlayer=False,CoopPlayerMaxNum=8,ServerPlayerMaxNum=32,ServerName="帕鲁集团",ServerDescription="这是一个问很危险的 Server",AdminPassword="Admin@123Admin@234",ServerPassword="admin",PublicPort=8211,RCONEnabled=True,RCONPort=25575,Region="",bUseAuth=True)
```

- 解析

```shell
Difficulty							简单：casual，普通：normal，困难：hard
DeathPenalty						死亡惩罚：None，Item，ItemAndEquipment，ALL
（None：无损失，Item：物品，ItemAndEquipment：丢失物品和设备，ALL：丢失物品、设备、帕鲁）

DayTimeSpeedRate					白天速度倍率
NightTimeSpeedRate					夜间速度倍率
ExpRate								经验倍率
WorkSpeedRate 						工作速率

PalCaptureRate						帕鲁捕获率
PalSpawnNumRate						帕鲁刷新率
EnemyDropItemRate					击败帕鲁爆率
CollectionDropRate					资源获取率
CollectionObjectRespawnSpeedRate	资源刷新率
CollectionObjectHpRate				资源HP倍率

PlayerStomachDecreaceRate			玩家饥饿消耗率
PlayerStaminaDecreaceRate			玩家耐力降低率
PalStomachDecreaceRate				帕鲁饥饿消耗率
PalStaminaDecreaceRate				帕鲁耐力降低率

PalDamageRateAttack					帕鲁攻击伤害倍率
PalDamageRateDefense				帕鲁受到伤害倍率
PlayerDamageRateAttack				玩家攻击伤害倍率
PlayerDamageRateDefense				玩家受到伤害倍率
PlayerAutoHPRegeneRate				玩家自动HP恢复率
PlayerAutoHpRegeneRateInSleep		玩家睡眠HP恢复率
PalAutoHPRegeneRate					帕鲁自动HP再生率
PalAutoHpRegeneRateInSleep			帕鲁睡眠HP再生率（在帕鲁box中）
BuildObjectDamageRate				建筑受到伤害倍率
BuildObjectDeteriorationDamageRate	建筑自然损坏速度

PalEggDefaultHatchingTime			孵蛋时间

BaseCampMaxNum						基地数量
BaseCampWorkerMaxNum				基地最大工人数
GuildPlayerMaxNum					公会最大成员数量
CoopPlayerMaxNum					小队最大成员数量

ServerPlayerMaxNum					服务器可以加入的最大人数
ServerName							服务器名称
ServerDescription					服务器描述
ServerPassword						服务器密码
PublicPort							公网端口
PublicIP							公网IP
RCONEnabled							开启游戏管理工具，需要设置管理员密码
AdminPassword						管理员密码
RCONPort							游戏管理工具端口
Region 								设置服务器地区
BanListURL							黑名单 Url
```

### 游戏内的管理工具

```shell-session
/Shutdown {Seconds} {MessageText}	服务器在秒数后关闭，可以发布消息通知
/DoExit								强制停止服务器
/Broadcast {MessageText}			向服务器中的所有玩家发送消息
/KickPlayer {SteamID}				将玩家从服务器上踢出
/BanPlayer {SteamID}				从服务器禁止玩家
/TeleportToPlayer {SteamID}			传送到目标玩家的当前位置
/TeleportToMe {SteamID}				目标玩家传送到您当前的位置
/ShowPlayers						显示所有已连接玩家的信息。
/Info								显示服务器信息。
/Save								保存这个世界的数据
```

## 备份

```shell
server {
       listen 80;
       listen [::]:80;

       server_name example.com;

       root /var/www/example.com;
       index index.html;

       location / {
               try_files $uri $uri/ =404;
       }
}
```

```shell
# 存档备份
每日备份：http://154.201.80.145/CngR8TLFi10z551EPeBS5DyZ6MoD8BwK/Daily_palworld_backups.tar.bz2
每周备份：http://154.201.80.145/CngR8TLFi10z551EPeBS5DyZ6MoD8BwK/Weekly_palworld_backups.tar.bz2
每月备份：http://154.201.80.145/CngR8TLFi10z551EPeBS5DyZ6MoD8BwK/Monthly_palworld_backups.tar.bz2
```





ask觉得好看青蛙似的和控球后卫克哈撒旦立刻就千里卢卡省的请尽快另外还可即将离开请问请问哈斯和 ed
