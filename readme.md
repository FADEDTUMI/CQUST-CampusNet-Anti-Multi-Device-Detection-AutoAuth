# 重庆科技大学校园网认证以及防多设备检测的教程文档

本文档包含三个部分：
1. 极路由4增强版刷机教程（注意：此路由器处理器太旧，我已经更换新的路由器：新路由3，newwifi3。若已经有一台处理器新的路由器且刷好openwrt固件请跳跃至防多设备检测教程处，若需要刷机教程，可参考极路由4刷机教程，各路由器刷机流程几乎相同。）
2. 防多设备检测教程
3. 锐捷校园网自动认证脚本（适用于校园网认证环境，此处以重庆科技大学校园网认证为例，认证方式为锐捷）

校园网多设备检测基本原理：

- **IPID**：IP Identification，用于唯一标识每个IP数据包。如果多个设备共用一个IP，但数据包ID不连续，会触发检测。
- **TTL**：Time To Live，数据包的生存时间。不同设备的TTL通常不同，共用IP时TTL不一致也会触发检测。
- **MAC**：MAC地址，最基础的检测法，相信我不用多说
- **UA（User-Agent）**：通过浏览器或设备发送的用户代理字符串来识别设备类型。
- **Flash cookie**：浏览器访问Flash网页时记录相关内容信息的一个存储区域，Flash Cookie并不会被浏览器轻易清除,用于跟踪用户设备


## 一、极路由4增强版刷机教程

### 1. 获取local_ssh程序

- **背景说明**：由于厂商倒闭关服，我们需要自行通过算法基于uuid以及local_token生成cloud_token，然后才能获取local_ssh。
- **操作**：下载获取local_ssh的程序。
- 开启SSH的程序源于：[https://www.right.com.cn/FORUM/thread-8268438-1-1.html]

### 2. 开启SSH服务

- 获取成功后，端口22开启，此时可通过终端工具选择ssh2协议进行连接。
  - **注意**：此时需要将ssh连接变为永久，因为通过算法开启的端口号只会维持5分钟。
  - **命令**：
    ```bash
    /etc/init.d/dropbear enable
    ```
  - 将SSH设置为永久。（这里我使用的终端工具是SecureCRT）
  - **图片参考**：
  - ![[image](https://github.com/FADEDTUMI/CQUST-CampusNet-Anti-Multi-Device-Detection-AutoAuth/readmeimage/开启极路由ssh.png)](https://github.com/FADEDTUMI/CQUST-CampusNet-Anti-Multi-Device-Detection-AutoAuth/blob/main/readmeimage/%E5%BC%80%E5%90%AF%E6%9E%81%E8%B7%AF%E7%94%B1ssh.png)
  - ![图片2](https://github.com/FADEDTUMI/CQUST-CampusNet-Anti-Multi-Device-Detection-AutoAuth/blob/main/readmeimage/securecrt%E6%88%AA%E5%9B%BE.png)

### 3. 备份路由器固件及相关数据

- **目标**：我们开启SSH的目的是给此路由器刷上第三方固件，因此需要先备份路由器的MAC地址以及原始固件。
- **备份MAC地址**：
  ```css
  ifconfig -a
  ```
  将打印出来的所有内容全部备份在txt中（此处不贴图，较为简单）。
- **备份原始固件**：
  1. 在终端中输入命令：
     ```bash
     cat /proc/mtd
     ```
     可看到我们需要备份的内容。
  2. 创建备份文件夹（路径在/tmp下），具体命令如下：
     ```bash
     cd /tmp
     mkdir firmwarebackup
     cd firmwarebackup
     ```
  3. 在此文件夹内开始备份，具体命令如下：
     ```bash
     dd if=/dev/mtd0 of=/tmp/firmwarebackup/u-boot.bin
     dd if=/dev/mtd1 of=/tmp/firmwarebackup/debug.bin
     dd if=/dev/mtd2 of=/tmp/firmwarebackup/Factory.bin
     dd if=/dev/mtd3 of=/tmp/firmwarebackup/firmware.bin
     dd if=/dev/mtd4 of=/tmp/firmwarebackup/kernel.bin
     dd if=/dev/mtd5 of=/tmp/firmwarebackup/rootfs.bin
     dd if=/dev/mtd6 of=/tmp/firmwarebackup/hw_panic.bin
     dd if=/dev/mtd7 of=/tmp/firmwarebackup/bdinfo.bin
     dd if=/dev/mtd8 of=/tmp/firmwarebackup/backup.bin
     dd if=/dev/mtd9 of=/tmp/firmwarebackup/overlay.bin
     dd if=/dev/mtd10 of=/tmp/firmwarebackup/firmware_backup.bin
     dd if=/dev/mtd11 of=/tmp/firmwarebackup/oem.bin
     dd if=/dev/mtd12 of=/tmp/firmwarebackup/opt.bin
     ```
  4. 检查备份是否成功：
     在Windows中打开PowerShell或者命令行，输入命令：
     ```ruby
     scp -r -o HostKeyAlgorithms=+ssh-rsa root@192.168.199.1:/tmp/firmwarebackup ./
     ```
     - **注意**：由于新版取消了ssh-rsa，而极路由4仅支持较旧的ssh-rsa主机密钥类型，我们可以采取临时启用ssh-rsa，安全的同时且仅单次命令有效。
  - **开启SSH的程序源于**：
    [https://www.right.com.cn/FORUM/thread-8268438-1-1.html](https://www.right.com.cn/FORUM/thread-8268438-1-1.html)
  - **图片参考**：
  - ![image](https://github.com/FADEDTUMI/CQUST-CampusNet-Anti-Multi-Device-Detection-AutoAuth/blob/main/readmeimage/breed%E6%B5%81%E7%A8%8B1.png)
  - ![image](https://github.com/FADEDTUMI/CQUST-CampusNet-Anti-Multi-Device-Detection-AutoAuth/blob/main/readmeimage/breed%E6%B5%81%E7%A8%8B2.png)
  - ![image](https://github.com/FADEDTUMI/CQUST-CampusNet-Anti-Multi-Device-Detection-AutoAuth/blob/main/readmeimage/breed%E6%B5%81%E7%A8%8B3.png)
  - ![image](https://github.com/FADEDTUMI/CQUST-CampusNet-Anti-Multi-Device-Detection-AutoAuth/blob/main/readmeimage/breed%E6%B5%81%E7%A8%8B4.png)
  - ![image](https://github.com/FADEDTUMI/CQUST-CampusNet-Anti-Multi-Device-Detection-AutoAuth/blob/main/readmeimage/breed%E6%B5%81%E7%A8%8B5.png)

## 二、防多设备检测教程

### 1. 定制固件及所需软件包

- **访问网址**：[https://openwrt.ai/](https://openwrt.ai/)
- **软件包参考**：
  ```rust
  luci-app-autotimeset luci-app-ttyd luci-theme-argon luci-app-broadbandacc kmod-rkp-ipid iptables-mod-filter iptables-mod-ipopt iptables-mod-u32 iptables-nft kmod-ipt-ipopt iptables-mod-conntrack-extra
  ```
- **自行更新openwrt固件**
### 2. 安装ShellClash

- **命令**：
  ```bash
  export url='https://fastly.jsdelivr.net/gh/juewuy/ShellCrash@master' && sh -c "$(curl -kfsSl $url/install.sh)" && source /etc/profile &> /dev/null
  ```
- **此处结合网上大多教程填充细节，例如crash作者的github[https://github.com/juewuy/ShellCrash/releases]**。
- crash添加#用于UA3F的Clash配置（无外部代理）
  - **配置文件地址**：
    [https://cdn.jsdelivr.net/gh/SunBK201/UA3F@master/clash/ua3f-cn.yaml](https://cdn.jsdelivr.net/gh/SunBK201/UA3F@master/clash/ua3f-cn.yaml)

### 3. 从URL安装UA3F

- **命令**：
  ```bash
  opkg update
  opkg install curl libcurl luci-compat
  export url='https://blog.sunbk201.site/cdn' && sh -c "$(curl -kfsSl $url/install.sh)"
  service ua3f reload
  ```
- **注意**：此处安装UA3F的命令，opkg或许可能出现failed download错误，请更新openwrt源！可更换国内源：
  ```swift
  src/gz openwrt_core https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/22.03.4/targets/x86/64/packages
  src/gz openwrt_base https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/22.03.4/packages/x86_64/base
  src/gz openwrt_luci https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/22.03.4/packages/x86_64/luci
  src/gz openwrt_packages https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/22.03.4/packages/x86_64/packages
  src/gz openwrt_routing https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/22.03.4/packages/x86_64/routing
  src/gz openwrt_telephony https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/22.03.4/packages/x86_64/telephony
  ```
  - **具体命令参考**：
    ```bash
    vi /etc/opkg/distfeeds.conf
    ```

### 4. 配置启动项

- **脚本**：
  ```nginx
  # 启动 UA3F
  uci set ua3f.enabled.enabled=1
  uci commit ua3f
  service ua3f enable
  service ua3f start

  # 防火墙规则：
  iptables -t nat -A PREROUTING -p udp --dport 53 -j REDIRECT --to-ports 53
  iptables -t nat -A PREROUTING -p tcp --dport 53 -j REDIRECT --to-ports 53

  # 防 IPID 检测
  iptables -t mangle -N IPID_MOD
  iptables -t mangle -A FORWARD -j IPID_MOD
  iptables -t mangle -A OUTPUT -j IPID_MOD
  iptables -t mangle -A IPID_MOD -d 0.0.0.0/8 -j RETURN
  iptables -t mangle -A IPID_MOD -d 127.0.0.0/8 -j RETURN
  # 由于本校局域网是 B 类网，所以我将这一条注释掉了，具体要不要注释结合你所在的校园网内网类型
  iptables -t mangle -A IPID_MOD -d 10.0.0.0/8 -j RETURN
  #iptables -t mangle -A IPID_MOD -d 172.16.0.0/12 -j RETURN
  iptables -t mangle -A IPID_MOD -d 192.168.0.0/16 -j RETURN
  iptables -t mangle -A IPID_MOD -d 255.0.0.0/8 -j RETURN
  iptables -t mangle -A IPID_MOD -j MARK --set-xmark 0x10/0x10

  # 防时钟偏移检测
  iptables -t nat -N ntp_force_local
  iptables -t nat -I PREROUTING -p udp --dport 123 -j ntp_force_local
  iptables -t nat -A ntp_force_local -d 0.0.0.0/8 -j RETURN
  iptables -t nat -A ntp_force_local -d 127.0.0.0/8 -j RETURN
  iptables -t nat -A ntp_force_local -d 192.168.0.0/16 -j RETURN
  iptables -t nat -A ntp_force_local -s 192.168.0.0/16 -j DNAT --to-destination 192.168.1.1

  # 通过 iptables 修改 TTL 值
  iptables -t mangle -A POSTROUTING -j TTL --ttl-set 64

  # iptables 拒绝 AC 进行 Flash 检测
  iptables -I FORWARD -p tcp --sport 80 --tcp-flags ACK ACK -m string --algobm --string " src=\"http://1.1.1." -j DROP
  ```

### 5. 修改添加OpenWrt的NTP时间服务器

- **NTP服务器**：
  ```css
  ntp.aliyun.com
  time1.cloud.tencent.com
  time.ustc.edu.cn
  cn.pool.ntp.org
  ```

### 6. 注意事项及问题解决

- 启动crash服务时可能会出现一堆报错，此时可将防火墙应用改成iptables。
- **参考网址**：[https://github.com/SunBK201/UA3F/issues/7](https://github.com/SunBK201/UA3F/issues/7)

### 7. 测试防检测功能已经运行

- **防UA检测生效**
- ![image](https://github.com/FADEDTUMI/CQUST-CampusNet-Anti-Multi-Device-Detection-AutoAuth/blob/main/readmeimage/%E9%98%B2UA%E6%A3%80%E6%B5%8B%E7%94%9F%E6%95%88.png)
- **防TTL检测生效**
- ![image](https://github.com/FADEDTUMI/CQUST-CampusNet-Anti-Multi-Device-Detection-AutoAuth/blob/main/readmeimage/%E9%98%B2TTL%E7%94%9F%E6%95%88.png)
- **防IPID检测生效**
- ![image](https://github.com/FADEDTUMI/CQUST-CampusNet-Anti-Multi-Device-Detection-AutoAuth/blob/main/readmeimage/%E9%98%B2IPID%E6%A3%80%E6%B5%8B%E5%B7%B2%E7%94%9F%E6%95%88.png)


## 三、重庆科技大学校园网自动化认证脚本部署

### 1. 环境说明

- 大部分防多设备检测已经构建完毕，此时可根据自身校园网认证进行自动化脚本部署。
- **本人校园网认证环境**：重庆科技大学校园网，认证方式为锐捷。

### 2. 部署步骤

#### 步骤一：下载所需文件脚本，记录自己的校园网认证网页的wanuserip数据。

- 假设，我的验证地址是：
  ```arduino
  http://1.1.1.1/eportal/index.jsp?wlanuserip=
  ```
  需要将wlanuserip=及后面所有内容都复制保存。

#### 步骤二：更改脚本内容

- 打开脚本文件中的`ruijie_template.sh`，只需修改以下两项：
  - **service**：修改为运营商urlencode编码（不需要填写则需要手动注释）。
  - **queryString**：修改为刚刚复制的wlanuserip=地址。

#### 步骤三：上传脚本到路由器

- 使用winscp软件建立与路由器之间的通信，将脚本放在`/etc/ruijie`目录下并给予足够的权限：755。
- **测试脚本**：
  ```bash
  /etc/ruijie/ruijie_template.sh 登录账号 密码
  ```
- 若出现permission denied，则权限不够，可直接添加执行权限：
  ```bash
  chmod +x /etc/ruijie/ruijie_template.sh
  ```

#### 步骤四：测试认证

- 测试返回内容为success，则说明认证脚本成功运行。
- **添加进OpenWrt启动项**：
  ```bash
  # 联网
  /etc/ruijie/ruijie_template.sh 登录账号 密码
  ```
- 保存并重启路由器，即可享受你的防多设备检测自动认证的网络环境！

## 个人吐槽

在此我要吐槽一下重庆科技大学的校园网速度，50M带宽，敷衍谁呢，当然不是说学校不好哈（害怕.jpg）。

## 文章引用

- 基于GPL开源协议
- 开启SSH的程序源于：[https://www.right.com.cn/FORUM/thread-8268438-1-1.html](https://www.right.com.cn/FORUM/thread-8268438-1-1.html)
- 参考文档：[https://blog.csdn.net/u010102747/article/details/124639593](https://blog.csdn.net/u010102747/article/details/124639593)
- 相关网址：
- [https://github.com/SunBK201/UA3F/issues/7](https://github.com/SunBK201/UA3F/issues/7)
- https://github.com/Zxilly/UA2F
- https://github.com/CHN-beta/rkp-ipid
- https://github.com/jerrykuku/luci-theme-argon
- https://github.com/jerrykuku/luci-app-argon-config
- https://github.com/lucikap/luci-app-ua2f
- https://github.com/linkease/istore
- https://github.com/openwrt/openwrt
- 叶小姐做的视频（超级无敌喜欢外加感谢）：[https://www.bilibili.com/video/BV1yr4meeENt]

