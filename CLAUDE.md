# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 작업 규칙 (필수 준수)

1. **모든 답변은 한국어로 작성한다.**
2. **사용자 호칭은 "대장님"으로 한다.**
3. **작업 시작 전 반드시 "바로작업할까요?"라고 묻고 대장님의 확인을 받은 뒤 진행한다.** 단순 질문/조회는 제외, 파일 수정·생성·삭제 등 변경 작업에 적용.
4. **중요 작업 전에는 반드시 백업을 만든다.** `index.html` 등 핵심 파일을 수정하기 전에 `index.html.backup-YYYYMMDD-HHMM` 형식으로 사본을 남긴다.
5. **한 번에 너무 많이 변경하지 않는다.** 변경 범위를 작게 쪼개고, 한 작업 단위가 끝나면 대장님께 확인받은 뒤 다음으로 넘어간다.
6. **탭은 번호로 소통한다.** 예: "9-1", "4-3 요척계산기". 대장님이 탭 번호를 말씀하시면 해당 위치를 즉시 식별해 작업한다.
7. **색상명 이중 체계에 주의한다.** 특히 9-1에서 **판매 색상명**(예: `골드/유광`)과 **도금공장 색상명**(예: `대금/유광`)이 분리되어 있다. 두 체계 사이에서는 `sale_to_factory_map`을 통한 **역변환을 반드시 거친다**. 변환 누락 시 공장 발주서가 잘못 나간다.
8. **버전 형식은 `v{yyyymmdd}.{숫자}`로 통일한다.** 같은 날 안에서는 숫자만 증가(`.01` → `.02` → …), 날짜가 바뀌면 숫자는 `.01`로 리셋한다. 예: `v20260502.03` 다음날 첫 배포 → `v20260503.01`.
9. **접속 방식은 `index.html?v=숫자` 형식만 사용한다.** 브라우저 캐시 무효화를 위해 쿼리스트링 `?v=` 파라미터로 접속하며, 다른 형식은 사용하지 않는다.
10. **프로젝트 호칭은 "laceroom앱"으로 통일한다.** 대화·문서·커밋 메시지·UI 라벨 어디에서든 이 프로젝트를 가리킬 때는 "laceroom앱"으로 부른다. (예전 명칭인 "LACEROOM통합관리", "LACEROOM비서", "도금작업지시서" 등 혼용 금지.)

## What this is

`index.html` is the entire application — a ~1.15 MB, ~15k-line single-file React 18 app in Korean named "LACEROOM통합관리" (LACEROOM Integrated Management). It started as a plating (도금) work-order tool for 주식회사 골드 and has grown into an integrated dashboard covering plating orders, receivables, vendors/fabric suppliers, freelancer labor, salary, fixed/business expenses, subsidies, memos, accessory barcodes, master product data, user/PC management, and change logs.

There is no build system, no test suite, no `package.json`. React, ReactDOM, Tailwind, Babel-standalone, ExcelJS, and SheetJS are all loaded from CDNs in the `<head>`. JSX is transpiled in the browser by Babel-standalone inside one giant `<script type="text/babel">`.

## Running and testing

- **Run**: open `index.html` in a browser. There is no dev server. For full localStorage / file:// access, the codebase originally instructed users to launch Chrome with `--allow-file-access-from-files` (see the alert in the `useEffect` that reads localStorage).
- **Test**: there are no automated tests. Verification means clicking through the relevant tab in the browser, watching the console, and inspecting localStorage.
- **Lint/build**: none. Edits to `index.html` are live on next page reload.

## Big-picture architecture

Everything lives in one `App` function component. Key shapes to know before editing:

**Single-component, many-useState pattern.** `App` declares ~120 `useState` hooks at the top, one cluster per domain (products, history, vendors, salary, etc.). Each cluster has a paired `saveX(data)` helper that does both `setX(data)` and `localStorage.setItem('<key>', JSON.stringify(data))`. When adding a new domain, follow the same triple: `useState` initializer that lazily reads localStorage, plus a `save*` helper. Do **not** call `setState` and `localStorage.setItem` separately at call sites — always go through the `save*` helper so the change-tracking counter (see below) stays in sync.

**Tabs are state, not routes.** Top-level navigation is `activeMainTab` (values include `dashboard`, `plating`, `cost`, `receivable`, `fabricSuppliers`, `labor`, `salary`, `china`, `memo`, `productMgmt`, `fixedExpense`, `subsidy`, `userMgmt`, `changeLog`). Each tab is a `{activeMainTab === '...' && (...)}` block in the JSX. There is no router.

**localStorage is the database.** Every domain persists to a snake_case key (e.g., `master_products`, `plating_history`, `plating_products`, `plating_receivables`, `vendor_bills`, `vendor_payments`, `vendor_items`, `salary_employees`, `salary_records`, `salary_payments`, `plating_freelancers`, `plating_freelancer_works`, `fabric_orders`, `fabric_suppliers`, `dashboard_links`, `laceroom_memos`, `laceroom_recurring_memos`, `soldout_items`, `today_off_items`, `accessory_barcodes`, `business_info`, `pc_name`, `name_change_logs`, `polish_stocks`, `fixed_expenses`, `expense_categories`, `biz_expenses`, `biz_categories`, `yocheok_input`, `timer_settings`). Treat the key names as a stable schema — renaming one orphans user data.

**Inline migrations gated by sentinel keys.** When the data shape changes, do the migration inline at app startup and write a sentinel like `master_migrated_v1`, `gocchang_metalLabels_migrated`, or `gocchang_color_metal_merged` so it only runs once. Existing examples are in the `masterProducts` initializer (~line 468). Follow that pattern rather than mutating data at call sites.

**Backup change-tracking.** `laceroom_last_changed_at`, `laceroom_last_backup_at`, and `laceroom_unbacked_count` form a lightweight "you have N unsaved changes" indicator surfaced on the dashboard. `markChanged()` / `markBacked()` helpers near the top maintain this — wrap any new mutation path so the counter remains meaningful.

## Domain model: colors and finishes

The plating order shape is non-obvious and central to the Excel export.

- `COLOR_DEFS` enumerates colors (대금, 니켈, 흑진주, 흑골드, 로즈골드 대용금, 은, 14k대용금, 동착) with a `finishType` of `both` / `glossOnly` / `none` (older snapshots used `hasFinish: bool`). Finishes are 유광 / 반무광.
- Display strings are split between **screen** (`getColorDisplayName` — includes the parenthetical Korean explanation, for staff) and **Excel** (`excelColorName(name, finish)` — terse factory-facing label, with a special case for `로즈골드 대용금` whose finish formatting is non-uniform). Always pick the right one for the surface.
- Grind options (`GRIND_OPTIONS`) are 연마 / 연마 제외 / 연마 완료. A product's `grindRequired` flag controls which is offered as default and whether the grind dropdown is editable in the order form.
- Order rows are unique on `(productId, grind)`; the order form auto-rotates to the next available grind if the user re-adds a product.

## Excel export specifics

`handleDownloadExcel(order)` (and the various per-domain XLSX builders) uses ExcelJS to build a workbook with merged cells per product (one merged block of the date/name/image/grind columns spanning the product's color rows, plus a per-product "총수량" sum row using a `SUM(G…)` formula) and embeds the product image via base64 with `editAs: 'oneCell'`. When changing column order, update both the `headers` array and the `worksheet.columns` widths and every `row.getCell(N)` index — they are positional and will silently misalign.

## Repository hygiene

The repo is full of timestamped snapshot files (`laceroom-v20260421.*.html`, `index_*.html`, `도금작업지시서_*.html`, `LACEROOM비서_*.html`, `index.html.backup-*`). These are historical backups, not active code. **Only `index.html` is live.** Do not edit the snapshots, do not chase consistency between them, and do not delete them without the user's say-so. `README.md` is itself a snapshot (an old full-source dump of the v20260418.8 build); do not treat it as documentation.
