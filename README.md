<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Invoice #100</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Merriweather:wght@400;700&family=Inter:wght@400;600&display=swap" rel="stylesheet">
<style>
  :root{
    --ink:#0b2545;      /* deep navy */
    --ink-2:#1d3d74;    /* table header */
    --muted:#6b7280;
    --border:#d1d5db;
    --banner-start:#0b5bd3;/* gradient banner substitute for image */
    --banner-end:#19324f;
    --accent:#0b5bd3;
  }
  *{box-sizing:border-box}
  html,body{margin:0}
  body{font-family:Inter,system-ui,-apple-system,Segoe UI,Roboto,Helvetica,Arial,sans-serif;color:var(--ink);background:white}
  .page{max-width:900px;margin:20px auto;padding:24px;border:1px solid #e5e7eb}

  /* Banner (you can swap with your own banner image) */
  .banner{height:110px;border:1px solid var(--border);background:linear-gradient(135deg,var(--banner-start),var(--banner-end));border-radius:2px;display:grid;align-content:center;padding:18px;color:#fff}
  .brand{font-family:Merriweather,serif;font-size:32px;font-weight:700}
  .brand small{display:block;font-family:Inter,system-ui;font-weight:400;font-size:12px;opacity:.9;margin-top:6px}

  .split{display:grid;grid-template-columns:1fr 1fr;gap:28px;margin-top:26px}
  h2{font-family:Merriweather,serif;font-weight:700;margin:0 0 6px 0}
  label{display:block;font-size:13px;color:var(--muted);margin-bottom:6px}
  input,textarea{width:100%;padding:10px 12px;border:1px solid var(--border);border-radius:4px;font-size:14px}
  textarea{min-height:52px;resize:vertical}

  .table{margin-top:26px;border:1px solid var(--ink-2)}
  .thead{display:grid;grid-template-columns:1.5fr .6fr;background:var(--ink-2);color:#fff;font-weight:700;padding:10px 12px;text-align:center}
  .row{display:grid;grid-template-columns:1.5fr .6fr}
  .row > div{border-top:1px solid var(--border)}
  .row input{border:0;border-left:1px solid var(--border)}
  .row input:first-child{border-left:0}
  .row > div:first-child{border-right:1px solid var(--border)}
  .row > div input{width:100%;padding:12px}

  .tools{margin:12px 0;display:flex;gap:10px}
  button{border:1px solid var(--border);background:#fff;padding:10px 14px;border-radius:6px;cursor:pointer}
  .primary{background:var(--accent);border-color:var(--accent);color:#fff;font-weight:600}

  .totals{display:grid;grid-template-columns:1fr 280px;gap:24px;margin-top:26px}
  .box{border:1px solid var(--border)}
  .box .line{display:grid;grid-template-columns:1fr 1fr}
  .box .line > div{padding:10px;border-top:1px solid var(--border)}
  .box .line:first-child > div{border-top:0}
  .right{text-align:right;font-variant-numeric:tabular-nums}
  .totalBar{background:var(--ink-2);color:#fff;font-weight:800;padding:12px 14px;display:flex;justify-content:space-between}
  .small{font-size:12px;color:var(--muted)}

  @media print{
    .tools{display:none}
    body{background:white}
    .page{border:0;margin:0}
  }
</style>
</head>
<body>
  <div class="page" id="page">
    <!-- Banner/Header -->
    <div class="banner">
      <div class="brand">Company Name
        <small>Address, City, ST, ZIP Code • Phone Number | Fax Number</small>
      </div>
    </div>

    <!-- Invoice meta -->
    <div class="split">
      <div>
        <h2>Invoice # <span id="invNo">100</span></h2>
        <div class="small">Date: <input id="invDate" type="date" style="display:inline-block;width:auto"></div>
      </div>
    </div>

    <!-- Bill To / For -->
    <div class="split">
      <div>
        <h2>Bill To</h2>
        <label>Name | Company</label>
        <input id="billName" type="text" placeholder="Name | Company">
        <label style="margin-top:10px">Address, City, ST, ZIP Code</label>
        <input id="billAddr" type="text" placeholder="Address, City, ST, ZIP Code">
        <label style="margin-top:10px">Phone</label>
        <input id="billPhone" type="text" placeholder="Phone">
      </div>
      <div>
        <h2>For</h2>
        <label>Product Description</label>
        <textarea id="forText" placeholder="Product Description"></textarea>
      </div>
    </div>

    <!-- Items -->
    <div class="table">
      <div class="thead"><div>Item Description</div><div>Amount</div></div>
      <div id="rows"></div>
    </div>

    <div class="tools">
      <button id="add" class="primary">Add Row</button>
      <button id="clear">Clear Rows</button>
      <button id="print">Print / Save PDF</button>
    </div>

    <!-- Totals area -->
    <div class="totals">
      <div></div>
      <div class="box">
        <div class="line"><div>Subtotal</div><div class="right" id="subtotal">$0.00</div></div>
        <div class="line"><div>Tax Rate</div><div><input id="taxRate" type="number" min="0" step="0.01" placeholder="%" style="width:100%;border:0;padding:0 8px;text-align:right"></div></div>
        <div class="line"><div>Other Costs</div><div><input id="other" type="number" min="0" step="0.01" value="0" style="width:100%;border:0;padding:0 8px;text-align:right"></div></div>
        <div class="totalBar"><span>Total Cost</span><span id="total">$0.00</span></div>
      </div>
    </div>
  </div>

<script>
(function(){
  const fmt = new Intl.NumberFormat(undefined,{style:'currency',currency:'USD'});
  const rows = document.getElementById('rows');
  const subtotalEl = document.getElementById('subtotal');
  const totalEl = document.getElementById('total');
  const taxRateEl = document.getElementById('taxRate');
  const otherEl = document.getElementById('other');

  function makeRow(desc='', amount=0){
    const r = document.createElement('div');
    r.className = 'row';
    const c1 = document.createElement('div');
    const c2 = document.createElement('div');
    const d = document.createElement('input'); d.type='text'; d.placeholder='Description'; d.value=desc;
    const a = document.createElement('input'); a.type='number'; a.step='0.01'; a.min='0'; a.value=amount;
    c1.appendChild(d); c2.appendChild(a); r.append(c1,c2);

    // remove on dblclick amount cell
    r.title='Double‑click a row to remove';
    r.addEventListener('dblclick', ()=>{ r.remove(); calc(); });

    [d,a].forEach(el=>el.addEventListener('input', calc));
    rows.appendChild(r);
    calc();
  }

  function calc(){
    let subtotal = 0;
    rows.querySelectorAll('.row input[type="number"]').forEach(i=>{
      const v = parseFloat(i.value); if(!isNaN(v)) subtotal += v;
    });
    const taxRate = parseFloat(taxRateEl.value||'0')/100;
    const other = parseFloat(otherEl.value||'0');
    const total = subtotal + subtotal*taxRate + other;
    subtotalEl.textContent = fmt.format(subtotal);
    totalEl.textContent = fmt.format(total);
  }

  // Controls
  document.getElementById('add').onclick = ()=> makeRow('',0);
  document.getElementById('clear').onclick = ()=>{ rows.innerHTML=''; calc(); };
  document.getElementById('print').onclick = ()=> window.print();
  [taxRateEl, otherEl].forEach(el=> el.addEventListener('input', calc));

  // initialize with 3 blank rows like the screenshot
  makeRow(); makeRow(); makeRow();
  // default date
  document.getElementById('invDate').valueAsDate = new Date();
})();
</script>
</body>
</html>
# calculadora
