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
.product { background:rgba(255,255,255,0.1); padding:10px; border-radius:8px; text-align:center; transition:0.2s; position: relative; overflow: hidden;}
.product:hover { transform:scale(1.03); }
.product img { max-width:100%; border-radius:5px; display:block; }
.buy-btn { display:inline-block; background:#4da3ff; color:white; padding:5px 10px; border-radius:5px; margin-top:5px; text-decoration:none;}
.skeleton { background: rgba(255,255,255,0.2); width: 100%; height: 200px; border-radius:5px; animation: pulse 1.5s infinite; }
@keyframes pulse {
  0% { opacity: 0.6; }
  50% { opacity: 1; }
  100% { opacity: 0.6; }
}
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
const WEBAPP_URL = "https://script.google.com/macros/s/XXXXXXXXXXXX/exec"; // ใส่ URL Web App ของคุณ

function driveDirectLink(url){
  if(!url) return "";
  const match = url.match(/\/d\/(.*?)\//);
  return match ? https://drive.google.com/uc?export=view&id=${match[1]} : url;
}

function createLazyImage(src, alt){
  const img = document.createElement("img");
  img.alt = alt;
  img.loading = "lazy";
  img.src = src;
  return img;
}

async function loadProducts(){
  try{
    const res = await fetch(WEBAPP_URL);
    if(!res.ok) throw new Error("Fetch ล้มเหลว");
    const items = await res.json();
    if(!Array.isArray(items)) throw new Error("ข้อมูลไม่ใช่ Array");

    const container = document.getElementById("product-list");
    container.innerHTML = "";
    const grid = document.createElement("div");
    grid.className = "product-grid";

    items.forEach(item => {
      const div = document.createElement("div");
      div.className = "product";

      const skeleton = document.createElement("div");
      skeleton.className = "skeleton";
      div.appendChild(skeleton);

      const name = item.Name || "ไม่มีชื่อ";
      const price = item.Price || "-";
      const pic = driveDirectLink(item.Picture || "");
      const waLink = https://wa.me/+8562099872754?text=${encodeURIComponent(`สั่งซื้อ: ${name} ราคา: ${price} บาท)}`;

      const img = createLazyImage(pic, name);
      img.onload = () => skeleton.remove();
      div.appendChild(img);

      const h3 = document.createElement("h3"); h3.innerText = name; div.appendChild(h3);
      const p = document.createElement("p"); p.innerText = ฿${price}; div.appendChild(p);
      const a = document.createElement("a"); a.className="buy-btn"; a.href=waLink; a.target="_blank"; a.innerText="สั่งซื้อ"; div.appendChild(a);

      grid.appendChild(div);
    });

    container.appendChild(grid);
  }catch(err){
    document.getElementById("product-list").innerText = "ดึงข้อมูลไม่ได้: " + err;
    console.error(err);
  }
}

loadProducts();
</script>
</body>
</html>
