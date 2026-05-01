# 餐饮管理系统 (Dining Management System)

> **项目简介：**
> 这是一个基于 Spring Boot 框架开发的在线外卖订购系统（一体化 SaaS 解决方案）。业务覆盖了用户端的小程序点餐、支付，以及商家端的员工管理、菜品/套餐管理、订单实时派发与财务数据报表等核心流程。
>
> **我的角色：全栈开发工程师**
> 负责该项目的后端架构设计与API开发，以及Web管理端的前端构建和小程序端的接入。主导了从数据库设计、核心业务逻辑实现、前后端通信(RESTful API)、性能优化(Redis缓存)到联调上线的完整生命周期。

---

## 技术栈选型

作为全栈开发，本项目在前后端均采用了现代企业级技术栈：

### 后端架构 (Backend)
- **核心框架：** Spring Boot (3.1.2), Spring MVC (Java 17)
- **持久层框架：** MyBatis
- **数据库：** MySQL 8.x
- **缓存与中间件：** Redis, Spring Cache
- **身份认证：** JWT (JSON Web Token)
- **实时通信：** WebSocket
- **接口文档：** Knife4j (Swagger)
- **其他组件：** 阿里云 OSS (对象存储)、Apache POI (报表导出)、HttpClient (微信支付/登录接口请求)

### 前端 Web 管理端 (Admin Web)
- **核心框架：** Vue.js 2.x
- **语言：** TypeScript & JavaScript
- **UI 组件库：** Element UI
- **请求/路由与状态管理：** Axios, Vue Router, Vuex
- **构建工具：** Webpack & vue-cli

### 用户端小程序 (Mini Program)
- **技术选型：** 微信原生小程序开发 (接入 Uni-app 组件生态)
- **核心能力：** 微信授权登录，微信支付，地理位置定位。

---

## 核心业务与技术亮点

### 1. 完善的权限控制与安全体系
- 通过 **JWT令牌** 技术实现了无状态的登录鉴权，同时结合 **Spring MVC 拦截器** 从底层控制所有未授权的 API 请求。
- 使用 `ThreadLocal` 技术存储当前线程的用户 ID，实现操作人追踪的零侵入式代码开发。

### 2. 高并发点餐场景的性能调优
- 针对用户端高频调用的“查看菜单”接口，整合 **Redis** 进行缓存层设计。
- 结合 **Spring Cache** 框架，实现基于注解的缓存自动更新（清除）机制响应菜品的上下架操作，极大减轻了 MySQL 数据库的读压力。

### 3. 多端实时通信能力 (WebSocket)
- 破除传统的 HTTP 轮询模式，采用 **WebSocket** 建立服务器与 Web 管理端的长连接。
- 当用户在小程序端下单并支付成功后，后端服务会**毫秒级推送信件**到商家前端控制台，实现实时语音播报和最新订单高亮，大大提升了后厨出餐效率。

### 4. 完整的微信生态接入
- 接入 **微信登录 API** (`wx.login`) 获取 `OpenId`，并实现静默注册/登录机制。
- 对接 **微信支付系统**，统筹处理预支付订单创建、支付回调校验验证、支付凭证处理等高难度交易逻辑。

### 5. 可视化数据统计与报表导出
- 后端使用复杂 SQL 配合业务代码完成营业额、订单完成率、新老用户等数据的 T+1 跑批与精细化统计。
- 前端利用 **ECharts** 进行数据直观展示。
- 使用 **Apache POI** 生成标准化 Excel 运营报表并支持业务方一键下载。

---

## 📁 核心目录结构

```text
├── sky-take-out/           # 后端 Spring Boot 核心服务模块 (主导开发)
│   ├── sky-server/         # 核心业务服务器 (Controller/Service/Mapper代码)
│   ├── sky-pojo/           # 实体类、DTO、VO统一管理
│   └── sky-common/         # 通用工具类、异常定义类、全局常量
├── project-dining-admin-vue-ts/    # Web 商家管理后台前端源码 
├── mp-weixin/              # 微信小程序端静态资源与代码
└── ...
```

---

## Windows / Mac 开发环境搭建与部署

### 1. 基础环境准备
- **Java 环境**：安装 Java JDK 17 并配置环境变量。
- **数据库**：安装 MySQL、Redis 数据库并创建相应数据库。
  - 创建 MySQL 数据库与表：运行 `mysql.sql`。
- **构建工具**：安装 Maven 构建工具。

### 2. 下载并导入项目
克隆项目到本地：
```bash
git clone https://github.com/你的用户名/你的仓库名.git
```

### 3. 后端服务配置
进入 `sky-take-out` 后端工程，修改核心配置文件 `application.yml`：
```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/你的数据库名
    username: root
    password: 数据库密码
  data:
    redis:
      password: redis数据库密码
```

在 `resources` 目录下新建/修改 `application-env.yml` 文件，写入微信集成所需配置：
```yaml
sky:
  wechat:
    appid: 申请微信小程序可获得
    secret: 申请微信小程序可获得
    mchid: 商户号
    mchSerialNo: 商户证书序列号
    privateKeyFilePath: 商户私钥路径
    apiV3Key: 
    weChatPayCertFilePath: 
    notifyUrl: 
    refundNotifyUrl: 
```

完成后，运行 `SkyApplication.java` 即可启动服务 (默认端口 :8080)。

### 4. 前后端 Nginx 反向代理配置 
下载安装 Nginx，并在 `http` 这一项下完成以下配置（处理跨域、WebSocket 与静态资源路由）：

```nginx
map $http_upgrade $connection_upgrade{
    default upgrade;
    '' close;
}

upstream webservers{
    server 127.0.0.1:8080 weight=90 ;
    #server 127.0.0.1:8088 weight=10 ;
}

server {
    listen       80;
    server_name  localhost;

    location / {
        root   html/sky;
        index  index.html index.htm;
    }

    # 反向代理，处理管理端发送的请求
    location /api/ {
        proxy_pass   http://localhost:8080/admin/;
        #proxy_pass   http://webservers/admin/;
    }

    # 反向代理，处理用户端发送的请求
    location /user/ {
        proxy_pass   http://webservers/user/;
    }

    # WebSocket 反向代理
    location /ws/ {
        proxy_pass   http://webservers/ws/;
        proxy_http_version 1.1;
        proxy_read_timeout 3600s;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "$connection_upgrade";
    }

    location /media {
        root 配置媒体文件位置; # eg: D:/static (对应上传目录)
        # 注：在 D:/static 目录下创建 media 文件夹
    }
}
```

### 5. Web 管理端运行 (开发前端专用)
_注：如果是为了开发管理端前端，推荐使用 NVM 将 Node 切换为 v16 进行编译运行。_
```bash
nvm use 16
cd project-rjwm-admin-vue-ts
yarn install --ignore-engines
npm run serve
```
