<!doctype html>
<html lang="th">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>สินค้า</title>
  <meta name="color-scheme" content="light only" />
  <style>
    :root{
      --bg:#304674;
      --card:#ffffff;
      --ink:#0f172a;
      --muted:#475569;
      --accent:#15a362;
      --accent-ink:#ffffff;
      --chip:#e2e8f0;
      --radius:14px;
      --shadow: 0 2px 6px rgba(0,0,0,.06), 0 8px 24px rgba(0,0,0,.08);
    }
    *{box-sizing:border-box}
    body{
      margin:0; font-family: ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, "Noto Sans Thai", "Noto Sans", Arial, sans-serif;
      background:var(--bg); color:#fff;
    }
    .wrap{
      max-width:1100px; margin:auto; padding:20px 14px 80px;
    }
    header{
      display:flex; align-items:center; justify-content:space-between; gap:12px;
      margin:8px 0 18px;
    }
    .brand{
      font-weight:700; letter-spacing:.2px; font-size:20px;
    }
    .grid{
      display:grid; gap:12px;
      /* ตามโจทย์: มือถือ 3 ชิ้น/แถว */
      grid-template-columns: repeat(3, 1fr);
    }
    /* จอใหญ่ขึ้น: ให้การ์ดหายอึดอัดโดยคงสัดส่วนสวยงาม */
    @media (min-width:640px){
      .grid{ gap:16px; grid-template-columns: repeat(4, 1fr); }
    }
    @media (min-width:960px){
      .grid{ gap:18px; grid-template-columns: repeat(5, 1fr); }
    }
    .card{
      background:var(--card); color:var(--ink);
      border-radius:var(--radius); box-shadow:var(--shadow); overflow:hidden;
      display:flex; flex-direction:column; min-height:100%;
    }
    .thumb{
      aspect-ratio: 1 / 1; width:100%; object-fit:cover; background:#cbd5e1;
    }
    .body{ padding:10px 10px 12px; display:flex; flex-direction:column; gap:8px; }
    .name{ font-weight:700; font-size:14px; line-height:1.25; min-height:2.5em; }
    .price{ font-weight:700; color:var(--accent); }
    .meta{ display:flex; gap:8px; align-items:center; color:var(--muted); font-size:12px; }
    .meta a{ color:inherit; text-decoration:none; border-bottom:1px dashed currentColor;}
    .btn{
      margin-top:auto;
      display:inline-flex; align-items:center; justify-content:center; gap:8px;
      padding:10px 12px; border-radius:10px; background:var(--accent); color:var(--accent-ink);
      text-decoration:none; font-weight:700; font-size:14px;
    }
    .btn:hover{ filter:brightness(1.05); }
    .status{
      margin:16px 0; font-size:14px; color:#dbeafe;
    }
    .skeleton{
      background:linear-gradient(90deg, #e2e8f0 25%, #f1f5f9 37%, #e2e8f0 63%);
      background-size:400% 100%; animation:shimmer 1.2s ease-in-out infinite;
    }
    @keyframes shimmer{0%{background-position:100% 0}100%{background-position:-100% 0}}
    .card.skel .thumb{ background:#e2e8f0 }
    .card.skel .name, .card.skel .price{ height:14px; border-radius:6px }
    .card.skel .name{ width:80%; }
    .card.skel .price{ width:50%; }
    footer{
      position:fixed; inset:auto 0 10px 0; display:flex; justify-content:center; pointer-events:none;
    }
    .chip{
      pointer-events:auto;
      background:rgba(255,255,255,.12); color:#e2e8f0;
      backdrop-filter: blur(6px);
      padding:8px 12px; border-radius:999px; font-size:12px;
    }
  </style>
</head>
<body>
  <div class="wrap">
    <header>
      <div class="brand">รายการสินค้า</div>
    </header>

    <div id="status" class="status">กำลังดึงข้อมูล…</div>
    <div id="grid" class="grid" aria-live="polite"></div>
  </div>

  <footer>
    <div class="chip">พื้นหลัง #304674 · 3 ชิ้น/แถวบนมือถือ</div>
  </footer>

  <script>
    // === ตั้งค่า ===
    const SHEET_ID = "1W7Ak78fUsG9RjHYMBWmj0x9GHKFWXpx8wLY5f04MMa0";
    const SHEET_NAME = "Sheet1"; // ต้องตรงกับชื่อแผ่นงาน
    const WHATSAPP_PHONE = "8562099872754"; // wa.me ต้องไม่มีเครื่องหมาย +

    // ใช้ gviz endpoint (ไม่ต้อง API key) แต่ต้อง Publish to the web
    const url = https://docs.google.com/spreadsheets/d/${SHEET_ID}/gviz/tq?sheet=${encodeURIComponent(SHEET_NAME)};

    const grid = document.getElementById('grid');
    const status = document.getElementById('status');

    // ใส่ skeleton ระหว่างโหลด
    function addSkeleton(count=6){
      grid.innerHTML = "";
      for(let i=0;i<count;i++){
        const c = document.createElement('div');
        c.className = 'card skel';
        c.innerHTML = `
          <div class="thumb skeleton"></div>
          <div class="body">
            <div class="name skeleton"></div>
            <div class="price skeleton"></div>
            <a class="btn" style="opacity:.5; pointer-events:none;">สั่งซื้อทาง WhatsApp</a>
          </div>`;
        grid.appendChild(c);
      }
    }

    function toRowsFromGviz(text){
      // gviz ตอบกลับเป็น JS function wrapper ต้องตัดออกก่อน
      const json = JSON.parse(text.substring(text.indexOf('{'), text.lastIndexOf('}') + 1));
      const table = json.table;
      const cols = table.cols.map(c => (c.label || c.id || '').trim());
      const rows = table.rows.map(r => r.c.map(cell => cell ? cell.v : ''));
      return { cols, rows };
    }

    function render(items){
      grid.innerHTML = "";
      if(!items.length){
        status.textContent = "ไม่พบสินค้า";
        return;
      }
      status.textContent = พบทั้งหมด ${items.length} รายการ;
      for(const it of items){
        const name = (it.Name ?? "").toString().trim();
        const price = (it.Price ?? "").toString().trim();
        const picture = (it.Picture ?? "").toString().trim();
        const link = (it.Link ?? "").toString().trim();

        const card = document.createElement('div');
        card.className = 'card';

        const img = document.createElement('img');
        img.className = 'thumb';
        img.alt = name || "รูปสินค้า";
        img.loading = "lazy";
        img.src = picture || "https://via.placeholder.com/600x600.png?text=No+Image";

        const body = document.createElement('div');
        body.className = 'body';

        const title = document.createElement('div');
        title.className = 'name';
        title.textContent = name || "ไม่ระบุชื่อสินค้า";

        const priceEl = document.createElement('div');
        priceEl.className = 'price';
        priceEl.textContent = price || "";

        const meta = document.createElement('div');
        meta.className = 'meta';
        if(link){
          const a = document.createElement('a');
          a.href = link; a.target="_blank"; a.rel="noopener";
          a.textContent = "ดูรายละเอียด";
          meta.appendChild(a);
        }

        const msg = encodeURIComponent(
          สวัสดีครับ/ค่ะ สนใจสั่งซื้อสินค้า: ${name}${price ? " ("+price+")" : ""} +
          ${link ? `\nลิงก์: ${link} : ""}`
        );
        const wa = document.createElement('a');
        wa.className = 'btn';
        wa.href = https://wa.me/${WHATSAPP_PHONE}?text=${msg};
        wa.target = "_blank"; wa.rel = "noopener";
        wa.innerHTML = `
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" aria-hidden="true">
            <path d="M20.5 3.5A11 11 0 0 0 3.4 20.6L2 22l1.7-.5A11 11 0 1 0 20.5 3.5Z" stroke="white" stroke-width="1.5"/>
            <path d="M8.5 9.5c.2 3 2.9 5.8 6 6l1.2-1.3c.2-.2.5-.2.8-.1l2 .8c.3.1.5.5.4.8-.8 2-3 3.3-5.2 3.3A7 7 0 1 1 15.8 7c.1.3 0 .6-.2.8L14.3 9c-.2.2-.5.2-.8.1l-2-1c-.3-.1-.6 0-.8.3l-1 1.1Z" fill="white"/>
          </svg>
          สั่งซื้อทาง WhatsApp
        `;

        body.appendChild(title);
        if(price) body.appendChild(priceEl);
        if(link) body.appendChild(meta);
        body.appendChild(wa);

        card.appendChild(img);
        card.appendChild(body);
        grid.appendChild(card);
      }
    }

    async function load(){
      addSkeleton(9);
      try{
        const res = await fetch(url);
        const text = await res.text();
        const { cols, rows } = toRowsFromGviz(text);

        // ทำแผนที่คอลัมน์แบบยืดหยุ่น (อาศัยชื่อหัวคอลัมน์)
        const idx = {
          Name: cols.findIndex(c => /^name$/i.test(c)),
          Price: cols.findIndex(c => /^price$/i.test(c)),
          Picture: cols.findIndex(c => /^picture$/i.test(c)),
          Link: cols.findIndex(c => /^link$/i.test(c)),
        };

        const items = rows.map(r => ({
          Name: r[idx.Name] ?? "",
          Price: r[idx.Price] ?? "",
          Picture: r[idx.Picture] ?? "",
          Link: r[idx.Link] ?? "",
        })).filter(x => (x.Name || x.Price || x.Picture || x.Link));

        render(items);
      }catch(err){
        console.error(err);
        status.textContent = "ดึงข้อมูลไม่สำเร็จ กรุณาตรวจสอบการตั้งค่า Sheet (ต้อง Publish เป็นสาธารณะ)";
        grid.innerHTML = "";
      }
    }

    load();
  </script>
</body>
</html>
