<script>
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';

  let productos = [];
  let categorias = [];
  let filtroCategoria = '';
  let busquedaNombre = '';
  let cargando = true;
  let error = '';

  // Redirección si no es admin
  onMount(() => {
    if (localStorage.getItem("isAdmin") !== "true") {
      window.location.href = "/";
    }
    cargarCategorias();
    cargarProductos();
  });

  async function cargarCategorias() {
    try {
      const res = await fetch('https://eltragolocorest.runasp.net/api/Categoria');
      if (!res.ok) throw new Error("Error al cargar categorías");
      categorias = await res.json();
    } catch (err) {
      categorias = [];
    }
  }

  async function cargarProductos() {
    cargando = true;
    try {
      const res = await fetch('https://eltragolocorest.runasp.net/api/Producto');
      if (!res.ok) throw new Error("Error al cargar productos");
      productos = await res.json();
      error = '';
    } catch (err) {
      productos = [];
      error = "❌ Error al cargar los productos.";
    }
    cargando = false;
  }

  $: productosFiltrados = productos.filter(p => {
    const coincideCategoria = !filtroCategoria || p.CAT_NOMBRE === filtroCategoria || p.CAT_ID === filtroCategoria;
    const coincideBusqueda = p.PROD_NOMBRE?.toLowerCase().includes(busquedaNombre.toLowerCase());
    return coincideCategoria && coincideBusqueda;
  });

  function editarProducto(productId) {
    window.location.href = `/app/productos/editar?id=${productId}`;
  }

  function verDetalleProducto(productId) {
    window.location.href = `/app/productos/detalle?id=${productId}`;
  }

  function agregarProducto() {
    window.location.href = `/app/productos/crear`;
  }

  function volverPanel() {
    goto("/dashboard");
  }
</script>

<svelte:head>
  <title>Lista de Productos</title>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" />
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.5/font/bootstrap-icons.css" />
  <link href="https://fonts.googleapis.com/css2?family=Raleway:wght@400;600&display=swap" rel="stylesheet" />
</svelte:head>

<main class="container mt-5">
  <div class="text-center mb-4">
    <h1 class="fw-bold text-warning">
      <i class="bi bi-box-seam-fill"></i> Lista de Productos
    </h1>
    <p class="text-light">
      Administra los productos de tu tienda con facilidad.
    </p>
  </div>

  <div class="d-flex justify-content-between mb-3 flex-wrap gap-2">
    <button class="btn btn-warning fw-bold" aria-label="Agregar nuevo producto" on:click={agregarProducto}>
      <i class="bi bi-plus-circle"></i> Agregar Producto
    </button>
  </div>

  <div class="mb-3 d-flex gap-3 flex-wrap">
    <!-- filtros aquí -->
  </div>

  <div class="card shadow-lg p-4 rounded-4 bg-secondary text-light card-lista">
    <div class="d-flex justify-content-end mb-3">
      <a href="/app/dashboard" class="btn btn-outline-light fw-bold">
        <i class="bi bi-arrow-left-circle-fill"></i> Volver al Panel
      </a>
    </div>
    <!-- Aquí tu tabla de productos -->
    {#if cargando}
      <div class="alert alert-info text-center m-0">Cargando productos...</div>
    {:else if error}
      <div class="alert alert-danger text-center m-0">{error}</div>
    {:else if productosFiltrados.length === 0}
      <div class="alert alert-info text-center m-0">📭 No hay productos registrados todavía.</div>
    {:else}
      <div class="table-responsive">
        <table class="table table-dark table-hover table-bordered text-center align-middle">
          <thead class="table-light text-dark">
            <tr>
              <th>ID</th>
              <th class="text-start">Nombre</th>
              <th>Precio</th>
              <th>Stock</th>
              <th>Acciones</th>
            </tr>
          </thead>
          <tbody>
            {#each productosFiltrados as prod}
              <tr>
                <td>{prod.PROD_ID}</td>
                <td class="text-start">{prod.PROD_NOMBRE}</td>
                <td>${parseFloat(prod.PROD_PRECIO).toFixed(2)}</td>
                <td>{prod.PROD_STOCK}</td>
                <td class="btn-icon-group">
                  <button
                    class="btn btn-sm btn-warning me-1"
                    title="Editar"
                    aria-label="Editar producto"
                    on:click={() => editarProducto(prod.PROD_ID)}
                  >
                    <i class="bi bi-pencil-fill"></i>
                  </button>
                  <button
                    class="btn btn-sm btn-info text-white"
                    title="Ver Detalle"
                    aria-label="Ver detalle del producto"
                    on:click={() => verDetalleProducto(prod.PROD_ID)}
                  >
                    <i class="bi bi-eye-fill"></i>
                  </button>
                </td>
              </tr>
            {/each}
          </tbody>
        </table>
      </div>
    {/if}
  </div>
</main>

<style>
  :global(body) {
    background-color: #23252b !important;
    color: #f8f9fa;
  }
  main {
    font-family: 'Raleway', Arial, sans-serif;
    background-color: #23252b;
    min-height: 100vh;
  }
  .shadow-lg {
    background-color: #23252b !important;
    color: #f8f9fa;
    border-radius: 1.5rem !important;
    box-shadow: 0 6px 18px rgba(212, 175, 55, 0.10);
    border: 2px solid #484b52;
  }
  .table-dark {
    --bs-table-bg: #23252b;
    --bs-table-striped-bg: #23252b;
    --bs-table-hover-bg: #23252b;
    color: #f8f9fa;
  }
  .table thead th {
    vertical-align: middle;
    background-color: #484b52 !important;
    color: #f0db7d !important;
    border-bottom: 2px solid #f0db7d;
  }
  .btn-icon-group .btn {
    min-width: 36px;
    min-height: 36px;
    font-size: 1.1rem;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    border-radius: 0.5rem;
  }
  .rounded-4 {
    border-radius: 1.5rem !important;
  }
  .text-warning {
    color: #f0db7d !important;
  }
  .text-light {
    color: #f8f9fa !important;
  }
  .fw-bold {
    font-weight: 700 !important;
  }
</style>