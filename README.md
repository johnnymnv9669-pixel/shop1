<!DOCTYPE html>
<html lang="th">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>shop1</title>
<style>
  body {
    font-family: sans-serif;
    background-color: #304674;
    color: white;
    margin: 0;
    padding: 20px;
  }
  h1 { color: #4da3ff; text-align: center; }
  .product-grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 15px;
    margin-top: 20px;
  }
  .product {
    background: rgba(255,255,255,0.1);
    padding: 10px;
    border-radius: 8px;
    text-align: center;
    transition: transform 0.2s;
  }
  .product:hover { transform: scale(1.03); }
  .product img { max-width: 100%; border-radius: 5px; }
  .buy-btn {
    display: inline-block;
    background: #4da3ff;
    color: white;
    padding: 5px 10px;
    border-radius: 5px;
    margin-top: 5px;
    text-decoration: none;
  }
  @media (max-width: 900px) { .product-grid { grid-template-columns: repeat(2, 1fr); } }
  @media (max-width: 600px) { .product-grid { grid-template-columns: repeat(1, 1fr); } }
</style>
</head>
<body>
<h1>shop1</h1>
<hr>
<h2 style="text-align:center;">รายการสินค้า</h2>
<div id="product-list" style="text-align:center;">กำลังดึงข้อมูล...</div>

<script>
// Google Sheet CSV URL
const SHEET_CSV = "https://docs.google.com/spreadsheets/d/e/2PACX-1vSRieAboeby5A7nHJhvVyG532EudqvvfvvPal-u2zO0rfAkRDw_03O06ZNdU0aptATWV83D5zSH9Vn2/pub?gid=0&single=true&output=csv";

// Proxy สำรองหลายตัว
const PROXIES = [
  url => https://api.allorigins.win/raw?url=${encodeURIComponent(url)},
  url => https://corsproxy.io/?${encodeURIComponent(url)},
  url => https://thingproxy.freeboard.io/fetch/${url}
];

// แปลง Google Drive Link เป็น Direct Link
function driveDirectLink(url) {
  const match = url.match(/\/d\/(.*?)\//);
  return match ? https://drive.google.com/uc?export=view&id=${match[1]} : url;
}

// ฟังก์ชัน fetch ด้วย proxy สำรอง
function fetchCSV(urls, index=0) {
  if(index >= urls.length) return Promise.reject("ไม่สามารถดึงข้อมูล CSV ได้จาก proxy ทั้งหมด");
  const proxyURL = urls[index](SHEET_CSV);
  return fetch(proxyURL)
    .then(res => {
      if(!res.ok) throw new Error("Fetch ล้มเหลว");
      return res.text();
    })
    .catch(err => {
      console.warn("Proxy ล้มเหลว:", proxyURL, err);
      return fetchCSV(urls, index+1); // ลอง proxy ตัวถัดไป
    });
}

// ดึงและแสดงข้อมูล
fetchCSV(PROXIES)
  .then(data => {
    const rows = data.trim().split("\n").map(r => r.split(","));
    const headers = rows.shift().map(h => h.trim()); // trim space
    const nameIdx = headers.indexOf("Name");
    const priceIdx = headers.indexOf("Price");
    const picIdx = headers.indexOf("Picture");
    const linkIdx = headers.indexOf("Link");

    if(nameIdx===-1 || priceIdx===-1 || picIdx===-1) throw new Error("ชื่อคอลัมน์ไม่ตรงกับ Sheet");

    const productList = document.createElement("div");
    productList.className = "product-grid";

    rows.forEach(row => {
      const div = document.createElement("div");
      div.className = "product";

      const name = row[nameIdx] || "ไม่มีชื่อ";
      const price = row[priceIdx] || "-";
      const pic = driveDirectLink(row[picIdx] || "");
      const waMessage = encodeURIComponent(สั่งซื้อ: ${name} ราคา: ${price} บาท);
      const waLink = https://wa.me/+8562099872754?text=${waMessage};

      div.innerHTML = `
        <img src="${pic}" alt="${name}">
        <h3>${name}</h3>
        <p>฿${price}</p>
        <a class="buy-btn" href="${waLink}" target="_blank">สั่งซื้อ</a>
      `;
      productList.appendChild(div);
    });

    const container = document.getElementById("product-list");
    container.innerHTML = "";
    container.appendChild(productList);
  })
  .catch(err => {
    document.getElementById("product-list").innerText = "ดึงข้อมูลไม่ได้: " + err;
  });
</script>
</body>
</html>
