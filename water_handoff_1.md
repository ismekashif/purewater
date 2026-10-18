# Clear Water — prototype handoff and decision log

## Package and operation
Five HTML documents: four role-specific single-page prototypes, plus a reviewer entry. Each role contains connected screen views, not a production multi-role application. Open the reviewer HTML first, or preview any role HTML directly. HTML embeds the shared theme as a fallback and also references the CSS theme. Google Fonts, Tailwind, Flowbite and Lucide are CDN resources. Core interaction logic is vanilla JavaScript; Flowbite is imported but does not supply the custom dialogs or navigation.

For the PWA demo, serve the whole extracted bundle over HTTPS or localhost. Example from the extracted folder: `python -m http.server 8000`, then open the reviewer entry at localhost:8000. The manifest and service worker need relative assets together. Installation varies by browser and platform and is not browser-certified in this delivery. The worker caches public demo documents only; NEVER deploy it for authenticated production pages. It does not cache APIs or payment traffic. Google Fonts/CDN scripts may be unavailable offline; core demo styling and actions do not depend on successful icon rendering. No production push subscription, OTP, payments, authorization, dispatch or persistence backend is implemented. Demo state is mostly in-memory and resets on reload. Customer and other role prototypes do not share a real data store or synchronized events.

## Information architecture
Customer: Home, Orders, Account; persistent cart access; checkout, auth, payment result, order tracking, saved address editor, deposit records.
Store: Orders, Dispatch; operational group filters, selected-order details, readiness and manual assignment/override dialogs.
Rider: Requests, Active deliveries, History; request outcomes, countdown and delivery completion confirmation.
Admin: Overview, Products, Staff, Orders, Deposits, Reports, Settings.
Reviewer entry is a review tool only, not customer role switching or role authorization.

## Ten acceptance-review scenarios
1. First customer: Home reviewer selector -> First-time customer -> Add 20 L bottle -> Cart -> Checkout -> Sign in. Demo OTP 123456 passes; another code fails; 000000 expires. Continue from checkout -> simulated payment -> confirmed outcome -> tracking.
2. Scheduled repeat: Orders -> Review reorder -> Checkout -> Schedule delivery -> choose date and window -> inspect summary -> pay simulation.
3. Existing bottle: one applicable bottle -> selected/unselected option -> inspect -₹200 and total. Multiple applicable bottles are blocked from checkout in the demo until policy is resolved.
4. Pending payment: payment reviewer selector -> Pending/unverified -> Check status. No second-payment action exists. Also inspect paid/order-unconfirmed and customer-exit states.
5. First rider declines: Store -> selected detail -> first-rider-declines state. Rider -> decline confirmation -> request becomes nonactionable. These are separate simulations, not synchronized clients.
6. All eligible riders exhausted: Store -> Needs attention or dispatch reviewer control -> exhausted -> manual assignment.
7. Override: Store -> assigned state -> Review assignment override. Enable stale assignment checkbox and confirm to inspect rejected write and fresh-state review.
8. Delay: Customer -> tracking -> delayed exception -> revised sample window and help placeholder.
9. Reorder changes: Orders -> changed-reorder control -> cart highlights sample reprice and explains an unavailable pack was omitted.
10. Push denial: Customer Account or Admin Settings -> preferences -> simulated denied -> in-app notifications remain usable.
Bonus: offline control pauses writes; rider expiry removes actionability; request unavailable and acceptance unverified are separate states.

## Journey maps
Customer: browse -> cart/bottle review -> address/timing -> task-preserving auth -> payment reconciliation -> confirmed tracking; exceptions stay distinguishable.
Store: review -> prepare -> ready -> monitor automatic request -> manual intervention if exhausted -> monitor confirmed fulfillment.
Rider: review -> confirmed accept -> assignment (not departure) -> validated operational departure -> confirm delivered -> confirmed completion.
Admin: find -> edit/preview -> validate -> server-authorized save; configuration is blocked when its semantics are unknown.

## Behavior and implementation annotations
- All values, timestamps, windows, addresses, people, prices and reports are illustrative. The ₹200 one-bottle deposit comes from the supplied brief; other product amounts and quantities are mock values.
- The example 20 L price is ₹280 inclusive of ₹200, so its product component is ₹80. Selected example deduction gives ₹80 with the mock ₹0 delivery charge. Do not treat a zero mock fee as approved free delivery.
- Possession, exchange eligibility and recorded deposit balance are separate data concepts. No deposit refund/redemption actions are included.
- Quantity increases above one applicable bottle clear the example deduction and block payment review. This is a prototype safeguard, NOT the proposed production business rule.
- Immediate and scheduled estimates are explicitly samples. Date availability, unavailable-date rendering and live slot invalidation are not fully implemented.
- Auth resend uses a demo countdown. Production expiry, rate limits and attempt limits must come from the server. The demo country code is +91; country support requires approval.
- No order or delivery-update SMS exists. Phone fields are never sent externally.
- Payment states are distinct from order states. No retry while payment is unresolved. Cashfree redirects, signatures/webhooks and reconciliation need backend implementation.
- Timeline events are proposed. Out for delivery requires a real departure event, not acceptance. Rider departure is a reviewer control because the real workflow is unapproved.
- Store concurrency requires a versioned or equivalent server-controlled write. Conflict never silently retries an override.
- Rider expiration uses a demo clock; production server expiry is authoritative. No rider eligibility count or two-rider dispatch model is assumed.
- Completion uses a demo confirmation and separate simulated server outcome. Actual status-transition eligibility requires approval.
- Notification destinations mark that item read; list viewing does not. Read state is currently in memory. Push dialogs simulate outcomes without requesting browser permission. Real push permissions/subscriptions require separate implementation.
- No live maps, photos, delivery OTP, signatures, navigation integrations, cash on delivery or fabricated support contacts are included.
- Product save is simulated; staff mutation and incomplete dispatch-setting save are intentionally disabled.
- Admin report definitions are explicitly review assumptions. Deposits are displayed separately, not added to mock sales.
- Search/filter prototypes are lightweight; combined filters, pagination and full audit history need implementation.

## Permissions and data contracts
All authentication and role restrictions MUST be enforced on the server. Demo reviewer access is not authorization.
Customer: own orders, addresses and supported deposit records. Store: authorized location orders and permitted dispatch actions. Rider: requests they may respond to and their confirmed deliveries. Admin: business-approved scoped management.
Required entities: product with price/deposit treatment/stock; address with coverage result; availability window with timezone/capacity; cart quote with currency/components/expiry; payment attempt with verified status; order with independent fulfillment status; assignment request with expiry and version; rider eligibility; operational event log; linked deposit records; per-user notifications; role permissions; versioned dispatch configuration.
Mutations require duplicate-submission protection and idempotency where appropriate, server validation, and authoritative response before success UI. Unknown outcomes must remain unknown until reconciled. Never automatically replay financial or fulfillment writes on reconnect.

## Shared components and tokens
Theme: background #F4F8F7, white cards, foreground #142D2A, primary #0F766E, muted text #526762. Warning #92400E on #FFFBEB; error #B42318 on #FEF3F2; success #166534 on #F0FDF4. DM Sans body, Manrope headings. 48px common actions; 56px rider actions. 4px spacing scale; 12px controls; 16px cards. Focus outline 3px with offset. Inline SVG product illustrations identify volume/pack type without invented photography.
Motion: quick 120ms, standard 180ms, panels up to 240ms; no bounce/shake/counter animation. CSS prefers-reduced-motion removes transitions and animations. Full keyboard, screen-reader, reflow, contrast and touch audits remain required on rendered views. Native dialogs provide keyboard modality; completion feedback is persistent in-view, while short cart acknowledgments may use a toast.

## Responsive layout
Mobile-first customer and rider. Customer product grid becomes two columns at 600px and three at 1024px. Store/admin switch to sidebar at 1024px and compact navigation below it. Tables have horizontal scrolling on narrow screens. Long text wraps; checkout stacks below desktop widths. Bottom navigation includes safe-area padding. Desktop customer bottom navigation is hidden; cart access remains available. Test at 360, 390, 768, 1024 and 1440 CSS pixels plus text zoom before sign-off.

## Decision log — unresolved, not silently implemented
D01 Deposit per bottle versus per order. Owner: business/finance.
D02 Multi-bottle deduction quantity and mixed eligibility. Owner: business.
D03 Empty bottle handover and missing-bottle treatment. Owner: operations/business.
D04 Existing deposit records and checkout eligibility. Owner: finance/business.
D05 Coverage validation and unsupported-address rules. Owner: operations.
D06 Immediate estimate source, cutoff, capacity, available dates, timezone. Owner: operations.
D07 Authentication gate, supported countries, recipient requirements. Owner: product/business.
D08 Cashfree reconciliation, retry eligibility, paid-but-order-unconfirmed recovery. Owner: engineering/finance.
D09 Fulfillment labels, ready/assignment/departure events and permissions. Owner: operations/engineering.
D10 Rider eligibility, actual count, sequential/parallel assignment and exhaustion. Owner: operations.
D11 Timeout trigger, expiry behavior and setting effective scope. Owner: operations/engineering.
D12 Assignment override authorization, replacement notification and race handling. Owner: operations/engineering.
D13 Customer-visible rider identity and approved help/contact route. Owner: privacy/business.
D14 Staff management actions and role permissions. Owner: business/security.
D15 Supported deposit fields, linkages and reconciliation. Owner: finance/engineering.
D16 Sales/order-volume definitions, deposit/fee/unresolved-payment exclusions. Owner: finance.
D17 Notification recipients, destination/read behavior and push implementation. Owner: product/engineering.
D18 Stock and price change acceptance, quote validity, scheduling revalidation. Owner: product/engineering.
D19 Install/offline production scope and secure local data retention. Owner: product/security.

## Coverage limitations and development readiness
This is a design-review package, not a finished production PWA or an end-to-end integrated system. Critical review scenarios are clickable using simulations. The CSV explicitly identifies handoff-only scheduling/connectivity states. Complete unavailable-date and slot-invalidated flows, real role enforcement, synchronized dispatch, support integration, robust address lists, real record-detail/history navigation, backend validation, audit logs and browser/device testing before development acceptance. Business decisions above remain sign-off blockers.

## Validation performed
Generated JavaScript is syntax-checked with Node. Dynamic screen rendering and generated control syntax are tested using a minimal DOM harness (not a browser). HTML links and packaging are statically checked. No full visual browser/device, screen-reader, live Cashfree, push, installability or security certification is claimed.
