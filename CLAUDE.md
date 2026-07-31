# 元素周期表复刻项目

## 图像识别（强制规则）

**当前模型不支持视觉识别。** 收到图片时：

### 找图片路径
1. 优先：`[Image: source: C:\Users\...]` 直接用
2. 找不到就取最新截图：
```bash
ls -t "C:\Users\熊艳霞\Pictures\Screenshots\*.png" "C:\Users\熊艳霞\AppData\Local\Packages\Microsoft.ScreenSketch_8wekyb3d8bbwe\TempState\Snips\*.png" 2>/dev/null | head -1
```
3. 禁止去 /tmp、data/uploads 搜索

### 调用 Kimi
```bash
cd "C:\Users\熊艳霞\.claude\kimi-vision" && node vision.js "拿到的路径" --fast --prompt "详细描述这张截图的内容" 2>&1
```

或一步到位：
```bash
cd "C:\Users\熊艳霞\.claude\kimi-vision" && node latest.js
```

## 网页爬取（强制规则）

**WebFetch 经常被拦。** 爬网页用 curl，不要用 WebFetch：
```bash
curl -sL --max-time 15 "网址" 2>&1 | head -100
```
GitHub 原始文件用 ghproxy 加速：
```bash
curl -sL "https://ghproxy.net/https://raw.githubusercontent.com/用户/仓库/分支/路径" 2>&1
```

---

## 项目目标
复刻 https://seanwong17.github.io/Future-Style-Periodic-Table/ —— 一个赛博朋克风格的可交互元素周期表。但**不是完全照搬**，具体需求见下方。

## 用户信息
- 编程新手，正在学 Vue 3 + TypeScript
- 喜欢水绿/青绿色、小清新马卡龙渐变风格
- 喜欢原版那种发光一样的设计效果，但不想雷同
- 项目做出来要**自己能理解和修改**，不能全丢给 AI 代写
- 每天约 8 小时，从现在（周三）到下周三，中间周六日放假

## 技术方案
- **技术栈**: Vue 3 + TypeScript + Vite（项目已在 D:\periodic-table\）
- **3D 电子排布**: CSS 3D（不用 Three.js，用户是新手）
- **数据**: 118 个元素已下载到 src/data/elements.json（来自原项目，含中文名）
- **布局**: CSS Grid，18 列 × 7 行 + 镧系锕系额外两行

## 设计风格
- **直接复刻原版**：深色赛博朋克风，不改配色
- 原版 CSS 已拿到：`:root` 变量 + 全部样式
- 深色背景 `#050505`，青色 `#00f3ff` + 粉色 `#ff00aa` 霓虹点缀

## 功能需求（全部要做）

### 核心功能（必须）
1. **周期表网格**: 118 个元素卡片，CSS Grid 布局，镧系锕系单独两行
2. **元素卡片**: 显示符号、名称、原子序数，发光悬停动效
3. **分类筛选**: 顶部彩色标签按钮（碱金属、稀有气体等），点击高亮同类，其他变灰
4. **详情面板**: 点击卡片弹出，分 4 个 Tab：
   - 基础信息：名称、符号、原子序数、原子质量、分类、电子排布
   - 物理属性：密度、熔点、沸点、相态
   - 化学属性：电负性、电离能
   - 历史：发现者、发现年份、命名来源
   - ~~媒体~~（原版没做好，跳过）
5. **热力图**: 在周期表上渐变显示原子半径/电负性/电离能，下拉或按钮切换模式
6. **3D 电子排布**: CSS 3D 旋转电子轨道，鼠标拖拽旋转 + 滚轮缩放
7. **搜索**: 输入符号/名称/原子序数快速定位元素
8. **中英切换**: 一键切换中文/英文界面
9. **只做电脑端**，不做移动端适配

### 优先级（如果时间不够）
3D 电子旋转 > 中英切换（这两项可延后）

## 教与学的原则
用户希望自己动手写代码，AI 负责：
- **先讲解**概念、原理、为什么要这样写
- **引导**用户自己动手敲代码
- 只有在**用户卡住或明确请求**时才帮忙写代码
- 写完代码后**解释**每一块的作用
- 不要大段贴代码让用户直接复制

## 数据说明
数据文件: src/data/elements.json
格式: { "elements": [ ... ] }
每个元素的关键字段:
- name: 中文名称
- symbol: 符号
- number: 原子序数
- period: 周期（行号 1-7）
- group: 族（列号 1-18）
- category: 分类（中英混，需要标准化）
- atomic_mass: 原子质量
- boil, melt: 沸点/熔点
- density: 密度
- electron_configuration / shells: 电子排布
- electronegativity_pauling: 电负性
- ionization_energies: 电离能
- discovered_by, named_by, year_discovered: 历史信息

## 5 天计划

### Day 1（今天）: 骨架 + 卡片
- CSS Grid 周期表布局（18 列，按 period/group 定位）
- 元素卡片组件（符号、名称、原子序数）
- 水绿色调 + 浅色背景 + 卡片发光

### Day 2（周四）: 交互 + 详情
- 点击卡片弹出详情面板，4 个 Tab
- 搜索功能
- 悬停发光动效

### Day 3（周五）: 筛选 + 热力图
- 顶部分类标签筛选
- 热力图模式切换 + 渐变映射

### Day 4（周一）: 3D 电子排布
- CSS 3D 轨道旋转
- 鼠标拖拽 + 滚轮缩放

### 当前进度（截至 Day 2）
- ✅ Grid 布局 + 卡片 + 分类筛选
- ✅ 详情弹窗（4 个 Tab，大小固定一致）
- ✅ 热力图模式（标准/半径/电负性/电离能/熔点/沸点/密度）
- ✅ 搜索功能
- ✅ 3D 电子轨道（小窗 + 全屏展开，拖拽旋转 + 滚轮缩放）⚠️ 缩放感知问题待优化
- ✅ 顶栏固定 + 元素周期表标题
- ❌ 中英切换
- ❌ 部署 GitHub Pages
