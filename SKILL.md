# 旅行攻略生成 V2

## 基本信息

- **名称**: 旅行攻略生成V2
- **版本**: 4.0.1
- **基于**: 旅行攻略生成 v2.6.0
- **描述**: 生成带有**交互式腾讯地图**的旅行攻略 HTML。保持 V1 完整页面结构（Hero→概览→天气→地图→行程→酒店/预算/贴士），在每日行程前嵌入腾讯地图 JSAPI GL 模块，五色 POI 分类、双层描边路线、序号标记、点击联动飞行。v4.0 固化全部迭代规则：途牛CLI查航班/车次、先了解景点再问需求、路线串联优化、酒店搜索选择器、房型按人数、统一poiIdx、每日专属贴士、固定 HTML 模板。v4.0.1 新增 meta 长途交通段规则。
- **模板文件**: `template-攻略HTML模板.html`（本目录下，成都5日特种兵攻略的成品格式，生成新攻略时以此为底，只改 dayData/HOTEL_CANDIDATES/HOTEL_RECS 等数据，保持 CSS/JS/结构不变）

---

## V2 核心变化（vs V1）

| 特性 | V1 (旅行攻略生成) | V2 (本技能) |
|------|------------------|------------|
| 地图引擎 | 高德 APP scheme 唤起 | **腾讯地图 JSAPI GL 内嵌** |
| 地图交互 | 点击按钮跳转外部 APP | **网页内地图：打点+双层描边连线+信息窗+飞行** |
| POI 分类 | 无分类 | **景点/美食/文化/购物/夜生活 五色标记** |
| 行程展示 | 纯文字时间线 | **V1 时间轴风格 + 五色序号 + 分类标签 + 拍摄 Tips** |
| 按天切换 | 滚动 | **地图 Tab 切换，联动下方行程区** |
| 视觉风格 | 深色 Hero + 纸色背景 | **继承 V1 视觉风格（深色 Hero + 纸色卡片）** |
| 页面结构 | V1 完整模块 | **V1 结构不变，在每日行程前嵌入地图** |
| 地图联动 | 无 | **地图 Tab⇄行程、卡片→飞行+信息窗、标记→信息窗** |

---

## HTML 生成规范（V2 新增）

### 0. 整体布局（V1 风格 · 地图嵌入）

**核心原则：保持 V1 的完整页面结构，在「每日行程」前嵌入腾讯地图模块**。

桌面端垂直单列，移动端流式响应。

```
┌──────────────────────────────────────────┐
│ 🔴 HERO（暗红渐变，标题+标签+元信息）      │
├──────────────────────────────────────────┤
│ 📋 行程概览（数字卡片 + 路线文字流图）     │
├──────────────────────────────────────────┤
│ 🌤 天气参考                              │
├──────────────────────────────────────────┤
│ 🗺 路线地图 ★ V2 核心模块                 │
│  ┌[DAY1 Tab][DAY2 Tab][DAY3 Tab]──────┐  │
│  │  腾讯地图 JSAPI GL（全宽，440px）    │  │
│  │  序号标记 · 双层描边路线 · 信息窗    │  │
│  └────────────────────────────────────┘  │
├──────────────────────────────────────────┤
│ 📅 每日行程（时间轴详情卡片）             │
│  D1 · 三国蜀韵                           │
│  08:00 ─ 武侯祠 [景点]                   │
│  09:30 ─ 锦里古街 [文化]                 │
│  ...                                     │
│  D2 · 国宝都市                           │
│  ...                                     │
├──────────────────────────────────────────┤
│ 🏨 住宿推荐 / 💰 费用预估 / 📱 实用贴士  │
├──────────────────────────────────────────┤
│ Footer：数据来源 + 生成时间               │
└──────────────────────────────────────────┘
```

**移动端（≤768px）**：地图高度 260-320px，所有模块单列堆叠，触摸友好。

### 1. Hero 区域（深色渐变 · 继承 V1 风格）

```html
<div class="hero" style="background:linear-gradient(160deg,#7e1214 0%,#b52830 40%,#d4483a 100%)">
  <div class="hero-tag">CULTURE · FOOD · NATURE</div>
  <h1>{{目的地}} {{天数}}日游</h1>
  <div class="hero-sub">{{一句话描述}}</div>
  <div class="hero-meta">
    <span>{{天数}}天 · {{POI数}}个打卡点</span>
    <span>预算 ¥{{预算}}</span>
    <span>{{出发地}}出发</span>
  </div>
</div>
```
- 背景：暗红色渐变（#7e1214→#b52830→#d4483a），配合低透明度几何纹理
- 标题大字 + 元信息标签行
- 与 V1 的 Hero 风格完全一致

### 2. POI 分类与配色（五色体系）

| 分类 | 英文 key | 主色 | 背景色 | 标记文字 | 适用场景 |
|------|---------|------|--------|---------|---------|
| 景点 | `sight` | `#6366f1` 靛蓝 | `#eef2ff` | 景 | 自然/人文景点、公园 |
| 美食 | `food` | `#f59e0b` 琥珀 | `#fffbeb` | 食 | 餐厅、小吃街、夜市 |
| 文化 | `culture` | `#10b981` 翠绿 | `#ecfdf5` | 文 | 博物馆、寺庙、历史街区 |
| 购物 | `shop` | `#ec4899` 粉红 | `#fdf2f8` | 购 | 商圈、商场、文创店 |
| 夜生活 | `night` | `#8b5cf6` 紫罗兰 | `#f5f3ff` | 夜 | 酒吧街、夜景、演出 |

**分类规则**：
- 每个 POI 选择**最匹配的 1 个分类**
- 自然景区/动物园 → `sight`；古街/寺庙/博物馆 → `culture`
- 餐厅/小吃街 → `food`；商场/购物街 → `shop`
- 酒吧/夜景/夜市 → `night`
- 每天至少覆盖 3 种分类

### 3. 时间轴卡片模板（继承 V1 的 timeline 风格）

```html
<!-- 每条行程一个 tl-item，保留 V1 的圆点+时间+内容卡片结构 -->
<div class="tl-item">
  <div class="tl-dot"></div>
  <div class="tl-time">{{time}}</div>
  <div class="tl-content" onclick="focusPoi({{dayIdx}}, {{poiIdx}})">
    <div class="tl-place">
      <span class="tl-seq seq-{{cat}}">{{序号}}</span>{{name}}
    </div>
    <div class="tl-desc">{{desc}}</div>
    <div class="tl-tags">
      <span class="tag tag-{{cat}}">{{catCN}}</span>
      <span class="tag tag-tip" v-if="tip">📷 {{tip}}</span>
      <span class="tag tag-cost" v-if="cost">{{cost}}</span>
    </div>
  </div>
</div>
```

**卡片样式**：五色分类标签（tag-sight/food/culture/shop/night）+ 可选的拍摄 Tips（tag-tip）+ 费用标签（tag-cost）。hover 微右移+阴影，点击高亮蓝色边框。
**左侧圆点**：红色 accent 圆点 `box-shadow` 模拟描边环（与 V1 一致，不按分类变色）。
**序号圆圈**：在 POI 名称前用 `tl-seq` 小圆标，颜色按分类（seq-sight/food/culture/shop/night），显示 1/2/3...

**v3.2 增强（马先生反馈）**：
1. **信息窗必须换行**：腾讯地图 InfoWindow 内容容器必须设 `max-width:240px; word-break:break-all; word-wrap:break-word; white-space:normal`，否则长文本伸出弹窗。
2. **每日起讫点为酒店/民宿（可填写）**：每天行程第一个 POI 是"起点酒店"、最后一个 POI 是"终点酒店"，渲染为可编辑卡（input 填酒店名，localStorage 持久化），酒店标记用金色/teal 色并给推荐坐标（如火车站旁/镇中心）；同城连续住宿默认沿用前一日。用户填写的酒店名即时同步到地图标记。
3. **交通类 POI 必须列班次详情**：用独立"交通卡"样式显示 `车次号 · 发车→到达 · 时长 · 二等座价格`，并给出备选班次（alt）。如 `C5746 · 06:14发→07:53到 · 1小时39分 · 二等座¥130`。
4. **每日专属贴士**：每个 day-block 底部挂"🎯 当日专属贴士"卡片（3-5 条），与当天景点/交通/预约/高反/摄影强相关，比全局贴士更细。
5. **酒店卡显示房型价格**：推荐酒店卡片按人数显示 `大床房 ¥xxx/晚`（1人）与 `双床房 ¥xxx/晚`（2人+）。
6. **移动端（≤768px）**：tips-grid/hotel-grid/wx-grid 强制单列（grid-template-columns:1fr），酒店输入框全宽。

**v3.3 增强（马先生反馈·2）**：
1. **房型按人数动态切换**（不再同时显示大床+双床）：住宿推荐顶部加"出行人数"按钮组（1人/2人/3人/4人+），点击切换并 `localStorage` 持久化（key=`guide_pax`，默认1人）。1人渲染大床房价格，2+人渲染双床房价格。
2. **统一 poiIdx 索引（修复点击错位bug）**：每天构建统一 `allPois = [hotelStart, ...pois, hotelEnd]` 数组，所有 `.tl-content` 的 `onclick` 用 `allPois` 中的索引，地图 `updateMap` 也用同一数组，cards 选择器用 `#itinerary-content .tl-content` 全集。点第 N 个 → 标记和高亮第 N 个。**不要**在 hotel/poi 分开用不同索引。
3. **酒店搜索+候选+坐标回传+持久化（核心需求）**：
   - 酒店 input 设为 `readonly` + `onclick` 触发下拉
   - 下拉包含：搜索框（input 监听 input 事件，300ms 防抖）+ 推荐酒店（预置 HOTEL_CANDIDATES 字典，每张酒店卡 3-5 个真实酒店名+真实坐标）+ 搜索结果区
   - **优先调用腾讯地图 JSAPI 前端搜索** `new TMap.service.Search({pageSize:8}).searchRegion({keyword, cityName, referenceLocation: TMap.LatLng})` —— 无需 WebService key，用现有 JSAPI key 即可；酒店类泛关键词必传 `referenceLocation` 才准
   - 搜索失败/降级：用预置 HOTEL_CANDIDATES 做关键词模糊匹配
   - 选中候选 → `setHotel(key, {name, lat, lng, address})` JSON 序列化存 `localStorage`（key=`guide_hotel_<key>`）→ 关闭下拉 → `renderItinerary()` 重渲染 + `updateMap()` 更新标点（地图标记用新坐标）
   - **沿用前一日酒店逻辑**（inheritMap）：`d2_start←d1_end, d3_start←d2_end, d4_start←d3_end, d5_start←d4_end`，通过 getHotel() 优先读 localStorage，fallback 到 inheritMap[当前key] 的 localStorage，再 fallback 到默认坐标
   - 预置 HOTEL_CANDIDATES 数据结构：`{ 'd1_start':[{name, lat, lng, address}, ...], ... }`
4. **下拉 UI**：`<div class="hotel-dropdown">` 固定在酒店卡内，sticky 搜索框 + max-height:380px 滚动 + hover 高亮 + 点击选中
5. **行程概览必含车次/航班号（强制）**：route-flow 文字流图里每一段的交通必须写明车次号/航班号/发到时间，例如：
   - 飞机：`✈ 长沙黄花→成都双流 19:00-21:00 · 约2h`（具体航班号写示例+让用户自填）
   - 高铁：`🚄 C5746 06:14→07:53 成都东→黄龙九寨站 · 1h39m · 二等座¥130`
   - 动车：`🚄 C字头动车 07:00→08:20 成都→峨眉山站`
   - 拼车：`🚐 拼车 06:00→11:00 成都→四姑娘山镇`
   **禁止**在 route-flow 写"高铁去黄龙"或"拼车去四姑娘山"这种无班次时间的描述。

### 4. 地图集成（三层降级初始化 · Codex 验证方案）

```html
<script>
  // 1. 定义所有数据和函数（initMap / updateMap / renderFallbackMap 等）
  const dayData = [...];
  function initMap() { ... }
  function updateMap() { ... }
  function renderFallbackMap(dayIdx) { ... }
  function renderLeafletMap(dayIdx) { ... }
  function renderStaticFallbackMap(dayIdx) { ... }
  function loadLeafletAssets(onReady, onError) { ... }

  // 2. DOM 就绪后：先渲染行程 → 再加载地图
  document.addEventListener('DOMContentLoaded', function() {
    renderItinerary(0);   // ★ 行程不依赖地图，先渲染

    if (location.protocol === 'file:') {
      renderFallbackMap(0);  // file:// 直接降级
      return;
    }

    // 动态加载腾讯地图（不阻塞页面）
    const script = document.createElement('script');
    script.src = 'https://map.qq.com/api/gljs?v=1&key=OB4BZ-D4W3U-B7VVO-4PJWW-6TKDJ-WPB77&callback=initMap';
    script.onerror = function() { renderFallbackMap(currentMapDay); };
    document.body.appendChild(script);

    // 3 秒超时兜底
    setTimeout(function() { if (!map) renderFallbackMap(0); }, 3000);
  });
</script>
```

**⚠️ 初始化铁律（Codex 验证通过）**：

| # | 规则 | 原因 |
|---|------|------|
| 1 | **行程先渲染** | `renderItinerary(0)` 在 `DOMContentLoaded` 第一行，不依赖地图 |
| 2 | **地图动态加载** | `document.createElement('script')` 而非 `<script src>` 标签 |
| 3 | **callback 机制** | `&callback=initMap` 保证 TMap 完全就绪后才回调 |
| 4 | **file:// 检测** | 本地文件协议下腾讯地图不工作，直接走降级 |
| 5 | **三层降级** | 腾讯地图 → Leaflet（unpkg CDN）→ 静态 SVG（纯内联，零依赖） |
| 6 | **3 秒超时** | `setTimeout` 检查 `map` 是否存在，超时自动降级 |
| 7 | **所有切换入口 try/catch** | `switchMapDay()` / `focusPoi()` 对 `updateMap()` 进行 try/catch，异常走 `renderFallbackMap()` |
| 8 | **绝不依赖 `window.onload`** | 也不轮询 `setInterval` 检测 `TMap` |

**地图初始化**：
```javascript
const map = new TMap.Map('map-container', {
  center: new TMap.LatLng({{城市中心lat}}, {{城市中心lng}}),
  zoom: 12,
  mapStyleId: "style8"  // 白浅风格，不要改
});
```

**POI 标记**（序号圆形 SVG，按分类着色，非分类文字）：
```javascript
// 使用序号 ①②③...，颜色按分类，白色圆形边框
function makeMarkerSvg(num, color) {
  return 'data:image/svg+xml,<svg xmlns="http://www.w3.org/2000/svg" width="32" height="32">'
    + '<circle cx="16" cy="16" r="14" fill="' + color + '" stroke="#fff" stroke-width="2"/>'
    + '<text x="16" y="21" text-anchor="middle" fill="white" font-size="14" font-weight="bold">' + num + '</text></svg>';
}
// 每个 POI 独立 style（因序号不同），五色：景点#6366f1 美食#f59e0b 文化#10b981 购物#ec4899 夜生活#8b5cf6
```

**连线**（双层 polyline：深色描边 + 浅蓝主体，醒目有层次）：
```javascript
const paths = pois.map(p => new TMap.LatLng(p.lat, p.lng));
// 下层：深色描边（宽 8px，半透明深蓝）
new TMap.MultiPolyline({
  map, styles: { 'stroke': new TMap.PolylineStyle({
    color: 'rgba(30,58,138,0.65)', width: 8, borderWidth: 0, lineCap: 'round', lineJoin: 'round'
  })}, geometries: [{ id:'s', styleId:'stroke', paths }]
});
// 上层：浅蓝主体（宽 4px，不透明亮蓝）
new TMap.MultiPolyline({
  map, styles: { 'fill': new TMap.PolylineStyle({
    color: 'rgba(59,130,246,0.85)', width: 4, borderWidth: 0, lineCap: 'round', lineJoin: 'round'
  })}, geometries: [{ id:'f', styleId:'fill', paths }]
});
```

**信息窗**（点击 marker 弹出）：
```javascript
infoWindow.setPosition(new TMap.LatLng(p.lat, p.lng));
infoWindow.setContent('<div style="padding:10px 14px">...' + p.name + '...' + p.desc + '</div>');
infoWindow.open();
```

**飞行定位**（点击卡片 → 地图飞到 POI）：
```javascript
map.easeTo({ center: new TMap.LatLng(p.lat, p.lng), zoom: 15 }, { duration: 800 });
```

### 5. 交互逻辑

**桌面端**：
```
用户操作                    地图响应                     卡片响应
─────────                  ─────────                   ─────────
点击 DAY Tab              重新渲染该天所有 POI          重新渲染下方卡片
                          fitBounds 自适应视野
点击下方卡片              飞到 POI 位置 zoom=15         高亮边框 + 滚动到视野
                          弹出信息窗
点击地图 Marker           弹出信息窗（名称+时间+描述）   对应卡片滚动到视野
```

**移动端（≤768px）**：
```
用户操作                    地图响应                     下方内容
─────────                  ─────────                   ─────────
点击 DAY Tab              重新渲染该天所有 POI          重新渲染卡片
                          fitBounds 自适应视野
点击地图 Marker           弹出信息窗（触摸友好）         无联动
滚动查看下方卡片           无变化                        自然滚动查看详情
点击卡片                   地图飞到对应 POI             高亮卡片
```

### 6. CSS 设计系统（继承 V1 视觉风格）

```css
:root {
  --ink: #1a1f36; --paper: #fafbfc; --cream: #f5f3f0;
  --accent: #c0392b; --accent2: #635bff;
  --gold: #b8860b; --muted: #697386; --border: #e3e8ee;
  --green: #2e7d32; --blue: #1565c0; --card-bg: #ffffff;
  --shadow: 0 2px 12px rgba(0,0,0,.06);
  --cat-sight: #6366f1; --cat-food: #f59e0b; --cat-culture: #10b981;
  --cat-shop: #ec4899; --cat-night: #8b5cf6;
}
```
- 深色 Hero（暗红渐变） + 纸色 body 背景（#fafbfc）
- 白色卡片 + 米色辅助底色（--cream）
- accent 为红色系（与 V1 一致），accent2 为蓝色（联动高亮）

### 7. POI 数据获取（V2 增强）

V2 继承 V1 全部数据获取流程（知识库 → CLI → 途牛 → 天气），但有以下增强：

#### 7.1 腾讯地图 POI 搜索（新增，必须执行）

**坐标获取优先级**：
1. **知识库 geocode_cache.json** → Grep 搜索景点名获取缓存坐标
2. **腾讯地图 POI 搜索** → `tmap_client.poi_search(name, region="城市")` 获取实时坐标
3. **腾讯地图地理编码** → `tmap_client.geocoder("地址")` 兜底

```python
import sys, os
sys.path.insert(0, os.path.expanduser('~/.workbuddy/skills/skill_2062731548497326080/scripts'))
from tmap_client import TmapClient
client = TmapClient()

# 批量搜索景点坐标
for name in poi_names:
    r = client.poi_search(name, region='成都')
    # 提取 lat/lng/address/category/id
```

#### 7.2 POI 分类自动推断

根据以下规则自动为每个 POI 分配分类：
- 腾讯地图 category 字段包含"旅游景点:国家级景区/公园" → `sight`
- 包含"美食:小吃快餐/中餐厅" → `food`
- 包含"文化场馆:博物馆/寺庙/教堂" → `culture`
- 包含"购物:商业步行街/商场" → `shop`
- 包含"娱乐休闲:酒吧/KTV/夜景" → `night`
- 名称含"小吃街/美食街/夜市" → `food`
- 名称含"酒吧/酒馆/夜" → `night`
- 名称含"博物馆/院/寺/庙/宫/祠" → `culture`

---

## 工作流程（V2 调整版）

### Step 0: 需求收集（同 V1）

必须收集：目的地、出行日期、人数、天数、主题、出发城市。

**⚠️ 先了解景点，再问需求（v3.2 强制规则）**：
1. 动手排路线前，必须先了解每个目标景点的基本情况：景区内有多条线路/沟谷吗？各线路的差异（观光车/徒步/骑马、强度、时长、看点）是什么？
2. 当存在**影响行程结构的多选**时，必须用 AskUserQuestion 先问用户，禁止默认替用户选。典型必问项：
   - 多沟谷景区选哪条线（如四姑娘山：双桥沟/长坪沟/海子沟）
   - 看日出类景点前一晚住哪档（如峨眉山：金顶/雷洞坪/山脚）
   - 交通方式档位（高铁/飞机/大巴/拼车）与预算档位
3. 询问时给出选项差异说明（强度/时间/价格/看点），并标注推荐项，让用户能快速决策。
4. 用户给出路线线索（如"某地到某地有直达车"）时，先核实交通事实再排线，不要想当然。

### Step 0.3: 路线优化（v3.2 新增）

排每日路线时主动优化交通串联，减少绕路与折返：
- 善用高铁直达/经停（如川青铁路黄龙九寨站→三星堆站直达，不必先回成都）
- 顺路景点合并（如黄龙上午 + 三星堆下午 + 当晚回成都，可省一天）
- 大型景区拆到 1 天半时，把"长途交通段"放上午或傍晚，把"景区游览"放光线好的时段
- 看日出需求 → 前一晚必须住在景区内/山腰，并在 D 天标注"05:xx 起床"时间点

### Step 0.5: 知识库查询（同 V1）

Grep 知识图谱/知识网络/geocode缓存。

### Step 1: POI 坐标获取（V2 增强）

```
对每个计划访问的 POI：
  1. 先查 geocode_cache.json（知识库缓存）
  2. 缓存未命中 → 用腾讯地图 tmap_client.poi_search() 搜索
  3. 提取 lat/lng/category/id
  4. 根据 category 自动推断 POI 分类（sight/food/culture/shop/night）
```

### Step 1.5: 交通班次查询（v4.0 强制，途牛 CLI）

**所有机票/火车票段必须用途牛 CLI 查真实班次号+时刻+价格，禁止写"C字头动车""航班约2h"这种含糊描述。**

```bash
export TUNIU_API_KEY='sk-d1c4f3597df34a8eb4afc5e26ba75223'   # 备用 sk-d16ce86adcfc469c87b699f6180d0ae2
# 航班（flight）：低价 / TIME时间范围
args='{"departureCityName":"长沙","arrivalCityName":"成都","departureDate":"2026-08-21"}'
tuniu call flight searchLowestPriceFlight -a "$args"
# 火车票（train）：departureTime 按出发时间筛选
args='{"departureCityName":"成都东","arrivalCityName":"峨眉山","departureDate":"2026-08-25","departureTime":"06:00-09:00"}'
tuniu call train searchLowestPriceTrain -a "$args"
```

**要点**：
- 完整 API 参考见 `tuniu-api-ref.md`（三、航班 / 四、火车票）
- 飞机必查**机场**：成都分双流/天府（返程选双流，离市区近）；预算写**含税价**（票面+机建燃油¥120）
- 高铁/动车站名要精确（"成都东""成都南"）
- 每段给出 1 个主选班次 + 2 个备选班次（含时刻/价格）
- 查不到/失败时标注"以12306/航司实时为准"，不得用占位描述

### Step 2-6: 同 V1（内容规划/避坑/酒店/餐食/预算）

### Step 7: HTML 生成（v4.0 固定模板）

**⚠️ 必须直接复用本 skill 目录下 `template-攻略HTML模板.html`（成都5日特种兵攻略成品格式），在模板基础上只改数据，保持 CSS/JS/交互逻辑不变。** 模板包含以下已固化的功能模块（全部实测验证过，不要重构）：

| 模板功能 | 说明 |
|---------|------|
| Hero 深红渐变 | 标题/标签/元信息（预算/天数/出发地） |
| 行程概览 | 数字卡 + route-flow 文字流图（**每段必含车次号/航班号/发到时间**，v3.3 强制） |
| 天气参考 | wx-grid 卡片 |
| 🗺 路线地图 | 腾讯地图 JSAPI GL + Day Tab + 序号标记 + 双层 polyline + 信息窗（**内容强制 word-break:break-all 换行**）+ 三层降级（腾讯→Leaflet→SVG） |
| 每日行程 | 统一 `allPois = [hotelStart, ...pois, hotelEnd]` 索引（**修复点击错位**）；交通卡显示 `车次号·发到时间·价格·备选`；酒店搜索选择器；`🎯 当日专属贴士` 挂每天底部 |
| 住宿推荐 | 按 `pax`（localStorage `guide_pax`，默认1）只显示一种房型价格（1人大床/2+双床，**不显示房型文字、不显示切换按钮**） |
| 费用预估 | budget-table 详细预算 |
| 通用贴士 | tips-grid（≤768px 单列） |
| Footer | 数据来源+生成时间 |

**生成步骤**：
1. 复制 `template-攻略HTML模板.html` 为 `<目的地><天数>日游.html`
2. 改 `dayData`（日期/名称/pois/tips）→ 改 `HOTEL_CANDIDATES`（每张酒店卡 3-5 个真实酒店+坐标）→ 改 `HOTEL_RECS`（房型价格）→ 改 hero/概览/天气/预算/贴士文本
3. 保留全部 JS 函数（initMap/updateMap/buildAllPois/renderItinerary/hotelDropdown/selectHotel/降级 等）不动
4. 用 Node `new Function()` 校验内嵌 JS 语法后再交付

所有 CSS/JS 内联，单个 HTML 文件可直接打开（file:// 协议地图自动降级）。

---

## V2 HTML 固定模板结构

```
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>{{目的地}} {{天数}}日游 · {{副标题}}</title>
  <script src="https://map.qq.com/api/gljs?v=1&key=OB4BZ-D4W3U-B7VVO-4PJWW-6TKDJ-WPB77"></script>
  <style>/* V2 CSS（深色 Hero + 纸色卡片 + 移动端响应式，~400 行） */</style>
</head>
<body>
  <!-- 1. Hero：深色红色渐变，title / subtitle / meta tags -->
  <div class="hero">...</div>

  <!-- 2. 行程概览：ov-card 数字统计 + 三色 route-flow 文字路线 -->
  <div class="section"><div class="section-title">行程概览</div>...</div>

  <!-- 3. 天气参考 -->
  <div class="section"><div class="section-title">天气参考</div>...</div>

  <!-- 4. 🗺 路线地图 ★ V2 核心 -->
  <div class="map-section">
    <div class="map-card">
      <div class="map-tabs">
        <button class="map-tab active" onclick="switchMapDay(0)">DAY 1</button>
        <button class="map-tab" onclick="switchMapDay(1)">DAY 2</button>
        ...
      </div>
      <div class="map-body">
        <div id="map-container" class="map-container">
          <div class="map-legend">...</div>
        </div>
      </div>
    </div>
  </div>

  <!-- 5. 每日行程：V1 timeline 风格 day-block -->
  <div class="section" id="itinerary-section">
    <div class="section-title">每日行程</div>
    <div id="itinerary-content">...</div>
  </div>

  <!-- 6. 住宿推荐 -->
  <div class="section"><div class="section-title">住宿推荐</div>...</div>

  <!-- 7. 费用预估 -->
  <div class="section"><div class="section-title">费用预估</div>...</div>

  <!-- 8. 实用贴士 -->
  <div class="section"><div class="section-title">实用贴士</div>...</div>

  <!-- 9. Footer -->
  <div class="footer">...</div>

  <script>
    const dayData = [...];  // POI 数据（含 lat/lng/cat/time/name/desc/tip）
    let map, markerLayer, polylineLayer, polylineLayer2, infoWindow;
    function initMap() {...}       // 地图初始化 + 初始渲染
    function updateMap(dayIdx) {...}  // 切换天：重建 marker + 双层 polyline + fitBounds
    function switchMapDay(dayIdx) {...} // Tab 切天：updateMap + renderItinerary + scrollTo
    function renderItinerary(dayIdx) {...} // 渲染下方 day-block timeline
    function focusPoi(d, i) {...}    // 点击卡片 → 切天(如需) → 飞行+信息窗+高亮
    function showInfoWindow(d, i) {...} // 弹出信息窗（序号+名称+时间+分类+描述+📷Tips）
  </script>
</body>
</html>
```

**关键约束**：
- marker 标记使用序号圆形 SVG（`data:image/svg+xml,...`），每个 POI 独立 styleId
- 路线使用**双层 MultiPolyline**（layer2 深色描边 width:8 + layer1 亮蓝主体 width:4）
- 地图 Tab 切换联动下方行程区（`switchMapDay` 同时调 `updateMap` + `renderItinerary` + `scrollIntoView`）
- 所有点击行为降级到 `focusPoi` 统一处理自动切天+飞行+高亮

---

## 依赖

- **腾讯地图 JSAPI GL**（内嵌，Key: `OB4BZ-D4W3U-B7VVO-4PJWW-6TKDJ-WPB77`）
- **腾讯地图助手 Skill**（`tencentmap-map-assistant`，用于 POI 坐标搜索）
- **旅行攻略生成 V1** 全部依赖（知识库/CLI/途牛/天气/高德API）

---

## 与 V1 的关系

- **V2 独立存在**，不影响 V1
- V1 适合：纯文本攻略 + 外部 APP 导航
- V2 适合：网页内交互地图 + Stripe 视觉风格
- 两版可同时安装，用户按需选择

---

## V2 示例输出

参考文件：`成都3日游攻略.html`（本 Skill 附带示例）

---

## 更新记录

- v3.1.0 (2026-06-29) — **固定 V1 页面结构 + 地图嵌入格式**
  - Hero 改为深色红渐变（继承 V1 风格）
  - 页面结构固定为 9 模块：Hero→概览→天气→**地图**→行程→酒店→预算→贴士→Footer
  - 路线改为双层 polyline（深色描边 width:8 + 亮蓝主体 width:4）
  - 时间轴卡片改为 V1 的 tl-item/tl-dot/tl-content 结构，五色序号+分类标签+拍摄Tips
  - 地图 Tab 切换联动下方行程区（switchMapDay 三合一）
  - 移动端 260-320px 地图，全模块流式堆叠
- v3.0.0 (2026-06-29) — V2 首发：交互式腾讯地图 HTML
  - 从静态高德导航 → 内嵌腾讯地图 JSAPI GL
  - 五色 POI 分类体系（景点/美食/文化/购物/夜生活）
  - 卡片时间轴 + 地图联动
  - 基于旅行攻略生成 v2.6.0 改造
