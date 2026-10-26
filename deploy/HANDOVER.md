# qinparts.com 国产车配件站 —— 交接文档

> 版本：2026-10-26 ｜ 本地路径：`C:\Users\123\WorkBuddy\qinparts-website`
> 仓库：`github.com/Shielby514/qinparts-website`（分支 `main`，最新 `37bf63c`）
>
> 一句话现状：**站建完了、门禁全绿、但没上线**（Pages 未启用 + 域名未解析），
> 卡在两张"非技术"的手续上，代码侧没有阻塞。

---

## 一、30 秒速览

| 项目 | 状态 |
|---|---|
| 页面 | **960 页**，全部生成完毕 |
| 站内链接 | 44,619 条，**断链 0** |
| 质量门禁 | **9 项全 PASS**（见第五节，改动后必跑） |
| 内容规范 | 已定稿，见 `../20260324111450/docs/qinparts-content-standard-2026-10-24.md` |
| 品牌视觉 | LOGO + 公司 Flag + 品牌红 `#990110` 已全站落地 |
| 产品图 | ⚠️ **最大遗留**：1,477 张全部来自竞品站，仅 4 件做了试点重绘 |
| 上线 | ❌ **未上线**：GitHub Pages 未启用，`www.qinparts.com` 未解析 |
| 每日任务 | 无。静态站，不需要 24 小时流水线 |

---

## 二、站点结构

```
/                       首页（品类网格 + 品牌入口）
/brands/{brand}/        14 个品牌页
/models/{brand-model}/  95 个车系页
/oe/{brand}-{oe}/       798 个 OE 详情页   ← 主体
/parts/{brand}/{cat}/   27 个品牌×品类组合页
/categories/{cat}/      9 个品类页
/blog/                  6 篇（含列表页）
/rfq/ /about/ /contact/ /faq/ /factory/ /shipping/ /quality-control/
/privacy/ /terms/       9 个静态页
/404.html
```

四层联动：`/` → `/brands/` → `/models/` → `/oe/`，加 `/parts/{brand}/{cat}/` 组合层。

**规则细节**（改动前务必理解，否则会造出断链）：

- 别名归一：去品牌前缀 + 去连字符做 key（`tiggo-5` = `tiggo5`，`yuanplus` → `atto3`）
- 跨品牌车型（Jetour / Exeed / Jaecoo 同属奇瑞集团）主品牌取配件数最多者，
  页内声明 "also listed under"
- 无车型记录归入 `{brand}-general`（显示 "General Applications"）
- 组合页 < 3 件不出页，`combo_link()` 自动回退到品类页/品牌页

---

## 三、部署状态（这里是唯一阻塞）

### 3.1 已经就绪的

- 远端仓库可推（`origin` 内嵌 PAT）
- `CNAME` 文件内容 = `www.qinparts.com`
- `sitemap.xml` 960 条，`robots.txt` 已就位
- 所有 `<a href>` / `<img src>` 是 **root-relative**（`/oe/xxx/`），
  只有 canonical / og:url / og:image / JSON-LD 的 url 用绝对地址

### 3.2 两个没做的动作

**① GitHub Pages 未启用**

建议走 **分支源模式**（Deploy from a branch，`main` / root），
不要走 GitHub Actions——本机 PAT **缺 `workflow` scope**，
推不了 `.github/workflows/*.yml`（原准备的文件备份在 `.workbuddy/deploy/github-pages-static.yml`）。

用 API 开启（本机没装 `gh`，用 curl）：

```bash
curl -X POST -H "Authorization: token <PAT>" \
  -H "Accept: application/vnd.github+json" \
  https://api.github.com/repos/Shielby514/qinparts-website/pages \
  -d '{"source":{"branch":"main","path":"/"}}'
```

**② 域名未解析**

`www.qinparts.com` 需要一条 CNAME 指向 `shielby514.github.io`。
解析生效前，页面上的产品图会裂——**不是因为图不存在**（磁盘上 1,477 张都在），
而是历史遗留过"绝对域名写法"，已于 2026-10-24 全站改为 root-relative 修好。
若上线后仍看到图裂，先查 CDN/DNS，不要再改回绝对地址。

### 3.3 上线后必须做的事

1. Google Search Console 添加资源 `https://www.qinparts.com/`（**带尾斜杠**）
2. 提交 `sitemap.xml`
3. 等 3-7 天看首批收录，再决定是否加量

---

## 四、内容规范（改任何页面之前先读）

文档：`C:\Users\123\WorkBuddy\20260324111450\docs\qinparts-content-standard-2026-10-24.md`

核心一句话：**AI 搜索取的是 chunk，成败取决于「这段被单独摘出去还成不成立」。**

### 答案块三段式（每个要拿排名的段落都按这个写）

```
直答句（40-60 词，禁止代词开头）
证据句（可核验的数字/事实）
行动句（下一步做什么）
```

### 实体归一（写错会被判重复内容）

| 类型 | 规则 |
|---|---|
| OE 号 | 全大写、去空格去连字符 |
| 品牌 | 8 个固定值，不要造新写法 |
| 车型 | 用官方写法；**别名首次出现必须声明**（`Atto 3 (also sold as Yuan Plus)`）|
| 品类 | 8 个固定值 |
| 商务条款 | `MOQ 50 pcs` / `FOB Guangzhou`（不要写 "MOQ: 50" 或"50个起订"）|

### JSON-LD 五层（缺一层不算合格）

`Organization` / `BreadcrumbList` / `Product` + `Offer` / `FAQPage` / `ItemList`

**禁止编造**：`foundingDate`、`telephone`、`email`、`price`。
没有的数据就整块不写，不要填假值。

**FAQPage 必须与页面可见文字逐字一致**，否则富媒体结果会被拒——门禁会查这一项。

---

## 五、质量门禁（改动后必跑，不过不许 push）

### 脚本在**主站**目录，不在本仓库

```bash
cd C:/Users/123/WorkBuddy/20260324111450
PYTHONPATH="C:/Users/123/.workbuddy/binaries/python/envs/kingmay/Lib/site-packages" \
"C:/Users/123/.workbuddy/binaries/python/envs/kingmay/Scripts/python.exe" \
  .workbuddy/scripts/verify_qinparts_pages.py
```

> ⚠️ 这是个**交接坑**：脚本物理位置在 `20260324111450\.workbuddy\scripts\`，
> 内部 `ROOT` 常量硬编码为 `C:\Users\123\WorkBuddy\qinparts-website`。
> 换机器必须先改这个常量，否则会校验错目录。

### 九项检查与当前基线

| # | 检查项 | 当前 | 说明 |
|---|---|---|---|
| 1 | title 是否跑到 body 后 | 0 | 曾经的 `rstrip('</head>')` 字符集 bug |
| 2 | JSON-LD 能否解析 | 0 | |
| 3 | 面包屑品类是否与页面一致 | 0 | |
| 4 | 断链 | 0（44,619 条链接） | |
| 5 | 未被任何车系页链接的 OE | 0 | 孤儿页 |
| 6 | 非中英文异常字符 | 0 | |
| 7 | FAQPage 与可见文本一致 | 0 | 富媒体资格 |
| 8 | div 开闭配平 | 0 | **2026-10-24 才补的** |

第 8 项是血的教训：前 7 项全绿的时候，站里同时躺着
**832 页页脚未闭合、798 页内容逸出 `<main>`、首页品类网格被劈断**。
**链接能通 ≠ 结构对**，所以加了这项。

> 当前基线：**960 页 / 44,619 链接 / 全 0 / GATE: PASS**

---

## 六、脚本清单

都在 `.workbuddy/scripts/`（该目录在 `.gitignore` 内，**不入库**，换机器要单独拷）。

| 脚本 | 作用 | 幂等 |
|---|---|---|
| `fix_qinparts_urls.py` | 绝对域名 → root-relative（限定 `img`/`a` 标签，canonical 保留绝对） | ✅ |
| `fix_qinparts_structure.py` | 补 footer `</div>`、删多余的 `</div></main>`、重建首页品类网格 | ✅ |
| `rebuild_oe_related.py` | 重建 OE 页"相关件"：同品牌+同品类，最多 12 条 | ✅ |
| `brand_header.py` | 960 页页头注入（只 1 种变体，靠 CSS order 控左右） | ✅ |
| `audit_div_balance.py` | 单独查 div 配平（门禁第 8 项的独立版） | 只读 |
| `audit_img_quality.py` / `classify_img.py` | 图片质量审计/分类 | 只读 |
| `finish_pilot.py` | 试点图：裁底部 14% 去水印 → 800×800 | ✅ |
| `install_pilot_imgs.py` | 试点图替换（显式映射表 + 原图备份） | ✅ |
| `make_logo.py` | 抠 LOGO、生成 favicon / apple-touch-icon | ✅ |
| `shot_site.js` / `shot_oe_gal.js` / `contact_sheet.py` | Playwright 整站实拍 + 拼总览图 | 只读 |

> 生成脚本（`build_*`）在**主站** `.workbuddy/scripts/` 下，
> 因为它们是主站流水线的一部分。

---

## 七、品牌视觉资产（已定稿，别改）

| 项 | 值 |
|---|---|
| LOGO | `assets/logo-qinparts.png`（267×68 @2x，显示高 38px，透明底），页头**左上角** |
| 公司 Flag | `China Auto Parts One-Stop Solution Provider`（页头可见文字 + Organization.slogan） |
| 品牌红 | `--accent:#990110` / `--accent-dark:#7a0d0d`（hover）/ `--accent-soft:#fbeaec` |
| 底色 | `--soft:#faf8f8`（配红不显冷；原冷灰已弃用） |
| 图标 | `favicon.png`（64，红底白 Q）/ `apple-touch-icon.png`（180） |

**页头左右布局只靠 CSS `order` 控制**：
`.brand{order:1;margin-right:auto}`、`nav.top{order:2;margin-left:auto}`。
DOM 里一直是 nav 在前——所以挪位置只改一个文件，不用刷 960 页。
`Get a Quote` 按钮用属性选择器 `nav.top a[href="/rfq/"]`，因为 DOM 里它没有 `class="cta"`。

---

## 八、产品图：最大的遗留问题

### 现状真相

| 项 | 数值 |
|---|---|
| 磁盘图片 | 1,554 张（63 MB），在 `assets/img/cn/` |
| 被页面引用 | **1,477 张** / 4,762 次 |
| 未被引用（死图） | 77 张 |
| **来源** | **全部来自 `imgs.partsail.com`（竞品站）** |
| 页面显示尺寸 | **200×200**（`object-fit:contain`），磁盘 700×700 |
| 已重绘 | **仅 4 件 8 张**（BYD 减震弹簧上座 / 门把手 / 电动水泵 / Geely 门锁体） |

原图普遍带：牛皮纸底、黄卷尺、中文说明、二维码、金属刷。

### 目标规格

800×800、纯白底、产品占画面 ~70%、无文字/卷尺/水印/外物，JPEG q88。

### 两条铁律（都是实测踩出来的）

1. **重绘必须 image-to-image**（`input_fidelity: high`）。
   纯文生图会画出"长得像但不是"的零件——这在汽配站是灾难。
2. **压在零件上的外物会被 AI 吸收成零件结构**。
   实测：门锁体上的金属刷被重绘成黑色泡沫垫。
   提示词必须点名并禁止再解释：
   > "the wire brush is a foreign object, remove it entirely —
   > do not turn it into foam, padding or any soft strip"

### 水印处理：用几何裁切，不要靠检测

- 白底图上"亮像素计数"**彻底失效**（右下角 35,301 个亮像素，无区分度）
- 极淡的浅灰水印像素计数只报 10 个 = 静默漏检
- **唯一可靠验收 = 把右下角裁出来放大目视**
- 落地方案：裁底部 14% → 居中裁方 → resize 800×800

### 三档预算（未决，等你拍）

| 方案 | 数量 | 预估积分 | 风险 |
|---|---|---|---|
| 只换主图、删细节图位 | 798 | 3,990–7,980 | 成对图只换一张会风格打架 |
| 主图 + 细节图都换 | 1,477 | 7,385–14,770 | 成本最高，效果最一致 |
| 门面优先（首页/品牌页/热销） | ~250 | 1,250–2,500 | 内页仍是竞品图 |

重做率按 20–30% 上浮预留积分。

---

## 九、待办清单

| 优先级 | 事项 | 阻塞点 |
|---|---|---|
| P0 | 启用 GitHub Pages（分支源模式） | 无，一条 API 调用 |
| P0 | `www.qinparts.com` CNAME 解析到 `shielby514.github.io` | 需要域名管理权限 |
| P1 | 产品图方案拍板并执行（见第八节三档） | 等决策 |
| P1 | 上线后接 GSC + 提交 sitemap | 依赖 P0 |
| P2 | 清理 `_test.jpg` 与 77 张死图 | 无（但先确认没被模板引用） |
| P2 | 补齐 `.workbuddy/scripts` 到新机器 | 换机时必做 |
| P3 | 增补 blog（现仅 6 篇） | 无 |

---

## 十、交接必知的坑

1. **门禁脚本不在本仓库**，在主站 `.workbuddy/scripts/`，且 `ROOT` 硬编码
2. **`.workbuddy` 不入库**（`.gitignore` 第 6 行），换机器必须整个目录拷过去
3. **PAT 缺 `workflow` scope**，不要试图加 GitHub Actions，用分支源部署
4. **图片引用必须 root-relative**。曾经全站写成绝对域名，域名未解析时
   **1,477 张图全裂、21,367 条内链点击即跳不可达地址**，而磁盘上文件都在——
   极易误判成"图片丢了"而重新生成
5. **批量替换域名要用 `<a` / `<img` 限定标签名**，别全局替换：
   canonical / og:url / og:image / JSON-LD 的 url **必须保持绝对**
6. **FAQPage 与可见文字必须逐字一致**，改 FAQ 时两处一起改
7. 根目录 `_prev_*.png`、`tmp_*` 都是本地验收产物，已被 `.gitignore`，**不要提交**
