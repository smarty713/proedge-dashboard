# ProEdge partner dashboard

Executive dashboard for Smarty and Tom Kelly. GitHub Pages serves `main` at <https://smarty713.github.io/proedge-dashboard/>.

The passcode gate is unchanged. Do not edit its markup or the gate script.

Every figure on the page comes from `data/current.json`. A weekly update is a new JSON snapshot. Leave `.nojekyll` in place so Pages serves the `data` directory as files.

Charts use Chart.js 4.4.6 from `vendor/chart.umd.min.js`, so the page does not depend on a third-party host.

## Update a week

1. Edit `data/current.json`. Set `as_of` to the snapshot date (`YYYY-MM-DD`).
2. Copy that file to `data/history/YYYY-MM-DD.json`.
3. Append the date to `snapshots` if it is not already there. Keep the list in chronological order.
4. Commit to `main` and push.

The chart plots each snapshot on the Monday of its week. If a week has more than one snapshot, the later date is the one shown. Weeks in the `as_of` month with no snapshot stay blank. They are not drawn as zero.

A money field set to `null`, or a profit with `profit_pending: true`, renders as `pending`. Pending profit is not treated as zero and is left out of the product-profit sum.

## JSON schema

`data/current.json` and each `data/history/YYYY-MM-DD.json` use the same object.

### Top level

| Field | Type | Meaning |
| --- | --- | --- |
| `schema` | number | Schema version. Current version is 1. |
| `as_of` | string | Snapshot date, `YYYY-MM-DD`. Shown in the header. |
| `company` | string | Legal name in the header and footer. |
| `partners` | array | `{ "id", "name" }`. Order is display order. |
| `split_rule` | object | How monthly profit is divided. |
| `kpis` | object | Hero totals. |
| `split` | object | This month's split, in dollars. |
| `products` | array | One object per product line, in display order. |
| `unique_customers` | number | Distinct customers. |
| `customer_status` | array | `{ "id", "label", "count" }`. |
| `past_due` | array | `{ "name", "amount", "failed" }`. `failed` is `YYYY-MM-DD` or `null`. |
| `snapshots` | array of strings | Dates that have a file in `data/history/`. |
| `unique_customer_note` | string, optional | Shown under the unique-customer count. |
| `weekly_actuals` | array | TD Bank weeks. Each item is `{ "week_start", "income", "expense", "profit" }`. `week_start` is `YYYY-MM-DD`. Money fields are numbers, or `null` when unknown. An empty array renders the pending state. |

### `split_rule`

| Field | Type | Meaning |
| --- | --- | --- |
| `base_partner` | string | Partner id who receives the base. |
| `base_amount` | number | First dollars of monthly profit that go to `base_partner`. |
| `remainder_share` | number | Each partner's share of the remainder. `0.5` is an even split. |
| `carryforward` | boolean | `false` means the base resets every month. Nothing carries into the next month. |

### `kpis`

| Field | Type | Meaning |
| --- | --- | --- |
| `revenue` | number | Total revenue. |
| `profit` | number | Total profit. |
| `affiliate_payouts` | number | Affiliate payouts. |
| `affiliate_note` | string | Short note under that figure. |
| `owed` | number | Amount past due. |

### `split`

| Field | Type | Meaning |
| --- | --- | --- |
| `profit` | number | Profit the split is applied to. Same as `kpis.profit` when the books tie. |
| `base` | number | Dollars assigned as the base this month. |
| `remainder` | number | Profit after the base. |
| `shares` | object | Keyed by partner id. Each value is `{ "base", "remainder", "total" }`. |

`total` is that partner's `base` plus `remainder`. The partner totals add up to `profit`.

### `products`

| Field | Type | Meaning |
| --- | --- | --- |
| `id` | string | Stable id. |
| `label` | string | Card and chart name. |
| `detail` | string, optional | Subtitle, such as a line that includes another offer. |
| `customers` | number | Customers on this line. |
| `revenue` | number | Revenue. |
| `vendor_cost` | number or null | Vendor cost. `null` renders as pending. |
| `vendor_cost_note` | string, optional | Shown under the card. |
| `affiliate_payouts` | number, optional | Shown on the card when this line pays affiliates. |
| `profit` | number or null | Profit. `null` when it is not known. |
| `profit_pending` | boolean | When true, profit renders as pending. |
| `note` | string, optional | Shown under the card. |
| `count_label` | string, optional | Word after the customer count. Defaults to `customers`. |

Health on the card is derived, not stored. Pending when profit is pending. Loss when profit is negative. Healthy when profit is greater than zero.

### History file

`data/history/YYYY-MM-DD.json` is the `current.json` document as of that date. The month-progress chart reads `as_of`, `kpis.revenue`, and `kpis.profit` from each file named in `snapshots`.

The 4-week Income / Expense / Profit chart reads `weekly_actuals` from `data/current.json` only. Leave that array empty until the TD Bank figures exist. Do not fill it with estimates.

### Tie-outs the page computes

The page adds product revenue, known product profit, past-due amounts, and customer-status counts. If a sum does not match the stated total or `unique_customers`, it prints the difference under that section. It does not add a plug figure to force a balance.

## November 12, 2026 snapshot

Product customer counts are the counts from the previous dashboard, with the November 12 adjustments below. CRM is one lower because one canceled CRM customer is no longer included.

Unique customers are 98. Robert Weichelt and Jeffery B Jewelers are excluded. Mark States is included (Seller Assist, $82.50/mo). That $82.50 is not added again on top of Seller Assist revenue.

`$1 SaaS seats` is $9.00 for 9 seats. With that line, product revenue matches the stated total. Its vendor cost and profit are pending.

Customer statuses are paying 86, past due 3, and not billed 9. Those add to 98.

CRM vendor cost is the GHL Founders Subscription from the previous dashboard. It is the amount left after affiliate payouts are taken out of CRM revenue to reach CRM profit.

Seller Lead Engine vendor cost is the half of revenue that is not profit.

Service Plans vendor cost is zero because profit equals revenue.

AI Employee profit and vendor cost are pending. An unconfirmed cost in the previous file was not carried forward.

Seller Assist has 60 paying customers. Vendor cost is 57 billable seats at $49, which is $2,793. Three payers have no vendor seat yet. Two internal accounts on the vendor list are not customers.

`weekly_actuals` is empty until TD Bank weekly income, expense, and profit are supplied. The page then draws grouped bars for income and expense and a line for profit.

Do not put a customer price-tier split on the page.
