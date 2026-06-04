# 久雀到综独立数据面实施计划

> 日期：2026-06-04
> 适用仓库：`/Users/adonis/Desktop/code/OpsInsightHub`
> 设计文档仓库：`/private/tmp/jiuque-daozong-data-design-pages`
> GitHub Pages HTML：`2026-06-04-jiuque-daozong-general-implementation-plan.html`
> GitHub Pages Markdown：`2026-06-04-jiuque-daozong-general-implementation-plan.md`

## 0. 总体决策

久雀到综按“共享控制面 + 独立数据面 + 独立查询/页面层”实现。

| 层级 | 决策 | 说明 |
| --- | --- | --- |
| 控制面 | 共享 | `stores`、用户、权限、订阅、`platform_connections`、`platform_authorizations`、业务部、区域、门店归属继续共用现有体系，通过 `business_vertical='general'` 隔离 |
| 数据面 | 全部拆开 | 9 个 Excel 全部进入 `*_general_*` 表，不再写入到餐现有事实表，避免字段语义、粒度和平台值污染 |
| 查询层 | 单独写 | 到综 UI/UX 与到餐不同，新增 general query service/API，围绕 `/daily`、`/users`、`/monitor`、`/stores/[store]` 组织 |
| RPA/worker | 平台共享，业态隔离 | scheduler/BullMQ/浏览器 worker 可复用，但任务入口必须按 `business_vertical` 过滤，connector factory 按 `(platform, business_vertical)` 路由 |

## 1. 本批 Excel 固定口径

| artifact_type | 文件 | 行数 | 粒度 | 一期用途 |
| --- | --- | ---: | --- | --- |
| `meituan_general_review_stats_daily` | `评价数据-20260601-20260603-282358728-1780557847362178.xlsx` | 5,049 | 日期 x 门店，平台值为 `ALL` | 评价数、好评数、差评数、差评回复率 |
| `meituan_general_review_comments` | `门店评价-2823587281780557898671.xlsx` | 309 | 评价明细 | 评价列表、星级、回复状态、文本分析 |
| `meituan_general_product_traffic_daily` | `商品分析-20260601-20260603-282358728-1780557959003187.xlsx` | 39,730 | 日期 x 门店 x 商品，平台值为 `ALL` | 商品访问、下单、访购率、商品偏好 |
| `meituan_general_transaction_type_daily` | `商品交易数据-20260601-20260603-282358728-178055800017962.xlsx` | 6 | 日期 x 商品类型，品牌级 | 品牌级团购/预订交易概览，不能用于单店 |
| `meituan_general_store_traffic_daily` | `客流数据-20260601-20260603-282358728-178055801869749.xlsx` | 5,049 | 日期 x 门店 | 曝光、访问、意向、下单、收藏、打卡 |
| `meituan_general_ranking_daily` | `榜单数据_仅展示上榜门店-20260601-20260603-282358728-1780558040174106.xlsx` | 26,569 | 日期 x 门店 x 榜单 | 城市/行政区/商圈排名监控 |
| `douyin_general_review_comments` | `门店评价_全部门店_2026_06_01-2026_06_03.xlsx` | 649 | 评价明细 | 评价列表、评分、回复状态 |
| `douyin_general_transaction_daily` | `交易_20260604.xlsx` | 8,891 | 日期 x 门店 x 商品 x 交易体裁 | 核销金额、核销券数、意向成交、退款、渠道分析 |
| `douyin_general_product_daily` | `商品_20260604.xlsx` | 38,783 | 日期 x 商品 x 投放渠道，无门店 ID | 品牌商品流量、直播/短视频/达人指标，不能伪造成单店商品流量 |

## 2. 目标架构

```text
platform_connections / stores / users / permissions
  -> shared control plane, filtered by business_vertical='general'

collection_runs / collection_artifacts
  -> shared artifact log, metadata.business_vertical='general'

*_general_* source tables
  -> exact Excel grain + header_hash + raw_row + parse warnings

apps/main/lib/jiuque-general/*
  -> scope resolver + daily + monitor + users + store detail queries

apps/main/app/api/jiuque-general/*
  -> API boundary for new Jiuque UI
```

## 3. 文件结构

### Database

- Create: `database/migrations/021_add_business_vertical_and_general_control_plane.sql`
- Create: `database/migrations/022_add_jiuque_general_source_tables.sql`
- Modify: `database/basic-store-schema.sql`
- Modify: `database/rpa-connections-schema.sql`

### Worker

- Create: `worker/src/parsers/general/excel.ts`
- Create: `worker/src/parsers/general/parse-utils.ts`
- Create: `worker/src/parsers/general/store-match.ts`
- Create: `worker/src/parsers/general/*-parser.ts`
- Create: `worker/src/connectors/meituan-general.ts`
- Create: `worker/src/connectors/douyin-general.ts`
- Create: `worker/src/scripts/dry-run-general-artifacts.ts`
- Modify: `worker/src/parsers/index.ts`
- Modify: `worker/src/scheduler.ts`
- Modify: `worker/src/index-browser.ts`

### Web App

- Create: `apps/main/lib/jiuque-general/types.ts`
- Create: `apps/main/lib/jiuque-general/scope.ts`
- Create: `apps/main/lib/jiuque-general/daily.ts`
- Create: `apps/main/lib/jiuque-general/monitor.ts`
- Create: `apps/main/lib/jiuque-general/users.ts`
- Create: `apps/main/lib/jiuque-general/store-detail.ts`
- Create: `apps/main/app/api/jiuque-general/*/route.ts`
- Create: `apps/main/app/jiuque/*/page.tsx`

## 4. Task Plan

### Task 1: 共享控制面业态隔离

**目标：** 给共享控制面补 `business_vertical`，让到餐和到综共用表但不混数据。

**Files:**
- Create: `database/migrations/021_add_business_vertical_and_general_control_plane.sql`
- Modify: `database/basic-store-schema.sql`
- Modify: `database/rpa-connections-schema.sql`

- [ ] Step 1: 新增业态字段和索引。

```sql
ALTER TABLE stores
  ADD COLUMN IF NOT EXISTS business_vertical VARCHAR(32) NOT NULL DEFAULT 'dining',
  ADD CONSTRAINT stores_business_vertical_check
    CHECK (business_vertical IN ('dining', 'general'));

ALTER TABLE platform_connections
  ADD COLUMN IF NOT EXISTS business_vertical VARCHAR(32) NOT NULL DEFAULT 'dining',
  ADD CONSTRAINT platform_connections_business_vertical_check
    CHECK (business_vertical IN ('dining', 'general'));

ALTER TABLE platform_authorizations
  ADD COLUMN IF NOT EXISTS business_vertical VARCHAR(32) NOT NULL DEFAULT 'dining',
  ADD CONSTRAINT platform_authorizations_business_vertical_check
    CHECK (business_vertical IN ('dining', 'general'));

CREATE INDEX IF NOT EXISTS idx_stores_business_vertical ON stores(business_vertical);
CREATE INDEX IF NOT EXISTS idx_platform_connections_platform_vertical
  ON platform_connections(platform, business_vertical, connection_status);
```

- [ ] Step 2: 新增业务部、区域、门店归属表。

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
```

- [ ] Step 3: 验证。

```bash
cd worker
npm test -- test/integration/migrations.test.ts
```

Expected: 现有到餐数据默认 `dining`，新建久雀连接可写 `general`。

### Task 2: 新增九张 `*_general_*` source 表

**目标：** 9 个 Excel 全量进入到综 source 表，不写到餐事实表。

**Files:**
- Create: `database/migrations/022_add_jiuque_general_source_tables.sql`

- [ ] Step 1: 所有 source 表必须包含审计字段。

```sql
source_run_id UUID REFERENCES collection_runs(id) ON DELETE SET NULL,
source_connection_id UUID REFERENCES platform_connections(id) ON DELETE SET NULL,
source_artifact_id UUID REFERENCES collection_artifacts(id) ON DELETE SET NULL,
source_file_name TEXT,
header_hash VARCHAR(64),
grain VARCHAR(64) NOT NULL,
raw_row JSONB NOT NULL DEFAULT '{}'::jsonb,
parse_warnings JSONB NOT NULL DEFAULT '[]'::jsonb,
needs_store_review BOOLEAN NOT NULL DEFAULT false,
created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
```

- [ ] Step 2: 建表清单。

```text
meituan_general_review_stats_daily
meituan_general_review_comments
meituan_general_product_traffic_daily
meituan_general_transaction_type_daily
meituan_general_store_traffic_daily
meituan_general_ranking_daily
douyin_general_review_comments
douyin_general_transaction_daily
douyin_general_product_daily
```

- [ ] Step 3: 唯一键规则。

| Table | Unique key |
| --- | --- |
| `meituan_general_review_stats_daily` | `(source_connection_id, stat_date, platform_store_id)` |
| `meituan_general_review_comments` | `(source_connection_id, platform_comment_id)` |
| `meituan_general_product_traffic_daily` | `(source_connection_id, stat_date, platform_store_id, product_id)` |
| `meituan_general_transaction_type_daily` | `(source_connection_id, stat_date, product_type)` |
| `meituan_general_store_traffic_daily` | `(source_connection_id, stat_date, platform_store_id)` |
| `meituan_general_ranking_daily` | `(source_connection_id, stat_date, platform_store_id, source_platform, ranking_name, ranking_city, ranking_district, ranking_business_district)` |
| `douyin_general_review_comments` | `(source_connection_id, platform_review_id)` |
| `douyin_general_transaction_daily` | `(source_connection_id, stat_date, platform_store_id, product_id, transaction_type_level1, transaction_type_level2)` |
| `douyin_general_product_daily` | `(source_connection_id, stat_date, product_id, channel)` |

- [ ] Step 4: 验证。

```bash
npm run type-check
cd worker && npm test -- test/integration/migrations.test.ts
```

Expected: migration test passes.

### Task 3: 通用 Excel parser 基础设施

**目标：** 所有到综 parser 使用同一套 strict reader、header sentinel、字段解析和门店匹配。

**Files:**
- Create: `worker/src/parsers/general/excel.ts`
- Create: `worker/src/parsers/general/parse-utils.ts`
- Create: `worker/src/parsers/general/store-match.ts`

- [ ] Step 1: 实现 Excel 读取结果。

```ts
export interface ExcelReadResult {
  sheetName: string
  headers: string[]
  headerHash: string
  rows: Array<Record<string, unknown>>
}
```

- [ ] Step 2: 实现 `assertHeaders(actual, expected, artifactType)`。

Required behavior: 缺列返回 `COLUMN_DRIFT`，不允许按列序静默解析。

- [ ] Step 3: 实现 parse helper。

```text
parseDateDay
parseInteger
parseDecimal
parsePercent
parseChineseBoolean
parseStarRating
normalizeBlank
stableHash
```

- [ ] Step 4: 实现严格门店匹配。

匹配顺序：平台门店 ID、`platform_connection_stores`、同城市标准化门店名；仍未命中则 `needs_store_review=true`。禁止 fallback 到任意门店。

### Task 4: 美团 6 个 parser

**目标：** 导入美团经营宝 6 个 Excel。

**Files:**
- Create: `worker/src/parsers/general/meituan-review-stats-parser.ts`
- Create: `worker/src/parsers/general/meituan-review-comments-parser.ts`
- Create: `worker/src/parsers/general/meituan-product-traffic-parser.ts`
- Create: `worker/src/parsers/general/meituan-transaction-type-parser.ts`
- Create: `worker/src/parsers/general/meituan-store-traffic-parser.ts`
- Create: `worker/src/parsers/general/meituan-ranking-parser.ts`

- [ ] Step 1: 每个 parser 固定 2026-06-01 到 2026-06-03 样本 header。
- [ ] Step 2: `商品交易数据` 强制标记 `grain='brand_date_product_type'`，不能作为单店数据。
- [ ] Step 3: `商品分析` 保留 `source_platform='ALL'`，页面展示为“点评+美团”。
- [ ] Step 4: `榜单数据` 保留一店一天多榜单明细，不压成 `store_rankings` 宽表。
- [ ] Step 5: dry-run 预期行数：`5049, 309, 39730, 6, 5049, 26569`。

### Task 5: 抖音 3 个 parser

**目标：** 导入抖音来客 3 个 Excel。

**Files:**
- Create: `worker/src/parsers/general/douyin-review-comments-parser.ts`
- Create: `worker/src/parsers/general/douyin-transaction-parser.ts`
- Create: `worker/src/parsers/general/douyin-product-parser.ts`

- [ ] Step 1: `douyin_general_review_comments` 用 `门店ID + 评价时间 + 用户昵称 + 文本内容 hash` 生成稳定 `platform_review_id`。
- [ ] Step 2: `douyin_general_transaction_daily` 按门店、商品、交易体裁导入。
- [ ] Step 3: `douyin_general_product_daily` 保持品牌/商品/渠道粒度，`store_id=null`。
- [ ] Step 4: 文件表头为 `周` 时返回 `UNSUPPORTED_PERIOD`。
- [ ] Step 5: dry-run 预期行数：`649, 8891, 38783`。

### Task 6: parser 路由与 dry-run CLI

**Goal:** route general artifacts and validate all nine files before writing DB rows.

**Files:**
- Modify: `worker/src/parsers/index.ts`
- Create: `worker/src/scripts/dry-run-general-artifacts.ts`
- Modify: `worker/package.json`

- [ ] Step 1: 路由 9 个 artifact type 到 `parseGeneralArtifact`。
- [ ] Step 2: 添加 `npm run dry-run:general`。
- [ ] Step 3: 跑 9 个样本，输出 `artifact_type / row_count / header_hash / grain / warnings`。

```bash
cd worker
npm run dry-run:general -- /Users/adonis/Downloads/评价数据-20260601-20260603-282358728-1780557847362178.xlsx /Users/adonis/Downloads/门店评价-2823587281780557898671.xlsx /Users/adonis/Downloads/商品分析-20260601-20260603-282358728-1780557959003187.xlsx /Users/adonis/Downloads/商品交易数据-20260601-20260603-282358728-178055800017962.xlsx /Users/adonis/Downloads/客流数据-20260601-20260603-282358728-178055801869749.xlsx /Users/adonis/Downloads/榜单数据_仅展示上榜门店-20260601-20260603-282358728-1780558040174106.xlsx /Users/adonis/Downloads/门店评价_全部门店_2026_06_01-2026_06_03.xlsx /Users/adonis/Downloads/交易_20260604.xlsx /Users/adonis/Downloads/商品_20260604.xlsx
```

Expected: no `COLUMN_DRIFT`.

### Task 7: Worker 业态隔离

**Goal:** dining/general worker 不能互相取任务。

**Files:**
- Modify: `worker/src/scheduler.ts`
- Modify: `worker/src/index-browser.ts`
- Modify: `worker/src/scheduler/reschedule-after-run.ts` if needed

- [ ] Step 1: `WORKER_BUSINESS_VERTICAL=dining|general` 必填。

```ts
const workerBusinessVertical = process.env.WORKER_BUSINESS_VERTICAL
if (workerBusinessVertical !== 'dining' && workerBusinessVertical !== 'general') {
  throw new Error('WORKER_BUSINESS_VERTICAL must be dining or general')
}
```

- [ ] Step 2: pending auth 和 scheduled SELECT 都加 `pc.business_vertical = $1`。
- [ ] Step 3: worker-browser 加载连接后二次校验 vertical。
- [ ] Step 4: 跑隔离测试。

```bash
cd worker
npm test -- test/integration/dispatcher.test.ts test/integration/assignWorker.test.ts
```

### Task 8: 到综 connector

**Goal:** 到综下载动作进入独立 connector，不塞进现有到餐 connector。

**Files:**
- Create: `worker/src/connectors/meituan-general.ts`
- Create: `worker/src/connectors/douyin-general.ts`
- Modify: `worker/src/index-browser.ts`

- [ ] Step 1: connector factory 改为 `(platform, business_vertical)` 路由。
- [ ] Step 2: `MeituanGeneralConnector` 产出 6 个美团 artifact。
- [ ] Step 3: `DouyinGeneralConnector` 产出 3 个抖音 artifact。
- [ ] Step 4: 每个 artifact 写入 `metadata.business_vertical='general'`、`period_type='day'`、`source_grain`。

### Task 9: 到综 query service

**Goal:** 单独服务久雀 UI，不复用到餐 report builder 查询。

**Files:**
- Create: `apps/main/lib/jiuque-general/types.ts`
- Create: `apps/main/lib/jiuque-general/scope.ts`
- Create: `apps/main/lib/jiuque-general/daily.ts`
- Create: `apps/main/lib/jiuque-general/monitor.ts`
- Create: `apps/main/lib/jiuque-general/users.ts`
- Create: `apps/main/lib/jiuque-general/store-detail.ts`

- [ ] Step 1: scope resolver 按 `general -> user access -> department -> region -> store -> date-valid assignment` 过滤。
- [ ] Step 2: `/daily` 返回总数据、平台卡片、数据可用性说明。
- [ ] Step 3: `/monitor` 只启用 Excel 支撑规则：营收下滑、曝光下滑、转化下降、退款异常、差评率、未回复、榜单下降。
- [ ] Step 4: `/users` 只返回弱信号，不能声明用户级留存/流失。
- [ ] Step 5: `/stores/[storeId]` 对品牌级 source 返回 `unsupported_for_store`。

### Task 10: 到综 API routes

**Goal:** 暴露新 query service。

**Files:**
- Create: `apps/main/app/api/jiuque-general/daily/route.ts`
- Create: `apps/main/app/api/jiuque-general/monitor/route.ts`
- Create: `apps/main/app/api/jiuque-general/users/route.ts`
- Create: `apps/main/app/api/jiuque-general/stores/[storeId]/route.ts`

- [ ] Step 1: 复用现有 session/auth/access-control pattern。
- [ ] Step 2: 校验 `startDate/endDate/departmentId/regionId/storeId`。
- [ ] Step 3: 测试 owner/admin/child user 权限和 `business_vertical='general'` 过滤。

### Task 11: 久雀页面

**Goal:** 做新业态 UI，不复制到餐页面结构。

**Files:**
- Create: `apps/main/app/jiuque/daily/page.tsx`
- Create: `apps/main/app/jiuque/monitor/page.tsx`
- Create: `apps/main/app/jiuque/users/page.tsx`
- Create: `apps/main/app/jiuque/stores/[storeId]/page.tsx`

- [ ] Step 1: `/jiuque/daily`：业务部/区域/门店筛选、总数据、平台卡片、覆盖说明。
- [ ] Step 2: `/jiuque/monitor`：风险总览、规则分类、门店风险列表、证据抽屉。
- [ ] Step 3: `/jiuque/users`：新客弱信号、商品包型偏好、渠道偏好、不支持生命周期说明。
- [ ] Step 4: `/jiuque/stores/[storeId]`：单店 KPI、渠道、商品、评价、榜单、不支持包间/用户模块说明。

### Task 12: 验收与观测

**Goal:** parser、worker、query、UI 有可验证闭环。

**Files:**
- Modify: `worker/src/parsers/index.ts`
- Create: `apps/main/lib/jiuque-general/coverage.ts`
- Create: `apps/main/app/api/jiuque-general/coverage/route.ts`

- [ ] Step 1: artifact parse metadata 记录 `header_hash`、`parsed_rows`、`inserted_rows`、`grain`。
- [ ] Step 2: coverage endpoint 返回每个指标 `supported/partial/unsupported` 和原因。
- [ ] Step 3: 全量验证。

```bash
npm run lint
npm run type-check
npm test
cd worker && npm test
```

## 5. 验收标准

1. 9 个 Excel dry-run 行数准确，header hash 稳定。
2. `*_general_*` parser 不写任何到餐事实表。
3. 所有 API/query 都过滤 `business_vertical='general'`。
4. 品牌级文件不展示成单店事实。
5. worker 入口按 `WORKER_BUSINESS_VERTICAL` 隔离。
6. 页面明确显示 unsupported 指标，不伪造包间、翻台、用户生命周期。
7. 重复导入同一 artifact 幂等。

## 6. 延后范围

- 包间数、房型、房态、包间使用率。
- 翻台率、单包间产值。
- 用户级留存、流失、生命周期和用户列表。
- 私域好友、企微触达。
- 设备合规、运营执行、财务、回本周期。
