# Food ordering & dispatch system — build plan

One web link you send to customers. They order, you receive a ticket, you send it to a driver,
the driver accepts with an ETA and delivers. You (moderator) control everything and the
accounts are kept automatically.

Status key: `[x]` done in prototype · `[ ]` to build · `[?]` needs a decision from you

---

## 1. Who uses it

| Role | How they get in | What they do |
|---|---|---|
| **Customer** | The link you send (no app install, no account needed to order) | Browse menu, use deals, order, track, message the shop |
| **Driver** | Their own driver link + login | Get tickets, accept with ETA or decline, mark picked up / delivered, message the shop |
| **Moderator (you + staff)** | Admin link + login | See every order, dispatch to drivers, edit menu, deals, drivers, settings, accounts |

---

## 2. Feature checklist

### Customer side
- [x] Menu by category, sold-out items greyed out
- [x] Basket with quantities
- [x] Meal deals at the top of the menu, spend deals applied automatically, promo codes
- [x] Checkout: name, phone, address, comment for the shop/driver
- [x] Order tracking page: received → being prepared → driver assigned (with ETA) → on the way → delivered
- [x] Chat with the shop on the order
- [x] Live notifications when the order changes (in-page in prototype)
- [x] "Shop closed" mode blocks new orders
- [ ] Real push notifications to the phone (PWA web push), SMS fallback
- [ ] Payment: customers pay the driver on delivery (cash or card reader). No online payments for now
- [ ] Optional customer account: saved address, reorder, order history
- [ ] Delivery area / postcode check and distance-based fee `[?]`
- [ ] Item options (sauces, sizes, extras) and allergen info
- [ ] Opening hours schedule (auto open/close)
- [ ] "Add to home screen" so it behaves like an app (PWA)

### Moderator side (full control)
- [x] Orders board with tabs: New · Sent to driver · On the way · Completed · Cancelled · All
- [x] Ticket view: customer details, items, totals, deal used, full status history
- [x] Send ticket to a driver with a note
- [x] Reassign if a driver declines or doesn't respond
- [x] Mark delivered / cancel order
- [x] Two separate comment threads per order: customer ↔ shop and driver ↔ shop
- [x] Notification bell + live pop-ups for new orders, accepts, declines, messages
- [x] Menu editor: add, edit price, mark sold out, delete
- [x] Deals you switch on and off whenever you like (optional end time):
  - Meal deals: pick items and quantities, set one price (e.g. 2× wings + fries for £17). Shown at the top of the menu
  - Spend deals: spend £X, get £ off / % off / free delivery. Applied automatically, customers are told how much more to spend
  - Promo codes you hand out yourself
- [x] Usage and customer savings for every deal
- [ ] Loyalty deal: e.g. every 5th order gets £5 off (needs customer accounts or phone number matching) `[?]`
- [x] Drivers: add, on/off duty, delivery counts
- [x] Settings: shop name, currency, delivery fee, open/closed
- [ ] Staff accounts with permissions (owner vs staff)
- [ ] Audit log (who changed what)
- [ ] Refunds and partial refunds
- [ ] Print/thermal printer ticket output for the kitchen `[?]`
- [ ] Auto-assign to nearest/free driver (later)

### Driver side
- [x] Own view showing only their tickets
- [x] Accept with ETA (quick buttons 10/15/20/30/45 min or custom) or decline
- [x] Address with "open in Maps" link, customer phone
- [x] Picked up / Delivered buttons
- [x] Chat with the shop per order
- [x] Today's deliveries and earnings
- [x] On/off duty
- [ ] Push notification + sound for a new ticket (must work with phone locked)
- [ ] Driver login (phone number + code)
- [ ] Live GPS location shared with the shop/customer (later, optional)
- [ ] Proof of delivery photo (later, optional)

### Accounting
- [x] Totals for Today / 7 days / 30 days / All: orders, sales, discounts, delivery fees, takings, average order
- [x] Daily breakdown table
- [x] Deliveries per driver
- [x] Deal usage and how much each deal cost you
- [x] CSV export (copy into Excel / Google Sheets)
- [ ] Record cash vs card each driver collected on delivery
- [ ] Driver pay tracking (later, once you decide how drivers are paid)
- [ ] Expenses entry (ingredients, fuel) for real profit
- [ ] VAT/tax report `[?]`

### Look & feel
- [x] Mobile-first, works on any phone browser, light and dark mode
- [ ] Your logo, colours and food photos `[?]`
- [ ] Your own domain, e.g. `order.yourkitchen.com` `[?]`

---

## 3. Recommended tech (free to start)

| Need | Tool | Cost to start |
|---|---|---|
| Website hosting | **Cloudflare Pages**, Netlify or Vercel | Free |
| Database, logins, live updates | **Supabase** (Postgres + Auth + Realtime) | Free tier |
| App feel on phones | **PWA** (installable web app, push notifications) | Free |
| SMS fallback (optional) | Twilio or similar | Pay as you go, pennies per text |
| Email receipts (optional) | Resend / Brevo free tier | Free at low volume |
| Domain (optional) | Any registrar | About £10 a year |

Realistic running cost: **£0/month** while testing and starting out. If you grow, the
first paid upgrade is usually Supabase Pro (about $25/month). Free tiers change, so check
current limits before launch. One catch to know: Supabase free projects pause after a week
with no activity, so a live business will want to keep it active or upgrade.

A native App Store / Play Store app isn't needed at first. A PWA can be added to the home
screen and get notifications. If we want store apps later, the same code can be wrapped with
Capacitor.

---

## 4. Data we store

- **orders**: number, time, customer (name, phone, address, note), items, subtotal, discount,
  deal code, delivery fee, total, status, driver, ETA, status history
- **messages**: order, thread (customer or driver), sender, text, time
- **menu_items**: name, category, description, price, available
- **deals**: kind (meal deal, spend deal, promo code), items + price for meal deals, reward type + amount, minimum spend, code, end time, on/off
- **drivers**: name, phone, on duty, login
- **notifications**: who it's for, text, order, time, read
- **settings**: shop name, currency, delivery fee, open/closed, hours

Security: row-level rules so customers only see their own order, drivers only see orders sent
to them, and only moderators can see everything and change settings.

---

## 5. Order flow

```
Customer places order ──► NEW (you get a notification)
You send ticket to driver ──► SENT TO DRIVER (driver gets a notification)
   Driver declines ──► back to NEW, you reassign
Driver accepts + ETA ──► ACCEPTED (you and the customer see the ETA)
Driver collects ──► OUT FOR DELIVERY
Driver delivers ──► COMPLETED (goes into accounts)
You can CANCEL at any point before completion.
```

---

## 6. Build phases

1. **Prototype** (this folder) — clickable demo of all three sides. Data lives in your browser only.
2. **Real backend** — Supabase database, logins for moderator and drivers, live updates across phones.
3. **Notifications** — PWA install + web push for drivers and customers, sound alert for new tickets.
4. **Cash-up** — record what each driver collected on delivery, end-of-day totals, refunds.
5. **Accounting+** — expenses, exports, tax report (driver pay when you're ready).
6. **Launch** — your branding, domain, real menu, test with a few customers and drivers.

---

## 7. Decisions needed from you

- [ ] Business name, logo, colours, food photos
- [ ] Real menu and prices, item options (sizes, sauces)
- [ ] Country / currency (prototype uses £, changeable in Settings)
- [ ] Delivery area and fee (flat or by distance), minimum order
- [ ] Opening hours
- [ ] Which deals to launch with, and whether you want loyalty deals (every Nth order)
- [ ] Customers order as guests, or create an account?
