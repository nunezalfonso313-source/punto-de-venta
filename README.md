<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Punto de Venta</title>
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-gray-100 font-sans min-h-screen flex flex-col">

  <header class="bg-blue-600 text-white p-4 shadow-lg flex justify-between items-center">
    <h1 class="text-xl font-bold">🛒 Sistema POS</h1>
    <span class="text-xs bg-blue-700 px-2 py-1 rounded">En línea</span>
  </header>

  <main class="flex-1 p-4 max-w-4xl mx-auto w-full grid grid-cols-1 md:grid-cols-3 gap-6">
    
    <section class="md:col-span-2 space-y-6">
      <div class="bg-white p-4 rounded-lg shadow-md">
        <h2 class="text-lg font-bold mb-3 text-gray-700">➕ Nuevo Producto</h2>
        <form id="form-product" class="grid grid-cols-2 gap-3">
          <input type="text" id="p-name" placeholder="Nombre (ej. Coca Cola)" class="border p-2 rounded col-span-2" required>
          <input type="text" id="p-cat" placeholder="Categoría" class="border p-2 rounded" required>
          <input type="number" step="0.5" id="p-price" placeholder="Precio ($)" class="border p-2 rounded" required>
          <input type="number" id="p-stock" placeholder="Stock" class="border p-2 rounded col-span-2" required>
          <button type="submit" class="col-span-2 bg-green-600 text-white font-bold py-2 rounded hover:bg-green-700 transition">
            Guardar en Inventario
          </button>
        </form>
      </div>

      <div class="bg-white p-4 rounded-lg shadow-md">
        <h2 class="text-lg font-bold mb-3 text-gray-700">📦 Catálogo de Productos</h2>
        <div id="product-list" class="grid grid-cols-1 sm:grid-cols-2 gap-3">
          <p class="text-gray-400 text-sm">Cargando productos...</p>
        </div>
      </div>
    </section>

    <section class="bg-white p-4 rounded-lg shadow-md flex flex-col justify-between h-fit">
      <div>
        <h2 class="text-lg font-bold mb-3 text-gray-700">🧾 Carrito de Compra</h2>
        <div id="cart-items" class="space-y-2 mb-4 max-h-60 overflow-y-auto">
          <p class="text-gray-400 text-sm">El carrito está vacío.</p>
        </div>
      </div>

      <div class="border-t pt-4">
        <div class="flex justify-between text-xl font-bold mb-4">
          <span>Total:</span>
          <span id="cart-total">$0.00</span>
        </div>
        <button id="btn-checkout" onclick="checkout()" class="w-full bg-blue-600 text-white font-bold py-3 rounded-lg hover:bg-blue-700 transition" disabled>
          Cobrar
        </button>
      </div>
    </section>

  </main>

  <script>
    const API_URL = "https://pos-backend-api.nunezalfonso313.workers.dev/api";
    let cart = [];
    let products = [];

    async function fetchProducts() {
      try {
        const res = await fetch(`${API_URL}/products`);
        products = await res.json();
        renderProducts();
      } catch (err) {
        console.error("Error al cargar productos:", err);
      }
    }

    function renderProducts() {
      const container = document.getElementById('product-list');
      if (products.length === 0) {
        container.innerHTML = '<p class="text-gray-400 text-sm col-span-2">No hay productos registrados.</p>';
        return;
      }
      container.innerHTML = products.map(p => `
        <div class="border p-3 rounded-lg flex justify-between items-center bg-gray-50">
          <div>
            <h3 class="font-bold text-gray-800">${p.name}</h3>
            <p class="text-xs text-gray-500">${p.category} | Stock: ${p.stock}</p>
            <p class="text-sm font-semibold text-green-600">$${p.price}</p>
          </div>
          <button onclick="addToCart('${p.id}')" class="bg-blue-500 text-white px-3 py-1 rounded text-sm hover:bg-blue-600">
            + Agregar
          </button>
        </div>
      `).join('');
    }

    document.getElementById('form-product').addEventListener('submit', async (e) => {
      e.preventDefault();
      const newProd = {
        id: "p_" + Date.now(),
        name: document.getElementById('p-name').value,
        category: document.getElementById('p-cat').value,
        price: parseFloat(document.getElementById('p-price').value),
        stock: parseInt(document.getElementById('p-stock').value)
      };

      await fetch(`${API_URL}/products`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(newProd)
      });

      e.target.reset();
      fetchProducts();
    });

    function addToCart(id) {
      const prod = products.find(p => p.id === id);
      if (!prod) return;
      
      const item = cart.find(c => c.id === id);
      if (item) {
        item.qty++;
      } else {
        cart.push({ ...prod, qty: 1 });
      }
      renderCart();
    }

    function renderCart() {
      const container = document.getElementById('cart-items');
      const totalEl = document.getElementById('cart-total');
      const btn = document.getElementById('btn-checkout');

      if (cart.length === 0) {
        container.innerHTML = '<p class="text-gray-400 text-sm">El carrito está vacío.</p>';
        totalEl.innerText = '$0.00';
        btn.disabled = true;
        return;
      }

      let total = 0;
      container.innerHTML = cart.map(item => {
        const subtotal = item.price * item.qty;
        total += subtotal;
        return `
          <div class="flex justify-between text-sm items-center border-b pb-1">
            <span>${item.name} x${item.qty}</span>
            <span class="font-semibold">$${subtotal.toFixed(2)}</span>
          </div>
        `;
      }).join('');

      totalEl.innerText = `$${total.toFixed(2)}`;
      btn.disabled = false;
    }

    function checkout() {
      alert("¡Venta completada con éxito!");
      cart = [];
      renderCart();
    }

    fetchProducts();
  </script>
</body>
</html>
