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
  h1 { color: #4da3ff; }
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
    .product-grid { grid-template-columns: repeat(1, 1fr); }
  }
</style>
<script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.4.1/papaparse.min.js"></script>
</head>
<body>
<h1>shop1</h1>
<hr>
<h2>รายการสินค้า</h2>
<div id="product-list">กำลังดึงข้อมูล...</div>

<script>
const SHEET_CSV = "https://docs.google.com/spreadsheets/d/e/2PACX-1vSRieAboeby5A7nHJhvVyG532EudqvvfvvPal-u2zO0rfAkRDw_03O06ZNdU0aptATWV83D5zSH9Vn2/pub?gid=0&single=true&output=csv";
const CSV_URL = "https://corsproxy.io/?" + encodeURIComponent(SHEET_CSV);

function convertImageLink(url) {
  if (!url) return "";
  url = url.trim();

  // Google Drive
  const driveMatch = url.match(/\/d\/(.*?)\//);
  if (driveMatch && driveMatch[1]) {
    return https://drive.google.com/uc?export=view&id=${driveMatch[1]};
  }

  // Google Photos
  if (url.includes("googleusercontent.com")) {
    return url;
  }

  // Dropbox
  if (url.includes("dropbox.com")) {
    return url.replace("?dl=0", "?raw=1");
  }

  // Direct image link
  if (/\.(jpg|jpeg|png|gif|webp)$/i.test(url)) {
    return url;
  }

  return url;
}

Papa.parse(CSV_URL, {
  download: true,
  header: true,
  complete: function(results) {
    const data = results.data;
    const productList = document.createElement("div");
    productList.className = "product-grid";

    data.forEach(item => {
      if (!item.Name || !item.Price) return; // ข้ามแถวว่าง
      let imgUrl = convertImageLink(item.Picture);
      const div = document.createElement("div");
      div.className = "product";
      div.innerHTML = `
        <img src="${imgUrl}" alt="${item.Name}">
        <h3>${item.Name}</h3>
        <p>฿${item.Price}</p>
        <a class="buy-btn" href="https://wa.me/+8562099872754?text=สั่งซื้อ: ${encodeURIComponent(item.Name)}" target="_blank">สั่งซื้อ</a>
      `;
      productList.appendChild(div);
    });

    document.getElementById("product-list").innerHTML = "";
    document.getElementById("product-list").appendChild(productList);
  },
  error: function(err) {
    document.getElementById("product-list").innerText = "ดึงข้อมูลไม่ได้: " + err;
  }
});
</script>
</body>
</html>
