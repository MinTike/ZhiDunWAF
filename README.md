# 智盾WAF v10.2 Ultra

基于 **Python (FastAPI) + HTML/CSS/JS (ECharts + Leaflet)** 的实时 DDoS/WAF 防御与可视化平台。

> **当前版本**: v10.2 Ultra 企业级安全版 | **更新日期**: 2026-09-26

## 📋 版本历史

| 版本          | 日期         | 主要更新                                                                                                                                                                                                  |
| ----------- | ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| v10.2 Ultra | 2026-09-26 | 企业级安全版：20 项安全修复(目录穿越/XSS/封禁失效/密钥轮换/ACL重写/AdminAPI鉴权) + ReDoS 正则治理 + 误报分级处置(XFF/UA/SSRF/LDAP) + 事件循环优化(批量检测线程池/登录页缓存) + 内存无界收口 + pickle 降级删除 + payload 限制落地 + L5 绑回环 + 集群 IP 白名单 + pytest 测试体系 + 版本号统一 |
| v9.8 Ultra  | 2026-09-19 | 黑匣子全量日志(哈希链)、域名真实白名单、五秒盾/附加人机最小检测≥5秒、Windows 服务+开机自启+GUI安装向导、API 版 WAF 升级、网站发布包编译                                                                                                                     |
| v9.7 Ultra  | 2026-09-18 | 附加人机验证/五秒盾端口域名防护、登录接口防护、API 地址豁免、修复 403 循环、系统服务化部署、右下角托盘后台任务                                                                                                                                          |
| v9.6 Ultra  | 2026-09-15 | MHDDoS 全方法专项防御、端口域名管理增强、统一告警通道强化、性能深度优化                                                                                                                                                               |
| v8.0 Ultra  | 2026-09-05 | 全面升级：五秒盾、L5动态端口跳转、AI周期分析、AI审计、AI人机检测、统一告警管理、系统防火墙、DDoS吞吐服务、欺骗防御、端口域名管理、代理防护、访问限制、崩溃保护、短信通知                                                                                                            |
| v7.0 Ultra  | 2026-08-29 | Bot 智能管理系统、API 安全防护、威胁情报系统、Webhook 告警集成、数据库性能优化                                                                                                                                                       |
| v6.0        | -          | AI 智能分析防御 + IP 全量访问记录                                                                                                                                                                                 |
| v5.0        | -          | 性能优化版：检测结果缓存、热点 IP 快速通道、批量检测优化、异步非阻塞优化                                                                                                                                                                |
| v4.0        | -          | 链路追踪 + 行为分析 + 攻击聚类 + 自适应阈值                                                                                                                                                                            |
| v3.0        | -          | 配置持久化 + 欺骗防御 + 性能均衡 + 配置预设                                                                                                                                                                            |
| v2.0        | -          | 安全响应头 + IP 信誉 + 取证增强 + 蜜罐                                                                                                                                                                             |
| v1.0        | -          | 初始版本：基础检测 + 封禁 + 限流 + 挑战                                                                                                                                                                              |

## ✨ 核心特性

### 🆕 v9.8 新增核心能力

#### 📦 黑匣子全量日志（服务器瘫痪后唯一黑匣子）

- **双流落盘**：`data/blackbox/audit.log`（审计事件，同步 fsync 崩溃零丢失）+ `data/blackbox/requests.log`（全量请求，异步批量落盘）
- **SHA-256 哈希链防篡改**：每一行携带 `prev_hash + hash`，任何篡改都会破坏整条链，可事后完整取证
- **自动轮转**：审计 100MB / 请求 500MB 自动轮转，保留最近 5 份备份
- **状态可视化**：`/api/performance/stats` 返回黑匣子统计；控制台「黑匣子」页一键查看

#### 🖥️ 集群管理控制台（CustomTkinter 桌面界面，非 Web）

- 实时显示 RPS / 活跃连接 / 封禁数 / 防御等级 / 攻击状态 / 最近攻击事件
- 集群节点在线/离线、Leader 选举、封禁同步、分区检测
- 一键加入集群、手动同步封禁列表
- 密钥与密码（`admin_token`、`cluster_secret`、API 密钥路径）一键复制
- Windows 下直接启动/停止/重启 `ZhiDunWAF` 服务

#### 🛡️ 端口域名真实白名单防护

- 默认配置**不再放行任意域名**，仅 `localhost / 127.0.0.1` 默认放行，外部业务域名必须显式配置
- 七层检测：端口白名单 → Host 头校验 → 编码绕过 → IP 直访 → 端口扫描 → 子域名枚举 → 域名+路径白名单
- 网关端口自动对齐，消除"网关放行但 WAF 误封"的配置割裂
- 按域名+路径路由到不同后端（`get_backend_for` 已接线）

#### ⏱️ 五秒盾 / 附加人机检测强制 ≥5 秒

- 前端 JS 计时 + 后端 `created_at` 双端校验，检测时长不足 5 秒直接拒绝
- 端口域名防护 / 登录接口防护 / API 地址豁免（可配置）

#### ⚙️ Windows 服务 + 开机自启 + GUI 安装向导

- `ZhiDunWAF` 系统服务（LocalSystem），同时守护网关与 WAF，崩溃 3 秒自动拉起
- 开机自启 + 登录后右下角托盘后台任务
- GUI 四步安装向导（欢迎/选项/进度/完成），一键部署

### 检测引擎 (8 大策略融合)

| 策略                | 检测目标          | 说明                    |
| ----------------- | ------------- | --------------------- |
| 速率检测 (HTTP Flood) | 高频请求          | 滑动窗口单 IP 请求量超阈值即封禁    |
| 限流检测              | 每秒配额          | 单 IP 每秒请求数超配额触发限流     |
| CC 检测             | URL 重复        | 同一 URL 高频访问识别为 CC 攻击  |
| 并发连接检测            | 连接型 Flood     | 单 IP 并发连接数过高即封禁       |
| 慢速攻击检测            | Slowloris     | 请求头读取超时识别慢速攻击         |
| 载荷异常检测            | 超大载荷          | 超大请求体自动拦截             |
| UA 异常检测           | 异常 User-Agent | 空/恶意 UA 特征识别          |
| 规则匹配              | 自定义正则         | SQL 注入/XSS/路径穿越/命令注入等 |

### 缓解策略

- **自动封禁**(带 TTL,到期自动解封)
- **永久封禁**(重复违规 5 次自动升级为永久)
- ️ **速率限流**(令牌桶)
- **人机验证挑战**(JS PoW 工作量证明)
- **白名单放行**

### 五级防御等级

| 等级 | 名称 | 触发条件                           | 行为                     |
| -- | -- | ------------------------------ | ---------------------- |
| L1 | 标准 | 默认                             | 正常检测                   |
| L2 | 加强 | 全局 RPS 超阈值                     | 加强规则匹配                 |
| L3 | 警戒 | 全局 RPS 或唯一 IP 超阈值              | 新 IP 强制挑战              |
| L4 | 高危 | 高 RPS / 大量唯一 IP / 脉冲尖峰 / 地毯式轰炸 | 分布式封禁                  |
| L5 | 锁定 | 极端攻击                           | 仅 授权IP 可管理,业务全拒 + 端口跳转 |

### 五秒盾 (Five Second Shield)

- 首次访问自动跳转至 5 秒验证页面
- JS 计算 + Cookie 验证双重机制
- 浏览器指纹采集与识别
- 自动放行合法浏览器，拦截自动化工具
- 可配置豁免路径与可信 IP
- 与防御等级联动（L3+ 自动激活）

### L5 动态端口跳转

- L5 锁定模式下自动切换至专用端口(默认 60000)
- 原业务端口全部关闭，仅保留管理通道
- 自动/手动升降级时平滑切换，用户无感知
- 授权 IP 白名单机制，紧急情况下可访问
- 端口可配置，支持自定义 L5 端口

### 分布式攻击防御

- 地毯式轰炸检测(低速率多 IP 聚合超阈值)
- 分布式 IP 扫描封禁(L4+ 维持期间每 3 秒持续扫描)
- 全局评估节流(每秒 1 次完整评估,防止性能耗尽)
- 自动跳过白名单/已封禁/长期不活跃 IP

### 链路追踪 (Trace Analyzer)

- XFF 链伪造检测(识别代理层伪造来源 IP)
- 代理/VPN 特征识别(数据中心 IP 检测)
- 客户端指纹关联分析(同一攻击者多 IP 关联)
- 请求头异常模式检测

### 取证中心 (Forensic)

- 攻击者画像(IP 位置/信誉/攻击历史/UA 分布/URL 分布)
- 攻击事件归档(SQLite 持久化)
- 登录审计与蜜罐触发记录
- 安全报告自动生成
- 攻击源地理位置地图(Leaflet 交互式地图,支持 6 种图层切换)
  - 高德街道 / 高德卫星 / Esri街道 / Esri卫星 / Esri深灰 / 本地离线地图

### 安全加固

- **登录三重认证**:API Key + 密码(PBKDF2-HMAC-SHA256) + 数字算术验证码
- **AES-256-GCM 加密**:应用层全链路加密,无需 HTTPS
- **防重放攻击**:每请求携带一次性 Nonce
- **会话绑定 IP**:防止 Token 被盗用
- **蜜罐机制**(L4+ 激活):高度伪装的诱饵路径,触发即永久封禁 + 全量取证
- **安全响应头**:HSTS/CSP/X-Frame-Options 等全自动注入

### 统一安全网关

- 网关作为唯一外部入口(WAF 后端绑定 127.0.0.1)
- TCP 层预拦截已封禁 IP(拒绝 HTTP 握手)
- 反向代理 body 透传(POST/PUT/PATCH)
- XFF 防伪(过滤伪造的 X-Forwarded-For)
- 连接数限制 + 端口复用

### 前端可视化大屏 (v8.0 SOC 控制台)

- 实时流量曲线(ECharts 双 Y 轴:请求/s + 拦截/s)
- 攻击类型分布饼图
- 实时攻击事件流(WebSocket 推送,毫秒级延迟)
- TOP 攻击源 IP 排行
- 封禁 IP 管理(增删解封)
- 自定义规则引擎(8 条预置规则 + 运行时增删改)
- 检测阈值在线调参
- 攻击者画像(含 Leaflet 地图定位)
- 链路追踪面板
- 内置攻击模拟器
- 统一告警抽屉(实时告警通知)
- 深色/浅色主题切换
- 16+ 功能模块 Tab 切换

### AI 智能分析防御

- **多模型支持**:DeepSeek / 阿里云 / 腾讯云 / Kimi / Bigmodel / 火山引擎 / 自定义模型
- **统一接口**:所有模型通过统一适配器调用,切换模型无需修改业务代码
- **异步分析**:不阻塞请求主流程,后台异步执行 AI 研判
- **智能缓存**:相同特征请求复用分析结果,降低 API 调用成本
- **熔断机制**:连续失败自动熔断,避免影响主业务
- **三级分析模式**:
  - `async` 异步模式(默认):仅记录分析结果,不直接干预
  - `sync` 同步模式:高置信度恶意请求直接拦截
  - `monitor` 监控模式:仅记录日志,不执行任何动作
- **自动封禁**:高置信度恶意请求可配置自动封禁(封禁时长加倍)
- **信誉联动**:AI 判定结果影响 IP 信誉分,形成闭环防御

### AI 周期分析 (v8.0)

- **定时自动分析**:可配置周期（小时/天/周）自动执行全量安全分析
- **多维度分析**:攻击趋势、风险评估、薄弱点识别、优化建议
- **报告生成**:自动生成结构化安全分析报告
- **历史对比**:与历史周期数据对比，识别异常变化
- **告警联动**:发现高风险问题自动触发告警通知
- **立即分析**:支持手动触发即时分析任务

### AI 审计 (v8.0)

- **操作行为审计**:所有管理操作全程记录，支持溯源
- **风险操作识别**:AI 识别高风险操作行为
- **异常行为检测**:偏离正常模式的操作自动标记
- **审计日志**:完整的审计日志，支持查询与导出

### AI 人机检测 (v8.0)

- **行为生物特征**:鼠标移动轨迹、键盘输入节奏、滚动模式
- **浏览器指纹**:Canvas 指纹、WebGL 指纹、字体检测
- **环境探测**:无头浏览器检测、自动化工具识别
- **挑战验证**:动态难度的人机验证挑战
- **风险评分**:0-100 人机概率评分

### IP 全量访问记录

- **全协议覆盖**:TCP / UDP 全端口访问记录
- **全 IP 版本**:IPv4 / IPv6 双栈支持
- **持久化存储**:SQLite 数据库持久化,支持 90 天留存
- **高性能写入**:LRU 缓存 + 异步批量写入,不阻塞请求
- **流量统计**:记录入站/出站字节数、访问次数、首次/末次访问时间
- **地理位置**:自动关联 IP 归属地与 ASN 信息
- **风险画像**:关联攻击类型、风险评分、封禁状态
- **标签系统**:支持自定义标签分类管理

### 🤖 Bot 智能管理系统

- **六维检测引擎**:UA 指纹匹配 + 浏览器头检测 + 行为分析 + JavaScript 检测 + robots.txt 遵守 + 挑战通过率
- **Bot 分类管理**:友好爬虫(Google/Bing/Baidu)/恶意爬虫/可疑 Bot 三级分类
- **五种处置策略**:允许 / 限速 / 挑战 / 封禁 / 欺骗
- **行为画像**:每 IP Bot 行为档案，包含访问频率、URL 模式、资源比例等
- **黑白名单**:Bot 专属白名单和黑名单，支持手动管理
- **实时统计**:Bot 访问统计、Top Bot IP 排行、类型分布

### 🔐 API 安全防护

- **API 滥用检测**:IP+API 粒度速率限制、突发流量检测、非工作时间访问、异常方法检测
- **参数安全检测**:SQL 注入 / XSS / 命令注入 / 路径穿越 / 参数污染 / SSRF / JSON 注入
- **递归解码防护**:支持 URL 编码、HTML 实体、Unicode 等多层编码绕过检测
- **越权访问检测**:水平越权 / 垂直越权识别、API 黑白名单、RBAC 基础框架
- **API 自动发现**:自动学习 API 接口、敏感 API 识别、调用统计排行
- **可信 IP 机制**:可信 IP 跳过 API 安全检测

### 📡 威胁情报系统

- **IP 信誉库**:SQLite 持久化 + LRU 内存缓存，信誉分 0-100 动态评分
- **12 种威胁标签**:恶意 IP、代理 IP、Tor 节点、数据中心、扫描器、僵尸网络等
- **多情报源支持**:STIX / OpenIOC / CSV / JSON / Plain 五种格式
- **定时同步**:可配置情报源定时拉取、自动去重合并
- **四维风险评估**:信誉 40% + 历史 30% + 地理 15% + ASN 15%
- **信誉自动衰减**:随时间推移信誉分自动恢复，避免永久标记
- **威胁历史**:完整的威胁事件历史记录，支持溯源分析

### 🔔 统一告警管理 (v8.0)

- **多通道告警**:Webhook(飞书/钉钉/企业微信/Slack) + 短信通知
- **五种告警类型**:攻击告警 / 系统告警 / 配置变更 / 封禁告警 / 自定义告警
- **四级告警级别**:INFO / WARNING / CRITICAL / EMERGENCY
- **智能告警管理**:去重 + 聚合 + 静默 + 频率限制，避免告警风暴
- **消息模板引擎**:各平台原生格式适配（飞书卡片、钉钉 Markdown、Slack Block Kit）
- **签名验证**:飞书/钉钉 HMAC-SHA256 签名机制
- **失败重试**:指数退避重试机制，保障送达率
- **异步发送**:后台线程发送，主流程零阻塞
- **实时告警抽屉**:前端统一告警面板，实时展示

### 📱 短信通知 (v8.0)

- **多服务商支持**:阿里云/腾讯云/华为云短信网关
- **模板化发送**:支持自定义短信模板
- **告警分级**:不同级别告警发送不同优先级
- **频率控制**:防止短信轰炸，每日上限控制
- **发送记录**:完整的短信发送日志与状态追踪

### 🔥 欺骗防御 (Deception)

- **蜜罐路径**:伪装的敏感路径，攻击者一碰就触发
- **伪造响应**:返回高度仿真的错误页面/登录框
- **数据污染**:注入虚假数据，误导攻击者
- **延迟陷阱**:渐进式延迟，拖慢攻击节奏
- **取证增强**:触发欺骗时全量数据包捕获

### 🌐 端口域名管理 (v8.0)

- **多端口监听**:支持同时监听多个业务端口
- **域名路由**:按域名分发到不同后端服务
- **SSL 证书管理**:多域名证书自动匹配
- **端口转发**:灵活的端口映射与转发规则
- **L5 专用端口**:锁定模式下独立管理端口

### 系统防火墙集成 (v8.0)

- **系统级封禁**:封禁 IP 直接写入系统防火墙(iptables/Windows Firewall)
- **自动同步**:WAF 封禁列表与系统防火墙双向同步
- **性能优化**:内核层拦截，比应用层更高效
- **安全加固**:防止攻击者绕过 WAF 直接攻击其他端口

### DDoS 吞吐服务 (v8.0)

- **流量清洗**:高并发下的智能流量清洗
- **请求排队**:过载时请求排队机制，保护后端
- **优先级调度**:重要业务优先处理
- **熔断保护**:后端异常时自动熔断
- **降级策略**:多级降级策略，保障核心可用

### 访问限制页面 (v8.0)

- **自定义拦截页**:可定制的拦截/挑战页面模板
- **多国语言**:支持多语言拦截页面
- **品牌定制**:Logo/配色/文案完全自定义
- **申诉通道**:集成 IP 申诉功能
- **倒计时显示**:封禁剩余时间可视化

### 🛡️ 崩溃保护 (Crash Guard)

- **进程守护**:主进程异常退出自动重启
- **内存监控**:内存占用过高自动告警与回收
- **磁盘监控**:磁盘空间不足自动告警
- **健康检查**:定期自检，异常自动恢复
- **崩溃日志**:完整的崩溃现场信息，便于排查

### ⚡ 性能优化

- **复合索引优化**:新增 5 个数据库复合索引，查询性能提升 3-5 倍
- **SQLite 调优**:WAL 模式 + 64MB 缓存 + 256MB MMAP + 内存临时表
- **VACUUM 整理**:定期数据库碎片整理，减少磁盘占用
- **ANALYZE 统计**:自动维护查询优化器统计信息
- **模块懒加载**:新模块按需延迟导入，启动速度提升

### MHDDoS 专项防御

<https://github.com/MatrixTM/MHDDoS>

- **Layer7 全方法检测**:覆盖 MHDDoS 全部 25 种 HTTP 攻击方法(GET/POST/CFB/BYPASS/NULL/COOKIE/PPS/APACHE/XMLRPC/RHEX/STOMP/BOT/DYN/SLOW/AVB/TOR/BOMB/DGB/DOWNLOADER 等)
- **Layer4 连接型防御**:TCP/UDP/SYN/CPS/CONNECTION 攻击检测与限流
- **CPS 速率限制**:单 IP 每秒新建连接数限制,超阈值自动封禁
- **并发连接限制**:单 IP 并发连接数上限控制
- **全局连接限制**:总并发连接数保护,防止资源耗尽
- **数据中心 IP 识别**:识别代理/IDC 发起的批量攻击
- **UA 指纹提取**:MHDDoS 特有 User-Agent 特征库匹配
- **请求一致性检测**:识别 MHDDoS 攻击流量的节律特征

### IP 地理位置定位

- 多在线 API 并行查询(ipinfo.io + pconline + ip-api)
- 内置城市坐标表(100+ 中国城市 + 国际主要城市)
- 坐标三级补全策略(API → 城市表 → 省份表)
- 内存缓存 + 磁盘持久化
- 后台异步查询(不阻塞请求)
- Unicode 规范化(解决特殊字符城市名翻译问题)

## 🚀 快速开始

### 1. 安装依赖

```bat
pip install -r requirements.txt
```

### 2. 启动服务

```bat
python run.py
```

> ★ v10.2: 启动 WAF 后会自动打开「集群管理控制台」桌面界面(可用 `--no-console` 关闭)。  
> 指定端口 / 附带托盘:

```bat
python run.py --port 9000 --no-console   :: 不弹桌面控制台(服务环境使用)
python run.py --tray                     :: 额外拉起系统托盘后台任务
```

### 3. 启动网关 (可选,推荐生产环境使用)

```bat
python gateway.py
```

### 4. 服务器部署

```bat
python deploy.py --ip 你的服务器IP
```

### 5. 编译宣传/演示/blog 网站发布包 (v9.8)

```bat
python deploy.py --build-websites
```

- 自动将 `website/` 下的 宣传站(index.html)、演示控制台(waf_console.html)、  
  API文档(api-docs.html)、技术博客(blog.html) 编译为单文件 HTML + 完整资源 ZIP，  
  输出到 `.trae-html-share-packages/website/`。
- 宣传站独立部署:

```bat
python website_deploy.py --ip 你的服务器IP --port 7000
python website\demo_server.py   # 启动演示站
```

### 6. 访问控制台

浏览器打开 → <http://127.0.0.1:8000/dashboard>

默认账号:

- 用户名: `admin`
- 密码: `admin123`
- API Key: 首次启动自动生成

### 7. 测试防御效果

- 进入「攻击模拟」Tab,选择攻击类型与数量,点击「发起模拟攻击」
- 或使用「快速测试」按钮,直接对被保护接口发起请求
- 切换到「实时监控」查看流量曲线、事件流与封禁列表
- 切换到「取证中心」查看攻击者画像与地理位置地图

## 📁 项目结构

```
DDos_WAF/
├── backend/                   # 后端 Python
│   ├── __init__.py
│   ├── config.py             # 全局配置(可运行时调参)
│   ├── models.py             # Pydantic 数据模型
│   ├── traffic_store.py      # 滑动窗口流量存储 + 时序数据
│   ├── mitigator.py          # 缓解引擎(封禁/限流/挑战)
│   ├── rule_engine.py        # 自定义规则引擎 + 预置规则库
│   ├── detector.py           # 8 策略融合检测引擎 + 分布式攻击检测
│   ├── defense_levels.py     # 五级防御等级定义
│   ├── trace_analyzer.py     # 链路追踪模块
│   ├── geo_locator.py        # IP 地理位置定位(多 API + 坐标补全)
│   ├── forensic_logger.py    # 取证日志(SQLite 持久化)
│   ├── honeypot.py           # 蜜罐诱饵机制
│   ├── challenge.py          # JS PoW 人机验证挑战
│   ├── five_second_shield.py # 五秒盾(v8.0 新增)
│   ├── auth.py               # 三重认证 + AES-256-GCM 加密
│   ├── state_persistence.py  # 状态持久化(加密存储)
│   ├── shared_state.py       # 多进程共享状态同步
│   ├── websocket_manager.py  # WebSocket 实时推送
│   ├── waf_middleware.py     # WAF 中间件(拦截所有请求)
│   ├── api_routes.py         # REST API 路由
│   ├── api_routes_v8.py      # v8.0 API 路由扩展
│   ├── main.py               # FastAPI 入口 + 后台任务
│   ├── ai_analyzer.py        # AI 智能分析引擎(多模型支持)
│   ├── ai_periodic_analyzer.py # AI 周期分析(v8.0 新增)
│   ├── ai_audit.py           # AI 审计(v8.0 新增)
│   ├── ai_human_detect.py    # AI 人机检测(v8.0 新增)
│   ├── ip_access_logger.py   # IP 全量访问记录器
│   ├── bot_manager.py        # Bot 智能管理系统
│   ├── api_security.py       # API 安全防护
│   ├── threat_intel.py       # 威胁情报系统
│   ├── alert_manager.py      # 统一告警管理(v8.0 新增)
│   ├── webhook_notifier.py   # Webhook 告警集成
│   ├── sms_notifier.py       # 短信通知(v8.0 新增)
│   ├── deception.py          # 欺骗防御(v8.0 新增)
│   ├── port_domain_manager.py # 端口域名管理(v8.0 新增)
│   ├── proxy_config_manager.py # 代理配置管理
│   ├── system_firewall.py    # 系统防火墙集成(v8.0 新增)
│   ├── ddos_absorber.py      # DDoS 吞吐服务(v8.0 新增)
│   ├── access_restriction.py # 访问限制页面(v8.0 新增)
│   ├── crash_guard.py        # 崩溃保护(v8.0 新增)
│   ├── perf_monitor.py       # 性能监控
│   ├── security_logger.py    # 安全日志
│   ├── url_validator.py      # URL 验证器
│   └── reverse_proxy.py      # 反向代理
├── frontend/                  # 前端
│   ├── index.html            # SOC 控制台主页(v8.0)
│   ├── login.html            # 登录页
│   ├── favicon.svg
│   └── static/
│       ├── css/style.css     # 深色科技风样式
│       ├── js/dashboard.js   # 基础前端逻辑
│       ├── js/v8_dashboard.js # v8.0 控制台主逻辑
│       ├── js/forensic.js    # 取证中心前端逻辑
│       ├── js/login.js       # 登录页逻辑
│       ├── js/echarts.min.js # ECharts 图表库
│       ├── json/world-map.json # 世界地图数据
│       └── lib/leaflet/      # Leaflet 地图库(本地托管)
├── website/                   # 宣传网站
│   ├── index.html            # 官网首页
│   ├── waf_console.html      # 控制台演示页
│   ├── api-docs.html         # API 文档
│   ├── css/                  # 样式文件
│   ├── js/                   # 脚本文件
│   ├── img/                  # 图片资源
│   └── fonts/                # 字体文件
├── waf_api_version/           # API 版 WAF(独立部署, 纯检测模式, v9.8)
│   ├── main.py               # FastAPI 入口
│   ├── waf_engine.py         # 检测引擎(42种攻击类型)
│   ├── api_routes.py         # 检测/规则/密钥/审计/黑匣子/集群 API
│   ├── blackbox.py           # 黑匣子全量日志(哈希链)
│   ├── alert_manager.py      # 统一告警(Webhook)
│   ├── audit.py              # AI 审计引擎
│   ├── auth.py               # Nonce + AES-256-GCM 认证
│   ├── config.py             # 配置管理
│   ├── key_manager.py        # 授权密钥管理
│   ├── mitigator.py          # 封禁管理器(参考)
│   ├── traffic_store.py      # 流量统计
│   ├── run.py                # 启动器
│   ├── static/               # 检测测试面板
│   └── waf_api_config.json   # 运行时配置
├── console/                   # 桌面控制台与托盘
│   ├── waf_console.py        # CustomTkinter 集群管理控制台(总览/集群/密钥/黑匣子/服务)
│   ├── anim.py               # 动效引擎：缓动/颜色插值/补间/帧调度(与Tk解耦可单测)
│   ├── widgets.py            # 动效控件：滑动指示条/视图转场/Toast/数字滚动/呼吸灯
│   ├── waf_tray.py           # 系统托盘后台任务(pystray)
│   ├── start_console.bat     # 一键启动控制台
│   └── start_tray.bat        # 一键启动托盘(加 --debug 保留窗口)
├── install/                   # 系统服务部署
│   ├── win_service.py        # Windows 服务宿主(守护 gateway+run)
│   ├── setup_all.bat         # 一键部署脚本
│   ├── install_windows_service.bat / remove_windows_service.bat
│   ├── enable_console_autostart.bat / disable_console_autostart.bat
│   └── install_linux_service.sh + waf.service.template
├── installer/                 # 安装向导
│   ├── 启动安装向导.bat       # GUI 安装向导入口(四步,自动申请管理员权限)
│   ├── waf_setup_wizard.ps1   # GUI 安装向导主程序(WinForms 四步)
│   ├── waf_deploy.ps1         # 部署引擎(依赖/服务/托盘/防火墙)
│   ├── waf_setup.iss          # Inno Setup 源脚本(编译 exe 安装包)
│   └── build_setup.bat        # 编译 exe 安装包(需 Inno Setup 6)
├── MHDDoS-main/              # MHDDoS 攻击工具(测试用)
├── gateway.py                 # 统一安全网关(TCP 层防御 + CPS 限流)
├── deploy.py                  # 服务器部署配置工具(含 --build-websites)
├── website_deploy.py          # 宣传站部署工具
├── run.py                     # 启动脚本
├── requirements.txt
└── README.md
```


## 🔌 API 一览

| 方法 | 路径 | 说明 |
|------|------|------|
| GET  | `/api/stats` | 实时统计快照 |
| GET  | `/api/traffic_series` | 流量时序数据 |
| GET  | `/api/attacks` | 最近攻击事件 |
| GET  | `/api/blocked` | 已封禁 IP 列表 |
| POST | `/api/block` | 手动封禁 IP |
| DELETE | `/api/block/{ip}` | 解封 IP |
| DELETE | `/api/block_all` | 清空封禁 |
| GET/POST/PUT/DELETE | `/api/rules` | 规则 CRUD |
| GET/PUT | `/api/config` | 配置读写 |
| GET/POST/DELETE | `/api/whitelist` | 白名单管理 |
| POST | `/api/simulate` | 攻击模拟 |
| GET | `/api/geo_attackers` | 攻击源 IP 地理分布 |
| GET | `/api/trace/stats` | 链路追踪统计 |
| GET | `/api/forensic/profile/{ip}` | 攻击者完整画像 |
| GET | `/api/forensic/attacks` | 攻击事件查询 |
| GET | `/api/forensic/logins` | 登录审计记录 |
| GET | `/api/forensic/honeypot` | 蜜罐触发记录 |
| GET | `/api/forensic/blocks` | 封禁历史 |
| GET | `/api/forensic/audit` | 操作审计日志 |
| GET | `/api/forensic/security_report` | 安全报告 |
| GET  | `/api/reputation/{ip}` | IP 信誉查询 |
| PUT | `/api/reputation/{ip}` | IP 信誉修改 |
| PUT | `/api/geo_block` | 地理封禁配置 |
| GET/POST/PUT/DELETE | `/api/ai/models` | AI 模型配置管理 |
| GET | `/api/ai/stats` | AI 分析统计 |
| POST | `/api/ai/analyze` | 触发 AI 分析 |
| GET | `/api/ai/periodic/status` | AI 周期分析状态(v8.0) |
| POST | `/api/ai/periodic/trigger` | 立即触发周期分析(v8.0) |
| GET | `/api/ai/periodic/reports` | 周期分析报告列表(v8.0) |
| GET | `/api/ip-access/recent` | 最近访问 IP 列表 |
| GET | `/api/ip-access/{ip}` | 单个 IP 详细信息 |
| GET | `/api/ip-access/stats` | IP 访问统计 |
| GET | `/api/bot/stats` | Bot 管理统计 |
| GET/POST/PUT/DELETE | `/api/bot/rules` | Bot 规则管理 |
| GET | `/api/api-security/stats` | API 安全统计 |
| GET | `/api/threat-intel/feed` | 威胁情报源管理 |
| GET | `/api/alerts` | 告警列表(v8.0) |
| POST | `/api/alerts/read` | 标记告警已读(v8.0) |
| GET | `/api/alerts/unread_count` | 未读告警数量(v8.0) |
| GET/POST/PUT/DELETE | `/api/v8/port-domain` | 端口域名管理(v8.0) |
| GET/POST | `/api/v8/five-second-shield` | 五秒盾配置(v8.0) |
| GET/POST | `/api/v8/defense/level` | 防御等级手动调整(v8.0) |
| GET | `/api/v8/system/health` | 系统健康状态(v8.0) |
| GET | `/api/cluster/status` | 集群节点状态(v9.8) |
| POST | `/api/cluster/join` | 手动加入集群(v9.8) |
| POST | `/api/cluster/sync-blocks` | 手动同步封禁(v9.8) |
| GET | `/api/performance/stats` | 性能统计 + 黑匣子状态(v9.8) |
| GET | `/api/ha/status` | 模块级高可用状态(v9.8) |
| POST | `/api/ha/modules/{module}/plan` | 手动切换模块应急方案(v9.8) |
| POST | `/api/ha/modules/{module}/reset` | 恢复模块主方案(v9.8) |
| WS   | `/ws` | 实时事件推送 |
| GET  | `/docs` | Swagger API 文档 |

## 🤖 API 版 WAF（独立部署，纯检测模式，v9.8）

`waf_api_version/` 是智盾WAF v9.8 的 API 化版本，**纯检测模式**：用户服务器拦截 HTTP 请求 → 调用本 API → WAF 全面检测 → 返回结果 → 用户自行处置。

```bash
# 安装依赖
pip install -r waf_api_version/requirements.txt

# 启动(默认 0.0.0.0:9100)
python -m waf_api_version.run
```

```python
import requests, time, secrets
headers = {
    "X-API-Key": "你的API密钥",
    "X-Nonce": secrets.token_hex(16),
    "X-Timestamp": str(int(time.time())),
    "Content-Type": "application/json",
}
resp = requests.post("http://your-server:9100/api/detect", headers=headers, json={
    "method": "GET", "path": "/index.php",
    "query": "id=1' UNION SELECT 1,2,3--",
    "headers": {"User-Agent": "Mozilla/5.0"},
    "body": "", "client_ip": "192.168.1.100",
})
if resp.json()["data"]["blocked"]:
    print("阻断!", resp.json()["data"]["rule_name"])
```

**v9.8 新增能力**：
- **黑匣子全量日志**：`data/blackbox/audit.log`（同步 fsync 零丢失）+ `requests.log`（全量请求），SHA-256 哈希链防篡改
- **统一告警**：`/api/detect` 命中高危时自动推送 Webhook（企业微信/钉钉/飞书）
- **集群状态端点**：`GET /api/cluster/status` 查看节点/黑匣子/防御配置
- **黑匣子统计端点**：`GET /api/blackbox/stats`
- **API 端点**：`/api/detect`、`/api/detect/batch`、`/api/audit/*`、`/api/keys/*`、`/api/rules/*`、`/api/admin/key/*`

## 🛡️ 生产部署建议

1. **HTTPS 强制**:本系统配置为 HTTP,生产环境必须前置 Nginx/Caddy 启用 TLS,所有数据传输加密
2. **多节点共享**:将 `traffic_store` 与 `mitigator` 的内存存储替换为 Redis,实现多 WAF 节点状态共享
3. **包级检测**:当前为 HTTP 层 WAF,SYN/UDP Flood 需在内核或网关层(eBPF/XDP/硬件抗 D 设备)处置
4. **管理面隔离**:生产部署应将 `/api/*` 与 `/dashboard` 置于独立端口 + 鉴权,避免被 WAF 规则误伤
5. **真实 IP 透传**:反向代理需正确设置 `X-Forwarded-For` / `X-Real-IP`,本系统已支持解析
6. **系统防火墙**:启用系统防火墙集成,在网络层拦截恶意 IP,提升防护效率
7. **L5 预案**:提前配置好 L5 锁定端口与授权 IP 白名单,应对极端攻击场景

---

## ⚙️ 系统服务化部署 (v9.8)

WAF 可以脱离控制台终端，以系统服务方式在后台运行，实现开机自启与崩溃自动恢复。

### Windows(系统服务,内核级后台)

```bat
:: ★★★ GUI 安装向导(推荐): 欢迎/选项/进度/完成 四步向导界面
install\启动安装向导.bat

:: ★ 一键部署脚本: 自动安装依赖/customtkinter/pystray/pillow、
::   注册服务、开机自启、启动服务、托盘自启
install\setup_all.bat

:: 仅安装服务(系统Python需已装项目依赖)
install\install_windows_service.bat

:: 卸载
install\remove_windows_service.bat

:: 编译专业 exe 安装包(需安装 Inno Setup 6)
installer\build_setup.bat
```

> 要求: 系统级 Python(非 venv)需已安装项目依赖 `uvicorn/fastapi/httpx`；
> 安装脚本会自动为该系统 Python 安装 pywin32 并注册服务。
> 服务进程使用系统 Python 作为子进程解释器(venv 的 pythonservice.exe 无法在系统账户下正常初始化)。

服务名 `ZhiDunWAF`，同时守护 `gateway.py` 与 `run.py`，任一崩溃 3 秒内自动拉起。

> ★ v9.8: 服务进程运行在后台(session 0,无桌面)，不会弹出界面。
> 安装脚本已同时开启「登录后右下角托盘后台任务」—— 登录 Windows 后系统托盘出现盾牌图标，
> 实时显示 WAF 服务状态，右键可打开控制台/启停服务/重启服务。
> 手动启用/取消: `install\enable_console_autostart.bat` / `install\disable_console_autostart.bat`

### Linux(systemd 自启动)

```bash
# 安装并开机自启(需要 root)
sudo bash install/install_linux_service.sh

# 查看状态/日志
systemctl status waf.service
journalctl -u waf.service -f
```

### 🔔 系统托盘后台任务(右下角任务图标)

登录后自动在右下角系统托盘显示盾牌图标，作为常驻后台任务：

```bat
pythonw console\waf_tray.py          :: 启动托盘程序(无窗口)
python console\waf_tray.py           :: 带窗口调试运行
console\start_tray.bat               :: 一键启动(自动选用 venv 解释器)
console\start_tray.bat --debug       :: 一键启动并保留控制台窗口
python -m console.waf_tray --status  :: 打印一次状态后退出
```

- **悬浮提示**: 实时显示 `ZhiDunWAF` 服务状态(运行中/已停止/未安装) + RPS / 防御等级 / 集群在线数
- **图标配色**: 绿=运行中、红=已停止、黄=启停中、灰=未安装；正在被打时叠加红色警示点
- **状态变更通知**: 服务启停、攻击状态变化、集群网络分区时弹出系统通知
- **右键菜单**:
  - 打开管理控制台(默认，双击亦打开)
  - 启动WAF服务 / 停止WAF服务 / 重启WAF服务
  - 打开黑匣子目录 / 打开项目目录
  - 退出
- **开机自启**: 注册表 `HKCU\...\Run\ZhiDunWAFTray`（安装脚本自动配置）

依赖: `pip install pystray pillow`（安装脚本已自动安装）

### 🖥️ 集群管理控制台(CustomTkinter 自定义桌面界面,非Web)

替代控制台终端的图形化管理入口，基于 **customtkinter(custom 库)** 实现现代深色主题界面，
实时显示 WAF 状态、集群节点、密钥密码、黑匣子日志：

```bat
python console\waf_console.py          :: 启动图形控制台
python console\waf_console.py --no-anim :: 关闭界面动效（低性能机器/远程桌面）
python -m console.waf_console --status :: 无图形界面时命令行查看状态
```

依赖: `pip install customtkinter`（控制台还需 Python 自带 tkinter）。

> ★ v10.2: 控制台支持**最小化到托盘** —— 点击关闭按钮不会退出程序，
> 而是隐藏到右下角托盘图标；双击托盘图标恢复窗口，右键可退出。
> （托盘后台任务由 `console\waf_tray.py` 提供，两者可共存。）

**界面动效**（v10.2，`console\anim.py` 引擎 + `console\widgets.py` 控件层）：

- **侧边栏滑动指示条**：切换视图时高亮条平滑滑过去，不再瞬移
- **视图转场**：新视图滑入 + 幕布向右擦除揭开（Tk 无逐控件 alpha，用颜色擦除实现）
- **指标数字滚动**：RPS/连接数/封禁数变化时滚动到新值，而不是硬跳
- **卡片悬停/按压反馈**：指标卡悬停提亮，按钮按压有高亮闪光
- **呼吸状态灯**：顶栏连接状态 + 服务控制页，运行=绿灯慢呼吸 / 遭受攻击=红灯快呼吸 / 离线=静止
- **Toast 浮动通知**：操作结果在右下角滑入停留后滑出，底栏同步提示
- **表格逐行揭示**：切进视图时表格逐行出现；3 秒轮询刷新是静默的，不会闪
- **入场辉光**：总览页六张指标卡依次点亮

动效可以关掉（三层优先级：命令行 > 环境变量 > 界面开关）：

| 方式 | 做法 |
|---|---|
| 界面开关 | 左下角「界面动效」开关，关掉后立即落终态 |
| 命令行 | `--no-anim` 全关 / `--reduced-motion` 只保留状态切换 |
| 环境变量 | `WAF_CONSOLE_NO_ANIM=1` / `WAF_CONSOLE_REDUCED_MOTION=1`（适合远程桌面/瘦客户端） |

能力边界：
- 所有网络请求在后台线程执行，主线程只负责渲染（Tkinter 非线程安全）
- 所有动效合并到**单条** `after()` 帧链路上（`anim.Ticker`），不会 N 个控件各起一条链拖垮主线程
- 后端不可达时降级为「离线」展示而非崩溃 —— 控制台最常见的使用场景恰恰是 WAF 挂了之后去看状态
- 敏感凭据默认脱敏，显式点击「显示」才可见；复制只写剪贴板，不落盘

- **状态总览**:RPS / 活跃连接 / 封禁数 / 防御等级 / 攻击状态 / 最近攻击事件
- **集群管理**:节点在线离线 / Leader / 同步计数 / 一键加入集群 / 手动同步封禁
- **密钥与密码**:`admin_token`、`cluster_secret`、API 密钥路径(一键复制)
- **黑匣子**:审计/请求日志大小、队列积压、打开日志目录
- **服务控制**:Windows 下可直接启动/停止/重启 `ZhiDunWAF` 服务

### 📦 黑匣子全量日志

服务器瘫痪后唯一的取证数据源，位于 `data/blackbox/`：

- `audit.log`:关键操作(封禁/解封/拦截/配置变更/登录/启停)，同步 fsync 落盘，崩溃不丢
- `requests.log`:全量请求明细，异步批量落盘，高吞吐
- 哈希链防篡改:每行 `prev_hash + hash`，任意篡改破坏整条链
- 状态接口:`/api/performance/stats` 返回 `blackbox` 字段

### 🧬 模块级高可用集群服务 (v9.8 Ultra)

为 WAF **所有核心模块**提供容灾安全方案。面对大规模攻击时，某个模块一旦故障/被击穿，
立即自动切换到备用方案，每个模块 **至少 3 个应急方案 + 1 个 fail-safe 兜底**：

- **23 个核心模块全覆盖**:检测引擎 / 缓解引擎(block、is_blocked) / 流量存储 / 五秒盾 /
  附加人机验证 / 端口域名管理 / 黑匣子 / 统一告警 / 取证 / 威胁情报 / Bot 管理 /
  API 安全 / IP 访问记录 / 地理位置 / 蜜罐 / 欺骗防御 / 系统防火墙 / DDoS 吞吐 /
  AI 审计 / 链路追踪 / WebSocket 推送 / Webhook 通知器
- **自动降级链**:`primary → fallback1 → fallback2 → fallback3 → fail_safe`
  - 同步热路径:主方案抛异常立即降级并重试下一方案(零延迟)
  - 异步健康监控:每 5 秒探测模块健康，连续失败 3 次自动降级
  - 自动恢复:健康恢复且冷却期(30s)过后逐级回滚到主方案
- **应急方案设计约定**:检测类模块 fail-closed(宁拦勿放)，验证类模块 fail-open(避免锁死)
- **黑匣子联动**:每次降级/恢复/手动切换均写入黑匣子审计日志(哈希链)
- **状态可观测**:`GET /api/ha/status` 返回全部模块当前方案/故障计数/降级历史

管理端点(需 `Authorization: Bearer <admin_token>`):

| 方法 | 路径 | 说明 |
| --- | --- | --- |
| GET | `/api/ha/status` | 全部模块高可用状态(v9.8) |
| POST | `/api/ha/modules/{module}/plan` | 手动切换到指定应急方案,body:`{"plan":"fallback1"}` |
| POST | `/api/ha/modules/{module}/reset` | 恢复主方案 |

## ⚙️ 技术栈

- **后端**:Python 3.11 + FastAPI + Uvicorn + Pydantic v2
- **前端**:原生 HTML/CSS/JS + ECharts 5 + Leaflet 1.9.4(零构建)
- **地图**:Leaflet + 高德/Esri 瓦片(库文件本地托管,支持离线回退,6 种图层)
- **通信**:WebSocket(实时推送)+ REST API
- **加密**:AES-256-GCM(应用层全链路加密)
- **认证**:API Key + PBKDF2 密码 + 数字验证码
- **数据库**:SQLite(WAL 模式 + 复合索引优化)
- **AI 模型**:DeepSeek / 阿里云 / 腾讯云 / Kimi / Bigmodel / 火山引擎
- **告警通道**:飞书 / 钉钉 / 企业微信 / Slack / 短信
- **依赖**:极简部署
- **开发投资**:6000元（RMB）
- **技术支持**:3630632874@qq.com

---

<div align="center">

**左熙宇 © 版权所有**

Copyright © 2026 左熙宇. All Rights Reserved.

</div>
