# 高等数学 · 自动控制原理 · 笔记

用 GitHub Pages 发布的可离线阅读的数学笔记。在 iPhone / iPad / 电脑上用浏览器直接打开，
公式是原生 MathML（不是图片），图表是内联 SVG，**每页零外部请求**。

## 站点结构

    /                      入口页（两门课 + 手机阅读说明）
    /gaoshu/               高等数学 · 149 页 · 13 章
    /zikong/               自动控制原理 · 12 页（持续更新中）

每门课下按课次分页：`P001-映射/index.html`、`P002-…/index.html` …

## 为什么是分页而不是「整本一个文件」

实测（Chromium，CPU 降频 6x 近似 iPhone 13 Pro）：

| 形态 | 首屏可见内容耗时 |
|---|---|
| 整本合并单文件（4 万个公式） | **55~84 秒** |
| 分页（单页 100~350 个公式） | **0.1~2 秒** |

瓶颈是 MathML 排版，不是网络（gzip 后仅 1 MB）。所以走 GitHub Pages 反而应该分页 ——
相对链接在 HTTP 下完全有效，不再受 iOS 本地文件沙箱隔离的限制。

## 排版约定

- **行内公式禁止 `overflow-x` 或 `inline-block`**：CSS 规定 overflow 非 visible 的
  inline-block 基线取「底边距边」，会让公式整体上浮约 4px，与文字错位。
  真正的对策是把过长的行内公式改写成独立公式块（`$$ … $$`）。
- **窄屏兜底只打标记、不动基线**：构建器在 390px 下实测，把**真正溢出**的那几条行内公式
  标记为 `.miw`（行内可滑动）或 `.miwb`（宽到整屏放不下，升级为独占一行的独立公式）。
- **目录页摘要里的公式允许换行**：摘要只是点缀。注意 P097 那条
  `∬ΣPdydz+Qdzdx+Rdxdy=∬Σ(Pcosα+Qcosβ+Rcosγ)dS` 其实是**纯文本**
  （原文没写 `$…$`），没有 `.mi` 也没有 `<math>`，所以要用
  `.li .sn { overflow-wrap: anywhere }` 兜底，而不是给 `.mi` 加规则。

## 本机重新构建

内容源在电脑上（不在这个仓库里），改完笔记后重新构建：

    python <课程目录>/pipeline/notebuild.py      # 构建全部课页 + 目录页
    python _build/publish.py                      # 导出到 _site
    python _build/audit_site.py                   # 390px 全站体检（需 Playwright）
    python _build/push_github.py                  # 提交并推送，自动开 Pages

`notebuild.py` 依赖 Playwright 做窄屏实测标记，需用装了 Playwright 的解释器
（`D:/Dev/Languages/python/Python314/python.exe`）。没有 Playwright 时会静默跳过标记，
不影响构建成功。

## 不发布的内容

源目录里的 `page.html`（LaTeX 片段）、`_design/`、`data/`、`pipeline/` 都不上线，
`publish.py` 只导出构建后的 `index.html`（CSS 已内联进每个页面）。