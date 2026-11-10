# 果果的个人网站

> 关于我 · 照片墙 · 随笔 · 作品
> 一个纯静态前端 + WorkBuddy 云服务（数据库 / 登录 / 文件存储）的个人网站。

**线上地址**：https://guoguo-site.app.workbuddy.host/?sharecode=SU2SV3WGKY

**源码备份**：WorkBuddy 资料库（`guoguo-site-源码备份-2026-11-10.zip`，含完整 git 提交历史）

---

## 目录结构

```
.
├── index.html      # 四个页面的 DOM 结构（关于我 / 照片墙 / 随笔 / 作品）
├── styles.css      # 全部样式，配色用 CSS 变量统一管理
├── app.js          # 云服务初始化、数据读写、页面渲染、登录与上传逻辑
├── package.json    # 仅用于标识项目名，没有构建步骤
└── .gitignore
```

没有构建步骤，没有 `npm install` 依赖。直接双击 `index.html` 或用任意静态服务器打开即可。

---

## 本地运行

```bash
# 任选其一
python -m http.server 8848
npx serve .
```

然后访问 `http://127.0.0.1:8848/`。

> 直接双击 `index.html`（`file://` 协议）也能看，但登录回调会失败——OAuth 回调只认 HTTPS 域名。

---

## 数据存在哪里

**不在 Git 仓库里，在 WorkBuddy 云服务的数据库里。**

| 表 | 存什么 |
|---|---|
| `site_profile` | 自我介绍、标题、副标题、城市、邮箱、头像 |
| `tracks` | 「我关注的方向」三条 |
| `essays` | 随笔（标题、摘要、正文、标签、日期） |
| `works` | 作品（名称、分类、说明、封面、年份） |
| `photos` | 照片（压缩后的图片数据、说明、排序） |

- 应用 ID：`wbapp_rgjGUVYr5JkDnaV8ZQHI61`
- 图片方案：浏览器端 Canvas 压缩后存进数据库（访客可直接读）；原图备份到云存储（仅登录后可取）

**所以「改内容」不需要改代码、不需要重新发布**——登录后在页面上点开编辑弹窗改，存进数据库，刷新即生效。

---

## ⚠️ 一个必须知道的限制：登录回调域名白名单

云服务对 OAuth 回调地址做**精确匹配**。实测结果：

| 回调地址 | 服务端响应 |
|---|---|
| `https://guoguo-site.app.workbuddy.host/` | ✅ 通过 |
| `https://<你的用户名>.github.io/` | ❌ `callback_url is not a registered domain of any published application` |

推论：

- **公开内容读取不受影响**。把请求的 Origin 改写成 `github.io` 实测，5 个数据接口全部返回 200，页面照常显示内容。
- **登录会失败**。所以站点如果托管在 GitHub Pages，别人能看，你**不能登录**——传不了照片、发不了随笔、改不了内容。

### 两条可选路线

**路线 A（推荐）：GitHub 只做代码仓库**
GitHub 存源码、做版本历史、随时回滚；网站继续跑在 `guoguo-site.app.workbuddy.host`，**所有功能完整可用**。

**路线 B：GitHub Pages 再开一份公开只读版**
访客从 `github.io` 也能看全部内容（数据来自同一个数据库，实时同步），但你只能在自己的站点上编辑。适合当「作品集展示页」。

---

## 安全设计

- **访客只读**：所有编辑 / 删除 / 上传入口带 `owner-only` 类，CSS 用 `body:not(.signed-in) .owner-only{display:none!important}` 隐藏。
- **数据库行级权限（RLS）**：读策略公开，写策略要求 `auth.role() = 'authenticated' AND owner_id = auth.uid()`。
  - 特别注意：匿名请求下 `auth.uid()` 返回字面量 `'anon'`，所以**只写 `owner_id = auth.uid()` 等于没保护**，必须额外加 `auth.role()` 判断。
- **种子内容认领**：初始内容以 `owner_id='seed'` 写入，首次登录后自动改归属本人。

---

## 字体

使用国内镜像 `fonts.googleapis.cn` / `fonts.gstatic.cn`，避免 Google Fonts 在国内首屏长时间等待。CSS 里保留了 PingFang SC / 微软雅黑兜底。

---

## 部署到 WorkBuddy 云（保留完整功能）

用 WorkBuddy 的发布能力，**复用同一个应用 ID**，域名保持 `guoguo-site` 不变——换域名会触发上面说的 Origin 白名单问题，导致登录失效。
