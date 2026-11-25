# Build prompt: Spendelora (HTML, CSS and JavaScript)

Copy everything below the line and paste it into your AI coding agent in VS Code (GitHub Copilot Chat in Agent mode, Cline, Cursor, Claude Code, etc.). Work through it in the build order at the end. Don't paste it all and walk away: check each step in the browser before moving on.

---

You're helping me build **Spendelora**, a cozy, friendly expense tracker with the tagline **"Spend wisely. Live beautifully."** I already have a finished design I like, and I want you to rebuild it faithfully with **plain HTML, CSS and vanilla JavaScript**. Don't use React, Vue, Tailwind or any build step for the front end.

**Audience:** everyone: students, families, people of any gender or age. Keep all wording, emoji and decoration warm and inclusive. Never use gendered copy (no "girly", "for her", "babe", "queen", etc.).

## 1. Tech rules

- Files: `index.html`, `css/styles.css`, and JS modules in `js/` loaded with `<script type="module">`. Suggested modules: `state.js` (load/save), `logic.js` (pure money rules), `render.js` (UI), `charts.js` (SVG charts), `chat.js` (chat panel), `app.js` (wiring).
- No front-end dependencies except the Google Font **Nunito** (weights 400, 600, 700, 800, 900), loaded with a `<link>` in `<head>`.
- All data lives in the browser's **localStorage** under the key `spendelora:v1`, as one JSON object (shape below). Keep load/save in `state.js` so a real backend could replace it later.
- It must run by opening the site through a simple local server (e.g. the VS Code "Live Server" extension) with no install step. The **only** exception is the chat helper's tiny server (section 9).
- Keep every money rule as a **pure function** in `logic.js` (no DOM access), so it can be tested.
- Use semantic HTML (`<header>`, `<main>`, `<button>`, `<dialog>`, `<label>`), and make it fully keyboard-usable with visible focus rings.

## 2. Data shape

```js
// AppState saved in localStorage "spendelora:v1"
{
  version: 1,
  currency: "USD",            // one of the codes below
  budget: 800,                // monthly budget; 0 = not set
  categoryBudgets: { dining: 120, groceries: 250 }, // optional per-category limits
  categories: [ { id, name, emoji, color } ],       // color is "cat-1".."cat-9"
  expenses: [ {
    id, amount,               // positive number
    categoryId,
    date,                     // "YYYY-MM-DD"
    note,                     // string, may be empty
    recurrence,               // "none" | "weekly" | "monthly" | "yearly"
    recurringParentId,        // only on auto-generated copies of a recurring expense
    sample                    // true on demo data
  } ],
  onboarded: false,
  theme: "light"              // or "dark"
}
```

**Default categories** (id, name, emoji, color):
groceries Groceries 🛒 cat-1 · dining Dining 🍽️ cat-2 · transport Transport 🚌 cat-3 · home Home 🏠 cat-4 · health Health 💊 cat-9 · shopping Shopping 🛍️ cat-6 · fun Fun 🎉 cat-7 · bills Bills 📄 cat-8 · other Other ✨ cat-5

**Currencies** (code, symbol): USD $ · EUR € · GBP £ · INR ₹ · PKR Rs · JPY ¥ · CAD C$ · AUD A$ · MXN MX$
Money format: symbol + number with 2 decimals and thousands commas (`$1,234.50`). JPY shows no decimals.

**Sample data on first run** (all `sample: true`, dated relative to today):
Weekly market run 42.80 groceries (today) · Ramen with friends 14.50 dining (−1 day) · Bus to campus 2.75 transport (−1 day) · Music streaming 9.99 fun (−3 days) · Phone plan 65.00 bills (−5 days) · Groceries 38.20 (last month) · Pizza night 22.00 dining (last month) · New sneakers 120.00 shopping (2 months ago). Default budget 800, category limits dining 120 and groceries 250.

## 3. Look and feel

The feel is a **cozy stationery shop**, not a bank: warm rose, balanced with plum and cream.

Put these exact colors in `:root` as CSS variables (modern browsers support `oklch`):

```css
:root {
  --radius: 1.25rem;
  --background: oklch(0.975 0.013 25);   /* blush cream paper */
  --foreground: oklch(0.3 0.06 350);     /* plum text */
  --card: oklch(0.995 0.005 30);
  --primary: oklch(0.63 0.165 8);        /* warm rose */
  --primary-foreground: oklch(0.99 0.008 20);
  --primary-soft: oklch(0.93 0.04 10);
  --secondary: oklch(0.945 0.025 15);
  --muted: oklch(0.955 0.014 25);
  --muted-foreground: oklch(0.53 0.03 20);
  --accent: oklch(0.94 0.03 12);
  --accent-foreground: oklch(0.36 0.09 355);
  --berry: oklch(0.44 0.12 355);         /* logo word + headings */
  --destructive: oklch(0.58 0.2 25);
  --warning: oklch(0.76 0.14 65);
  --success: oklch(0.66 0.11 165);
  --border: oklch(0.915 0.025 18);
  --ring: oklch(0.63 0.165 8);
  --cat-1: oklch(0.8 0.1 150);  --cat-2: oklch(0.76 0.12 40);  --cat-3: oklch(0.75 0.1 235);
  --cat-4: oklch(0.78 0.09 75); --cat-5: oklch(0.74 0.12 15);  --cat-6: oklch(0.72 0.12 330);
  --cat-7: oklch(0.82 0.12 100);--cat-8: oklch(0.7 0.08 280);  --cat-9: oklch(0.8 0.06 190);
  --gradient-hero: linear-gradient(140deg, oklch(0.66 0.16 10) 0%, oklch(0.55 0.15 355) 60%, oklch(0.45 0.12 340) 100%);
  --gradient-paper: radial-gradient(1200px 600px at 100% -10%, oklch(0.94 0.04 15) 0%, transparent 60%),
                    radial-gradient(900px 500px at -10% 110%, oklch(0.95 0.03 60) 0%, transparent 60%);
  --shadow-soft: 0 1px 2px oklch(0.6 0.1 10 / 0.06), 0 8px 24px -8px oklch(0.6 0.12 10 / 0.18);
  --shadow-lift: 0 2px 4px oklch(0.6 0.1 10 / 0.08), 0 18px 40px -12px oklch(0.55 0.14 10 / 0.3);
  --shadow-fab: 0 10px 30px -6px oklch(0.6 0.17 8 / 0.55);
}
html.dark {
  --background: oklch(0.2 0.035 340);   --foreground: oklch(0.95 0.02 20);
  --card: oklch(0.255 0.045 342);       --primary: oklch(0.73 0.14 8);
  --primary-foreground: oklch(0.2 0.04 345); --primary-soft: oklch(0.33 0.07 350);
  --secondary: oklch(0.31 0.05 345);    --muted: oklch(0.29 0.04 343);
  --muted-foreground: oklch(0.74 0.03 15); --accent: oklch(0.33 0.06 348);
  --accent-foreground: oklch(0.94 0.03 15); --berry: oklch(0.82 0.08 10);
  --destructive: oklch(0.68 0.17 22);   --warning: oklch(0.8 0.13 70);
  --success: oklch(0.74 0.1 165);       --border: oklch(0.34 0.05 345); --ring: oklch(0.73 0.14 8);
  --cat-1: oklch(0.72 0.1 150); --cat-2: oklch(0.72 0.12 40); --cat-3: oklch(0.7 0.1 235);
  --cat-4: oklch(0.76 0.09 75); --cat-5: oklch(0.72 0.13 15); --cat-6: oklch(0.7 0.12 330);
  --cat-7: oklch(0.8 0.11 100); --cat-8: oklch(0.68 0.09 280); --cat-9: oklch(0.76 0.06 190);
  --gradient-hero: linear-gradient(140deg, oklch(0.5 0.14 8) 0%, oklch(0.4 0.12 350) 60%, oklch(0.32 0.09 335) 100%);
  --gradient-paper: radial-gradient(1200px 600px at 100% -10%, oklch(0.28 0.06 350) 0%, transparent 60%),
                    radial-gradient(900px 500px at -10% 110%, oklch(0.25 0.05 320) 0%, transparent 60%);
  --shadow-soft: 0 1px 2px oklch(0 0 0 / 0.2), 0 8px 24px -8px oklch(0.1 0.05 340 / 0.6);
}
```

- **Body:** `background: var(--background)` plus `background-image: var(--gradient-paper)`, fixed. Nunito everywhere.
- **Cards:** `background: var(--card)`, 1px `var(--border)` border, corner radius about 34px, `box-shadow: var(--shadow-soft)`. Nothing has sharp corners. Buttons are fully rounded pills.
- **Numbers:** always `font-variant-numeric: tabular-nums`, so amounts line up.
- **Category colors:** use `var(--cat-N)` for chart slices, chips and category dots, so dark mode recolors them automatically.
- **Logo:** a rounded-square rose tile (radius 10 out of 32) with a cream heart inside and a thin rose line down the middle of the heart. Put the word "Spendelora" next to it in extra-bold `var(--berry)`. Use the same SVG as the favicon.
- **Motion:** cards "rise in" (fade + 14px slide up, 0.55s, staggered), the add sheet and logo "pop in" (scale 0.92→1), progress bars grow from the left, chart bars grow from the bottom. Turn all animation off under `prefers-reduced-motion: reduce`.
- **Layout:** phone-first (designed at about 390px wide), max width about 1024px. On wide screens use two columns. Left: hero, budget, quick stats, insights, donut. Right: 6-month bars, category budgets, expense list.

## 4. Screen, top to bottom

1. **Header:** logo; currency dropdown (pill); a rose **"Ask"** pill button with a chat-heart icon that opens the chat (icon only on phones); a dark-mode toggle (moon/sun); an "Export CSV" pill (hidden on phones); a "⋯" menu with Export CSV, Import CSV, "Clear sample data" (only while sample data exists) and a red "Reset all data".
2. **Hero card** (`--gradient-hero`, cream text): "THIS MONTH" eyebrow, the month name ("November 2026"), a huge total, and the tagline under it. Round glassy ‹ › arrows browse months; the › arrow is disabled on the current month. Two soft translucent circles decorate the corners.
3. **Monthly budget card:** "$135.04 of $800.00", an animated progress bar and a message such as "You've used 17% of your budget · $664.96 left". The bar is rose under 85%, amber (`--warning`) from **85%**, and red (`--destructive`) from **100%**. An "Edit" button turns it into an inline number field (Enter saves, Escape cancels).
4. **Quick stats:** three small cards: 📅 daily average, the top category (emoji + name), and 🧾 the number of expenses. Daily average = total ÷ days so far this month (or ÷ days in the month for past months).
5. **Little insights:** a dashed-border card with one or two friendly lines (rules in section 7).
6. **"Where it went" donut:** an SVG donut by category for the selected month, with the total in the middle. The legend shows the top 5: color dot, emoji, name, amount and %.
7. **"Last 6 months" bars:** six rounded bars of monthly totals, ending at the selected month. The selected month's bar is rose and the others are soft. Clicking a bar jumps to that month.
8. **Category budgets:** a mini progress bar per category that has a limit (same 85%/100% colors). "Edit limits" opens a small form to set or clear each limit.
9. **Expense list:** a search box (matches note or category name). Expenses are grouped by day, newest first, with headers "Today", "Yesterday" or "Mon, Nov 16" and a total per day. Each row shows a colored emoji badge, the note (or the category name when empty), the category, a 🔁 badge for recurring items, and the amount. Tapping a row opens it for editing.
10. **Floating "+" button:** a 64px rose circle with `--shadow-fab`, fixed at the bottom right.
11. **Empty states:** never leave a blank card. Example: a big emoji + "No expenses yet — tap + to add your first one".

## 5. Add / edit expense sheet

A bottom sheet on phones and a centered dialog on desktop. It closes with Escape or by clicking the backdrop.
- A big amount field with the currency symbol, auto-focused, `inputmode="decimal"`.
- Category chips (emoji + name, tinted with the category color), plus a "+ New" chip. It opens an inline mini-form: name, emoji, and one of the 9 colors.
- Date (defaults to today; when browsing a past month, defaults to the 1st of that month), an optional note, and Repeats: Never / Weekly / Monthly / Yearly.
- Validation is inline and friendly: "Please enter an amount above 0". Never use `alert()`.
- In edit mode, show a red "Delete" with a confirm step. After deleting, show a toast "Expense deleted" with **Undo** for about 5 seconds.
- Toasts appear at the bottom center as rounded pills: "Expense added 💗", "Saved 💗", "CSV downloaded 📄".

## 6. Onboarding (first launch only)

A friendly dialog with 3 steps: (1) welcome + tagline, (2) choose currency, (3) set a monthly budget. Every step has **Skip**. Finishing or skipping sets `onboarded: true`.

## 7. Money rules (pure functions in `logic.js`)

- `budgetStatus(spent, budget)` → `{ pct, level }`. Level is `"ok"` under 85%, `"warning"` from 85%, `"danger"` from 100%. A budget of 0 is always ok.
- `generateRecurring(expenses, todayISO)` → **new** expenses only. For each template with recurrence ≠ none (and no `recurringParentId`), step forward from its date (weekly +7 days, monthly +1 month, yearly +1 year). Create one copy per step up to today, with `recurrence: "none"` and `recurringParentId = template.id`. Never duplicate an existing parent+date pair. Cap at 520 steps. Run it on load and after saving an expense.
- `toCsv(expenses, categories)` → header `date,category,note,amount`, sorted by date, amount with 2 decimals. Quote cells that contain `,`, `"` or a newline. Filename: `spendelora-YYYY-MM.csv` (current month).
- `fromCsv(text, categories)` → parses the same format, skips bad rows (bad date or amount ≤ 0), and maps unknown categories to "other".
- `loadState(raw)` → **never throws**. Missing, corrupt or foreign data starts fresh with defaults and samples. Wrap every localStorage call in try/catch; if storage is blocked, keep working in memory.
- `insights(state, month, today)` → at most 2 lines:
  - Compare categories with last month. On the first category with a change of 20% or more, say "Dining is up 32% from last month — worth a peek." or "You spent 25% less on Fun than last month. Nice!"
  - For the current month with a budget: ok → "You're under budget with 12 days to go. Keep it up 🌷"; warning → "Getting close to your budget — $80.00 left for 6 days."; danger → "You've passed this month's budget. No stress — next month is a fresh page ✨"
  - Fallback: "Groceries is your top category this month."
- Reset all data: confirm first, then clear everything but keep the theme, with an Undo toast.

## 8. Tests

Add `tests/logic.test.html`, a page that imports `logic.js` and prints ✅/❌ per test (or use `node --test` if Node is available). Cover:
1. 84.9 of 100 → ok; 85 → warning; 100 → danger.
2. A monthly template dated 2026-08-15, with today 2026-11-20 → creates 2026-09-15, 2026-10-15, 2026-11-15. Running it again creates nothing.
3. CSV export quotes a note containing a comma, and import reads it back.
4. `loadState("{not json")` returns a fresh valid state.

## 9. "Ask Spendelora" chat helper

A friendly AI helper that answers questions about the user's own spending.

**Panel UI:** a right-side drawer (full width on phones, about 448px on desktop) that opens from the header's "Ask" button.
- Header: the logo tile, "Ask Spendelora", "Questions about your spending, answered kindly", a "↺ New" button (clears the chat) and a close ✕.
- Empty state: a big logo, "Hi there! 👋", "I can see your expenses and budget. Ask me anything — or try one of these:", and 4 rounded suggestion buttons:
  "Where did most of my money go this month?" · "Am I on track with my budget?" · "Give me 3 easy ways to save this week" · "Which expenses repeat every month?"
- Messages: the user's messages are rose bubbles with cream text, aligned right. The helper's replies have **no bubble**: plain plum text on the page with simple markdown (bold, bullet lists).
- While waiting, show a shimmering "Thinking about your spending..." line. Show the reply word by word as it streams in.
- Composer: a rounded text box ("Ask about your spending…") with a round send button in its bottom-right corner. Enter sends and Shift+Enter makes a new line. While the reply is streaming, the send button becomes a Stop button. Keep the text box focused after sending.
- Small print under the box: "AI can make mistakes. Your chat is saved on this device only."
- **One conversation, saved in this browser**: store the messages array in localStorage under `spendelora:chat:v1`.
- Errors appear as a soft red message in the chat. Say "Too many questions right now, try again in a moment" for rate limits, and "Can't reach the helper. Check your internet." when offline.

**Spending snapshot:** with every message, the browser sends a plain-text summary built by a pure `buildChatContext(state, today)` function:
today's date, currency, monthly budget, spent this month (total + count), spent last month, amount left, this month by category (with limits), and the 60 most recent expenses (`date · amount · emoji category · note · repeats monthly`).

**Server (required: never put an AI key in browser JavaScript).** Create a tiny `server/` folder with one endpoint, `POST /api/chat`. Use one of these:
- Node + Express (`server/index.js`, `npm install express` + your AI provider's SDK), which also serves the static site, **or**
- a Cloudflare Worker / Netlify Function / Vercel Function if I deploy there.

The endpoint receives `{ messages: [{role, content}], context }`. Trim to the last 40 messages and cap the context at 12,000 characters. It calls the AI provider with the system prompt below and **streams** the answer back as plain text chunks (or Server-Sent Events). The browser reads the stream with `fetch` + `response.body.getReader()`. Read the key from an environment variable (e.g. `AI_API_KEY` in a `.env` file listed in `.gitignore`). Ask me which AI provider I want to use (e.g. Google Gemini, OpenAI or Anthropic) before writing this part.

System prompt to use:

```
You are the friendly helper inside Spendelora, a cozy expense tracker ("Spend wisely. Live beautifully.").
Spendelora is for everyone: students, families, people of any gender or age. Keep your tone warm, inclusive and never gendered.
Help the person understand their spending, stay within their budget and build gentle money habits.
- Base answers about their money ONLY on the spending snapshot below. If something isn't in it, say so kindly.
- Always use their currency when mentioning amounts.
- Keep replies short: a few sentences or a small bullet list. Use **bold** for key numbers.
- Be encouraging, never judgmental. No financial, tax or investment advice beyond everyday budgeting tips.
- You can't change their data; if they want to add or edit an expense, tell them to use the pink + button or tap the expense in the list.

## Spending snapshot
{context}
```

## 10. Quality bar

- Clean, commented, modular code; no inline `onclick` attributes, and one render function per section.
- Dialogs trap focus, close on Escape and return focus to the button that opened them.
- Tap targets at least 40px; every icon-only button has an `aria-label`.
- No stuck loading screens; local actions feel instant.
- A `README.md` with how to run the site (Live Server), how to run the chat server (`npm install`, set `AI_API_KEY`, `npm start`) and the feature list.

## Build order (stop and show me after each step)

1. Project skeleton, `styles.css` with all variables, header + logo + favicon, dark mode toggle.
2. `logic.js` + `state.js` + the tests page (all tests green).
3. Hero, budget card, quick stats, insights.
4. Donut + 6-month bar charts (pure SVG).
5. Expense list with search + the add/edit sheet + toasts with Undo.
6. Category budgets, recurring expenses, month navigation.
7. CSV export/import, clear samples, reset, onboarding.
8. Chat panel UI working against a fake reply first, then the real `server/` endpoint.
9. Final pass: phone at 390px, desktop, dark mode, keyboard only, reduced motion.
