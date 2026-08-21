# 途牛 CLI 调用参考

> **安装方式**：`npm install -g tuniu-cli@latest`（已安装 v1.0.6）
> **认证方式**：环境变量 `$env:TUNIU_API_KEY = 'sk-d1c4f3597df34a8eb4afc5e26ba75223'`
> **备用 Key**：`sk-d16ce86adcfc469c87b699f6180d0ae2`
> **文档地址**：https://open.tuniu.com/mcp/docs/apidoc/cli/intro.html
> **⚠️ 注意**：途牛走的是 CLI（`tuniu call`），不是 HTTP API！

---

## 一、通用调用方式

### 基本格式

```powershell
# 1. 设置 API Key（每次会话必须）
$env:TUNIU_API_KEY = 'sk-d1c4f3597df34a8eb4afc5e26ba75223'

# 2. 查看可用服务
tuniu list

# 3. 查看某服务下的工具
tuniu list train
tuniu list hotel
tuniu list ticket
tuniu list flight

# 4. 调用工具（核心格式）
tuniu call <server> <tool> -a '<JSON参数>'
```

### ⚠️ PowerShell 引号陷阱（必读）

PowerShell 中直接 `-a '{"key":"value"}'` 会导致双引号被吃掉，JSON 解析失败。
**正确写法：先将 JSON 赋值给变量，再用 `-a $args`：**

```powershell
# ✅ 正确方式
$args = '{\"key\":\"value\"}'
tuniu call server tool -a $args

# ❌ 错误方式（PowerShell 会吃掉 JSON 的双引号）
tuniu call server tool -a '{"key":"value"}'
```

### 调试模式

```powershell
tuniu call server tool -a $args -d   # -d 打印完整请求/响应
```

### 服务映射

| 服务 | CLI 命令前缀 | 用途 |
|------|------------|------|
| `train` | `tuniu call train` | 火车票搜索/详情/预订 |
| `hotel` | `tuniu call hotel` | 酒店搜索/详情/预订 |
| `ticket` | `tuniu call ticket` | 门票查询/预订 |
| `flight` | `tuniu call flight` | 机票搜索/舱位/预订 |
| `cruise` | `tuniu call cruise` | 邮轮搜索/预订 |
| `holiday` | `tuniu call holiday` | 度假产品搜索/预订 |

---

## 二、酒店

### 工具列表

| 工具名 | 功能 |
|--------|------|
| `tuniu_hotel_search` | 酒店搜索（支持翻页） |
| `tuniu_hotel_detail` | 酒店详情查询（房型报价） |
| `tuniu_hotel_create_order` | 创建预订订单 |

### 2.1 酒店搜索 `tuniu_hotel_search`

**请求参数**

| 参数 | 类型 | 必需 | 说明 |
|:---|:---|:---|:---|
| `cityName` | string | ✅ | 城市名称（如"黄山"） |
| `checkIn` | string | ✅ | 入住日期 YYYY-MM-DD |
| `checkOut` | string | ✅ | 离店日期 YYYY-MM-DD |
| `prices` | string | ❌ | 价格区间（如"100-400"）⚠️ **参数名是 prices，不是 priceRange** |
| `keyword` | string | ❌ | 关键词（酒店名、地标） |
| `adultNum` | number | ❌ | 成人数（默认2） |
| `queryId` | string | 翻页用 | 首次搜索返回的ID |
| `pageNum` | number | 翻页用 | 页码（从2开始） |

**调用示例**

```powershell
$env:TUNIU_API_KEY = 'sk-d1c4f3597df34a8eb4afc5e26ba75223'
$args = '{\"cityName\":\"黄山\",\"checkIn\":\"2026-06-25\",\"checkOut\":\"2026-06-26\"}'
tuniu call hotel tuniu_hotel_search -a $args

# 翻页
$args = '{\"queryId\":\"1368250179952896\",\"pageNum\":2}'
tuniu call hotel tuniu_hotel_search -a $args
```

**响应关键字段**

```json
{
  "queryId": "1368250179952896",
  "totalPageNum": 2,
  "currentPageNum": 1,
  "cityInfo": { "cityCode": 113, "cityName": "黄山" },
  "hotels": [{
    "hotelId": 1964176219,
    "hotelName": "桔子酒店（黄山风景区南大门汤口店）",
    "starName": "高档型",
    "brandName": "桔子酒店",
    "business": "汤口镇",
    "commentScore": 4.8,
    "commentDigest": "黄山脚下，尽享便捷与舒适。",
    "lowestPrice": 231,
    "address": "205国道天都大市场B楼105号",
    "roomName": "大床房",
    "roomArea": "28㎡",
    "roomWindow": "有窗",
    "meal": "无早餐",
    "refund": "限时取消",
    "firstPic": "https://m.tuniucdn.com/..."
  }]
}
```

### 2.2 酒店详情 `tuniu_hotel_detail`

**请求参数**

| 参数 | 类型 | 必需 | 说明 |
|:---|:---|:---|:---|
| `hotelId` | number | ⚠️二选一 | 酒店ID |
| `hotelName` | string | ⚠️二选一 | 酒店名称 |
| `checkIn` | string | ✅ | 入住日期 YYYY-MM-DD |
| `checkOut` | string | ✅ | 离店日期 YYYY-MM-DD |

**调用示例**

```powershell
$args = '{\"hotelId\":1964176219,\"checkIn\":\"2026-06-25\",\"checkOut\":\"2026-06-26\"}'
tuniu call hotel tuniu_hotel_detail -a $args
```

**响应关键字段**

```json
{
  "hotelId": 1964176219,
  "hotelName": "桔子酒店（黄山风景区南大门汤口店）",
  "starName": "高档型",
  "commentScore": 4.8,
  "roomTypes": [{
    "roomTypeId": "rt_001",
    "roomTypeName": "标准房",
    "bedType": "大床",
    "ratePlans": [{
      "ratePlanName": "可取消",
      "rmbPrices": "298",
      "mealText": "含早",
      "cancelDesc": "免费取消",
      "preBookParam": "eyJ0eXAiOiJKV1Q..."  // ⚠️ 下单必需，有效期30分钟
    }]
  }]
}
```

### 2.3 创建酒店订单 `tuniu_hotel_create_order`

| 参数 | 类型 | 必需 | 说明 |
|:---|:---|:---|:---|
| `hotelId` | string | ✅ | 酒店ID |
| `roomId` | string | ✅ | 房型ID（roomTypeId） |
| `preBookParam` | string | ✅ | 预订参数（来自详情） |
| `checkInDate` | string | ✅ | 入住日期 |
| `checkOutDate` | string | ✅ | 离店日期 |
| `roomCount` | number | ✅ | 房间数量 |
| `roomGuests` | array | ✅ | `[{"guests":[{"firstName":"三","lastName":"张"}]}]` |
| `contactName` | string | ✅ | 联系人姓名 |
| `contactPhone` | string | ✅ | 联系人电话 |

---

## 三、火车票

### 工具列表

| 工具名 | 功能 |
|--------|------|
| `searchLowestPriceTrain` | 车次搜索 |
| `queryTrainDetail` | 车次详情（含 seatInfo.resId） |
| `bookTrain` | 预订下单 |
| `queryTrainOrderDetail` | 查询订单 |
| `cancelOrder` | 取消订单 |

### 3.1 车次搜索 `searchLowestPriceTrain`

**请求参数**

| 参数 | 类型 | 必需 | 说明 |
|:---|:---|:---|:---|
| `departureCityName` | string | ✅ | 出发城市（如"长沙"） |
| `arrivalCityName` | string | ✅ | 到达城市（如"黄山"） |
| `departureDate` | string | ✅ | 出发日期 YYYY-MM-DD |
| `departureTime` | string | ❌ | 出发时间范围，如"08:00-12:00" |
| `pageNum` | number | 翻页用 | 页码（从2开始） |
| `queryId` | string | 翻页用 | 快照ID |

**调用示例**

```powershell
$args = '{\"departureCityName\":\"长沙\",\"arrivalCityName\":\"黄山\",\"departureDate\":\"2026-06-25\"}'
tuniu call train searchLowestPriceTrain -a $args
```

**实测返回**（长沙→黄山北 2026-06-25）

| 车次 | 出发 | 到达 | 耗时 | 二等座 | 一等座 |
|------|------|------|------|--------|--------|
| G2760 | 07:23 | 11:16 | 3h53m | ¥391 | ¥640 |
| G2186 | 16:59 | 20:32 | 3h33m | ¥395 | ¥639 |
| G1184 | 11:24 | 19:18 | 7h54m | ¥695 | ¥1111 |

**响应关键字段**

```json
{
  "successCode": true,
  "queryId": "79970923708449",
  "totalPageNum": 1,
  "data": [{
    "trainNum": "G2760",
    "departStationName": "长沙南",
    "destStationName": "黄山北",
    "departureTime": "2026-06-25 07:23",
    "arrivalTime": "2026-06-25 11:16",
    "duration": "3时53分",
    "price": {
      "edzPrice": "391",     // 二等座
      "ydzPrice": "640",     // 一等座
      "swzPrice": "1349",    // 商务座
      "wzPrice": "391"       // 无座
    },
    "seatAvailable": {
      "edzNum": 0,           // 二等座余票（0=没票了！）
      "ydzNum": 0,
      "swzNum": 0,
      "wzNum": 0
    }
  }]
}
```

### 3.2 车次详情 `queryTrainDetail`

**请求参数**

| 参数 | 类型 | 必需 | 说明 |
|:---|:---|:---|:---|
| `departureStationName` | string | ✅ | 出发站名（如"长沙南"） |
| `arrivalStationName` | string | ✅ | 到达站名（如"黄山北"） |
| `departureDate` | string | ✅ | 出发日期 yyyy-MM-dd |
| `trainNum` | string | ✅ | 车次号（如"G2760"） |

**调用示例**

```powershell
$args = '{\"departureStationName\":\"长沙南\",\"arrivalStationName\":\"黄山北\",\"departureDate\":\"2026-06-25\",\"trainNum\":\"G2760\"}'
tuniu call train queryTrainDetail -a $args
```

**响应关键字段**

```json
{
  "successCode": true,
  "data": {
    "departsDate": "2026-06-25",
    "seatInfo": [{
      "seatName": "二等座",
      "price": 391,
      "resId": 1980477395,    // ⚠️ 下单必需
      "leftNumber": 99,
      "seatStatus": "有"
    }]
  }
}
```

---

## 四、门票

### 工具列表

| 工具名 | 功能 |
|--------|------|
| `query_cheapest_tickets` | 门票查询 |
| `create_ticket_order` | 创建订单 |

### 4.1 门票查询 `query_cheapest_tickets`

**请求参数**

| 参数 | 类型 | 必需 | 说明 |
|:---|:---|:---|:---|
| `scenic_name` | string | ✅ | 景点名称（如"黄山风景区"） |

**调用示例**

```powershell
$args = '{\"scenic_name\":\"黄山风景区\"}'
tuniu call ticket query_cheapest_tickets -a $args
```

**响应关键字段**

```json
{
  "scenic_name": "黄山风景区",
  "tickets": [{
    "productId": 321837770,
    "scenicName": "黄山风景区",
    "resId": "2126810981",
    "resName": "北大门(太平方向)门票成人票_10:00-11:00入园",
    "startPrice": "190.0",
    "personTypeName": "成人票",
    "ticketTypeName": "单票",
    "lossName": "随时退",
    "scenicSpotStar": 5,
    "satisfaction": 100.0,
    "detailAddr": "安徽省黄山市黄山区205国道",
    "scenicOpenInfo": {
      "showOpenStartTime": "07:00",
      "showOpenEndTime": "17:10"
    }
  }]
}
```

---

## 五、机票

### 工具列表

| 工具名 | 功能 |
|--------|------|
| `searchLowestPriceFlight` | 航班搜索（6种模式） |
| `multiCabinDetails` | 舱位价格查询 |
| `getBookingRequiredInfo` | 获取预订必填信息 |
| `saveOrder` | 创建订单 |
| `cancelOrder` | 取消订单 |

### 5.1 航班搜索 `searchLowestPriceFlight`

**请求参数**

| 参数 | 类型 | 必需 | 说明 |
|:---|:---|:---|:---|
| `departureCityName` | string | ✅ | 出发城市 |
| `arrivalCityName` | string | ✅ | 到达城市 |
| `departureDate` | string | ✅ | 出发日期 YYYY-MM-DD |

**调用示例**

```powershell
$args = '{\"departureCityName\":\"长沙\",\"arrivalCityName\":\"黄山\",\"departureDate\":\"2026-06-25\"}'
tuniu call flight searchLowestPriceFlight -a $args
```

---

## 六、核心命令速查

### 全部 9 个命令

| 命令 | 功能 | 常用示例 |
|------|------|----------|
| `tuniu help` | 帮助（支持查看工具参数） | `tuniu help ticket query_cheapest_tickets` |
| `tuniu list` | 列出服务/工具 | `tuniu list` / `tuniu list train` |
| `tuniu call` | 调用工具（**核心**） | `tuniu call train searchLowestPriceTrain -a $args` |
| `tuniu health` | 健康检查 | `tuniu health` / `tuniu health --parallel` |
| `tuniu schema` | 导出工具 Schema | `tuniu schema --output json` |
| `tuniu config` | 配置管理 | `tuniu config init` / `tuniu config show` |
| `tuniu skill` | Skill 安装 | `tuniu skill install --agent cursor` |
| `tuniu completion` | Shell 补全 | `tuniu completion bash --install` |
| `tuniu discovery` | 服务发现 | `tuniu discovery refresh` |

### 6.1 查看工具参数：`tuniu help`

```powershell
# 查看工具的功能说明、参数列表、必填项
tuniu help ticket query_cheapest_tickets
tuniu help hotel tuniu_hotel_search
tuniu help train searchLowestPriceTrain
```

### 6.2 全局选项

| 选项 | 短选项 | 说明 | 示例 |
|------|--------|------|------|
| `--detail` | `-d` | 调试模式，打印完整请求/响应 | `tuniu call ... -d` |
| `--output` | `-o` | 输出格式：json / table / yaml | `tuniu list -o table` |
| `--profile` | `-p` | 环境配置（默认 production） | `tuniu list -p development` |
| `--config` | `-c` | 指定配置文件路径 | `tuniu config show -c ./config.json` |
| `--timeout` | `-t` | 超时时间（秒） | `tuniu call ... -t 60` |
| `--version` | `-V` | 显示版本号 | `tuniu -V` |

### 6.3 模拟执行：`--dry-run`

```powershell
# 不实际调用服务，仅验证参数格式
$args = '{\"scenic_name\":\"黄山风景区\"}'
tuniu call ticket query_cheapest_tickets -a $args --dry-run
```

---

## 七、调试与错误处理

### 7.1 退出码速查

| 退出码 | 含义 | 操作 |
|--------|------|------|
| `0` | 成功 | 解析 stdout JSON |
| `101` | 连接失败 | 重试或检查网络 |
| `102` | 工具不存在 | 运行 `tuniu list <server>` 检查工具名 |
| `103` | 参数错误 | 运行 `tuniu help <server> <tool>` 查看参数 |
| `104` | 认证失败 | 检查 `$env:TUNIU_API_KEY` |
| `105` | 超时 | 增加 `-t 60` 或检查网络 |
| `106` | 服务器错误 | 联系途牛或稍后重试 |
| `107` | 配置错误 | 运行 `tuniu config show` 检查 |
| `108` | 未配置 API Key | 设置 `$env:TUNIU_API_KEY` 环境变量 |
| `199` | 未知错误 | 使用 `-d` 查看详情 |

### 7.2 调试命令

```powershell
# 打印完整请求/响应
tuniu call train searchLowestPriceTrain -a $args -d

# 增加超时
tuniu call hotel tuniu_hotel_search -a $args -t 60

# 检查连通性
tuniu health --parallel

# 查看配置文件
tuniu config show

# 重新初始化配置（Key 过期/更换时）
tuniu config init --force
```

---

## 八、高级功能

### 8.1 服务发现 `tuniu discovery`

```powershell
# 查看服务发现状态与缓存信息
tuniu discovery status

# 列出当前可用服务
tuniu discovery list

# 刷新服务列表（服务端新增工具时使用）
tuniu discovery refresh
```

### 8.2 导出 Schema `tuniu schema`

```powershell
# 导出所有工具的参数定义（供 Agent 初始化）
tuniu schema --output json

# 仅导出指定服务
tuniu schema ticket --output json

# Markdown 格式
tuniu schema --output markdown
```

### 8.3 配置管理 `tuniu config`

```powershell
# 初始化配置文件 ~/.tuniu-mcp/config.json
tuniu config init

# 强制覆盖已有配置
tuniu config init --force

# 查看当前配置
tuniu config show
```

### 8.4 健康检查 `tuniu health`

```powershell
# 检查默认服务
tuniu health

# 并行检查所有服务（ticket/hotel/flight/train/cruise/holiday）
tuniu health --parallel -o table
```

---

## 九、踩坑经验

1. **PowerShell 引号**：绝对不能 `-a '{"key":"value"}'`，双引号会被吃掉。必须用 `$args = '{\"key\":\"value\"}'`
2. **Key 设置**：每次新 PowerShell 会话都要重新 `$env:TUNIU_API_KEY = '...'`
3. **酒店价格筛选**：参数名是 `prices`（复数），不是 `priceRange`
4. **火车票**：`departureCityName` 传城市名（如"长沙"），不需要加"南"等站名
5. **余票判断**：`seatAvailable.edzNum: 0` 表示该座位类型已售罄
6. **下单 URL**：所有下单接口返回后必须提醒用户点击 `paymentUrl` / `orderDetailUrl` 完成支付
7. **截止时间过短**：实测正常可用，不是 Key 或 API 问题
8. **退出码优先看 104/108**：认证问题最常见，确认 `$env:TUNIU_API_KEY` 已设置
9. **--dry-run 先测试**：不确定参数格式时，先用 `--dry-run` 验证再正式调用
10. **推荐先跑 schema**：每次生成攻略前执行 `tuniu schema --output json` 获取最新工具定义

---

*最后更新：2026-05-27（已整合官方使用指南、FAQ、CLI命令列表三份文档）*

---

## 三、航班（2026-08-20 实测可用）

### 工具列表

| 工具名 | 功能 |
|--------|------|
| `searchLowestPriceFlight` | 查询国内机票航班列表（6种查询模式） |
| `multiCabinDetails` | 查询航班舱位价格（下单用 cabinPriceId） |
| `getBookingRequiredInfo` | 下单前获取必填信息 |
| `saveOrder` | 提交机票订单 |
| `cancelOrder` | 取消订单 |

### 3.1 航班搜索 `searchLowestPriceFlight`

**请求参数**

| 参数 | 类型 | 必需 | 说明 |
|:---|:---|:---|:---|
| `departureCityName` | string | ✅ | 出发城市（如"长沙"） |
| `arrivalCityName` | string | ✅ | 到达城市（如"成都"） |
| `departureDate` | string | ✅ | 出发日期 yyyy-MM-dd |
| `searchType` | string | ❌ | 查询模式：不传=低价；`TIME`=按时间范围（配 departureTime/arrivalTime）；`PRICE`=价格区间；`TRANSFER`=中转 |
| `departureTime` | string | ❌ | 起飞时间范围（如"18:00-23:59"），配 searchType=TIME |
| `priceRange` | string | ❌ | 价格区间（如"500-1000"），配 searchType=PRICE |

**返回字段**：`flightNumber`(航班号)、`airlineCompany`(航司)、`departureTime/arrivalTime`(起降时间)、`departureAirport/arrivalAirport`(机场，注意区分**双流/天府**)、`basePrice`(票面价)、`totalTax`(机建燃油税)、`remainingSeats`、`type`(直飞/中转)

**调用示例**

```bash
export TUNIU_API_KEY='sk-d1c4f3597df34a8eb4afc5e26ba75223'
# 低价查询（默认）
args='{"departureCityName":"长沙","arrivalCityName":"成都","departureDate":"2026-08-21"}'
tuniu call flight searchLowestPriceFlight -a "$args"
# 时间范围查询（如晚班机）
args='{"departureCityName":"成都","arrivalCityName":"长沙","departureDate":"2026-08-26","searchType":"TIME","departureTime":"18:00-23:59"}'
tuniu call flight searchLowestPriceFlight -a "$args"
```

**⚠️ 实战要点（马先生行程验证）**：
- 成都分**双流/天府**两个机场：返程选双流起飞（离市区近，CA4393 19:55→21:50 等）；天府机场离市区1h+，赶飞机必须预留
- 长沙→成都周五晚（18:00后）性价比班次少：EU1802 21:00→23:00 ¥670+税120 是最优晚班机
- 机票预算应写**含税价**（票面价+机建燃油¥120）

---

## 四、火车票（动车/高铁/普速，2026-08-20 实测可用）

### 工具列表

| 工具名 | 功能 |
|--------|------|
| `searchLowestPriceTrain` | 查询火车票低价车次列表 |
| `queryTrainDetail` | 查询车次详情（含 seatInfo，预订用 resId） |
| `bookTrain` | 火车票预订 |
| `queryTrainOrderDetail` | 订单详情查询 |

### 4.1 车次搜索 `searchLowestPriceTrain`

**必传**：departureCityName(出发站，如"成都东")、arrivalCityName(到达站，如"峨眉山")、departureDate(yyyy-MM-dd)
**可选**：departureTime（如"06:00-09:00"按出发时间筛选）、arrivalTime（按到达时间筛选）

**返回**：data 数组中每项含 trainNum(车次号)、departureTime、arrivalTime、totalRunTime(历时)、secondSeatPrice(二等座)等

**调用示例**：
```bash
export TUNIU_API_KEY='sk-d1c4f3597df34a8eb4afc5e26ba75223'
# 早班动车
args='{"departureCityName":"成都东","arrivalCityName":"峨眉山","departureDate":"2026-08-25","departureTime":"06:00-09:00"}'
tuniu call train searchLowestPriceTrain -a "$args"
```

**实战要点（马先生行程验证）**：
- 成都⇄峨眉山动车 1h-1h15m，途牛可查实班次号
- 站名要精确到"成都东"或"成都南"（成都站主用东站/南站；12306 也搜"成都"但返回多个站）
- 返程动车务必留 1.5h+ 余量（景区下山+打车到站+候车）

---

## 四、火车票（动车/高铁/普速，2026-08-20 实测可用）

### 工具列表

| 工具名 | 功能 |
|--------|------|
| `searchLowestPriceTrain` | 查询火车票低价车次列表 |
| `queryTrainDetail` | 查询车次详情（含 seatInfo，预订用 resId） |
| `bookTrain` | 火车票预订 |
| `queryTrainOrderDetail` | 订单详情查询 |

### 4.1 车次搜索 `searchLowestPriceTrain`

**必传**：departureCityName(出发站，如"成都东")、arrivalCityName(到达站，如"峨眉山")、departureDate(yyyy-MM-dd)
**可选**：departureTime（如"06:00-09:00"按出发时间筛选）、arrivalTime（按到达时间筛选）

**返回**：data 数组中每项含 trainNum(车次号)、departureTime、arrivalTime、totalRunTime(历时)、secondSeatPrice(二等座)等

**调用示例**：
```bash
export TUNIU_API_KEY='sk-d1c4f3597df34a8eb4afc5e26ba75223'
# 早班动车
args='{"departureCityName":"成都东","arrivalCityName":"峨眉山","departureDate":"2026-08-25","departureTime":"06:00-09:00"}'
tuniu call train searchLowestPriceTrain -a "$args"
```

**实战要点（马先生行程验证）**：
- 成都⇄峨眉山动车 1h-1h15m，途牛可查实班次号
- 站名要精确到"成都东"或"成都南"（成都站主用东站/南站；12306 也搜"成都"但返回多个站）
- 返程动车务必留 1.5h+ 余量（景区下山+打车到站+候车）
