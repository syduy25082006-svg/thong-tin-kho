<!DOCTYPE html>
<html lang="vi">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Quản Lý Xuất Nhập Tồn Kho</title>

<style>
:root { --bg-purple: #5c5cfc; --accent: #3498db; --danger: #e74c3c; --success: #27ae60; }

body { font-family: 'Segoe UI', sans-serif; margin: 0; background: #f4f6f9; }

.header { background: var(--bg-purple); padding: 20px; color: white; text-align: center; }

.container {
    max-width: 1200px;
    margin: 20px auto;
    padding: 0 20px;
    display: grid;
    grid-template-columns: 1fr 300px;
    gap: 20px;
}

.card {
    background: white;
    padding: 20px;
    border-radius: 12px;
    box-shadow: 0 2px 10px rgba(0,0,0,0.05);
}

table { width: 100%; border-collapse: collapse; margin-top: 15px; }
th { background: #fafafa; padding: 12px; font-size: 12px; border-bottom: 1px solid #eee; }
td { padding: 12px; border-bottom: 1px solid #f2f2f2; }

.input-section {
    background: white;
    padding: 20px;
    border-radius: 12px;
    margin: -40px auto 20px;
    max-width: 1100px;
    display: flex;
    gap: 15px;
    flex-wrap: wrap;
}

.input-group { display: flex; flex-direction: column; flex: 1; }

input {
    padding: 10px;
    border: 1px solid #ddd;
    border-radius: 6px;
}

button {
    padding: 10px 20px;
    border-radius: 6px;
    border: none;
    cursor: pointer;
    font-weight: bold;
}

.btn-nhap { background: var(--success); color: white; }
.btn-xuat { background: var(--danger); color: white; }

.badge { padding: 3px 8px; border-radius: 4px; font-size: 11px; color: white; }
.bg-nhap { background: var(--success); }
.bg-xuat { background: var(--danger); }

.stock-status {
    background: #fff;
    padding: 15px;
    border-radius: 12px;
}

.stock-item {
    display: flex;
    justify-content: space-between;
    padding: 10px 0;
    border-bottom: 1px solid #eee;
}

.stock-qty { font-weight: bold; color: var(--bg-purple); }

.invoice-box {
    margin-top: 15px;
    padding: 10px;
    background: #fff3cd;
    border: 1px solid #ffeeba;
    border-radius: 8px;
    font-size: 14px;
}
</style>
</head>

<body>

<div class="header">
    <h2>HỆ THỐNG QUẢN LÝ KHO</h2>
</div>

<div class="input-section">
    <div class="input-group">
        <label>SẢN PHẨM</label>
        <input type="text" id="item-name">
    </div>

    <div class="input-group">
        <label>SỐ LƯỢNG</label>
        <input type="number" id="item-qty" value="1">
    </div>

    <div class="input-group">
        <label>ĐƠN GIÁ</label>
        <input type="number" id="item-price">
    </div>

    <div style="display:flex; gap:10px; align-items:end;">
        <button class="btn-nhap" onclick="handleStock('NHẬP')">NHẬP</button>
        <button class="btn-xuat" onclick="handleStock('XUẤT')">XUẤT</button>
    </div>
</div>

<div class="container">

<div class="card">
    <h3>Lịch Sử</h3>
    <table>
        <thead>
            <tr>
                <th>Loại</th>
                <th>Tên</th>
                <th>SL</th>
                <th>Tiền</th>
                <th></th>
            </tr>
        </thead>
        <tbody id="history-data"></tbody>
    </table>

    <div id="invoice"></div>
</div>

<div class="stock-status">
    <h3>📦 Tồn Kho</h3>
    <div id="stock-list">Chưa có hàng</div>
</div>

</div>

<script>

let inventory = {};
let inventoryValue = {};

function formatMoney(num) {
    return new Intl.NumberFormat('vi-VN').format(num) + ' ₫';
}

function handleStock(type) {

    let name = document.getElementById('item-name').value.trim().toUpperCase();
    let qty = parseInt(document.getElementById('item-qty').value);
    let price = parseInt(document.getElementById('item-price').value) || 0;

    if (!name || !qty) {
        alert("Nhập thiếu!");
        return;
    }

    if (type === 'XUẤT') {

        if (!inventory[name] || inventory[name] < qty) {
            alert("Không đủ hàng!");
            return;
        }

        inventory[name] -= qty;

        let total = qty * price;

        // 👉 HIỂN THỊ HÓA ĐƠN
        document.getElementById('invoice').innerHTML = `
            <div class="invoice-box">
                <b>HÓA ĐƠN</b><br>
                Tên: ${name}<br>
                SL: ${qty}<br>
                Giá: ${formatMoney(price)}<br>
                Tổng: <b>${formatMoney(total)}</b>
            </div>
        `;

    } else {

        inventory[name] = (inventory[name] || 0) + qty;
        inventoryValue[name] = (inventoryValue[name] || 0) + qty * price;

    }

    // thêm lịch sử
    let tr = document.createElement('tr');
    tr.innerHTML = `
        <td><span class="badge ${type==='NHẬP'?'bg-nhap':'bg-xuat'}">${type}</span></td>
        <td>${name}</td>
        <td>${qty}</td>
        <td>${formatMoney(qty * price)}</td>
        <td><button onclick="this.parentElement.parentElement.remove()">Xóa</button></td>
    `;
    document.getElementById('history-data').prepend(tr);

    renderStock();

    document.getElementById('item-name').value = '';
    document.getElementById('item-qty').value = 1;
    document.getElementById('item-price').value = '';
}

function renderStock() {

    let html = "";

    for (let item in inventory) {
        if (inventory[item] > 0) {
            html += `
                <div class="stock-item">
                    <span>${item}</span>
                    <span class="stock-qty">
                        ${inventory[item]} cái<br>
                        💰 ${formatMoney(inventoryValue[item] || 0)}
                    </span>
                </div>
            `;
        }
    }

    document.getElementById('stock-list').innerHTML = html || "Kho trống";
}

</script>

</body>
</html>
