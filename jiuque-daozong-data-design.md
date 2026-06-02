# 久雀到综数据口径与开发文档

> 日期：2026-05-29  
> 客户：久雀  
> 业务线：到综 / 到店综合 / 棋牌茶楼  
> demo：https://jiuque.jingyingcanmou.cn/  
> 依据：9 张后台截图 + 9 个实际下载 Excel 样本

## 0. 结论

本批样本已经足够启动一期 parser 和数据表设计。美团经营宝侧可以覆盖门店客流、商品访问、商品交易、评价汇总、评价明细、榜单；抖音来客侧可以覆盖交易、商品、评价明细。

表设计按最新对齐口径修正：`meituan`、`dianping`、`douyin` 继续作为平台维度；“到店综合”不新增为平台，不再设计综合平台码。新增业态维度 `business_vertical`，其中 `dining` 表示到店餐饮，`general` 表示到店综合，例如棋牌、KTV、休闲娱乐。

一期建议建设“平台 Excel 经营看板”，覆盖 demo 的日常数据跟进、平台数据卡片、部分异常监控。demo 中的单包间产值、翻台率、私域好友、用户标签、用户列表、设备合规、回本周期，不能从本批美团/抖音 Excel 直接得到，需要二期接入小程序/POS/房型/会员/企微/财务数据。

截图说明：HTML 版本已把 `Image #1` 到 `Image #9` 放到 `docs/assets/jiuque-daozong/`，并支持点击截图预览大图。Markdown 版本保留同样的图号锚点，方便和 HTML 对照。

## 1. 入口与样本总表

| 图号 | 平台 | 后台入口 | 下载动作 | Excel 样本 | 行数 | 数据粒度 | 建议 artifact_type |
| --- | --- | --- | --- | --- | ---: | --- | --- |
| Image #1 | 美团经营宝 | 经营参谋 > 评价分析 > 评价概览 | 下载明细表格 | `/Users/adonis/Downloads/评价数据-20260526-20260528-282358728-1780037217695339.xlsx` | 5,061 | 日期 x 门店 | `meituan_general_review_stats_daily` |
| Image #2 | 美团经营宝 | 评价管理 > 门店评价 > 评价明细 | 下载评论 | `/Users/adonis/Downloads/门店评价-2823587281780037356972.xlsx` | 279 | 评价明细 | `meituan_general_review_comments` |
| Image #3 | 美团经营宝 | 经营参谋 > 商品分析 | 下载明细 | `/Users/adonis/Downloads/商品分析-20260526-20260528-282358728-1780037485558215.xlsx` | 41,655 | 日期 x 门店 x 商品 | `meituan_general_product_traffic` |
| Image #4 | 美团经营宝 | 经营参谋 > 交易分析 | 下载明细表格 | `/Users/adonis/Downloads/商品交易数据-20260526-20260528-282358728-1780037645844689.xlsx` | 16,962 | 日期 x 门店 x 商品 | `meituan_general_product_transaction` |
| Image #5 | 美团经营宝 | 经营参谋 > 客流分析 > 客流概览 | 下载明细表格 | `/Users/adonis/Downloads/客流数据-20260522-20260528-282358728-1780037925110274.xlsx` | 11,809 | 日期 x 门店 | `meituan_general_store_traffic` |
| Image #6 | 美团经营宝 | 经营参谋 > 榜单分析 | 导出榜单明细 | `/Users/adonis/Downloads/榜单数据_仅展示上榜门店-20260526-20260528-282358728-178003805511437.xlsx` | 26,421 | 日期 x 门店 x 榜单 | `meituan_general_ranking` |
| Image #7 | 抖音来客 | 店铺管理 > 评价管理 | 右上角导出，弹窗选择门店和评价时间 | `/Users/adonis/Downloads/门店评价_全部门店_2026_05_26-2026_05_28.xlsx` | 802 | 评价明细 | `douyin_general_review` |
| Image #8 | 抖音来客 | 报表集市 > 报表下载 > 交易 | 交易 tab 导出，汇总方式选“按日” | `/Users/adonis/Downloads/2026_05_28_交易 (1).xlsx` | 16,820 | 日期 x 门店 x 商品 x 交易体裁 | `douyin_general_transaction` |
| Image #9 | 抖音来客 | 报表集市 > 报表下载 > 商品 | 商品 tab 导出，汇总方式选“按日” | `/Users/adonis/Downloads/2026_05_28_商品 (1).xlsx` | 27,224 | 日期 x 商品 x 投放渠道 | `douyin_general_product` |

关键限制：

1. 抖音交易和商品后台支持“按日”汇总。到综 demo 按日展示时，RPA 必须在报表集市选择“按日”后导出；如果文件表头仍出现 `周`，应判定为导出配置错误或历史样本，不作为日粒度口径入库。
2. 抖音商品样本没有门店 ID，只有 `适用门店数`。它适合品牌/商品层看板，不适合直接做单店商品访问分析。单店抖音商品流量需要导出时勾选门店维度，或只用交易表做单店交易口径。
3. `Image #7` 已替换为抖音评价管理导出弹窗截图。开发时仍需确认导出按钮触发后的下载任务、文件生成耗时和文件命名规则。

## 2. 图文入口说明

### 图 1：美团评价分析汇总

```text
截图锚点：Image #1
页面路径：美团经营宝 > 经营参谋 > 评价分析
关键筛选：门店、时间（日/月/自定义）
下载按钮：页面右侧“下载明细表格”
```

用途：给 demo 的“总好评数、评价数、差评率/差评回复率、评价异常监控”提供门店日汇总。

Excel 表头：

```text
日期, 平台, 省份, 城市, 点评门店id, 门店名称,
新增评价数, 新增差评数, 差评回复率, 新增好评数, 累计评价数, 已拦截违规评价数
```

入库建议：

- 复用表：`store_reviews`
- 周期：按 `date` 日粒度入库，周期元信息写入 artifact metadata。
- 唯一键：沿用现有 `UNIQUE(store_id, date, platform)`。
- 口径：`platform` 保持 `meituan` / `dianping`，业态从 `stores.business_vertical='general'` 或连接继承。

### 图 2：美团门店评价详情

```text
截图锚点：Image #2
页面路径：美团经营宝 > 评价管理 > 门店评价 > 评价明细
关键筛选：门店、平台、日期范围、星级、回复状态、投诉状态、来源
下载按钮：页面右侧“下载评论”
```

用途：评价明细、AI 话术、差评识别、回复状态、评价内容检索。

Excel 表头：

```text
评价时间, 城市, 评价门店, 点评门店ID, 美团门店ID, 用户昵称,
星级, 评分, 评价内容, 评价正文字数, 图片数, 视频数,
商家是否已经回复, 商家首次回复时间, 是否消费后评价, 消费时间
```

入库建议：

- 复用表：`store_comments`
- 星级：`5.0星` 解析为 `5.0`
- 子评分：`服务:3.5, 设施:4.5, 划算:3.5` 分别映射到 `service_score`、`environment_score` 等现有字段；`划算` 没有稳定字段时原文进 `raw_data`。
- 去重：没有稳定评价 ID，使用内容 hash 生成 `platform_comment_id`。

### 图 3：美团商品分析

```text
截图锚点：Image #3
页面路径：美团经营宝 > 经营参谋 > 商品分析
关键筛选：时间范围、商品类型、服务项目、适用门店、用户来源
下载按钮：页面右侧“下载明细”
```

用途：商品访问、商品下单、商品访购率、商品下单金额。适合支撑 demo 的“商品/套餐漏斗”和“价格敏感度”弱口径。

Excel 表头：

```text
日期, 平台, 点评门店id, 门店名称, 商品ID, 商品名称,
商品最新售价, 商品访问人数, 商品下单人数, 商品访购率,
商品下单券数, 商品退款券数, 商品下单金额
```

入库建议：

- 复用表：`product_detail`
- 周期：按 `date` 日粒度入库，周期元信息写入 artifact metadata。
- 唯一键：沿用现有 `UNIQUE(store_id, date, platform, product_id)`。
- 注意：样例中存在大量空值，parser 需要把空字符串解析为 null/0，不能抛错。

### 图 4：美团交易分析

```text
截图锚点：Image #4
页面路径：美团经营宝 > 经营参谋 > 交易分析
关键筛选：交易概览、核销及退款分析、商品下单人数排名
下载按钮：页面右侧“下载明细表格”
```

用途：美团侧“营业额/订单量/核销收入”的核心来源。

Excel 表头：

```text
日期, 商品类型, 商品ID, 商品名称, 省份, 城市, 点评门店ID, 门店名称,
下单人数, 下单券数, 下单金额（原价）, 下单金额,
核销人数, 核销券数, 核销金额（原价）, 核销金额,
退款券数, 退款金额（原价）
```

入库建议：

- 复用表：`product_detail`，必要时按门店日聚合回写 `store_transactions` / `refunds`
- 查询口径：按 `business_vertical='general'` 过滤，不新增综合平台
- 默认营业额口径：`核销金额`
- GMV 备用口径：`下单金额`
- 订单量默认口径：`核销券数`

### 图 5：美团客流分析

```text
截图锚点：Image #5
页面路径：美团经营宝 > 经营参谋 > 客流分析 > 客流概览
关键筛选：门店、平台（点评+美团）、日/月、日期范围
下载按钮：页面右侧“下载明细表格”
```

用途：曝光、访问、意向转化、下单、收藏、打卡。对应 demo 平台卡片里的流量漏斗。

Excel 表头：

```text
日期, 省份, 城市, 门店ID, 门店名称,
曝光人数, 曝光次数, 访问人数, 访问次数,
曝光访问转化率, 意向转化人数, 意向转化率,
下单人数, 留资人数, 累计收藏人数, 新增收藏人数, 新增打卡人数
```

入库建议：

- 复用表：`store_traffic`
- `门店ID` 在本表未区分美团/点评，入库时先通过 artifact 平台和门店映射确认 `stores.id`。
- `留资人数` 可作为 demo 里“线索/咨询”类指标的候选来源。

### 图 6：美团榜单分析

```text
截图锚点：Image #6
页面路径：美团经营宝 > 经营参谋 > 榜单分析
关键筛选：门店、日期、平台、榜单 tab
下载按钮：页面右侧“导出榜单明细”
```

用途：城市/行政区/商圈榜单排名，支撑 demo 的榜单排名和竞争环境监控。

Excel 表头：

```text
平台, 日期, 省份, 城市, 点评门店ID, 美团门店ID, 门店名称,
榜单名称, 榜单一级分类, 榜单二级分类,
榜单评选城市, 城市排名,
榜单评选行政区, 行政区排名,
榜单评选商圈, 商圈排名
```

入库建议：

- 复用表：`store_rankings`
- 排名字段允许文本：数字、`未上榜`、空值都要保留。
- demo 默认展示商圈排名；如果商圈排名为 `未上榜`，降级展示行政区或城市排名。

### 图 7：抖音评价

```text
截图锚点：Image #7
页面路径：抖音来客 > 店铺管理 > 评价管理
下载动作：右上角“导出”，弹窗选择所属门店和评价时间后点击“导出”
```

Excel 表头：

```text
门店名称, 门店ID, 门店备注名, 门店所在省份, 门店所在城市, 门店所在区域,
用户昵称, 用户等级, 评价时间, 推荐等级, 文本内容, 图片内容, 视频内容,
是否为消费后评价, 评价来源, 商品ID, 消费商品,
商家是否回复, 商家回复内容, 商家回复时间
```

入库建议：

- 复用表：`douyin_review`
- `推荐等级`：`5星` 解析为 `5.0`
- 图片和视频内容可能是多 URL 换行字符串，先原样保存到 text 字段，后续再拆数组。
- `评价来源` 样例为 `抖音团购`，可作为渠道字段。

### 图 8：抖音交易报表

```text
截图锚点：Image #8
页面路径：抖音来客 > 报表集市 > 报表下载 > 交易
关键筛选：汇总方式、统计周期、门店范围、商品范围
维度勾选：区域、省份、城市、门店、商品、交易体裁一级、交易体裁二级
```

用途：抖音侧核销金额、意向成交金额、退款金额，适合做营业数据和渠道/交易体裁分析。

目标按日导出表头：

```text
日期, 区域户ID, 门店省份, 门店城市, 门店ID, 商品ID,
交易体裁(一级), 交易体裁(二级), 区域, 门店, 商品,
核销金额, 核销券数, 核销人数, 核销客单价,
意向成交金额, 意向成交券数, 意向成交人数, 意向成交客单价,
意向退款金额, 意向退款券数, 意向退款人数
```

入库建议：

- 复用表：`douyin_transactions_detail`
- 查询口径：按日导出、按日入库；如果导出结果是 `周` 字段，需要重新按日导出。
- 周期：`period_type='day'`
- `交易体裁(一级/二级)` 继续使用现有字段，用于渠道分析。

### 图 9：抖音商品报表

```text
截图锚点：Image #9
页面路径：抖音来客 > 报表集市 > 报表下载 > 商品
关键筛选：汇总方式、统计周期、投放渠道、商品范围、商品品类、履约方式
指标范围：商品信息、流量、交易、直播、短视频、达人、送礼、品类
```

用途：抖音商品曝光/访问/成交/新客、直播和短视频带货表现。当前样本没有门店维度；如果 demo 要做单店商品流量，需要在导出时补门店维度或另取门店商品报表。

目标按日导出表头分组：

```text
基础：日期, 商品ID, 商品名称, 投放渠道, 商品类型, 划线价, 售价, 适用门店数
流量：商品曝光次数/人数, 商品访问次数/人数, 不含商品卡曝光/访问
交易：商品成交金额/券数/人数, 商品核销金额/券数/人数, 商品退款金额/券数/人数, 成交新客数
直播：上架直播间数, 直播间观看人数, 直播间商品曝光/点击/成交
短视频：成交视频数, 视频播放次数, 视频商品曝光/点击/成交
达人：达人成交金额/券数, 达人视频/直播成交金额与券数
分类：商品一级/二级/三级品类名称
```

入库建议：

- 复用表：`douyin_product_detail`
- 如果导出缺少门店维度，只能落品牌/商品层或进入待匹配队列，不能伪造成单店商品数据。
- 直播、短视频、达人字段先入 JSONB，避免第一版表字段过宽。

## 3. Demo 指标对应关系

统一表格字段说明：

| 字段 | 含义 |
| --- | --- |
| 标记 | 对应截图上的编号，或本节内部指标编号 |
| demo 模块或指标 | demo 页面或卡片里的展示字段 |
| 当前 Excel 支撑度 | 支撑 / 部分支撑 / 弱支撑 / 支撑查询层 / 不支撑 |
| 数据源 | 当前样本、已设计维表或二期待接入系统 |
| 计算方式 | 一期查询层或二期接入后的计算口径 |
| 需要客户确认/补数 | 需要客户确认的口径、阈值、字段和补充数据 |

### 3.1 日常数据跟进

| 标记 | demo 模块或指标 | 当前 Excel 支撑度 | 数据源 | 计算方式 | 需要客户确认/补数 |
| --- | --- | --- | --- | --- | --- |
| D1 | 营业额 | 支撑 | 美团交易 `核销金额`；抖音交易 `核销金额` | 默认 `sum(核销金额)`；GMV 备用口径用 `sum(下单金额 / 意向成交金额)` | 展示实收/核销，还是成交 GMV |
| D2 | 订单量 | 支撑 | 美团交易 `核销券数`；抖音交易 `核销券数` | 默认 `sum(核销券数)`；下单口径用 `sum(下单券数 / 意向成交券数)` | 订单量按券数、订单数还是用户数 |
| D3 | 单均 | 支撑 | 美团交易、抖音交易 | `营业额 / 订单量`，默认按核销口径派生 | 分母是否按核销券数 |
| D4 | 总好评数 | 支撑 | 美团评价汇总 `新增好评数`；抖音评价明细 `推荐等级` | 美团/点评优先用汇总；抖音按 `推荐等级 >= 4星` 聚合 | 抖音 4 星是否算好评；中评/差评分界 |
| D5 | 单包间产值 | 不支撑 | 当前 9 个平台 Excel 不提供包间数和房态 | 二期接入后按 `营业额 / 可售包间数` 或客户确认的分母计算 | 可售包间数、营业包间数、实际开台数用哪个做分母 |
| D6 | 翻台率 | 不支撑 | 当前 9 个平台 Excel 不提供包间、场次、时段 | 二期接订单/房态后按 `场次数 / 可售包间数` 或房型维度计算 | 按包间、房型还是门店营业时长计算 |
| D7 | 私域好友数 | 不支撑 | 当前 9 个平台 Excel 不提供企微/小程序会员数据 | 二期接企微或小程序后读取外部累计值/净增值 | 私域好友是累计好友、净增好友还是可触达用户 |

### 3.2 平台数据卡片

| 标记 | demo 模块或指标 | 当前 Excel 支撑度 | 数据源 | 计算方式 | 需要客户确认/补数 |
| --- | --- | --- | --- | --- | --- |
| P1 | 美团/点评流量：曝光、访问、意向、下单 | 支撑 | 美团客流数据：`曝光人数`、`访问人数`、`意向转化人数`、`下单人数` | 按 `日期 x 门店 x 平台` 汇总；如 Excel 平台列为“点评/美团”，分别写 `dianping` / `meituan` | 卡片合并点评+美团，还是保留平台切换 |
| P2 | 美团/点评营业：营业额、订单、退款、单均 | 支撑 | 美团商品交易数据：下单、核销、退款字段 | 营业额默认 `核销金额`，订单量默认 `核销券数`，单均派生；GMV 保留下单金额 | 核销金额和下单金额的展示取舍 |
| P3 | 评价：好/中/差、好评率、星级、回复状态 | 部分支撑 | 美团评价汇总、门店评价详情、抖音评价明细 | 好评/差评优先用汇总；中评用差额或明细星级；回复状态从评价明细聚合；星级评分可用明细均值弱口径 | 中评计算方式；星级评分是否接受明细均值 |
| P4 | 榜单排名 | 部分支撑 | 美团榜单数据：`城市排名`、`行政区排名`、`商圈排名` | 默认展示商圈排名；缺失时降级展示行政区或城市。抖音暂无本批榜单样本 | 优先展示商圈、行政区还是城市榜；抖音榜单是否需要补入口 |
| P5 | 经营评分、ROS、牌级 | 不支撑 | 当前 9 个平台 Excel 不提供 | 暂标待接入，可人工维护或补导出入口 | 这些字段是否必须一期上线 |
| P6 | 抖音流量与成交：访问、下单、成交分析、营业额 | 部分支撑 | 抖音商品报表、抖音交易报表 | 交易建议用按日交易报表；商品流量如需单店，需要重新导出带门店维度样本 | 下单人数取意向成交人数还是核销人数；商品流量是否必须单店展示 |
| P7 | 抖音货架点击、门店页来源、经营分 | 不支撑 | 当前交易/商品/评价样本不提供明确字段 | 需要补抖音门店流量来源、货架点击和经营分导出入口或 API | 货架点击对应哪个后台指标；推荐/搜索/同城来源是否必须一期上线 |
| P8 | 小程序平台卡片 | 不支撑 | 当前 9 个平台 Excel 不提供小程序流量、订单、投诉、活动数据 | 二期接小程序埋点、订单、投诉、弹窗/卡券点击数据 | 小程序是否纳入一期；若纳入需补样本和字段说明 |
| P9 | 一期可启用异常告警：营业额、曝光、转化、退款、差评、评价未回复、榜单下降 | 支撑 | 美团/抖音交易、美团客流、评价汇总与明细、榜单数据 | 按门店和日期范围做环比/同比、阈值判断和排名变化检测 | 告警阈值、比较周期、严重/预警/关注分级 |
| P10 | 二期待接入异常告警：翻台、包间、设备、合规、回本、用户身份 | 不支撑 | 当前平台 Excel 不提供房态、设备、督导、财务、用户身份 | 只预留风险分类，不纳入一期 Excel 看板 | 二期数据系统和字段责任方 |

### 3.3 `/daily` demo 字段级口径确认

下面两张图是 `https://jiuque.jingyingcanmou.cn/daily` 当前 demo 页面截图，HTML 版已做编号标记并可点击预览大图。这个表用于和客户确认“页面字段到底从哪个 Excel 来、哪些字段一期不能由平台 Excel 支撑”。

![久雀 /daily 顶部筛选与总数据](assets/jiuque-daozong/demo-daily-top.png)

#### 图 A：顶部筛选与总数据

| 标记 | demo 模块或指标 | 当前 Excel 支撑度 | 数据源 | 计算方式 | 需要客户确认/补数 |
| --- | --- | --- | --- | --- | --- |
| A1 | 业务部、区域、门店筛选 | 支撑查询层 | `store_business_departments`、`store_operation_regions`、`store_region_assignments` | 先按 `business_vertical='general'` 限定到综门店，再按业务部、区域得到门店集合 | 业务部/区域清单、门店归属、生效日期 |
| A2 | 日期、工作日/周末、同比、环比 | 支撑查询层 | 各 Excel 的 `日期`；工作日/周末由日历派生 | 日期范围过滤事实表；同比/环比在查询层计算 | 节假日口径；环比取上一周期还是上周同期 |
| A3 | 营业额 | 支撑 | 美团交易 `核销金额`；抖音交易 `核销金额`；小程序/POS 待补 | 默认 `sum(核销金额)`，GMV 另保留下单/意向成交金额 | 展示实收/核销，还是成交 GMV |
| A4 | 订单量 | 支撑 | 美团交易 `核销券数`；抖音交易 `核销券数` | 默认按核销券数；下单口径另保留下单券数/意向成交券数 | 订单量按券数、订单数还是用户数 |
| A5 | 单均 | 支撑 | 美团交易、抖音交易 | `营业额 / 订单量`，默认按核销口径 | 分母是否按核销券数 |
| A6 | 单包间产值 | 不支撑 | 当前 9 个平台 Excel 不提供 | 二期需要房型/包间数量、小程序或 POS 订单 | 分母取可售包间数、营业包间数还是实际开台数 |
| A7 | 翻台率 | 不支撑 | 当前 9 个平台 Excel 不提供 | 二期需要包间、场次、订单时段数据 | 按包间、房型还是门店营业时长计算 |
| A8 | 私域好友数 | 不支撑 | 当前 9 个平台 Excel 不提供 | 二期接企微/小程序会员数据 | 累计好友、净增好友还是可触达用户 |
| A9 | 总好评数 | 支撑 | 美团评价汇总 `新增好评数`；抖音评价明细 `推荐等级 >= 4星` | 按门店、日期、平台汇总；美团/点评优先用汇总 | 抖音 4 星是否算好评；中评/差评分界 |

![久雀 /daily 平台数据卡片](assets/jiuque-daozong/demo-daily-platforms.png)

#### 图 B：平台数据卡片

| 标记 | demo 模块或指标 | 当前 Excel 支撑度 | 数据源 | 计算方式 | 需要客户确认/补数 |
| --- | --- | --- | --- | --- | --- |
| B1 | 大众点评/美团：曝光人数、访问人数、下单人数 | 支撑 | 美团客流数据 | 按日期、门店、平台汇总 `曝光人数`、`访问人数`、`下单人数` | demo 卡片是否合并展示点评+美团，还是保留平台切换 |
| B2 | 大众点评/美团：牌级、经营评分 | 不支撑 | 当前 9 个 Excel 不提供 | 暂标待接入，可人工维护或补导出入口 | 是否必须一期上线 |
| B3 | 大众点评/美团：评价好/中/差、好评率、星级评分 | 部分支撑 | 美团评价汇总、门店评价详情 | 好评/差评优先用汇总；中评用差额或明细星级；星级评分用明细均值或外部评分 | 中评计算方式；星级评分是否接受明细均值 |
| B4 | 大众点评/美团：榜单排名 | 部分支撑 | 美团榜单数据 | 默认展示商圈排名，缺失时可降级行政区/城市 | 优先展示商圈、行政区还是城市榜 |
| B5 | 大众点评/美团：营业额、订单量、单均 | 支撑 | 美团商品交易数据 | 营业额默认核销金额，订单量默认核销券数，单均派生 | 核销金额和下单金额的展示取舍 |
| B6 | 抖音：访问人数、下单人数 | 部分支撑 | 抖音商品报表、抖音交易报表 | 交易建议用按日交易报表；商品流量如需单店，需要重新导出带门店维度样本 | 下单人数取意向成交人数还是核销人数 |
| B7 | 抖音：货架点击人数 | 不支撑 | 当前样本没有明确同名字段 | 候选为商品访问人数或不含商品卡访问人数，需确认后固定 | 货架点击在抖音后台对应哪个指标 |
| B8 | 抖音：经营分、评分、评价数 | 部分支撑 | 评价数来自抖音评价明细；经营分无字段；评分可用推荐等级弱口径 | 推荐等级均值只能近似评分，不等于平台门店评分 | 经营分导出入口；评分是否接受近似 |
| B9 | 抖音：门店页流量来源 | 不支撑 | 当前交易/商品/评价样本不提供来源占比 | 需要补抖音门店流量来源导出或 API | 推荐/搜索/同城/其他是否必须一期上线 |
| B10 | 抖音：成交分析、营业额、订单量、单均 | 支撑 | 抖音交易报表 | 成交分析可用意向成交；营业额默认核销金额；订单量默认核销券数 | 成交分析展示意向成交还是核销 |
| B11 | 小程序全部流量和营业字段 | 不支撑 | 当前 9 个平台 Excel 不提供 | 二期接小程序埋点、订单、投诉、活动点击数据 | 小程序是否纳入一期；若纳入需补样本 |

自查后需要对齐的口径风险：

1. 当前平台 Excel 不能覆盖 demo 里的全部字段，尤其是小程序、单包间产值、翻台率、私域好友、经营分、牌级、抖音门店页流量来源。
2. 抖音交易/商品必须按日导出。如果拿到的样本仍是 `周` 字段，不能导入日表，只能退回重新导出。
3. 美团评价汇总和美团评价明细不是同一类数据：前者是 `日期 x 门店` 汇总，后者是评论明细。现在 artifact 已分别命名为 `meituan_general_review_stats_daily` 和 `meituan_general_review_comments`。
4. `/daily` 的业务部、区域、门店筛选是久雀内部运营组织维度，不应写进平台字段，也不应改事实表唯一键。

### 3.4 `/users` demo 字段级口径确认

![久雀 /users 用户分析](assets/jiuque-daozong/demo-users.png)

`/users` 页面核心是用户分层和行为分布。当前 9 个平台 Excel 只有聚合交易、商品、评价数据，不包含稳定用户 ID、订单 ID、订单时间、包间、消费时长，因此这页多数指标不能由一期平台 Excel 严格支撑，只能做少量弱口径或二期待接入。

| 标记 | demo 模块或指标 | 当前 Excel 支撑度 | 数据源 | 计算方式 | 需要客户确认/补数 |
| --- | --- | --- | --- | --- | --- |
| U1 | 用户总数、新客、熟客、老客 | 弱支撑 | 抖音商品 `成交新客数`；美团本批无新/熟/老客字段 | 只可按平台原始定义做弱口径；不建议用评价昵称反推用户 | 是否接受各平台原始用户定义；需订单级匿名用户 ID |
| U2 | 流失人数、流失率、复购人数、复购率 | 不支撑 | 当前 9 个平台 Excel 不提供用户历史消费序列 | 不能严肃计算流失/复购 | 需订单 ID、用户匿名 ID、最近消费日期、历史消费次数 |
| U3 | 下单周期 | 不支撑 | 当前 9 个平台 Excel 只有日/商品/门店聚合 | 需要同一用户多次下单间隔，当前无法计算 | 需用户级订单流水 |
| U4 | 消费水平分布 | 部分支撑 | 美团/抖音交易和商品金额字段 | 可用门店/商品客单价做门店层粗分；不能得到用户消费水平分布 | 需用户累计消费、单次订单金额 |
| U5 | 优惠敏感度 | 部分支撑 | 商品类型、商品名称、售价/划线价、团购/套餐字段 | 可做商品层弱分类，不能还原用户级优惠敏感度 | 需订单级优惠、券、原价支付记录 |
| U6 | 包型消费偏好 | 部分支撑 | 美团/抖音商品名称 | 可从商品名称解析 `特惠/中/大/尊/4H/12H` 等标签 | 需标准包型字段，避免靠商品名解析 |
| U7 | 下单时段 | 不支撑 | 当前 9 个平台 Excel 不提供小时级下单时间 | 无法按小时、闲时/高峰或预约提前量计算 | 需订单创建时间、预约开始时间、消费开始/结束时间 |
| U8 | 工作日/周末、同比、环比筛选 | 支撑查询层 | 日期字段、查询层日历 | 工作日/周末由日历派生；同比/环比由查询层计算 | 节假日是否独立分组 |

### 3.5 `/monitor` demo 字段级口径确认

![久雀 /monitor 异常监控](assets/jiuque-daozong/demo-monitor.png)

`/monitor` 页面本质是规则引擎：先用 `/daily`、评价、榜单、订单/房态、财务、设备、督导等指标生成风险项，再按严重/预警/关注聚合。当前平台 Excel 能支撑其中一部分经营、流量、评价、榜单类告警；设备、合规、回本、用户身份类告警需要外部数据。

| 标记 | demo 模块或指标 | 当前 Excel 支撑度 | 数据源 | 计算方式 | 需要客户确认/补数 |
| --- | --- | --- | --- | --- | --- |
| M1 | 风险总览：总风险项、严重、预警、关注、健康门店占比 | 依赖规则结果 | 规则触发结果，不是 Excel 原始字段 | 基于下方规则触发结果聚合 | 阈值、风险等级、健康门店定义 |
| M2 | 经营业绩：营业额环比、同店增长 | 支撑 | 美团交易、抖音交易核销金额 | 按门店和日期范围比较核销金额 | 同店增长比较周期和门店范围 |
| M3 | 经营业绩：翻台率、包间使用率、单包间产值 | 不支撑 | 当前 9 个平台 Excel 不提供包间和场次 | 只能标待接入 | 需房态、包间数、场次、营业时长 |
| M4 | 收入核查：退款异常率 | 支撑 | 美团交易退款金额/券数；抖音意向退款金额/券数 | 按金额或券数计算退款占比并触发阈值 | 阈值按金额、券数还是订单数 |
| M5 | 0 元体验订单占比、店主下单订单占比 | 不支撑 | 当前 9 个平台 Excel 不提供订单实付明细和下单人身份 | 无法识别 0 元订单和店主身份 | 需订单级支付金额、用户身份、员工/店主标识 |
| M6 | 流量：曝光下滑、星级评分、商圈排名/榜单 | 部分支撑 | 美团客流、抖音商品、评价明细、榜单 Excel | 曝光和排名可直接算；星级用评价明细弱口径 | 经营分、牌级当前无导出字段 |
| M7 | 用户服务：差评率、评价未回复、客诉响应 | 部分支撑 | 评价汇总、评价明细 | 差评率和评价回复状态可算；投诉率和客诉时长需投诉系统 | 投诉和评价是否合并为用户服务风险 |
| M8 | 竞争环境：周边竞品、价格敏感度、竞品差评提及 | 部分支撑 | 榜单 Excel、评价文本、商品价格字段 | 商圈排名可用榜单；差评提及竞品可用评价文本 NLP；价格敏感度需竞品价 | 竞品清单和竞品价格来源 |
| M9 | 设备、运营执行、合规、回本周期 | 不支撑 | 当前 9 个平台 Excel 不提供 | 只预留规则分类，不纳入一期 Excel 看板 | 需设备档案/工单、督导评分、财务/保险/税务、投资回本数据 |

### 3.6 `/stores/上海徐汇店` demo 字段级口径确认

![久雀 /stores/上海徐汇店 单店深度分析](assets/jiuque-daozong/demo-store-shanghai-xuhui.png)

单店页是最细的经营诊断页，展示单店订单行为、包型、消费能力、留存、画像标签。当前平台 Excel 能支撑“平台交易、商品/套餐、评价、榜单”这些单店弱口径；凡是涉及用户、包间、预约、换房、消费时长、留存标签的图，都需要二期订单/房态/会员数据。

| 标记 | demo 模块或指标 | 当前 Excel 支撑度 | 数据源 | 计算方式 | 需要客户确认/补数 |
| --- | --- | --- | --- | --- | --- |
| S1 | 单店筛选与顶部卡片：营业额、客单价 | 支撑 | 美团/抖音交易核销金额、核销券数 | 营业额默认核销金额；客单价 `核销金额 / 核销券数` | 单店页是否只看选中门店，是否合并平台 |
| S2 | 顶部卡片：场次数、续单率、单包间产值、翻台率 | 不支撑 | 当前 9 个平台 Excel 不提供场次、续单、包间 | 无法计算，只能标待接入 | 需 POS/小程序订单、房态、包间数、续单标识 |
| S3 | 用户下单行为：消费时段、闲时/高峰、预订习惯、消费时长 | 不支撑 | 当前 9 个平台 Excel 不提供小时、预约提前量、消费开始/结束时间 | 无法计算时段和时长 | 需订单创建时间、预约时间、开台/离店时间 |
| S4 | 消费渠道偏好 | 部分支撑 | 美团/点评/抖音交易金额和券数 | 按平台交易金额/券数占比计算；高德/快手/小程序当前无样本 | 是否合并点评与美团；是否补高德/快手/小程序 |
| S5 | 包型偏好、包型趋势、包型 x 时段热力图 | 部分支撑 | 美团/抖音商品名称、交易日期 | 包型可从商品名称解析；趋势按日期聚合；时段热力图不能做 | 需标准包型、订单时段、房型字段 |
| S6 | 不同包型客单价、换房行为 | 部分/不支撑 | 商品交易弱口径；当前无换房事件 | 包型客单价可用商品交易弱算；换房行为不支撑 | 需订单包间、换房事件、前后包型 |
| S7 | 消费水平、时段客单价、时长 x 客单价 | 部分/不支撑 | 商品/交易金额；当前无时段和消费时长 | 消费水平可用订单/商品金额弱分布；时段和时长相关不支撑 | 需订单级金额、时间、消费时长 |
| S8 | 价格敏感度、消费能力客群分层 | 部分/不支撑 | 商品名称、商品类型、售价/划线价；当前无用户累计消费 | 可用团购/套餐/原价商品弱分类；客群分层需用户累计消费 | 需优惠明细、用户 ID、累计消费 |
| S9 | 新老用户、复购率、消费频次、生命周期 | 不支撑 | 当前 9 个平台 Excel 不提供用户历史序列 | 无法计算 | 需用户匿名 ID、订单历史 |
| S10 | 流失用户、沉默用户分析 | 不支撑 | 当前 9 个平台 Excel 不提供用户最近一次消费 | 无法识别沉默或流失 | 需用户最近消费日期和召回规则 |
| S11 | 用户画像标签、筛选命中、用户列表 | 不支撑 | 当前 9 个平台 Excel 不提供会员/标签/触达授权 | 只能作为二期标签系统设计，不应一期伪造 | 需会员/订单/渠道/触达授权 |
| S12 | 运营动作：推送、企微、导出名单、创建营销活动 | 不支撑 | 营销系统能力，不是平台 Excel 字段 | 一期不纳入平台 Excel 看板 | 需触达系统、权限、合规规则 |

### 3.7 用户分析

本批平台 Excel 不包含稳定用户 ID、订单 ID、包间、消费开始/结束时间，因此不能支撑完整用户画像。可先做弱口径：

| 标记 | demo 模块或指标 | 当前 Excel 支撑度 | 数据源 | 计算方式 | 需要客户确认/补数 |
| --- | --- | --- | --- | --- | --- |
| UX1 | 新客 | 部分支撑 | 抖音商品 `成交新客数`；美团本批无新客字段 | 只按抖音平台原始定义展示，不能跨平台合并用户 | 是否接受平台原始新客定义 |
| UX2 | 熟客、老客、流失、复购 | 不支撑 | 当前 9 个平台 Excel 不提供用户历史消费序列 | 不建议反推 | 用户匿名 ID、订单历史、最近消费日期 |
| UX3 | 消费水平 | 部分支撑 | 商品/门店交易金额 | 用商品/门店客单价粗分，不能形成用户级画像 | 用户累计消费、单次订单金额 |
| UX4 | 优惠敏感度 | 部分支撑 | 商品类型、售价、原价、下单/核销金额差额 | 做商品层弱分类 | 订单级优惠和券明细 |
| UX5 | 包型偏好 | 部分支撑 | 商品名称 | 从 `特惠 / 中包 / 大包 / 尊享 / 12H / 4H` 等标签解析 | 标准房型字段 |
| UX6 | 下单时段 | 不支撑 | 当前 9 个平台 Excel 不提供小时字段 | 无法计算 | 订单创建时间 |

### 3.8 异常监控

| 标记 | demo 模块或指标 | 当前 Excel 支撑度 | 数据源 | 计算方式 | 需要客户确认/补数 |
| --- | --- | --- | --- | --- | --- |
| R1 | 营业额环比/同比 | 支撑 | 美团/抖音交易核销金额 | 核销金额按门店、平台、日期范围比较 | 比较周期和阈值 |
| R2 | 曝光下滑 | 支撑 | 美团客流；抖音商品曝光按品牌/商品 | 美团按门店环比；抖音如需单店需补门店维度商品样本 | 抖音商品是否必须单店展示 |
| R3 | 访问转化下降 | 支撑 | 美团客流 `曝光访问转化率`、`意向转化率` | 按门店和日期范围比较转化率 | 阈值和告警等级 |
| R4 | 退款异常 | 支撑 | 美团/抖音退款金额和券数字段 | 退款金额 / 下单金额，退款券数 / 下单券数 | 按金额、券数还是订单数触发 |
| R5 | 差评率、评价未回复 | 支撑 | 美团评价汇总、评价明细、抖音评价明细 | 差评率按差评数/评价数；未回复按明细回复状态聚合 | 抖音星级分界和回复超时阈值 |
| R6 | 榜单下降 | 支撑 | 美团榜单数据 | 城市/行政区/商圈排名环比 | 优先展示和监控哪个榜单层级 |
| R7 | 经营分、牌级 | 不支撑 | 当前 9 个平台 Excel 不提供 | 暂不可做 | 补导出入口或人工维护字段 |

## 4. 入库模型设计

### 4.1 修正后的建模原则

现有项目里已经有两类成熟模型：

1. `basic-store-schema.sql`：到店通用指标表，例如 `store_traffic`、`store_transactions`、`product_traffic`、`store_reviews`、`store_rankings`，字段是业务口径，带 `store_id + date + platform`。
2. 平台专表：例如 `meituan_waimai_store_daily`、`taobao_shangou_store_daily`、`douyin_product_detail`、`douyin_review`，字段贴近平台导出，带 `connection_id + stat_date + 平台门店ID`，并保留 `raw_metrics/raw_data`。

结合当前参考意见，久雀到综不应复制一套“综合平台”。正确拆法是：

- `platform` 表示数据来源，继续使用现有 key：`meituan`、`dianping`、`douyin`。
- `business_vertical` 表示业态，新增枚举：`dining` 到店餐饮，`general` 到店综合。
- 到店餐饮和到店综合不会在同一个实体门店上共存，因此 `UNIQUE(store_id, date, platform)`、`UNIQUE(store_id, date, platform, product_id)` 等唯一键短期保持不变。
- 一期优先复用现有到店分析表；字段能映射的直接入现有表，字段差异先放 `raw_data` / `metadata` 或少量 nullable 字段中。
- 后续如果确认到店综合字段结构长期明显不同，再新增 `in_store_general_store_daily`、`in_store_general_product_daily` 这类业态专表。

### 4.2 新增业态字段

建议新增 3 个字段，默认值先给 `dining`，保证现有数据行为不变：

```sql
ALTER TABLE stores
  ADD COLUMN IF NOT EXISTS business_vertical VARCHAR(32) NOT NULL DEFAULT 'dining'
  CHECK (business_vertical IN ('dining', 'general'));

ALTER TABLE platform_connections
  ADD COLUMN IF NOT EXISTS business_vertical VARCHAR(32) NOT NULL DEFAULT 'dining'
  CHECK (business_vertical IN ('dining', 'general'));

ALTER TABLE platform_authorizations
  ADD COLUMN IF NOT EXISTS business_vertical VARCHAR(32) NOT NULL DEFAULT 'dining'
  CHECK (business_vertical IN ('dining', 'general'));

CREATE INDEX IF NOT EXISTS idx_stores_business_vertical
  ON stores(business_vertical);

CREATE INDEX IF NOT EXISTS idx_platform_connections_platform_vertical
  ON platform_connections(platform, business_vertical, connection_status);

CREATE INDEX IF NOT EXISTS idx_platform_authorizations_platform_vertical
  ON platform_authorizations(platform, business_vertical, authorization_status);
```

`stores.business_vertical` 是业务归属的最终判断；`platform_connections.business_vertical` 和 `platform_authorizations.business_vertical` 用于授权、采集和 parser 路由。创建连接时应从门店继承，品牌级连接则由用户选择。

### 4.3 platform 与 artifact

`platform_connections.platform` 不新增 `meituan_general`、`douyin_general`。示例：

```text
platform='meituan', business_vertical='general'
platform='dianping', business_vertical='general'
platform='douyin', business_vertical='general'
```

兼容说明：`platform_connections.platform`、`platform_authorizations.platform` 使用英文平台 key；部分既有事实表历史上可能保存中文平台值，例如 `product_detail.platform='美团'/'点评'`。到综开发时不要新增 `meituan_general` 这类平台码；如果复用历史表，parser 和查询层需要做 `normalize_platform_key` 兼容，把 `美团 -> meituan`、`点评 -> dianping`、`抖音 -> douyin` 统一到平台 key 再比较。

artifact 命名分两类：

- 格式和到餐一致：继续使用现有 artifact type，例如 `meituan_overall`、`dianping_overall`、`meituan_dianping_review`、`douyin_review`、`douyin_transaction`、`douyin_product`，并在 `collection_artifacts.metadata.business_vertical='general'`。
- 格式和到餐不同：使用带业态的 artifact type，例如本文 9 个样本建议使用 `meituan_general_*` / `douyin_general_*`，但 parser 入库时仍写 `platform='meituan'/'dianping'/'douyin'`。

本文样本先按“格式不同”处理，避免误走现有到餐 parser：

```text
meituan_general_review_stats_daily
meituan_general_review_comments
meituan_general_product_traffic
meituan_general_product_transaction
meituan_general_store_traffic
meituan_general_ranking
douyin_general_review
douyin_general_transaction
douyin_general_product
```

### 4.4 复用表与字段落点

| 样本 | 复用目标 | platform key / 事实表取值 | business_vertical | 说明 |
| --- | --- | --- | --- | --- |
| 美团评价汇总 | `store_reviews` | key 为 `meituan` / `dianping`；历史事实表兼容 `美团` / `点评` | `general` | `新增评价数`、`新增好评数`、`新增差评数`、`差评回复率` 可直接映射 |
| 美团评价明细 | `store_comments` | key 为 `meituan` / `dianping`；历史事实表兼容 `美团` / `点评` | `general` | 无稳定评价 ID 时用内容 hash 生成 `platform_comment_id` |
| 美团商品访问 | `product_detail` | key 为 `meituan` / `dianping`；历史事实表兼容 `美团` / `点评` | `general` | 商品 ID、售价、访问、下单、访购率、退款券数 |
| 美团商品交易 | `product_detail`，必要时聚合到 `store_transactions` / `refunds` | key 为 `meituan` / `dianping`；历史事实表兼容 `美团` / `点评` | `general` | 下单、核销、退款字段按商品粒度保留 |
| 美团客流 | `store_traffic` | key 为 `meituan` / `dianping`；历史事实表兼容 `美团` / `点评` | `general` | 曝光、访问、意向、下单、收藏、打卡 |
| 美团榜单 | `store_rankings` | key 为 `meituan` / `dianping`；历史事实表兼容 `美团` / `点评` | `general` | 现有字段不足以表达多榜单明细时，先把原始榜单行放 `raw_data` |
| 抖音评价 | `douyin_review` | `douyin` | `general` | 保留 AI/RPA 回复扩展空间 |
| 抖音交易 | `douyin_transactions_detail` | `douyin` | `general` | RPA 选择“按日”导出；若文件出现 `周` 字段则重新导出 |
| 抖音商品 | `douyin_product_detail` | `douyin` | `general` | 商品表缺门店维度时只能做品牌/商品层指标 |

本阶段不预先列具体 ALTER。开发计划里先和用户确认数据口径，再做字段对照：

1. 能稳定映射到现有字段的，直接写现有表。
2. 差异较小、只是少量附加字段的，用现有表兼容。
3. 差异明显、会让现有表语义变乱的，再新建业态专表。

真正的隔离仍来自 `stores.business_vertical` 和 `platform_connections.business_vertical`，不是靠复制平台。

### 4.5 门店运营组织维度

demo 的日常数据页顶部有 `业务部 -> 区域 -> 门店` 三级筛选。这个维度不属于平台，也不属于业态；它是久雀内部的运营组织口径。建议单独建维表，不要把区域写进交易、客流、评价等事实表唯一键。

推荐结构：

```sql
CREATE TABLE IF NOT EXISTS store_business_departments (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  organization_id UUID REFERENCES organizations(id) ON DELETE SET NULL,
  customer_code VARCHAR(64) NOT NULL DEFAULT 'jiuque',
  business_vertical VARCHAR(32) NOT NULL DEFAULT 'general'
    CHECK (business_vertical IN ('dining', 'general')),
  department_name VARCHAR(100) NOT NULL,
  department_code VARCHAR(64),
  sort_order INTEGER NOT NULL DEFAULT 0,
  status VARCHAR(20) NOT NULL DEFAULT 'active'
    CHECK (status IN ('active', 'disabled')),
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(customer_code, business_vertical, department_name)
);

CREATE TABLE IF NOT EXISTS store_operation_regions (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  department_id UUID NOT NULL REFERENCES store_business_departments(id) ON DELETE CASCADE,
  parent_region_id UUID REFERENCES store_operation_regions(id) ON DELETE SET NULL,
  region_name VARCHAR(100) NOT NULL,
  region_code VARCHAR(64),
  sort_order INTEGER NOT NULL DEFAULT 0,
  status VARCHAR(20) NOT NULL DEFAULT 'active'
    CHECK (status IN ('active', 'disabled')),
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(department_id, region_name)
);

CREATE TABLE IF NOT EXISTS store_region_assignments (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  store_id UUID NOT NULL REFERENCES stores(id) ON DELETE CASCADE,
  region_id UUID NOT NULL REFERENCES store_operation_regions(id) ON DELETE CASCADE,
  valid_from DATE NOT NULL DEFAULT CURRENT_DATE,
  valid_to DATE,
  is_primary BOOLEAN NOT NULL DEFAULT true,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  CHECK (valid_to IS NULL OR valid_to >= valid_from),
  UNIQUE(store_id, region_id, valid_from)
);

CREATE INDEX IF NOT EXISTS idx_store_region_assignments_store_date
  ON store_region_assignments(store_id, valid_from, valid_to);

CREATE INDEX IF NOT EXISTS idx_store_region_assignments_region
  ON store_region_assignments(region_id);
```

口径说明：

- `store_business_departments` 对应筛选器里的“业务部”。
- `store_operation_regions` 对应筛选器里的“区域”，归属于业务部；如果以后有大区/小区层级，可用 `parent_region_id` 扩展。
- `store_region_assignments` 是门店和区域的生效关系。门店调区时关闭旧记录 `valid_to`，新增一条新记录，历史报表按统计日期 join 对应区域。
- `stores.province/city/business_district` 仍表示地理位置，不承担运营区域筛选。
- 如果同一门店同一日期只能归属一个主区域，一期用应用层校验防重；后续需要强约束时再加 PostgreSQL exclusion constraint，避免出现重叠生效区间。

按区域查询门店范围：

```sql
WITH scoped_stores AS (
  SELECT DISTINCT sra.store_id
  FROM store_region_assignments sra
  JOIN store_operation_regions r ON r.id = sra.region_id
  JOIN store_business_departments d ON d.id = r.department_id
  WHERE d.business_vertical = 'general'
    AND ($4::uuid IS NULL OR d.id = $4)
    AND ($5::uuid IS NULL OR r.id = $5)
    AND sra.valid_from <= $2::date
    AND (sra.valid_to IS NULL OR sra.valid_to >= $1::date)
)
SELECT *
FROM scoped_stores;
```

接口层筛选顺序：

1. 先按 `business_vertical='general'` 限定到综门店。
2. 再按业务部和区域算出 `store_id` 集合。
3. 最后把 `store_id` 集合传给日常数据、用户分析、异常监控、单店深度分析查询。

### 4.6 新建专表的判断边界

一期默认先复用现有到店表。满足以下任一条件时，再新增业态专表：

- 确认后的字段和现有表差异明显，强行兼容会让字段含义不稳定。
- 到综需要长期展示的核心字段，在现有表中没有合理落点。
- 榜单这类一门店一天多榜单、多区域、多分类的数据需要完整明细查询。
- 同一实体在同一日期、同一平台下需要多行事实数据，现有唯一键无法表达。

备选专表命名：

```text
in_store_general_store_daily
in_store_general_product_daily
in_store_general_review_detail
in_store_general_ranking_daily
```

## 5. Parser 与字段映射

### 5.1 artifact 路由

Parser 路由按 `platform + business_vertical + artifact_type` 判断。平台只表示来源，业态决定走到餐逻辑还是到综逻辑。

```ts
const vertical = connection.business_vertical ?? 'dining'

if (vertical === 'general') {
  switch (artifactType) {
    case 'meituan_general_review_stats_daily':
    case 'meituan_general_review_comments':
    case 'meituan_general_product_traffic':
    case 'meituan_general_product_transaction':
    case 'meituan_general_store_traffic':
    case 'meituan_general_ranking':
    case 'douyin_general_review':
    case 'douyin_general_transaction':
    case 'douyin_general_product':
      return parseGeneralInStoreArtifact(...)
  }
}
```

如果未来确认某个到综 Excel 与现有到餐 artifact 完全一致，可以复用现有 artifact type，但必须在 `metadata.business_vertical='general'` 且 parser 分支里读 connection vertical。每个 parser 必须做表头 sentinel 校验；表头不匹配时返回 `COLUMN_DRIFT`，不要静默按错列导入。

### 5.2 周期解析

```text
美团日期：YYYY-MM-DD -> period_type=day, period_start=period_end=日期
抖音日期：YYYY-MM-DD -> period_type=day, period_start=period_end=日期
```

抖音交易和商品导出时必须选择“按日”。如果 parser 读到字段名 `周` 或值形如 `YYYY-MM-DD~YYYY-MM-DD`，不要伪造成日数据，应返回导出配置错误并提示重新导出。

### 5.3 门店匹配

优先级：

1. `stores.meituan_store_id` / `stores.dianping_store_id` / `stores.douyin_store_id` 精确匹配。
2. 同平台 store mapping 表匹配。
3. 同城市 + 标准化门店名匹配。
4. 无法匹配时创建候选门店，但标记 `metadata.needs_store_review=true`。

标准化门店名时只去除全角/半角括号、空格、间隔号，不要去掉“久雀”“雀渝汇”等品牌词，否则不同品牌门店可能误合并。

### 5.4 关键指标映射

| normalized_metric_key | 美团字段 | 抖音字段 | 聚合 |
| --- | --- | --- | --- |
| `exposure_users` | 客流：曝光人数 | 商品：商品曝光人数 | sum |
| `exposure_count` | 客流：曝光次数 | 商品：商品曝光次数 | sum |
| `visit_users` | 客流：访问人数 / 商品：商品访问人数 | 商品：商品访问人数 | sum |
| `visit_count` | 客流：访问次数 / 商品：商品访问次数 | 商品：商品访问次数 | sum |
| `intent_users` | 客流：意向转化人数 | 交易：意向成交人数 | sum |
| `order_users` | 交易：下单人数 | 交易：意向成交人数 | sum |
| `order_voucher_count` | 交易：下单券数 | 交易：意向成交券数 | sum |
| `order_amount` | 交易：下单金额 | 交易：意向成交金额 | sum |
| `redeemed_users` | 交易：核销人数 | 交易：核销人数 | sum |
| `redeemed_voucher_count` | 交易：核销券数 | 交易：核销券数 | sum |
| `redeemed_amount` | 交易：核销金额 | 交易：核销金额 | sum |
| `refund_voucher_count` | 交易：退款券数 | 交易：意向退款券数 | sum |
| `refund_amount` | 交易：退款金额（原价） | 交易：意向退款金额 | sum |
| `review_count` | 评价汇总：新增评价数 | 评价明细 count | sum |
| `positive_review_count` | 评价汇总：新增好评数 | 推荐等级 >= 4星 | sum |
| `negative_review_count` | 评价汇总：新增差评数 | 推荐等级 <= 2星 | sum |
| `negative_reply_rate` | 评价汇总：差评回复率 | 明细计算 | weighted_avg |
| `ranking_city_rank` | 榜单：城市排名 | 无 | latest |

## 6. Demo 查询设计

### 6.1 总数据

查询层必须同时过滤门店业态和连接业态，避免同平台到餐数据混入到综 demo。`/daily` 顶部的业务部、区域、门店筛选先通过 `store_region_assignments` 算出门店集合，再进入指标查询。

```sql
WITH scoped_stores AS (
  SELECT DISTINCT s.id AS store_id
  FROM stores s
  LEFT JOIN store_region_assignments sra
    ON sra.store_id = s.id
   AND sra.valid_from <= $2::date
   AND (sra.valid_to IS NULL OR sra.valid_to >= $1::date)
  LEFT JOIN store_operation_regions r ON r.id = sra.region_id
  LEFT JOIN store_business_departments d ON d.id = r.department_id
  WHERE s.business_vertical = 'general'
    AND ($3::uuid[] IS NULL OR s.id = ANY($3::uuid[]))
    AND ($4::uuid IS NULL OR d.id = $4::uuid)
    AND ($5::uuid IS NULL OR r.id = $5::uuid)
),
connected_stores AS (
  SELECT DISTINCT pcs.store_id, pc.platform
  FROM platform_connection_stores pcs
  JOIN platform_connections pc ON pc.id = pcs.connection_id
  JOIN scoped_stores ss ON ss.store_id = pcs.store_id
  WHERE TRUE
    AND pc.business_vertical = 'general'
    AND pc.platform = ANY(ARRAY['meituan', 'dianping', 'douyin'])
),
meituan_dianping_tx AS (
  SELECT
    SUM(product_verified_amount) AS revenue,
    SUM(product_vouchers_verified) AS orders,
    SUM(product_revenue_after) AS gmv
  FROM product_detail pd
  JOIN stores s ON s.id = pd.store_id
  WHERE EXISTS (
    SELECT 1
    FROM connected_stores cs
    WHERE cs.store_id = pd.store_id
      AND cs.platform = CASE pd.platform
        WHEN '美团' THEN 'meituan'
        WHEN '点评' THEN 'dianping'
        ELSE pd.platform
      END
  )
    AND pd.platform IN ('meituan', 'dianping', '美团', '点评')
    AND pd.date BETWEEN $1 AND $2
),
douyin_tx AS (
  SELECT
    SUM(redeemed_amount) AS revenue,
    SUM(redeemed_voucher_count) AS orders,
    SUM(intent_revenue) AS gmv
  FROM douyin_transactions_detail dt
  JOIN stores s ON s.id = dt.store_id
  WHERE EXISTS (
    SELECT 1
    FROM connected_stores cs
    WHERE cs.store_id = dt.store_id
      AND cs.platform = 'douyin'
  )
    AND dt.date BETWEEN $1 AND $2
)
SELECT
  COALESCE(meituan_dianping_tx.revenue, 0) + COALESCE(douyin_tx.revenue, 0) AS revenue,
  COALESCE(meituan_dianping_tx.orders, 0) + COALESCE(douyin_tx.orders, 0) AS orders,
  COALESCE(meituan_dianping_tx.gmv, 0) + COALESCE(douyin_tx.gmv, 0) AS gmv
FROM meituan_dianping_tx, douyin_tx;
```

### 6.2 美团/点评平台卡片

来源优先级：

1. 客流：曝光、访问、意向、下单。
2. 商品交易：营业额、订单量、退款。
3. 评价汇总和评价明细：评价数量、好评/差评、回复状态。
4. 榜单：商圈/城市/行政区排名。

### 6.3 抖音平台卡片

来源优先级：

1. 交易：核销金额、核销券数、意向成交、退款。
2. 商品：曝光、访问、成交新客、直播/短视频/达人指标。
3. 评价：评分、评价数、回复状态。

限制：当前商品报表缺门店维度，只能做品牌/商品级流量；单店抖音商品流量需要补一份带门店维度的导出样本。

### 6.4 单店深度分析

一期只展示可由平台 Excel 支撑的部分：

- 渠道偏好：美团/点评/抖音核销金额占比。
- 商品偏好：按商品名称解析 `特惠 / 中包 / 大包 / 尊享 / 12H / 4H` 等标签。
- 评价画像：评价关键词、差评内容、回复情况。
- 榜单表现：城市/行政区/商圈排名。

不展示或标注“待接入”的部分：

- 包间使用率、翻台率。
- 下单时段。
- 用户列表与标签。
- 消费生命周期和流失用户。

## 7. Worker 隔离设计

到综 worker 和现有餐饮 worker 并存时，必须在入口 SELECT 按 `business_vertical` 过滤。这里不能再靠新平台码隔离，因为 `platform` 必须保持 `meituan`、`dianping`、`douyin`。

| worker / 资源 | `business_vertical=dining` 连接 | `business_vertical=general` 连接 | 到餐 artifact | 到综 artifact | 到餐数据 | 到综数据 |
| --- | --- | --- | --- | --- | --- | --- |
| 现有到餐 worker | 允许 | 禁止 | 允许 | 禁止 | 允许 | 禁止 |
| 久雀到综 worker | 禁止 | 允许 | 禁止 | 允许 | 禁止 | 允许 |

```sql
SELECT *
FROM platform_connections
WHERE connection_status = 'connected'
  AND platform IN ('meituan', 'dianping', 'douyin')
  AND business_vertical = 'general'
FOR UPDATE SKIP LOCKED;
```

不要在 dispatch 里做 fallback 到餐饮 connector。能力边界必须在取任务入口挡死；同一个 `platform='meituan'`，`dining` 和 `general` 可以走不同下载逻辑。

## 8. 开发任务拆分

### Phase 1：字段冻结与 dry-run

- 新增 `stores.business_vertical`、`platform_connections.business_vertical`、`platform_authorizations.business_vertical`。
- 新增门店运营组织维表：`store_business_departments`、`store_operation_regions`、`store_region_assignments`，支撑 `/daily` 的业务部、区域、门店三级筛选。
- 不新增平台码，`platform` 仍为 `meituan`、`dianping`、`douyin`。
- 按本批 9 个样本暂定新增 9 个 `*_general_*` artifact type；如果后续确认格式一致，可回收为现有 artifact type + metadata。
- 新增各 artifact 的 header sentinel。
- 编写 dry-run 脚本，读取本批 9 个 Excel，输出 row count、insert candidate count、skipped count。
- 记录每个 artifact 的 `business_vertical`、`header_hash`、`period_type`、`period_start`、`period_end`。

验收：

- 9 个样本全部 dry-run 成功。
- 行数与本文入口总表一致。
- 抖音交易/商品按日导出；如果导出成周粒度，dry-run 明确报错。

### Phase 2：表结构与导入

- 复用现有 `store_traffic`、`product_detail`、`store_reviews`、`store_comments`、`store_rankings`、`douyin_transactions_detail`、`douyin_product_detail`、`douyin_review`。
- 和用户确认数据口径后做字段对照：差异小则兼容现有表，差异大则新建业态专表；不改现有唯一键。
- 实现美团 6 类 parser。
- 实现抖音 3 类 parser。
- 对金额、百分比、空值、`未上榜`、`-` 做统一 parse。

验收：

- 每个 Excel 可以入库。
- 重复导入幂等，不产生重复明细。
- 门店无法匹配时进入 review 状态，不静默丢数据。

### Phase 3：demo API 查询层

- 新增 general in-store 查询模块。
- 总览指标按核销口径返回，同时附带 GMV 备用字段。
- 平台卡片区分美团/点评/抖音。
- 所有查询都按 `business_vertical='general'` 过滤。
- 异常监控只启用已接数据源规则。

验收：

- demo 的日常数据、平台数据和部分异常监控可由真实 Excel 生成。
- `/users`、`/monitor`、单店深度分析里不能由当前 Excel 支撑的模块，明确显示“待接入小程序/POS/订单/房态/会员数据”。

## 9. 还需要补充的材料

为了后续开发更稳，建议再补 4 类材料：

1. 截图更新：当前 HTML 已嵌入 Image #1 到 Image #9，以及 `/daily`、`/users`、`/monitor`、`/stores/上海徐汇店` demo 截图；后续如后台 UI 变化，覆盖 assets 同名图片即可。
2. 抖音评价导出动作复测：Image #7 已补评价导出弹窗；开发时仍需确认导出按钮点击后的下载任务和文件命名规则。
3. 抖音按日导出样本复测：交易、商品都选择“按日”后重新导出一份样本，作为 parser 的固定样本。
4. 小程序/POS/订单样本：如果要做包间、翻台、用户画像、用户标签、流失/复购，需要订单级数据，至少包含订单 ID、用户匿名 ID、门店、包间/房型、开始/结束时间、支付金额、退款金额、渠道。

这 4 类不是阻塞一期“平台 Excel 经营看板”的开发，但会影响图文文档完整度和二期能力边界。

## 10. 对客户口径说明

建议对外这样表述：

> 一期接入美团经营宝和抖音来客可下载 Excel，优先建设门店经营、平台流量、商品交易、评价、榜单和基础异常监控。营业额默认采用核销金额，更贴近实际消费；同时保留成交金额作为 GMV 备用口径。  
> 用户画像、包间翻台、私域好友、设备合规、回本周期不属于本批平台 Excel 的直接字段，需要补充小程序/POS/订单/房型/企微/财务数据后作为二期建设。
