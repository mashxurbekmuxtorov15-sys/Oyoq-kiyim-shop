<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>SNEAKER HUB — Telegram Mini App</title>
    <script src="https://telegram.org/js/telegram-web-app.js"></script>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Outfit:wght@700;900&family=Plus+Jakarta+Sans:wght@400;600;700&display=swap');
        body { font-family: 'Plus Jakarta Sans', sans-serif; -webkit-tap-highlight-color: transparent; }
        h1, h2, h3 { font-family: 'Outfit', sans-serif; }
        .no-scrollbar::-webkit-scrollbar { display: none; }
    </style>
</head>
<body class="bg-gray-100 text-gray-900 pb-24">
    <header class="sticky top-0 z-30 bg-white/95 backdrop-blur border-b border-gray-200 px-4 py-3 flex justify-between items-center">
        <div>
            <h1 class="text-xl font-black tracking-tighter">SNEAKER<span class="text-orange-600">HUB</span></h1>
            <p id="user-greeting" class="text-xs text-gray-500">Магазин оригинальных кроссовок</p>
        </div>
        <button onclick="toggleCart()" class="relative bg-gray-100 p-2.5 rounded-full">
            🛒 <span id="cart-badge" class="absolute -top-1 -right-1 bg-orange-600 text-white text-xs font-bold w-5 h-5 flex items-center justify-center rounded-full">0</span>
        </button>
    </header>

    <div class="px-4 pt-4">
        <div class="bg-black text-white rounded-2xl p-5 flex items-center justify-between relative overflow-hidden">
            <div class="z-10 max-w-[65%]">
                <span class="text-orange-400 text-[11px] font-bold uppercase tracking-wider">Хит сезона</span>
                <h2 class="text-xl font-black mt-1 leading-tight">СКИДКИ ДО 20% НА NIKE</h2>
                <p class="text-xs text-gray-400 mt-1">Быстрая доставка с примеркой</p>
            </div>
        </div>
    </div>

    <div class="flex gap-2 overflow-x-auto no-scrollbar px-4 py-4" id="filters">
        <button onclick="filterProducts('Все')" class="filter-btn shrink-0 bg-black text-white px-4 py-2 rounded-full text-xs font-bold transition">Все</button>
        <button onclick="filterProducts('Nike')" class="filter-btn shrink-0 bg-white text-gray-700 border border-gray-200 px-4 py-2 rounded-full text-xs font-bold transition">Nike</button>
        <button onclick="filterProducts('Adidas')" class="filter-btn shrink-0 bg-white text-gray-700 border border-gray-200 px-4 py-2 rounded-full text-xs font-bold transition">Adidas</button>
        <button onclick="filterProducts('New Balance')" class="filter-btn shrink-0 bg-white text-gray-700 border border-gray-200 px-4 py-2 rounded-full text-xs font-bold transition">New Balance</button>
    </div>

    <main id="product-grid" class="px-4 grid grid-cols-1 sm:grid-cols-2 gap-4"></main>

    <div id="bottom-bar" class="fixed bottom-0 left-0 right-0 p-4 bg-gradient-to-t from-gray-100 via-gray-100 to-transparent z-30 hidden">
        <button onclick="toggleCart()" class="w-full bg-black text-white font-bold py-4 px-6 rounded-2xl shadow-xl flex justify-between items-center active:scale-95 transition">
            <span class="flex items-center gap-2">🛒 Перейти в корзину (<span id="bottom-count">0</span>)</span>
            <span id="bottom-total" class="text-orange-400 font-black">0 сум</span>
        </button>
    </div>

    <div id="cart-modal" class="fixed inset-0 bg-black/60 z-50 hidden flex flex-col justify-end">
        <div class="bg-white w-full rounded-t-3xl max-h-[90vh] flex flex-col justify-between p-5 overflow-y-auto">
            <div>
                <div class="flex justify-between items-center border-b pb-3">
                    <h3 class="text-xl font-black">Ваш заказ</h3>
                    <button onclick="toggleCart()" class="bg-gray-100 w-8 h-8 rounded-full font-bold text-gray-600">✕</button>
                </div>
                <div id="cart-items" class="my-4 space-y-3 max-h-[35vh] overflow-y-auto"></div>
            </div>
            <div class="border-t pt-4">
                <div class="flex justify-between text-lg font-black mb-4">
                    <span>Итого к оплате:</span>
                    <span id="cart-total" class="text-orange-600">0 сум</span>
                </div>
                <form id="checkout-form" onsubmit="sendOrder(event)" class="space-y-2.5">
                    <input type="text" id="customer-name" required placeholder="Ваше имя" class="w-full bg-gray-50 border border-gray-200 rounded-xl px-4 py-3 text-sm focus:outline-none focus:border-black">
                    <input type="tel" id="customer-phone" required placeholder="Номер телефона (+998 ...)" class="w-full bg-gray-50 border border-gray-200 rounded-xl px-4 py-3 text-sm focus:outline-none focus:border-black">
                    <input type="text" id="customer-address" required placeholder="Адрес доставки или ориентир" class="w-full bg-gray-50 border border-gray-200 rounded-xl px-4 py-3 text-sm focus:outline-none focus:border-black">
                    <button type="submit" id="submit-btn" class="w-full bg-orange-600 text-white font-bold py-4 rounded-xl shadow-lg active:scale-95 transition mt-2">
                        ✅ Подтвердить заказ
                    </button>
                </form>
            </div>
        </div>
    </div>

    <script>
        const tg = window.Telegram?.WebApp;
        if (tg) tg.expand();

        const TELEGRAM_BOT_TOKEN = '9213370190:AAGZ4C_2b-ShzuSjHqoqU8DQI7Q9M7gTDYU';
        const ADMIN_CHAT_ID = '9213370190';
        const CURRENCY = 'сум';

        const products = [
            { id: 1, brand: 'Nike', name: 'Nike Air Max Red', price: 1536000, sizes: [40, 41, 42, 43, 44] },
            { id: 2, brand: 'Nike', name: 'Nike Dunk High Retro', price: 1728000, sizes: [39, 40, 41, 42, 43] },
            { id: 3, brand: 'New Balance', name: 'New Balance 574 Grey', price: 1408000, sizes: [40, 41, 42, 43, 44] },
            { id: 4, brand: 'Adidas', name: 'Adidas Forum Low', price: 1344000, sizes: [38, 39, 40, 41, 42] }
        ];

        let cart = [];
        let selectedSizes = {};
        products.forEach(p => selectedSizes[p.id] = p.sizes[0]);

        const tgUser = tg?.initDataUnsafe?.user;
        if (tgUser) {
            document.getElementById('user-greeting').innerText = 'Привет, ' + tgUser.first_name + '! 👋';
            document.getElementById('customer-name').value = (tgUser.first_name + ' ' + (tgUser.last_name || '')).trim();
        }

        function vibrate() {
            if (tg?.HapticFeedback) tg.HapticFeedback.impactOccurred('light');
        }

        function renderProducts(filterBrand = 'Все') {
            const grid = document.getElementById('product-grid');
            grid.innerHTML = '';
            const filtered = filterBrand === 'Все' ? products : products.filter(p => p.brand === filterBrand);
            filtered.forEach(product => {
                const currentSize = selectedSizes[product.id];
                const sizesHtml = product.sizes.map(size => `
                    <button onclick="selectSize(${product.id}, ${size})" 
                        class="w-9 h-9 rounded-xl text-xs font-bold border transition ${currentSize === size ? 'bg-black text-white border-black' : 'bg-gray-50 text-gray-700 border-gray-200'}">
                        ${size}
                    </button>
                `).join('');

                grid.innerHTML += `
                    <div class="bg-white rounded-2xl p-4 shadow-sm border border-gray-100 flex flex-col justify-between">
                        <div>
                            <span class="text-xs text-gray-500 font-semibold">${product.brand}</span>
                            <div class="flex justify-between items-start mt-1">
                                <h3 class="font-bold text-base">${product.name}</h3>
                                <span class="font-black text-orange-600">${product.price.toLocaleString()} ${CURRENCY}</span>
                            </div>
                            <p class="text-[11px] text-gray-400 font-semibold uppercase mt-3">Размер:</p>
                            <div class="flex flex-wrap gap-1.5 mt-1">${sizesHtml}</div>
                        </div>
                        <button onclick="addToCart(${product.id})" class="w-full mt-4 bg-black active:bg-orange-600 text-white font-bold py-3 rounded-xl text-sm transition">
                            + В корзину
                        </button>
                    </div>
                `;
            });
        }

        function selectSize(productId, size) {
            vibrate();
            selectedSizes[productId] = size;
            const activeBrand = document.querySelector('.filter-btn.bg-black').innerText;
            renderProducts(activeBrand);
        }

        function filterProducts(brand) {
            vibrate();
            document.querySelectorAll('.filter-btn').forEach(btn => {
                btn.className = btn.innerText === brand
                    ? 'filter-btn shrink-0 bg-black text-white px-4 py-2 rounded-full text-xs font-bold transition'
                    : 'filter-btn shrink-0 bg-white text-gray-700 border border-gray-200 px-4 py-2 rounded-full text-xs font-bold transition';
            });
            renderProducts(brand);
        }

        function addToCart(productId) {
            vibrate();
            const product = products.find(p => p.id === productId);
            const size = selectedSizes[productId];
            const existing = cart.find(item => item.id === productId && item.size === size);
            if (existing) existing.quantity += 1;
            else cart.push({ ...product, size, quantity: 1 });
            updateCartUI();
        }

        function removeFromCart(index) {
            vibrate();
            cart.splice(index, 1);
            updateCartUI();
        }

        function updateCartUI() {
            const totalCount = cart.reduce((sum, i) => sum + i.quantity, 0);
            const totalSum = cart.reduce((sum, i) => sum + i.price * i.quantity, 0);

            document.getElementById('cart-badge').innerText = totalCount;
            document.getElementById('bottom-count').innerText = totalCount;
            document.getElementById('bottom-total').innerText = totalSum.toLocaleString() + ' ' + CURRENCY;
            document.getElementById('cart-total').innerText = totalSum.toLocaleString() + ' ' + CURRENCY;

            const bottomBar = document.getElementById('bottom-bar');
            if (totalCount > 0) bottomBar.classList.remove('hidden');
            else bottomBar.classList.add('hidden');

            const itemsContainer = document.getElementById('cart-items');
            if (cart.length === 0) {
                itemsContainer.innerHTML = '<p class="text-center text-gray-400 py-6 text-sm">Корзина пуста</p>';
                return;
            }

            itemsContainer.innerHTML = cart.map((item, idx) => `
                <div class="flex justify-between items-center bg-gray-50 p-3 rounded-xl">
                    <div>
                        <p class="font-bold text-sm">${item.name}</p>
                        <p class="text-xs text-gray-500">Размер: ${item.size} | ${item.quantity} шт. × ${item.price.toLocaleString()} ${CURRENCY}</p>
                    </div>
                    <button onclick="removeFromCart(${idx})" class="text-red-500 font-bold px-2 py-1 text-xs">Удалить</button>
                </div>
            `).join('');
        }

        function toggleCart() {
            vibrate();
            const modal = document.getElementById('cart-modal');
            modal.classList.toggle('hidden');
        }

        async function sendOrder(event) {
            event.preventDefault();
            if (cart.length === 0) return;

            const btn = document.getElementById('submit-btn');
            btn.disabled = true;
            btn.innerText = '⏳ Отправка заказа...';

            const name = document.getElementById('customer-name').value.trim();
            const phone = document.getElementById('customer-phone').value.trim();
            const address = document.getElementById('customer-address').value.trim();
            const username = tgUser?.username ? '@' + tgUser.username : 'Не указан';
            const totalSum = cart.reduce((sum, i) => sum + i.price * i.quantity, 0);

            const itemsList = cart.map((i, idx) => `${idx + 1}. ${i.name} (Размер: ${i.size}) — ${i.quantity} шт.`).join('\n');
            const text = `👟 НОВЫЙ ЗАКАЗ (Telegram Mini App)\n\n👤 Имя: ${name}\n📱 Телефон: ${phone}\n💬 Telegram: ${username}\n📍 Адрес: ${address}\n\n🛒 Товары:\n${itemsList}\n\n💰 ИТОГО: ${totalSum.toLocaleString()} ${CURRENCY}`;

            try {
                await fetch(`https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage`, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ chat_id: ADMIN_CHAT_ID, text })
                });
                cart = [];
                updateCartUI();
                toggleCart();
                document.getElementById('checkout-form').reset();
                if (tg?.showAlert) tg.showAlert('✅ Заказ успешно отправлен! Мы скоро свяжемся с вами.');
            } catch (e) {
                if (tg?.showAlert) tg.showAlert('Ошибка отправки. Проверьте токен бота.');
            } finally {
                btn.disabled = false;
                btn.innerText = '✅ Подтвердить заказ';
            }
        }

        renderProducts();
    </script>
</body>
</html>
