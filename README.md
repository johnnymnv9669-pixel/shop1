<!DOCTYPE html>
<html lang="th">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>shop1</title>
<style>
body { font-family: sans-serif; background:#304674; color:white; margin:0; padding:20px;}
h1 { color:#4da3ff; text-align:center; }
.product-grid { display:grid; grid-template-columns:repeat(3,1fr); gap:15px; margin-top:20px;}
.product { background:rgba(255,255,255,0.1); padding:10px; border-radius:8px; text-align:center; transition:0.2s;}
.product:hover { transform:scale(1.03); }
.product img { max-width:100%; border-radius:5px; }
.buy-btn { display:inline-block; background:#4da3ff; color:white; padding:5px 10px; border-radius:5px; margin-top:5px; text-decoration:none;}
@media(max-width:900px){.product-grid{grid-template-columns:repeat(2,1fr);}}
@media(max-width:600px){.product-grid{grid-template-columns:repeat(1,1fr);}}
</style>
</head>
<body>
<h1>shop1</h1>
<hr>
<h2 style="text-align:center;">รายการสินค้า</h2>
<div id="product-list" style="text-align:center;">กำลังดึงข้อมูล...</div>

<script>
const WEBAPP_URL = "https://script.google.com/macros/s/AKfycbwj6LodpxRbRsb70mgxgZguge2GITWBDq5Z5_0qG28cFWEfuNVyWPLmGZ69ClGSy9Uwnw/exec"; // ใส่ URL Web App ของคุณ

function driveDirectLink(url){
  const match = url.match(/\/d\/(.*?)\//);
  return match ? https://drive.google.com/uc?export=view&id=${match[1]} : url;
}

fetch(WEBAPP_URL)
  .then(res => res.json())
  .then(items => {
    const container = document.getElementById("product-list");
    container.innerHTML = "";
    const grid = document.createElement("div");
    grid.className = "product-grid";
    
    items.forEach(item => {
      const div = document.createElement("div");
      div.className = "product";
      const name = item.Name || "ไม่มีชื่อ";
      const price = item.Price || "-";
      const pic = driveDirectLink(item.Picture || "");
      const waLink = https://wa.me/+8562099872754?text=${encodeURIComponent(`สั่งซื้อ: ${name} ราคา: ${price} บาท)}`;
      
      div.innerHTML = `
        <img src="${pic}" alt="${name}">
        <h3>${name}</h3>
        <p>฿${price}</p>
        <a class="buy-btn" href="${waLink}" target="_blank">สั่งซื้อ</a>
      `;
      grid.appendChild(div);
    });
    container.appendChild(grid);
  })
  .catch(err => {
    document.getElementById("product-list").innerText = "ดึงข้อมูลไม่ได้: "+err;
  });
</script>
</body>
