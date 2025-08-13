<html lang="th">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>รายการสินค้า</title>
  <style>
    :root{
      --bg:#304674; --card:#fff; --ink:#0f172a; --accent:#15a362; --accent-ink:#fff;
      --radius:14px; --shadow: 0 2px 6px rgba(0,0,0,.06), 0 8px 24px rgba(0,0,0,.08);
    }
    body{margin:0; font-family:sans-serif; background:var(--bg); color:#fff;}
    .wrap{max-width:1100px; margin:auto; padding:20px 14px 80px;}
    header{margin-bottom:18px; font-weight:700; font-size:20px;}
    .grid{display:grid; gap:12px; grid-template-columns:repeat(3, 1fr);}
    @media(min-width:640px){.grid{grid-template-columns:repeat(4,1fr); gap:16px;}}
    @media(min-width:960px){.grid{grid-template-columns:repeat(5,1fr); gap:18px;}}
    .card{background:var(--card); color:var(--ink); border-radius:var(--radius);
      box-shadow:var(--shadow); overflow:hidden; display:flex; flex-direction:column;}
    .thumb{aspect-ratio:1/1; width:100%; object-fit:cover; background:#cbd5e1;}
    .body{padding:10px; display:flex; flex-direction:column; gap:8px;}
    .name{font-weight:700; font-size:14px; line-height:1.25; min-height:2.5em;}
    .price{font-weight:700; color:var(--accent);}
    .btn{margin-top:auto; display:inline-flex; justify-content:center; padding:10px;
      border-radius:10px; background:var(--accent); color:var(--accent-ink);
      text-decoration:none; font-weight:700; font-size:14px;}
    .btn:hover{filter:brightness(1.05);}
    .status{margin:16px 0; font-size:14px; color:#dbeafe;}
  </style>
</head>
<body>
  <div class="wrap">
    <header>รายการสินค้า</header>
    <div id="status" class="status">กำลังดึงข้อมูล…</div>
    <div id="grid" class="grid"></div>
  </div>

  <script>
  // ใส่ลิงก์ CSV ที่ได้จาก Publish to the web
  const CSV_URL = "https://docs.google.com/spreadsheets/d/e/2PACX-1vSRieAboeby5A7nHJhvVyG532EudqvvfvvPal-u2zO0rfAkRDw_03O06ZNdU0aptATWV83D5zSH9Vn2/pub?gid=0&single=true&output=csv";
  const WHATSAPP_PHONE = "8562099872754";

  async function load(){
    try{
      const res = await fetch(CSV_URL);
      const text = await res.text();
      const rows = text.trim().split("\n").map(r => r.split(","));
      const headers = rows[0].map(h => h.trim());
      const data = rows.slice(1).map(r => {
        const obj = {};
        headers.forEach((h,i) => obj[h] = r[i] ? r[i].trim() : "");
        return obj;
      });
      render(data);
      document.getElementById('status').textContent = พบ ${data.length} รายการ;
    }catch(err){
      console.error(err);
      document.getElementById('status').textContent = "โหลดข้อมูลไม่สำเร็จ";
    }
  }

  function render(items){
    const grid = document.getElementById('grid');
    grid.innerHTML = "";
    items.forEach(it => {
      const card = document.createElement('div');
      card.className = 'card';
      card.innerHTML = `
        <img class="thumb" src="${it.Picture}" alt="${it.Name}" />
        <div class="body">
          <div class="name">${it.Name}</div>
          <div class="price">${it.Price}</div>
          <a class="btn" target="_blank"
             href="https://wa.me/${WHATSAPP_PHONE}?text=${encodeURIComponent('สนใจ: ' + it.Name + ' (' + it.Price + ')')}">
            สั่งซื้อทาง WhatsApp
          </a>
        </div>
      `;
      grid.appendChild(card);
    });
  }

  load();
  </script>
</body>
</html>
