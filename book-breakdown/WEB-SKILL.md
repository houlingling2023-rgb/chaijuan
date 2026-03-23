---
name: chaijuan-web-generator
description: |
  拆卷网页生成技能：将拆书师输出的结构化 Markdown 转换为单文件 HTML 阅读页面。
  当收到拆书内容（Markdown/JSON 格式）需要转换为网页时使用此技能。
  也适用于优化已有拆书页面的视觉效果、交互体验、图表渲染等场景。
---

# 拆卷 · 网页生成技术 SKILL

本 SKILL 指导网页生成 Agent 将结构化拆书内容转换为高质量的单文件 HTML 页面。

**职责边界**：你只负责技术实现，不负责内容质量。内容由拆书师 Agent 提供，质量由主编 Agent 把关。你的任务是把好内容变成好网页。

---

## 一、技术架构

### 1.1 基础栈
- 单文件 HTML（所有代码、样式、数据内联）
- React 18 + Babel（CDN 引入，零打包）
- 无外部依赖（除 CDN 字体和 React）

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>[书名] · 深度拆解</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/react/18.2.0/umd/react.production.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/react-dom/18.2.0/umd/react-dom.production.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/babel-standalone/7.23.9/babel.min.js"></script>
  <!-- 字体根据主题选择 -->
</head>
<body>
  <div id="root"></div>
  <script type="text/babel">
    // 1. THEME（主题配置）
    // 2. DATA（从拆书内容转换的数据对象）
    // 3. COMPONENTS（通用组件库）
    // 4. PAGES（页面组件）
    // 5. APP（应用入口）
  </script>
</body>
</html>
```

### 1.2 数据结构

拆书内容统一转换为以下数据结构：

```javascript
const DATA = {
  meta: {
    title: "书名",
    author: "作者",
    rating: "8.5",           // 豆瓣评分（可选）
    tagline: "一句话定位",
    audience: "适合谁读",
  },
  overview: {
    summary: "全书核心论点（2-3 段）",
    architecture: "知识架构描述",
    readingGuide: "阅读建议",
  },
  chapters: [
    {
      id: "模块ID",
      icon: "🧭",
      title: "模块名",
      tagline: "一句话副标题",
      color: "主色（从主题色板分配）",
      logicChain: ["概念A", "概念B", "概念C", "结论"],
      sections: [
        {
          title: "小节名",
          type: "methodology|framework|action|tool|reality|decision",
          formula: "一句话公式",
          insight: "原创洞察段落",
          steps: [
            { step: "步骤名", detail: "具体说明" }
          ],
          cases: [
            {
              label: "案例标题",
              scene: "场景",
              conflict: "冲突",
              turning: "转折",
              result: "结果",
              // 或简写模式（用于较短案例）：
              // desc: "完整描述"
            }
          ],
          charts: [
            {
              type: "flow|compare|structure|timeline",
              title: "图表标题",
              data: { /* 图表数据，结构因类型而异 */ }
            }
          ],
          exercise: "行动指南"
        }
      ]
    }
  ]
};
```

---

## 二、通用组件库

以下组件是所有主题共享的。样式参数从主题配置（THEME 对象）读取。

### 2.1 布局组件

| 组件 | 用途 | 必要 Props |
|---|---|---|
| `ReadBar` | 顶部阅读进度条 | `progress` (0-100) |
| `TabNav` | Tab 导航栏 | `tabs[], current, onSelect` |
| `Sidebar` | 侧栏目录（可选，根据主题） | `chapters[], current, onSelect` |
| `PageNav` | 底部翻页 | `current, total, onPrev, onNext` |

### 2.2 内容组件

| 组件 | 用途 | 必要 Props |
|---|---|---|
| `Card` | 内容卡片容器 | `children, color` |
| `SectionTitle` | 章节标题 | `icon, title, sub` |
| `LogicChain` | 逻辑链 | `steps[]` |
| `Formula` | 核心公式高亮 | `text, color` |
| `Insight` | 洞察高亮块 | `text, color` |
| `Step` | 步骤项 | `n, text, detail, color` |
| `Quote` | 引用块 | `text, color` |
| `Para` | 正文段落 | `children` |

### 2.3 案例组件

| 组件 | 用途 | 必要 Props |
|---|---|---|
| `CaseStudy` | 完整案例（四要素） | `title, scene, conflict, turning, result` |
| `CaseCard` | 简短案例（标题+描述） | `label, desc` |

### 2.4 图表组件（SVG）

| 组件 | 用途 | 必要 Props |
|---|---|---|
| `InkFlow` | 流程图 | `title, steps[], colors[]` |
| `InkCompare` | 对比图 | `title, items[{label, value, color}]` |
| `InkStructure` | 结构图 | `title, data{center, nodes[]}` |
| `InkTimeline` | 时间线 | `title, events[{year, text}]` |

---

## 三、多风格主题系统

### 3.1 主题配置结构

```javascript
const THEME = {
  name: "ink-wash",  // 主题标识
  
  // 色板
  colors: {
    bg: "#f5f0e8",
    paper: "#faf7f0",
    text: "#2c2418",
    textSoft: "#6a5d4e",
    textMuted: "#a89a88",
    accent: "#c85030",        // 主强调色
    chapterColors: [          // 各章节分配色
      "#a87a28", "#3a7090", "#3a8a6a", "#a84a58", "#b89a28"
    ],
  },
  
  // 字体
  fonts: {
    heading: "'Noto Serif SC', 'PingFang SC', Georgia, serif",
    body: "'Noto Serif SC', 'PingFang SC', Georgia, serif",
    accent: "'ZCOOL XiaoWei', serif",  // 装饰用字体（可选）
    mono: "monospace",
  },
  
  // 导航模式
  nav: "sidebar",  // "sidebar" | "tabs" | "tabs+sidebar"
  
  // 装饰元素
  decorations: {
    sealIcon: true,           // 印章组件
    landscape: true,          // 山水 SVG 背景
    paperTexture: true,       // 纸张纹理
    inkAnimations: true,      // 水墨动画
  },
  
  // 组件样式覆盖
  card: {
    borderRadius: 10,
    borderLeft: true,         // 左侧色条
    shadow: "0 1px 8px rgba(120,100,70,0.06)",
    hoverLift: true,
  },
};
```

### 3.2 预设主题

#### 🏔️ 水墨国风（ink-wash）
- **适合**：人文、心理、哲学、传记类
- **代表作**：了不起的我
- **特点**：宣纸底色、印章装饰、山水 SVG 背景、ZCOOL XiaoWei 字体
- **导航**：侧栏目录 + 底部翻页
- **色板**：暖棕色系 + 朱红强调

#### 📜 古典书卷（classic-scroll）
- **适合**：商业、历史、科技史类
- **代表作**：浪潮之巅
- **特点**：暖色纸张底色、竖线时间轴、红色点缀、Noto Serif SC
- **导航**：顶部 Tab 切换
- **色板**：米色系 + 深红/深蓝强调

#### ⚡ 现代极简（modern-clean）
- **适合**：方法论、工具书、科普类
- **代表作**：不上班咖啡馆、天才的学习方法
- **特点**：冷灰白背景、彩色卡片、无装饰元素、Inter + Noto Serif SC
- **导航**：首页卡片 + 章节视图（或顶部 Tab）
- **色板**：灰白底 + 多彩章节色

#### 🔮 科技未来（tech-future）
- **适合**：AI、编程、前沿科技类
- **特点**：深色/暗色背景可选、渐变色块、代码风字体、动态粒子
- **导航**：顶部 Tab
- **色板**：深色底 + 霓虹色强调

### 3.3 风格选择规则

1. 如果用户指定风格 → 使用指定风格
2. 如果主编 Agent 指定风格 → 使用指定风格
3. 否则根据书籍类型自动选择（参考 3.2 的「适合」标签）
4. 不确定时默认使用「水墨国风」

---

## 四、SVG 图表技术规范

### 4.1 布局数学：先算再写

**在写任何 SVG 代码之前，必须先计算所有元素的坐标和边界。**

```
举例：N 个等宽节点，每个宽 w，间距 gap，viewBox 宽 V
totalWidth = N * w + (N-1) * gap
startX = (V - totalWidth) / 2
每个节点 x = startX + i * (w + gap)
最右边界 = startX + (N-1) * (w + gap) + w

如果最右边界 > V → 溢出！必须缩小 w 或 gap 或增大 V
```

### 4.2 对齐规则

- 同层级节点 y 坐标统一
- 节点间连线用水平直线（不用斜线）
- 所有文字 `textAnchor="middle"` 居中
- 超长文字用 `<foreignObject>` 包裹允许换行，或截断 + "…"

### 4.3 交互规则

- 可点击节点：`<g>` 加 `cursor: pointer` + `onClick`
- 内部文字加 `pointerEvents: "none"` 确保点击穿透
- Hover 效果通过 React state 控制（不用 CSS :hover，因为 SVG 兼容性问题）

### 4.4 图表尺寸

- `InkFlow`：viewBox 宽 600，高 160（节点 ≤ 6 个）或 高 200（节点 > 6 个）
- `InkCompare`：viewBox 宽 600，高 = 50 + items.length × 40
- `InkStructure`：viewBox 宽 600，高 250
- `InkTimeline`：viewBox 宽 600，高 = 40 + events.length × 60
- 所有图表外层设 `width="100%"`，自适应容器宽度

---

## 五、导航与响应式

### 5.1 Tab 导航规范

Tab 数量 ≥ 5 时：
- 外层 `overflowX: "auto"`
- 隐藏原生滚动条（`-webkit-scrollbar` display:none + `scrollbar-width: none`）
- 右侧渐变遮罩提示可滚动
- Tab 设 `flexShrink: 0`
- 紧凑化 padding（10px 14px），字号 13px

第一个 Tab（全书导读）与后续章节 Tab 之间用视觉分隔（竖线 `borderRight`）。

### 5.2 LogicChain 断行

步骤和箭头必须包在同一个 `inline-flex` 容器里：

```jsx
// ✅ 正确
{steps.map((step, i) => (
  <span key={i} style={{display:"inline-flex", alignItems:"center", gap:4}}>
    <span>{step}</span>
    {i < steps.length - 1 && <span style={{color: accent}}>→</span>}
  </span>
))}
```

### 5.3 最小宽度

- 设计基准：375px（iPhone SE）
- 所有内容在 375px 下不被裁切
- 图表在窄屏下通过 `width="100%"` + viewBox 自动缩放
- **InkCompare 双栏**：窄屏（<500px）必须用 `flexWrap:"wrap"` 自动换成上下排列，每栏设 `flex:"1 1 280px"` + `minWidth:0`

### 5.4 模块间过渡卡片（试跑新增）

每个模块末尾（ActionBox 之后）添加一个过渡卡片，引向下一个模块：

```jsx
<Card color={nextChapterColor} style={{marginTop:32,background:`linear-gradient(135deg,${nextColor}04,${P.paperLight})`,borderLeft:`3px solid ${nextColor}40`}}>
  <Para>[过渡文字，从拆解内容的模块递进连接句中获取] →</Para>
</Card>
```

最后一个模块不加过渡卡片。

---

## 六、组件联动一致性

添加/修改组件时，必须同步更新所有引用点：

**检查清单**：
- CHS 数组的长度 === chContent 数组的长度
- 新组件的 props 在 App 层正确传递
- PageNav 的 total、ReadBar 的 progress 依赖 CHS.length
- 侧栏目录的章节列表与实际章节同步

---

## 七、合规与版权

每个页面底部必须包含：

```
⚠ 免责声明：本页面内容仅供个人学习与读书笔记整理之用，不用于任何商业目的。
书籍版权归原作者及出版社所有。如有侵权，请联系博主，将第一时间删除处理。
```

---

## 八、自审清单

每次生成页面后，逐项检查：

### A. 代码完整性
- [ ] 括号平衡（`{}`、`()`、`[]`、反引号差值为 0）
- [ ] CHS.length === chContent.length
- [ ] 所有组件 props 正确传递
- [ ] Script 标签开闭匹配
- [ ] React 渲染无报错（ReactDOM.createRoot → render）

### B. SVG 布局
- [ ] 每个 SVG：计算元素坐标，确认无溢出 viewBox
- [ ] 同层节点 y 坐标一致
- [ ] 文字不超出节点边界

### C. 导航与交互
- [ ] Tab ≥ 5 时有滚动遮罩
- [ ] 导读 Tab 与章节 Tab 有视觉区分
- [ ] 章节速览卡片可点击跳转
- [ ] LogicChain 箭头不孤立在行首
- [ ] 侧栏（如有）开关正常

### D. 响应式
- [ ] 375px 宽度下无内容裁切
- [ ] 图表自适应缩放
- [ ] 长文本不溢出容器

### E. 主题一致性
- [ ] 所有组件使用 THEME 色板（不硬编码颜色）
- [ ] 字体与主题配置一致
- [ ] 装饰元素与主题匹配（水墨风有印章/山水，现代风没有）

### F. 试跑经验（必读）
- [ ] InkCompare 在窄屏下能换行（flexWrap + flex-basis）
- [ ] 模块间有过渡卡片（最后一个模块除外）
- [ ] 底部出版信息包含 ISBN
- [ ] 生成超时设置至少 15 分钟（大文件 80KB+ 需要充足时间）
- [ ] 文件名不要用连字符+中文混合（如 `浪潮之巅-完整知识图谱.html`），纯中文即可
