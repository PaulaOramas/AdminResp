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

<div class="card card-form shadow">
  <h2 class="text-center mb-4 text-dorado fw-bold">
    <i class="bi bi-plus-circle-fill"></i> Crear Nuevo Producto
  </h2>

  <form on:submit|preventDefault={crearProducto} autocomplete="off">
    <div class="mb-3">
      <label for="ProdNombre" class="form-label">Nombre del Producto</label>
      <input id="ProdNombre" class="form-control" bind:value={nombre} required />
    </div>

    <div class="mb-3">
      <label for="ProdDescripcion" class="form-label">Descripción</label>
      <textarea id="ProdDescripcion" class="form-control" bind:value={descripcion} required></textarea>
    </div>

    <div class="mb-3">
      <label for="ProdPrecio" class="form-label">Precio ($)</label>
      <input id="ProdPrecio" type="number" class="form-control" bind:value={precio} min="0" step="0.01" required />
    </div>

    <div class="mb-3">
      <label for="ProdStock" class="form-label">Stock</label>
      <input id="ProdStock" type="number" class="form-control" bind:value={stock} min="0" required />
    </div>

    <div class="mb-3">
      <label for="categoriaSelect" class="form-label">Categoría</label>
      <select id="categoriaSelect" class="form-select" bind:value={categoria} required>
        <option value="">Seleccione una categoría</option>
        {#each categorias as cat}
          <option value={cat.CAT_ID}>{cat.CAT_NOMBRE}</option>
        {/each}
      </select>
    </div>

    <div class="mb-3">
      <label for="ProdImg" class="form-label">Imagen URL</label>
      <input id="ProdImg" class="form-control" bind:value={img} required />
    </div>

    <div class="text-center">
      <button type="submit" class="btn btn-dorado">
        <i class="bi bi-plus-circle"></i> Crear Producto
      </button>
    </div>
  </form>

  <div class="mt-4">
    <a href="/app/productos" class="btn btn-secondary">
      <i class="bi bi-arrow-left"></i> Volver a la lista de productos
    </a>
  </div>
</div>

<style>
  .card-form {
    max-width: 600px;
    margin: 2rem auto;
    padding: 2rem;
    border-radius: 1rem;
    background-color: #fff;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  }

  .text-dorado {
    color: #c59d3b;
  }

  .btn-dorado {
    background-color: #c59d3b;
    color: #fff;
  }

  .btn-dorado:hover {
    background-color: #b58a2a;
  }

  .form-label {
    font-weight: 500;
  }

  .form-control, .form-select {
    border-radius: 0.5rem;
    box-shadow: none;
    border: 1px solid #ced4da;
  }

  .form-control:focus, .form-select:focus {
    border-color: #c59d3b;
    box-shadow: 0 0 0 0.2rem rgba(197, 157, 59, 0.25);
  }
</style>