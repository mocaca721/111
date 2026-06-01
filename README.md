<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>離線 POS 系統</title>
    <script src="https://unpkg.com/dexie/dist/dexie.js"></script>
    <link rel="manifest" href="data:application/json;base64,ewogICJnameIjogIumbouW9miBQT1Mg57O757WxIiwKICAic2hvcnRfbmFtZSI6ICJQT1MiLAogICJzdGFydF91cmwiOiAiLiIsCiAgImRpc3BsYXkiOiAic3RhbmRhbG9uZSIsCiAgImJhY2tncm91bmRfY29sb3IiOiAiI2ZmZmZmZiIKfQ==">
    <style>
        body { font-family: sans-serif; display: flex; margin: 0; height: 100vh; background: #f4f4f4; }
        #menu { flex: 2; padding: 20px; display: grid; grid-template-columns: repeat(3, 1fr); gap: 10px; overflow-y: auto; }
        #cart { flex: 1; background: white; border-left: 1px solid #ddd; display: flex; flex-direction: column; padding: 20px; }
        .product-btn { background: #007bff; color: white; border: none; padding: 20px; border-radius: 8px; font-size: 18px; cursor: pointer; }
        .cart-item { display: flex; justify-content: space-between; margin-bottom: 10px; border-bottom: 1px solid #eee; padding-bottom: 5px; }
        #checkout-btn { background: #28a745; color: white; border: none; padding: 20px; font-size: 20px; margin-top: auto; border-radius: 8px; }
        h2 { margin-top: 0; }
    </style>
</head>
<body>

<div id="menu">
    </div>

<div id="cart">
    <h2>購物車</h2>
    <div id="cart-items"></div>
    <hr>
    <h3 id="total-price">總計: $0</h3>
    <button id="checkout-btn" onclick="checkout()">確認結帳</button>
</div>

<script>
    // 1. 初始化本地資料庫 (代替 base44 API)
    const db = new Dexie("POS_DB");
    db.version(1).stores({
        products: "id, name, price",
        orders: "++id, total_amount, created_date"
    });

    // 2. 預設一些測試商品 (如果資料庫是空的)
    db.products.count(count => {
        if (count === 0) {
            db.products.bulkAdd([
                { id: "1", name: "美式咖啡", price: 60 },
                { id: "2", name: "拿鐵咖啡", price: 80 },
                { id: "3", name: "起司蛋糕", price: 100 }
            ]);
            renderMenu();
        }
    });

    // 3. 渲染商品選單
    async function renderMenu() {
        const products = await db.products.toArray();
        document.getElementById('menu').innerHTML = products.map(p => `
            <button class="product-btn" onclick="addToCart('${p.name}', ${p.price})">
                ${p.name}<br>$${p.price}
            </button>
        `).join('');
    }

    // 4. 購物車邏輯
    let cart = [];
    function addToCart(name, price) {
        cart.push({ name, price });
        updateCartUI();
    }

    function updateCartUI() {
        const cartDiv = document.getElementById('cart-items');
        const totalDiv = document.getElementById('total-price');
        cartDiv.innerHTML = cart.map(item => `
            <div class="cart-item"><span>${item.name}</span><span>$${item.price}</span></div>
        `).join('');
        const total = cart.reduce((sum, item) => sum + item.price, 0);
        totalDiv.innerText = `總計: $${total}`;
    }

    // 5. 結帳邏輯 (存入本地資料庫)
    async function checkout() {
        if (cart.length === 0) return alert("購物車是空的");
        const total = cart.reduce((sum, item) => sum + item.price, 0);
        
        await db.orders.add({
            total_amount: total,
            created_date: new Date().toISOString(),
            items: cart
        });

        alert("結帳成功！訂單已存入本地資料庫");
        cart = [];
        updateCartUI();
    }

    renderMenu();
</script>
</body>
</html>
