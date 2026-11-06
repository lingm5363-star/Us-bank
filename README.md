<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Ledger Dashboard</title>
<style>
  :root {
    --bg: #eef2f4; --surface: #ffffff; --ink: #14272e; --muted: #5d727a;
    --line: #d5dee2; --accent: #0b6e72; --accent-ink: #ffffff;
    --in: #16794a; --out: #b3392f;
  }
  @media (prefers-color-scheme: dark) {
    :root {
      --bg: #0f1b20; --surface: #17272e; --ink: #e6eef0; --muted: #93a8af;
      --line: #2a3f47; --accent: #3fb5b9; --accent-ink: #072427;
      --in: #5fd18f; --out: #ff8a7e;
    }
  }
  * { box-sizing: border-box; }
  body { margin: 0; background: var(--bg); color: var(--ink);
    font: 16px/1.5 Georgia, "Times New Roman", serif; }
  h1, h2, label, button, th, input, select, .num { font-family: "Trebuchet MS", "Segoe UI", sans-serif; }
  [hidden] { display: none !important; }
  .wrap { max-width: 760px; margin: 0 auto; padding: 24px 16px; }

  /* Login */
  .login { min-height: 100vh; display: grid; place-items: center; padding: 16px; }
  .login form { width: 100%; max-width: 360px; background: var(--surface);
    border: 1px solid var(--line); border-radius: 10px; padding: 28px 24px; }
  h1 { font-size: 1.5rem; margin: 0 0 4px; }
  .sub { color: var(--muted); margin: 0 0 20px; }
  label { display: block; font-size: .9rem; font-weight: 600; margin: 14px 0 4px; }
  input, select { width: 100%; padding: 10px 12px; font-size: 1rem; color: var(--ink);
    background: var(--bg); border: 1px solid var(--line); border-radius: 6px; }
  :focus-visible { outline: 3px solid var(--accent); outline-offset: 2px; }
  button { cursor: pointer; font-size: 1rem; font-weight: 600; padding: 10px 16px;
    border-radius: 6px; border: 1px solid var(--accent); background: var(--accent); color: var(--accent-ink); }
  button.ghost { background: transparent; color: var(--accent); }
  .login button { width: 100%; margin-top: 20px; }
  .error { color: var(--out); font-family: "Trebuchet MS", "Segoe UI", sans-serif; font-size: .9rem; min-height: 1.3em; margin: 12px 0 0; }
  .hint { color: var(--muted); font-size: .85rem; margin: 16px 0 0; }

  /* Dashboard */
  header { display: flex; justify-content: space-between; align-items: center; gap: 12px; flex-wrap: wrap; margin-bottom: 20px; }
  header h1 { margin: 0; }
  .balance { background: var(--accent); color: var(--accent-ink); border-radius: 10px; padding: 22px 24px; margin-bottom: 20px; }
  .balance .num { font-size: 2.4rem; font-weight: 700; display: block; }
  .stats { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; margin-bottom: 20px; }
  .stat { background: var(--surface); border: 1px solid var(--line); border-radius: 10px; padding: 14px 16px; }
  .stat .num { font-size: 1.3rem; font-weight: 700; display: block; }
  .stat small { color: var(--muted); }
  section { background: var(--surface); border: 1px solid var(--line); border-radius: 10px; padding: 18px; margin-bottom: 20px; }
  h2 { font-size: 1.1rem; margin: 0 0 12px; }
  .scroll { overflow-x: auto; }
  table { width: 100%; border-collapse: collapse; min-width: 440px; }
  th, td { text-align: left; padding: 10px 8px; border-bottom: 1px solid var(--line); }
  th { font-size: .85rem; color: var(--muted); font-weight: 600; }
  td.amt { text-align: right; font-family: "Trebuchet MS", "Segoe UI", sans-serif; font-weight: 600; white-space: nowrap; }
  th:last-child { text-align: right; }
  .in { color: var(--in); } .out { color: var(--out); }
  .row { display: grid; grid-template-columns: 2fr 1fr 1fr auto; gap: 10px; align-items: end; }
  .row label { margin-top: 0; }
  @media (max-width: 560px) { .row { grid-template-columns: 1fr 1fr; } .row button { grid-column: span 2; } }
  .empty { color: var(--muted); padding: 12px 0; }
</style>
</head>
<body>

<!-- LOGIN -->
<main class="login" id="loginView">
  <form id="loginForm" autocomplete="on">
    <h1>Sign in</h1>
    <p class="sub">Access your dashboard.</p>
    <label for="user">Username</label>
    <input id="user" name="username" autocomplete="username" required>
    <label for="pass">Password</label>
    <input id="pass" name="password" type="password" autocomplete="current-password" required>
    <p class="error" id="err" role="alert"></p>
    <button type="submit">Sign in</button>
    <p class="hint">Demo login: <b>admin</b> / <b>password123</b></p>
  </form>
</main>

<!-- DASHBOARD -->
<div class="wrap" id="dashView" hidden>
  <header>
    <h1>Welcome, <span id="who"></span></h1>
    <button class="ghost" id="logout" type="button">Sign out</button>
  </header>

  <div class="balance"><small>Current balance</small><span class="num" id="balance"></span></div>

  <div class="stats">
    <div class="stat"><small>Money in</small><span class="num in" id="totalIn"></span></div>
    <div class="stat"><small>Money out</small><span class="num out" id="totalOut"></span></div>
  </div>

  <section>
    <h2>Recent transactions</h2>
    <div class="scroll">
      <table>
        <thead><tr><th>Date</th><th>Description</th><th>Category</th><th>Amount</th></tr></thead>
        <tbody id="tx"></tbody>
      </table>
    </div>
    <p class="empty" id="empty" hidden>No transactions yet. Add one below.</p>
  </section>

  <section>
    <h2>Add a transaction</h2>
    <form id="txForm" class="row">
      <div><label for="desc">Description</label><input id="desc" required maxlength="60"></div>
      <div><label for="amount">Amount</label><input id="amount" type="number" step="0.01" min="0.01" required></div>
      <div><label for="type">Type</label>
        <select id="type"><option value="in">Money in</option><option value="out">Money out</option></select></div>
      <button type="submit">Add</button>
    </form>
  </section>
</div>

<script>
/*
  DEMO ONLY: this login runs in the browser, so anyone can read the password
  in the page source. For a real app, verify logins on a server.
*/
const USERS = { admin: "password123" };   // change these

const money = n => (n < 0 ? "-" : "") + "$" + Math.abs(n).toLocaleString("en-US", { minimumFractionDigits: 2, maximumFractionDigits: 2 });
const $ = id => document.getElementById(id);

const seed = [
  { date: "2026-11-05", desc: "Customer payment", cat: "Sales", amt: 420.00 },
  { date: "2026-11-04", desc: "Packaging supplies", cat: "Supplies", amt: -86.50 },
  { date: "2026-11-03", desc: "Online order #1042", cat: "Sales", amt: 135.25 },
  { date: "2026-11-02", desc: "Electricity bill", cat: "Utilities", amt: -112.40 },
  { date: "2026-11-01", desc: "Refund issued", cat: "Refunds", amt: -24.99 }
];

function load(key, fallback) {
  try { const v = localStorage.getItem(key); return v ? JSON.parse(v) : fallback; }
  catch (e) { return fallback; }
}
function save(key, val) {
  try { localStorage.setItem(key, JSON.stringify(val)); } catch (e) {}
}

let transactions = load("ledger_tx", seed);

function render() {
  const body = $("tx");
  body.textContent = "";
  transactions.slice(0, 10).forEach(t => {
    const tr = document.createElement("tr");
    [t.date, t.desc, t.cat].forEach(v => {
      const td = document.createElement("td"); td.textContent = v; tr.appendChild(td);
    });
    const a = document.createElement("td");
    a.className = "amt " + (t.amt < 0 ? "out" : "in");
    a.textContent = (t.amt > 0 ? "+" : "") + money(t.amt);
    tr.appendChild(a);
    body.appendChild(tr);
  });
  $("empty").hidden = transactions.length > 0;
  const tin = transactions.filter(t => t.amt > 0).reduce((s, t) => s + t.amt, 0);
  const tout = transactions.filter(t => t.amt < 0).reduce((s, t) => s + t.amt, 0);
  $("totalIn").textContent = money(tin);
  $("totalOut").textContent = money(tout);
  $("balance").textContent = money(tin + tout);
}

function showDash(name) {
  $("who").textContent = name;
  $("loginView").hidden = true;
  $("dashView").hidden = false;
  render();
}

$("loginForm").addEventListener("submit", e => {
  e.preventDefault();
  const u = $("user").value.trim(), p = $("pass").value;
  if (Object.prototype.hasOwnProperty.call(USERS, u) && USERS[u] === p) {
    try { sessionStorage.setItem("ledger_user", u); } catch (err) {}
    $("err").textContent = "";
    $("pass").value = "";
    showDash(u);
  } else {
    $("err").textContent = "Wrong username or password. Check both and try again.";
  }
});

$("txForm").addEventListener("submit", e => {
  e.preventDefault();
  const amount = parseFloat($("amount").value);
  if (!(amount > 0)) return;
  const isIn = $("type").value === "in";
  transactions.unshift({
    date: new Date().toISOString().slice(0, 10),
    desc: $("desc").value.trim(),
    cat: isIn ? "Income" : "Expense",
    amt: isIn ? amount : -amount
  });
  save("ledger_tx", transactions);
  e.target.reset();
  render();
});

$("logout").addEventListener("click", () => {
  try { sessionStorage.removeItem("ledger_user"); } catch (err) {}
  $("dashView").hidden = true;
  $("loginView").hidden = false;
  $("user").focus();
});

(function init() {
  let u = null;
  try { u = sessionStorage.getItem("ledger_user"); } catch (e) {}
  if (u && USERS[u] !== undefined) showDash(u);
})();
</script>
</body>
</html>
