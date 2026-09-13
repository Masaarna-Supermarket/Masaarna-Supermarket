# Masaarna Supermarket — Shop Handbook

Everything: putting it online, connecting the pieces, and running the shop day to day.

- **732** products · **18** categories · **AED 0** to run · **3** files

---

## 1. What you have

Three files make the whole shop. No server to keep running, no database to back up,
nothing that can go down at night.

| File | What it is |
|---|---|
| `index.html` | The shop customers see — search, categories, Arabic, basket, WhatsApp button |
| `products.json` | Your 732 products. **The only file you ever edit** |
| `admin.html` | Your manager. Edits `products.json` for you so you never touch the raw file |
| `assets/` | Logo and the three QR codes. Upload this folder too |

Plus, optionally, a `worker/` folder — a small piece that records orders so you get a
sales history and a customer list. Everything works without it; you just would not have
order records.

Companion file: `MASAARNA-TEMPLATES.xlsx` — Products, Offers, Categories, Orders and
Access sheets, all pre-filled.

### How an order actually flows

1. Customer browses and taps `+` on what they want
2. They type their name, phone and area, and press the green button
3. WhatsApp opens on their phone with the whole order written out
4. They press send; it arrives in your WhatsApp
5. At the same moment, the recorder writes the order down (if you set it up)
6. You reply to confirm, agree payment, and deliver

No customer accounts, no card gateway, no passwords for shoppers. The thing people
already have on their phone does the work.

---

## 2. Accounts and keys

Nothing here needs a card.

| Account | What it gives you | Cost | Needed? |
|---|---|---|---|
| **GitHub** | Hosts the shop, gives you a public link | free | Yes |
| **Cloudflare** | Your own domain, the order recorder, a real login on the manager | free | Recommended |
| **ImgBB** | Hosts product photos, gives you links | free | Only for new photos |
| **A domain** | `masaarnasupermarket.ae` instead of a github.io address | ~AED 130/yr | Optional |

### Where each key lives

| Key | Where you get it | Where it goes |
|---|---|---|
| WhatsApp number | Your own phone | `products.json` → `"whatsapp"` |
| Order recorder address | Printed when you deploy the worker | `products.json` → `"orderApi"` |
| Admin key | You invent it | Cloudflare secret + typed into `admin.html` once |
| ImgBB key | imgbb.com → About → API | Typed into `admin.html` once |
| GitHub token | GitHub → Settings → Developer settings | Only if git asks for a password |

> **Never put these in products.json:** the admin key, the ImgBB key, the GitHub token.
> `products.json` is public — anyone can open it. Only the WhatsApp number and the
> recorder address belong there, and both are meant to be public.

---

## 3. Test on your computer first

Always. Nothing goes online until you have seen it working here.

```powershell
cd "D:\Masaarna supermarket\simple-shop"
npx serve
```

Then open:

| Page | Address |
|---|---|
| The shop | `http://localhost:3000` |
| The manager | `http://localhost:3000/admin.html` |

`Ctrl+C` stops it.

> **Double-clicking `index.html` will not work.** That gives you `file:///…` and the page
> cannot read `products.json` next to it — you get "Could not load the product list".
> Always go through the `localhost` address.

---

## 4. Go live on GitHub Pages

Free with no time limit, never sleeps. About ten minutes, and **no command line** —
everything is drag and drop in the browser.

### First: make a NEW repository

> **Do not use `masaarna-supermarket`.** That repository holds the older Node.js
> version of this project — a completely different thing, with no `index.html` at its
> root. Pointing GitHub Pages at it would serve the wrong files. Leave it alone as a
> backup of that work and start clean.

At github.com press **New repository**:

| Field | Value |
|---|---|
| Repository name | `masaarna-shop` |
| Visibility | **Public** — GitHub Pages needs this on a free account |
| Add a README | leave unticked, you have your own |

Press **Create repository**.

### Then: upload exactly these

On the empty repository page click **uploading an existing file**, then drag these in
from `D:\Masaarna supermarket\simple-shop`:

| Upload | | Why |
|---|---|---|
| `index.html` | **yes** | The shop itself |
| `products.json` | **yes** | Your 732 products |
| `assets` folder | **yes** | Logo and QR codes — drag the whole folder, it keeps its name |
| `README.md` | **yes** | So whoever opens the repo knows what it is |

And leave these behind:

| Do not upload | Why |
|---|---|
| `admin.html` | **Anyone who found it could change your prices.** See the warning below |
| `worker/` | Deploys to Cloudflare separately in section 5, not here |
| `MASAARNA-TEMPLATES.xlsx` | Your working spreadsheet, not part of the website |
| `FIX-THESE-33-LINKS.csv` | A to-do list for you, not part of the website |

**Nothing is deleted from your computer.** "Do not upload" just means do not drag those
into the browser. Every file stays exactly where it is.

Press **Commit changes** at the bottom.

### Then: switch Pages on

**Settings** (top of the repository) → **Pages** (left sidebar):

| Setting | Choose |
|---|---|
| Source | Deploy from a branch |
| Branch | `main` |
| Folder | `/ (root)` |

Press **Save**. Wait about a minute, refresh the page, and your address appears:

```
https://saadaasalrushed-dev.github.io/masaarna-shop/
```

That link is your shop. Send it to anyone.

### Do you need to give anyone your GitHub password?

**No.** You sign in yourself, and you upload through the browser. There is no token to
create and no command line, because you are not using `git` for any of this — just drag
and drop. Never share your GitHub password or a personal access token with anyone.

> **Why `admin.html` stays off the internet.** A web page has no login unless something
> in front of it provides one. Upload the manager and anyone who guesses the address can
> rewrite every price in your shop, and you would not know until a customer told you. It
> works perfectly from your own computer with `npx serve`. If you genuinely need it
> online later, put Cloudflare Access in front of it first — section 6.

---

## 5. Set up the order recorder

The only server piece. Free: 100,000 requests a day and a 5 GB database. A shop taking
200 orders a day uses about 0.2% of that.

```powershell
npx wrangler login

cd "D:\Masaarna supermarket\simple-shop\worker"
npx wrangler d1 create masaarna
```

Paste the printed `database_id` into `wrangler.toml` over `PASTE_YOUR_DATABASE_ID_HERE`.

```powershell
npx wrangler d1 execute masaarna --remote --file=schema.sql
npx wrangler secret put ADMIN_KEY      # invent a long password, 20+ characters
npx wrangler deploy
```

Check it: open `https://masaarna-orders.YOUR-NAME.workers.dev/health` — you want `{"ok":true,...}`.

Then in `products.json`:

```json
"orderApi":"https://masaarna-orders.YOUR-NAME.workers.dev",
```

Re-upload `products.json`. Orders start being recorded.

> **If the recorder is ever down, the order still goes through.** WhatsApp opens as normal
> — only the bookkeeping is lost. Reaching you always matters more than the record.

---

## 6. Cloudflare: your own domain, and a real login

Two separate things, both free, both optional.

### Your own domain

Plain `.ae` is an **unrestricted** zone — no trade licence, no residency, no documents.
Around AED 130 a year. (It is `.co.ae` that demands a licence, and you do not need it.)

1. Register `masaarnasupermarket.ae` at a TDRA-accredited registrar
2. Add the domain to Cloudflare (free plan), point the registrar at Cloudflare's nameservers
3. In Cloudflare DNS add a `CNAME`: `www` → `YOUR-USERNAME.github.io`
4. In GitHub: **Settings → Pages → Custom domain**, enter it, tick **Enforce HTTPS**

### A real login on the manager

This is the honest answer to "can the admin panel have a password". It cannot on its own —
but **Cloudflare Access** can put one in front of it. Free for up to 50 people, with Google
sign-in or a code sent to their email.

1. Cloudflare dashboard → **Zero Trust** → **Access** → **Applications** → **Add an application**
2. Choose **Self-hosted**, point it at where `admin.html` lives
3. Add a policy: *Allow* → *Emails* → list exactly who may enter
4. Save. Anyone else gets a login screen they cannot pass

That is where "users and privileges" actually lives — not in a spreadsheet, and not in the
page itself. Removing someone takes effect immediately.

---

## 7. Photo uploading (ImgBB key)

Only needed for photos not online yet. Links already in your Excel need none of this.

1. Sign up free at imgbb.com
2. **About → API** → generate a key
3. `admin.html` → **Photos** → paste it → **Remember key**

Stored in your browser only, never in any uploaded file.

---

## 8. Shop details, delivery range and links

All of this lives at the top of `products.json` under `"shop"`.

### Your details, already filled in

```json
"name":"Masaarna Supermarket",
"nameAr":"سوبرماركت مسارنا",
"tagline":"Trust & Heritage",
"location":{
  "lat":25.2619552,
  "lng":55.5870278,
  "mapUrl":"https://maps.app.goo.gl/b316qJY514DqY6J26",
  "areaName":"",
  "areaNameAr":""
}
```

The coordinates came from your own Google Maps link. **Fill in `areaName`** with how
you would say it to a customer — "Al Warqa 4", "near the mosque on the main road" —
and it appears in the footer. I left it blank rather than guess your area wrong.

### The 7 km delivery range

```json
"delivery":{ "radiusKm":7, ... }
```

A green bar sits under the categories saying you deliver within 7 km, with a
**Check my address** button. Pressing it asks the phone for its location and works out
the straight-line distance from your shop — no map service, no API key, nothing to pay.

- **Inside 7 km** — the bar turns green: "you are 2.4 km away, we deliver to you"
- **Outside** — it turns red and points them at Talabat or noon
- **They refuse to share location** — nothing is blocked; they type their area as normal

Whichever happens, the result is written into the WhatsApp message you receive
(`✓ Inside delivery range` or `⚠ Outside the 7 km range`) and stored with the order,
so you know before you reply.

To change the range, edit `radiusKm`. To change the wording, edit `note` and
`outsideNote` (and their `…Ar` versions).

> **It measures straight-line distance, not driving distance.** That is deliberately
> optimistic — better to say "looks like we deliver, let us confirm" than to turn away
> someone you could actually reach. Treat it as a filter, not a promise.

### Your review, TikTok and WhatsApp links

```json
"links":{ "googleReview":"", "tiktok":"", "whatsappCommunity":"" }
```

The footer shows your three QR codes from `assets/`. **Paste the real web addresses in
here and each one becomes a tappable link instead** — which matters, because nobody
scans a QR code off the phone they are already holding.

To find each address: open the QR sheet on your computer, scan a code with your phone,
and copy the link it opens.

---

## 9. Items, one by one

Open `admin.html`. Every product is a row; every shaded box can be typed in directly.

| To do this | Do this |
|---|---|
| Change a price | Click the Price box, type, click away |
| Change stock | Same, in Stock. **0 hides it from the shop automatically** |
| Rename a product | Type over the name — this is what customers read |
| Move category | Type the category name exactly as it appears elsewhere |
| Add a photo | Paste a link into Image link |
| Show or hide | The green switch in the first column |
| Find something | Type any part of the name, or the barcode, in search |
| Sort | Click a column heading; click again to reverse |

**Adding a new product** — there is no "add" button, deliberately: one at a time by hand is
slower than the spreadsheet route. **Export Excel**, add rows at the bottom, **Import Excel**.
A new row needs at minimum a Barcode, a Product name and a Price.

**Deleting** — hide it instead. It disappears from the shop but keeps its barcode, price and
photo for when you stock it again. To truly remove one, delete its row in Excel and re-import.

> **Nothing is saved until you download.** An **unsaved** badge appears the moment you change
> anything. Press **Download products.json** and replace the file. Close the tab without
> doing that and your edits are gone.

---

## 10. Many at once

Every bulk action applies to **whatever the filter currently shows** — not the whole catalog.
Set the filter first, check the count, then act.

Filters: search, category, shown/hidden, has-photo/no-photo, on-offer/not. The grey label
tells you how many match, e.g. *"67 of 732 match"*.

| Action | What it does | Example |
|---|---|---|
| Set price to | One price across the group | All 1.5L drinks to 6.00 |
| Change price by % | Up or down proportionally | `10` raises 10%, `-5` cuts 5% |
| Set stock to | One number everywhere | After a stock count |
| Add to stock | Adds or subtracts | `24` after a delivery |
| Move to category | Re-files the group | Splitting a category |
| Put on offer, % off | Sets the offer, remembers the old price | `20` for 20% off |
| Show all / Hide all | On or off together | Hiding a seasonal range |
| Hide the ones with no photo | Keeps the shelf looking full | Before sending the link out |
| Clear all offers | Ends a promotion | After the weekend |

> **The habit that avoids accidents:** read the match count out loud before pressing Apply.
> "67 of 732" — is 67 what you meant? A percentage applied to the whole catalog because the
> filter was empty is a bad afternoon.

---

## 11. Photos

Three ways, in order of how much work they are.

**A. From your Excel — the main way.** Press **Import Excel**. It reads `Image 1`, `Image 2`,
`Image 3` and `Photo link` columns — **including links hidden behind cells** that only display
"123" or "Photo 1". Matches on barcode. Nothing needs renaming.

**B. Paste a list.** Under **Photos → Or paste a list of links**, one per line:

```
6281007070775,https://i.ibb.co/xxxx/photo.jpg
9555701508702,https://i.ibb.co/yyyy/photo.jpg
```

Tab-separated works too, so you can copy two columns straight out of Excel.

**C. Upload from your computer.** Only for photos not online yet. Needs the ImgBB key. Files
named by barcode or by product name attach themselves; anything unmatched is listed.

> **Two kinds of ibb.co link.** `i.ibb.co/ABC/photo.jpg` is the **image** and works.
> `ibb.co/ABC` is the **web page** and will not display. If a photo does not appear, that is
> almost always why — open the page link, right-click the photo, *Copy image address*.

> **Borrowed photos are a slow leak.** Many links point at Amazon, Talabat and other shops.
> Those can change or be blocked any day and your shelf goes blank without warning — and
> using a competitor's photography on your own shop is shaky ground. Re-upload the good ones
> to your own ImgBB over time.

---

## 12. Offers and promotions

An offer is simply a product whose **Was** price is higher than its **Price**. The shop works
out the rest: old price struck through, a green **OFFER −20%** badge, and an **Offers** chip
at the top of the shop.

- **One product** — type the old price into **Was (offer)**, lower the **Price**
- **A whole group** — filter, choose **Put on offer, % off**, type `20`, Apply. It records the
  pre-offer price, so running it twice does not compound
- **Ending it** — filter to **On offer only**, press **Clear all offers**

The **Offers** sheet in `MASAARNA-TEMPLATES.xlsx` works out the saving and percentage as you
type, so you can see the margin before committing.

---

## 13. Orders

Orders arrive in two places at once: a WhatsApp message you reply to, and a row in your records.

Open `admin.html` → **Orders**. Paste the recorder address and admin key once, press
**Remember**. Then:

- **Load orders** — every order with customer, phone, area, items and total
- **Download orders for Excel** — one row per order *line*, so you can pivot by product as
  well as by order. Arabic reads correctly

Each order holds: reference (`MSA-260913-A91X`), date and time, customer, phone, area, note,
every line with barcode/product/qty/unit price/line total, and an order total **recalculated
on the server**, not taken from the browser.

### Testing it properly

1. Open the shop, add two or three things
2. Type a name, phone and area you will recognise
3. Press the green button — WhatsApp should open with the order written out
4. In the manager press **Load orders** — your test order should be at the top
5. Press **Download orders for Excel** and open it

If WhatsApp opens but no order appears, `orderApi` is missing or wrong in `products.json`.

> **Stock does not move by itself.** Selling something does not reduce its number — this shop
> does not count stock for you. Update it when you restock, or import from your till.

---

## 14. Excel in and out

1. **Export Excel** — every product, every column, Arabic readable
2. **Edit** — change anything, add rows at the bottom for new products.
   **Do not rename the column headings** — that is how the import finds things
3. **Import Excel** — matches on barcode: known ones updated, unknown ones added.
   It then tells you exactly what it did

| Column | Meaning |
|---|---|
| Barcode | How products are matched. Never change it |
| Product name | What customers read |
| Category | Must match an existing category exactly |
| Price | What they pay. Numbers only |
| Was price (offer) | Fill in to create an offer; empty for none |
| Stock | Whole numbers. 0 hides it |
| Weight, Origin | Shown under the name |
| Image link | A web address, or blank |
| Show on shop (1/0) | 1 shows, 0 hides |

> **Keep a copy before importing.** Import overwrites matching products. The old
> `products.json` is the only way back. Keep the last few — `products-2026-09-13.json`.

---

## 15. Publishing your changes

Nothing you do in the manager reaches the live shop until you upload the file. That is
deliberate — you can work on prices all week and publish once.

1. **Download products.json** in the manager
2. On GitHub: open the repository, click `products.json`, pencil icon, delete everything,
   paste the new contents, **Commit changes**.
   (Or simpler: **Add file → Upload files**, drag the new one in)
3. Wait a minute, then refresh the shop with `Ctrl+F5` to defeat your browser's cache

---

## 16. Test checklist

Work down this on your computer first, then again on the live link.

**The shop**

- [ ] Homepage loads with the logo
- [ ] Category chips show counts that are not zero
- [ ] Clicking a category lists products with AED prices
- [ ] Search finds something by name *and* by barcode
- [ ] Photos load (coloured initial tiles are correct for products with no photo)
- [ ] The Arabic button flips the whole page right to left
- [ ] It works in a phone-sized window (F12, then Ctrl+Shift+M)
- [ ] A product on offer shows the old price struck through
- [ ] The logo appears in the header and the footer
- [ ] **Check my address** reports a distance and turns green or red
- [ ] The footer map link opens your shop in Google Maps
- [ ] The three QR codes appear in the footer

**Ordering**

- [ ] Adding to the basket updates the bar at the bottom
- [ ] Changing quantity recalculates the total correctly
- [ ] The basket survives a page refresh
- [ ] The green button opens WhatsApp with the order written out
- [ ] It reaches *your* number, not an empty chat
- [ ] The order appears in the manager's Orders list
- [ ] Download orders for Excel opens with Arabic readable

**The manager**

- [ ] Changing a price shows **unsaved** at the top
- [ ] Download products.json saves a file
- [ ] Export Excel then Import Excel reports 0 errors
- [ ] Hiding a product removes it from the shop after publishing
- [ ] A bulk % change applied to a filtered group, and only that group

---

## 17. When it breaks

| What you see | What it means | Fix |
|---|---|---|
| "Could not load the product list" | The page cannot reach `products.json` | Both files must sit together. If local, use `npx serve` |
| WhatsApp opens an empty chat | The number is wrong | Digits only, no `+`, starting `971` |
| Shop shows nothing | Every product is hidden | Filter to Hidden in the manager, switch some on |
| Photos are grey boxes | Page links, not image links | Use `i.ibb.co/…/photo.jpg`, not `ibb.co/…` |
| Arabic shows as `Ø§Ù„Ø­Ù„` | Excel dropped the encoding | Open via Data → From Text/CSV, choose UTF-8 |
| Changes not showing live | Old file, or browser cache | Re-upload `products.json`, then `Ctrl+F5` |
| Orders say "admin key is wrong" | Key mismatch | It must match the Cloudflare secret exactly |
| Orders list is empty | No `orderApi`, or no orders yet | Check `products.json` has the recorder address |
| `D1_ERROR: no such table` | The database has no table | Re-run the `d1 execute … --remote` step |
| GitHub Pages shows 404 | Still building, or wrong branch | Wait a minute; confirm **main** and **/ (root)** |
| Import says "no Barcode column" | A heading was renamed | Export a fresh file and edit that |
| Logo and QR codes missing | The `assets/` folder was not uploaded | Upload the whole folder, keeping the name `assets` |
| "Check my address" does nothing | The browser blocked location | Only works over `https://` — fine once live, and on `localhost` |
| It says everyone is out of range | `lat`/`lng` are wrong | They should be `25.2619552` / `55.5870278` |

---

## 18. Safety rules

**Never share:** your admin key (it opens every customer's name, phone and address), your
ImgBB key, your GitHub token.

**Safe to share:** the shop link, `products.json` (it is public anyway), your WhatsApp number.

**Keep copies.** Before any import or bulk change, save the current `products.json` with the
date in the name. It is the only undo there is.

> **The one rule that matters most:** do not upload `admin.html` to a public address without
> Cloudflare Access in front of it. Anyone who finds it could change every price in your shop,
> and you would not know until a customer told you.

**Giving someone else access:** send them the shop link freely. For the manager, either hand
them the files to run on their own computer, or add their email in Cloudflare Access. Never
give out your admin key so someone can "have a look".
