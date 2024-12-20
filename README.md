# 01 描述

使用Docker安装部署一个Chrome浏览器服务，实现浏览器里面使用浏览器<俄罗斯套娃🤣>好作：不使用远程工具中转，直接访问内网的网页服务。

# 02 部署步骤

## 2.1 编辑docker-compose.yaml配置文件

```yaml
services:
  chromium:
    image: lscr.io/linuxserver/chromium:latest
    container_name: chromium
    hostname: chromium
    shm_size: "1gb"
    restart: unless-stopped
    networks:  # 设置容器的网络，service网络已创建好，继续使用service网络。
      - service
    security_opt:
      - seccomp:unconfined # 仅对于 Docker 引擎，如果没有它，Chromium 将在无沙箱测试模式下运行。
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Shanghai  # 设置容器所属时区
      - CHROME_CLI=https://m.hesc.info/ # 默认打开的网址
      - LANGUAGE=zh_CN:zh
      - LANG=zh_CN.UTF-8
      - LANG=C.UTF-8  # 设置本地语言为"C.UTF-8",让KasmVNC支持中文输入
      - TITLE=需要哈气的纸飞机  # 设置页面的标题
    volumes:
      - ./config:/config
      - ./fonts:/usr/share/fonts  # 配置本地中文字体挂载，解决chromium浏览器中文无法显示的问题
      - /var/run/docker.sock:/var/run/docker.sock  # 挂载docker.sock配置文件

    ports:
      - 3010:3000
      - 3011:3001
    devices:
      - /dev/dri/:/dev/dri  # 使用GPU加速


networks:
  service:
    external: true
```

### 2.1.1 Parameters<参数>

> 容器使用在运行时传递的参数进行配置（如上所述）。这些参数由冒号分隔，分别表示 `<external>:<internal>` 。例如， `-p 8080:80` 将暴露容器内部的 `80` 端口，使其可以从容器外部的宿主 IP 地址上的端口 `8080` 访问。

| 参数 | 参数说明                                                                                                                                                                                                                                                                                                                               |
| :----: | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|   `-p 3000:3000`   | Chromium desktop gui.<br />Chromium HTTP桌面 GUI。<br />                                                                                                                                                                                                                                                                                       |
|   `-p 3001:3001`   | HTTPS Chromium desktop gui.<br />HTTPS Chromium 桌面 GUI。<br />                                                                                                                                                                                                                                                                               |
|   `-e PUID=1000`   | for UserID - see below for explanation.<br />对于 UserID - 请见下文解释<br />                                                                                                                                                                                                                                                                  |
|   `-e PGID=1000`   | for GroupID - see below for explanation.<br />对于 GroupID - 请见下文解释<br />                                                                                                                                                                                                                                                                |
|   `-e TZ=Etc/UTC`   | specify a timezone to use, see this [list](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones#List).<br />指定要使用的时间区域，请参阅此列表。<br />                                                                                                                                                                                                                                                          |
|   `-e CHROME_CLI=https://www.linuxserver.io/`   | Specify one or multiple Chromium CLI flags, this string will be passed to the application in full. 指定一个或多个 Chromium CLI 标志，此字符串将完整地传递给应用程序。                                                                                                                                                                  |
|   `-v /config`   | Users home directory in the container, stores local files and settings.<br />容器中用户的主目录，存储本地文件和设置。<br />                                                                                                                                                                                                                    |
|   `-v /usr/share/fonts`   | 设置中文字体目录，让Chromium浏览器支持中文。                                                                                                                                                                                                                                                                                           |
|   `--shm-size=`   | This is needed for any modern website to function like youtube.<br />这是任何现代网站像 YouTube 一样运行所必需的。<br />                                                                                                                                                                                                                       |
|   `--security-opt seccomp=unconfined`   | For Docker Engine only, many modern gui apps need this to function on older hosts as syscalls are unknown to Docker. Chromium runs in no-sandbox test mode without it.<br />仅适用于 Docker Engine，许多现代 GUI 应用程序需要此功能在较旧的宿主机上运行，因为系统调用对 Docker 来说是未知的。没有它，Chromium 将在无沙盒测试模式下运行。<br /> |

### 2.1.1 Optional environment variables<可选环境变量>

|         Variable         | Description                                                                                                                                                           |
| :-------------------------: | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|      CUSTOM\_PORT      | Internal port the container listens on for http if it needs to be swapped from the default 3000.<br />内部端口号，如果需要从默认的 3000 端口更改为 HTTP 监听。<br />          |
| CUSTOM\_HTTPS\_PORT | Internal port the container listens on for https if it needs to be swapped from the default 3001.<br />内部端口，如果需要从默认的 3001 更改为 https，则容器监听此端口。<br /> |
|      CUSTOM\_USER      | HTTP Basic auth username, abc is default.<br />HTTP 基本认证用户名，abc 为默认。<br />                                                                                        |
|         PASSWORD         | HTTP Basic auth password, abc is default. If unset there will be no auth<br />HTTP 基本认证密码，abc 为默认值。如果未设置，则没有认证<br />                                   |
|         SUBFOLDER         | Subfolder for the application if running a subfolder reverse proxy, need both slashes IE `/subfolder/`<br />子文件夹用于运行子文件夹反向代理的应用程序，需要两个斜杠 IE /subfolder/<br />  |
|           TITLE           | The page title displayed on the web browser, default "KasmVNC Client".<br />网页浏览器上显示的页面标题，默认为“KasmVNC 客户端”。<br />                                      |
|        FM\_HOME        | This is the home directory (landing) for the file manager, default "/config".<br />这是文件管理器的家目录（着陆点），默认为"/config"。<br />                                  |
|     START\_DOCKER     | If set to false a container with privilege will not automatically start the DinD Docker setup.<br />如果设置为 false，具有特权的容器将不会自动启动 DinD Docker 设置。<br />   |
|          DRINODE          | If mounting in /dev/dri for [DRI3 GPU Acceleration](https://www.kasmweb.com/kasmvnc/docs/master/gpu_acceleration.html) allows you to specify the device to use IE `/dev/dri/renderD128`<br />如果将 DRI3 GPU 加速的挂载位置设置为/dev/dri，允许您指定要使用的设备 IE `/dev/dri/renderD128`<br />                  |
|     DISABLE\_IPV6     | If set to true or any value this will disable IPv6<br />如果设置为 true 或任何值，这将禁用 IPv6<br />                                                                         |
|        LC\_ALL        | Set the Language for the container to run as IE `fr_FR.UTF-8` `ar_AE.UTF-8`<br />设置容器运行的语言为 IE `fr_FR.UTF-8` `ar_AE.UTF-8`<br />                                                                                        |
|       NO\_DECOR       | If set the application will run without window borders in openbox for use as a PWA.<br />如果设置，应用程序将在 openbox 中以无窗口边框的方式运行，用作 PWA。<br />            |
|        NO\_FULL        | Do not autmatically fullscreen applications when using openbox.<br />不要在使用 openbox 时自动全屏应用程序。<br />                                                            |

### 2.1.2 Optional run configurations<可选运行配置>

| Variable | Description                                                                                                                                                                                                                                                                                                                                                                            |
| :--------: | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|     `--privileged`     | Will start a Docker in Docker (DinD) setup inside the container to use docker in an isolated environment. For increased performance mount the Docker directory inside the container to the host IE `-v /home/user/docker-data:/var/lib/docker`.<br />将在容器内启动 Docker in Docker (DinD)设置，以在隔离环境中使用 Docker。为了提高性能，将 Docker 目录挂载到主机上 IE -v /home/user/docker-data:/var/lib/docker 。<br />                |
|     `-v /var/run/docker.sock:/var/run/docker.sock`     | Mount in the host level Docker socket to either interact with it via CLI or use Docker enabled applications.<br />在主机级别的 Docker 套接字中挂载，以便通过 CLI 与之交互或使用 Docker 支持的应用程序。<br />                                                                                                                                                                                  |
|     `--device /dev/dri:/dev/dri`     | Mount a GPU into the container, this can be used in conjunction with the `DRINODE` environment variable to leverage a host video card for GPU accelerated applications. Only **Open Source** drivers are supported IE (Intel,AMDGPU,Radeon,ATI,Nouveau)<br />将 GPU 安装到容器中，这可以与 `DRINODE` 环境变量结合使用，以利用主机显卡进行 GPU 加速应用程序。仅支持开源驱动程序 IE（英特尔、AMDGPU、Radeon、ATI、Nouveau）<br /> |

## 2.2 使用docker-compose 命令启动服务

```shell
~ ] docker-compose up     # 临时启动chromium容器
~ ] docker-compose up -d  # 后台启动chromium容器
~ ] docker-compose ps -a  # 查看chromium容器运行情况
NAME       IMAGE                                 COMMAND   SERVICE    CREATED       STATUS         PORTS
chromium   lscr.io/linuxserver/chromium:latest   "/init"   chromium   3 hours ago   Up 5 seconds   0.0.0.0:3010->3000/tcp, [::]:3010->3000/tcp, 0.0.0.0:3011->3001/tcp, [::]:3011->3001/tcp
```
