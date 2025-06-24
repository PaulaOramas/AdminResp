<script>
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';

  let productoId = '';
  let producto = null;
  let cargando = true;
  let error = '';

  onMount(async () => {
    // Redirección si no es admin
    if (localStorage.getItem("isAdmin") !== "true") {
      goto('/');
      return;
    }

    productoId = new URLSearchParams(window.location.search).get("id");
    if (!productoId) {
      error = "❌ No se proporcionó ID del producto.";
      cargando = false;
      return;
    }

    try {
      const res = await fetch(`https://eltragolocorest.runasp.net/api/Producto/${productoId}`);
      if (!res.ok) throw new Error('Producto no encontrado');
      producto = await res.json();
    } catch (e) {
      error = e.message;
    }
    cargando = false;
  });
</script>

<svelte:head>
  <title>Detalle de Producto</title>
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet" />
  <link href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.5/font/bootstrap-icons.css" rel="stylesheet" />
  <link href="https://fonts.googleapis.com/css2?family=Raleway:wght@400;600&display=swap" rel="stylesheet" />
</svelte:head>

<div class="container mt-5">
  <div class="text-center mb-4">
    <h1 class="fw-bold text-dorado">
      <i class="bi bi-eye-fill"></i> Detalle de Producto
    </h1>
  </div>

  <div class="card detalle-card shadow">
    {#if cargando}
      <div class="alert alert-info text-center">Cargando...</div>
    {:else if error}
      <div class="alert alert-danger text-center">{error}</div>
    {:else if producto}
      <div class="mb-3">
        <label for="prodId" class="form-label">ID</label>
        <input id="prodId" class="form-control" value={producto.PROD_ID} disabled />
      </div>
      <div class="mb-3">
        <label for="prodNombre" class="form-label">Nombre</label>
        <input id="prodNombre" class="form-control" value={producto.PROD_NOMBRE} disabled />
      </div>
      <div class="mb-3">
        <label for="prodDescripcion" class="form-label">Descripción</label>
        <textarea id="prodDescripcion" class="form-control" rows="3" disabled>{producto.PROD_DESCRIPCION}</textarea>
      </div>
      <div class="mb-3">
        <label for="prodPrecio" class="form-label">Precio</label>
        <input id="prodPrecio" class="form-control" value={producto.PROD_PRECIO?.toFixed(2)} disabled />
      </div>
      <div class="mb-3">
        <label for="prodStock" class="form-label">Stock</label>
        <input id="prodStock" class="form-control" value={`${producto.PROD_STOCK} unidades`} disabled />
      </div>
      <div class="mb-4">
        <label for="prodImagen" class="form-label">Imagen</label>
        <div class="text-center">
          {#if producto.PROD_IMG}
            <img id="prodImagen" src={producto.PROD_IMG} alt="Imagen del producto" class="img-fluid rounded border shadow-sm detalle-img" />
          {:else}
            <div class="alert alert-warning">No hay imagen disponible</div>
          {/if}
        </div>
      </div>
      <div class="mb-3">
        <label for="prodCategoria" class="form-label">Categoría</label>
        <input id="prodCategoria" class="form-control" value={producto.CAT_ID || ''} disabled />
      </div>
      <div class="text-center">
        <a href="/app/productos" class="btn btn-secondary fw-bold">
          <i class="bi bi-arrow-left-circle"></i> Volver a la lista
        </a>
      </div>
    {/if}
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
  .text-dorado {
    color: var(--dorado);
  }
  .detalle-card {
    max-width: 700px;
    margin: auto;
    background-color: var(--gris-medio);
    padding: 2rem;
    border-radius: 1rem;
    box-shadow: 0 6px 18px rgba(212, 175, 55, 0.25);
    color: var(--dorado-claro);
  }
  label.form-label {
    color: var(--dorado-claro);
    font-weight: 600;
  }
  input.form-control,
  textarea.form-control {
    background-color: var(--gris-oscuro);
    color: var(--dorado-claro);
    border: 1.5px solid var(--dorado);
    border-radius: 0.4rem;
    transition: border-color 0.3s ease;
  }
  input.form-control:disabled,
  textarea.form-control:disabled {
    background-color: var(--gris-medio);
    color: var(--dorado-claro);
    opacity: 1;
  }
  .detalle-img {
    max-height: 300px;
    border: 2px solid var(--dorado);
    background: #fff;
    margin: 0 auto;
    display: block;
    box-shadow: 0 2px 8px #f0db7d33;
  }
  .btn-secondary {
    background: var(--gris-oscuro);
    color: var(--dorado);
    border: 2px solid var(--dorado);
    font-weight: 600;
    transition: background 0.2s, color 0.2s;
  }
  .btn-secondary:hover, .btn-secondary:focus {
    background: var(--dorado);
    color: #23252b;
  }
</style>