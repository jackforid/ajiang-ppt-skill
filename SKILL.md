# ajiang-ppt-skill

横向翻页网页 PPT 生成 Skill，基于 React + Vite + Tailwind CSS v4 + Framer Motion + Lucide React。

## 描述

生成横向翻页网页 PPT（单 HTML 文件），含 WebGL 背景、章节幕封、数据大字报、图片网格等模板。

提供两种风格：
- **电子杂志 × 电子墨水**：衬线字体 + 流体背景 + 暖色调
- **瑞士国际主义**：无衬线字体 + 网格点阵 + IKB蓝/柠檬黄/柠檬绿/安全橙高亮

## 触发条件

当用户需要制作分享/演讲/发布会风格的网页 PPT，或提到以下关键词时使用：
- "杂志风 PPT"
- "瑞士风 PPT"
- "Swiss Style"
- "horizontal swipe deck"

---

## 技术栈

- React 19 + Vite（端口自定）
- Tailwind CSS v4（通过 `@tailwindcss/vite` 插件接入）
- Framer Motion（页面切换动画 + 元素入场动画）
- Lucide React（图标库）

---

## 全局设计规范

用 CSS 变量统一配色，所有组件通过 `var(--xxx)` 引用，方便一键换肤：

```css
:root {
  --primary-100: #019b98;   /* 主色 */
  --primary-200: #55ccc9;   /* 主色浅 */
  --primary-300: #c1ffff;   /* 主色最浅 */
  --accent-100: #dd0025;    /* 强调色 */
  --accent-200: #ffbfab;    /* 强调色浅 */
  --text-100: #014e60;      /* 正文深色 */
  --text-200: #3f7a8d;      /* 正文浅色 */
  --bg-100: #fbfbfb;        /* 背景 */
  --bg-200: #f1f1f1;        /* 背景次级 */
  --bg-300: #c8c8c8;        /* 背景最深 */
}
```

---

## 视觉风格要素

1. **毛玻璃卡片**（glass-card）：`background: rgba(255,255,255,0.7)` + `backdrop-filter: blur(12px)` + 半透明边框
2. **光晕背景**（glow-bg）：绝对定位的大圆形色块，`filter: blur(100px)`，半透明，营造氛围感
3. **网格底纹**：用 `linear-gradient` 画 40px 间距的浅色网格线作为全局背景
4. **颜色渐变横幅**：页面底部的结论性文字用渐变色或强调色的圆角大卡片承载

---

## 页面切换机制

- 使用 Framer Motion 的 `AnimatePresence` + `mode="wait"` 实现左右滑入滑出
- 切换时根据方向（direction）决定从左还是右进入/退出
- 动画用 spring 弹簧（`stiffness: 300, damping: 30`）

```jsx
const slideVariants = {
  enter: (direction) => ({ x: direction > 0 ? '100%' : '-100%', opacity: 0, scale: 0.95 }),
  center: { x: 0, opacity: 1, scale: 1 },
  exit: (direction) => ({ x: direction < 0 ? '100%' : '-100%', opacity: 0, scale: 0.95 })
};
```

---

## 导航交互

- **键盘**：← → 和空格键切页
- **触控**：监听 touchstart/touchmove/touchend，滑动距离 > 50px 触发切页
- **底部控制栏**：左右箭头按钮 + 圆点指示器（当前页为拉长的胶囊形），点击圆点可跳转
- **顶部进度条**：细长条，宽度 = `(当前页 / 总页数) * 100%`
- **页码显示**：右上角 `01 / 10` 格式

---

## 每页幻灯片的结构模式

每个 Slide 是一个独立组件，遵循统一的动画容器模式：

```jsx
<motion.div variants={containerVariants} initial="hidden" animate="show" className="h-full flex flex-col">
  {/* 可选：glow-bg 光晕装饰 */}
  <TitleArea title="主标题" subtitle="副标题（可选）" />
  <div className="flex-1 flex flex-col items-center justify-center gap-8 pb-8">
    {/* 内容卡片区域 */}
    <ConclusionBanner text="一句话总结" />
  </div>
</motion.div>
```

子元素使用 `staggerChildren: 0.15` 依次入场，每个子元素用 `itemVariants`（从下方 20px 淡入）。

---

## 常见的内容布局

1. **三列特性卡片**：`grid grid-cols-3`，每张卡片有图标 + 标题 + 描述
2. **四步流程线**：`grid grid-cols-4`，顶部圆形步骤编号 + 下方卡片，中间用进度条连接
3. **两列问题列表**：`grid grid-cols-2`，左侧红色边框 + 警告图标 + 文字
4. **横向流程图**：一行 4 个图标方块，中间用箭头 `>` 连接
5. **纵向清单**：竖排大卡片，左侧圆形图标 + 右侧标题描述

---

## 项目结构

```
src/
  main.jsx
  App.jsx              # 导航状态、切页逻辑、整体布局
  index.css            # Tailwind 入口 + CSS 变量 + 自定义样式类
  animations.js        # slideVariants / itemVariants / containerVariants
  components/
    ui/
      TitleArea.jsx          # 标题 + 副标题
      ConclusionBanner.jsx   # 底部结论横幅
      Icons.jsx              # 自定义 SVG 图标（如果 Lucide 没有的话）
    slides/
      Slide1.jsx ~ SlideN.jsx  # 每页一个独立组件
```

---

## 关键注意事项

- `body` 设置 `overflow: hidden` 防止原生滚动
- 内容区域使用 `flex-col items-center justify-center` 让内容垂直居中，避免大片空白
- 结论横幅不要用 `mt-auto`（会被推到最底部），让它紧跟内容自然排列
- 所有幻灯片用 `absolute inset-0` 定位在同一容器内，靠 AnimatePresence 切换
- 首尾页 disable 对应的翻页按钮

---

## 使用示例

**示例 1：创建AI发展趋势PPT**
```
创建一个关于AI发展趋势的PPT，瑞士风格，10页
```
输出：瑞士国际主义风格的AI发展趋势演示PPT，包含封面、目录、6个核心内容页和总结页。采用无衬线字体、网格点阵背景，使用IKB蓝、柠檬黄、安全橙等高对比色强调重点数据。

**示例 2：产品介绍PPT**
```
做一个产品介绍PPT，杂志风格
```
输出：电子杂志风格的产品介绍演示，采用衬线字体营造阅读感，配合流体渐变背景和暖色调配色，包含产品亮点、功能特性、用户案例等模块。

---

## 标签

`ppt` `presentation` `react` `framer-motion` `tailwindcss` `web-design` `horizontal-scroll`
