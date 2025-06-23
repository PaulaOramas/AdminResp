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
          {#if producto.PROD_IMAGEN}
            <img id="prodImagen" src={producto.PROD_IMAGEN} alt="Imagen del producto" class="img-fluid rounded" />
          {:else}
            <div class="alert alert-warning">No hay imagen disponible</div>
          {/if}
        </div>
      </div>
      <div class="text-center">
        <a href="/app/productos" class="btn btn-primary">
          <i class="bi bi-arrow-left"></i> Volver a la lista
        </a>
      </div>
    {/if}
  </div>
</div>

<style>
  .text-dorado {
    color: #f0db7d;
  }

  .detalle-card {
    background-color: #23252b;
    color: #f8f9fa;
    padding: 1.2rem 1.5rem;         /* Menos padding */
    border-radius: 1rem;
    box-shadow: 0 6px 18px rgba(212, 175, 55, 0.15);
    max-width: 480px;               /* Más angosto */
    margin: 0 auto;                 /* Centrado horizontal */
  }

  .detalle-card .form-label {
    font-weight: 600;
    color: #f0db7d;
  }

  .detalle-card input,
  .detalle-card textarea {
    background-color: #23252b;
    color: #f8f9fa;
    border: 1.5px solid #f0db7d;
    font-size: 1rem;
    padding: 0.4rem 0.8rem;
  }

  .detalle-card input:disabled,
  .detalle-card textarea:disabled {
    background-color: #23252b;
    color: #f8f9fa;
    opacity: 1;
  }

  .detalle-card .btn {
    min-width: 140px;
    font-size: 1rem;
    padding: 0.4rem 1.2rem;
  }
</style>