[23:09, 15/8/2568] Nit: <!DOCTYPE html>
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
  h1 {
    color: #4da3ff;
  }
  .product-grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 15px;
  }
  .product {
    background: rgba(255,255,255,0.1);
    padding: 10px;
    border-radius: 8px;
    text-align: center;
  }
  .product img {
    max-width: 100%;
    border-radius: 5px;
  }
  .buy-btn {
    display: inline-block;
    background: #4da3ff;
    color: white;
    padding: 5px 10px;
    border-radius: 5px;
    margin-top: 5px;
    text-d…
[23:20, 15/8/2568] Nit: <!DOCTYPE html>
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
  h1 {
    color: #4da3ff;
  }
  .product-grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 15px;
  }
  .product {
    background: rgba(255,255,255,0.1);
    padding: 10px;
    border-radius: 8px;
    text-align: center;
  }
  .product img {
    max-width: 100%;
    border-radius: 5px;
  }
  .buy-btn {
    display: inline-block;
    background: #4da3ff;
    color: white;
    padding: 5px 10px;
    border-radius: 5px;
    margin-top: 5px;
    text-decoration: none;
  }
  @media (max-width: 600px) {
    .product-grid {
      grid-template-columns: repeat(1, 1fr);
    }
  }
</style>
</head>
<body>
<h1>shop1</h1>
<hr>
<h2>รายการสินค้า</h2>
<div id="product-list">กำลังดึงข้อมูล...</div>

<script>
const SHEET_CSV = "https://docs.google.com/spreadsheets/d/e/2PACX-1vSRieAboeby5A7nHJhvVyG532EudqvvfvvPal-u2zO0rfAkRDw_03O06ZNdU0aptATWV83D5zSH9Vn2/pub?gid=0&single=true&output=csv";
const CSV_URL = "https://corsproxy.io/?" + encodeURIComponent(SHEET_CSV);

// ฟังก์ชันแปลง Google Drive URL → Direct link
function convertDriveLink(url) {
  const match = url.match(/\/d\/(.*?)\//);
  if (match && match[1]) {
    return https://drive.google.com/uc?export=view&id=${match[1]};
  }
  return url; // ถ้าไม่ใช่ Google Drive ให้คืนค่าเดิม
}

fetch(CSV_URL)
  .then(res => res.text())
  .then(data => {
    const rows = data.trim().split("\n").map(r => r.split(","));
    const headers = rows.shift();
    const nameIdx = headers.indexOf("Name");
    const priceIdx = headers.indexOf("Price");
    const picIdx = headers.indexOf("Picture");
    const linkIdx = headers.indexOf("Link");

    const productList = document.createElement("div");
    productList.className = "product-grid";

    rows.forEach(row => {
      let imgUrl = convertDriveLink(row[picIdx].trim());
      const div = document.createElement("div");
      div.className = "product";
      div.innerHTML = `
        <img src="${imgUrl}" alt="${row[nameIdx]}">
        <h3>${row[nameIdx]}</h3>
        <p>฿${row[priceIdx]}</p>
        <a class="buy-btn" href="https://wa.me/+8562099872754?text=สั่งซื้อ: ${encodeURIComponent(row[nameIdx])}" target="_blank">สั่งซื้อ</a>
      `;
      productList.appendChild(div);
    });

    document.getElementById("product-list").innerHTML = "";
    document.getElementById("product-list").appendChild(productList);
  })
  .catch(err => {
    document.getElementById("product-list").innerText = "ดึงข้อมูลไม่ได้: " + err;
  });
</script>
</body>
</html>
