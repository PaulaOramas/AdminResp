<script>
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';

  let nombre = '';
  let descripcion = '';
  let precio = '';
  let stock = '';
  let categoria = '';
  let img = '';
  let categorias = [];

  // Redirección si no es admin
  onMount(() => {
    if (localStorage.getItem("isAdmin") !== "true") {
      goto('/');
    }
    cargarCategorias();
  });

  async function cargarCategorias() {
    try {
      const res = await fetch('https://eltragolocorest.runasp.net/api/Categoria');
      if (!res.ok) throw new Error("Error al obtener categorías");
      categorias = await res.json();
    } catch (err) {
      categorias = [];
      alert("❌ No se pudieron cargar las categorías.");
    }
  }

  async function crearProducto(e) {
    e.preventDefault();

    if (!categoria || !nombre.trim() || !descripcion.trim() || !img.trim() ||
        isNaN(parseFloat(precio)) || isNaN(parseInt(stock)) ||
        parseFloat(precio) < 0 || parseInt(stock) < 0) {
      alert("⚠️ Por favor complete todos los campos correctamente.");
      return;
    }

    const producto = {
      CAT_ID: categoria,
      PROD_NOMBRE: nombre.trim(),
      PROD_DESCRIPCION: descripcion.trim(),
      PROD_IMG: img.trim(),
      PROD_PRECIO: parseFloat(precio),
      PROD_STOCK: parseInt(stock)
    };

    try {
      const res = await fetch('https://eltragolocorest.runasp.net/api/Producto', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(producto)
      });
      if (!res.ok) throw new Error("Error al registrar el producto.");
      alert("✅ Producto registrado exitosamente.");
      // Redirige a la lista de productos
      goto('/app/productos');
    } catch (err) {
      alert("❌ Error al registrar el producto.");
    }
  }
</script>

<svelte:head>
  <title>Crear Producto</title>
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet" />
  <link href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.5/font/bootstrap-icons.css" rel="stylesheet" />
  <link href="https://fonts.googleapis.com/css2?family=Raleway:wght@400;600&display=swap" rel="stylesheet" />
</svelte:head>

<div class="container mt-5">
  <div class="card card-form shadow">
    <h2 class="text-center mb-4 text-dorado fw-bold">
      <i class="bi bi-plus-circle-fill"></i> Crear Nuevo Producto
    </h2>

    <form on:submit|preventDefault={crearProducto} autocomplete="off">
      <div class="mb-3">
        <label for="ProdNombre" class="form-label">Nombre del Producto</label>
        <input id="ProdNombre" class="form-control gold-input" bind:value={nombre} required />
      </div>

      <div class="mb-3">
        <label for="ProdDescripcion" class="form-label">Descripción</label>
        <textarea id="ProdDescripcion" class="form-control gold-input" bind:value={descripcion} required></textarea>
      </div>

      <div class="mb-3">
        <label for="ProdPrecio" class="form-label">Precio ($)</label>
        <input id="ProdPrecio" type="number" class="form-control gold-input" bind:value={precio} min="0" step="0.01" required />
      </div>

      <div class="mb-3">
        <label for="ProdStock" class="form-label">Stock</label>
        <input id="ProdStock" type="number" class="form-control gold-input" bind:value={stock} min="0" required />
      </div>

      <div class="mb-3">
        <label for="categoriaSelect" class="form-label">Categoría</label>
        <select id="categoriaSelect" class="form-select gold-input" bind:value={categoria} required>
          <option value="">Seleccione una categoría</option>
          {#each categorias as cat}
            <option value={cat.CAT_ID}>{cat.CAT_NOMBRE}</option>
          {/each}
        </select>
      </div>

      <div class="mb-4">
        <label for="ProdImg" class="form-label">Imagen del Producto (URL)</label>
        <input id="ProdImg" class="form-control gold-input" bind:value={img} required />
      </div>

      <div class="d-flex justify-content-between gap-2">
        <button type="submit" class="btn btn-warning fw-bold flex-fill">
          <i class="bi bi-plus-circle"></i> Guardar Producto
        </button>
        <a href="/app/productos" class="btn btn-secondary fw-bold flex-fill">
          <i class="bi bi-arrow-left-circle"></i> Cancelar
        </a>
      </div>
    </form>
  </div>
</div>

<style>
  :root {
    --dorado: #f0db7d;
    --dorado-claro: #ffe082;
    --gris-medio: #23252b;
    --gris-oscuro: #1a1b1f;
  }
  :global(body) {
    background: var(--gris-oscuro) !important;
    min-height: 100vh;
  }
  .container {
    background: transparent !important;
  }
  .card-form {
    max-width: 600px;
    margin: 2rem auto;
    padding: 2.5rem 2rem;
    border-radius: 1rem;
    background-color: var(--gris-medio);
    box-shadow: 0 6px 18px rgba(212, 175, 55, 0.18);
    color: var(--dorado-claro);
  }
  .text-dorado {
    color: var(--dorado);
  }
  .form-label {
    color: var(--dorado);
    font-weight: 600;
  }
  .gold-input,
  .form-control,
  .form-select {
    border: 2px solid var(--dorado) !important;
    background-color: var(--gris-oscuro) !important;
    color: #fff !important;
    border-radius: 0.5rem !important;
    font-size: 1.08rem;
    box-shadow: 0 0 0 0.08rem var(--dorado-claro, #f0db7d33);
    transition: border-color 0.2s, box-shadow 0.2s;
  }
  .gold-input:focus,
  .form-control:focus,
  .form-select:focus {
    border-color: var(--dorado-claro) !important;
    box-shadow: 0 0 0 0.18rem var(--dorado-claro, #f0db7d55);
    background-color: #23252b !important;
    color: #fff !important;
  }
  .form-control::placeholder,
  textarea::placeholder {
    color: #f0db7d99 !important;
    opacity: 1;
  }
  .btn-warning {
    background: var(--dorado);
    color: #23252b;
    font-weight: 700;
    border: none;
    box-shadow: 0 2px 8px #f0db7d33;
    transition: background 0.2s, color 0.2s;
  }
  .btn-warning:hover, .btn-warning:focus {
    background: #ffe082;
    color: #23252b;
  }
  .btn-secondary {
    background: #23252b;
    color: var(--dorado);
    border: 2px solid var(--dorado);
    font-weight: 600;
    transition: background 0.2s, color 0.2s;
  }
  .btn-secondary:hover, .btn-secondary:focus {
    background: var(--dorado);
    color: #23252b;
  }
  .card-form .btn {
    min-width: 170px;
    border-radius: 0.5rem;
  }
</style>