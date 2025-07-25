<script>
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';

  let prodId = '';
  let nombre = '';
  let descripcion = '';
  let precio = '';
  let stock = '';
  let img = '';
  let categoria = '';
  let categorias = [];
  let categoriaMap = new Map();
  let cargando = true;
  let error = '';

  onMount(async () => {
    if (localStorage.getItem("isAdmin") !== "true") {
      goto('/');
      return;
    }

    const id = new URLSearchParams(window.location.search).get('id');
    if (!id) {
      error = "❌ No se proporcionó el ID del producto.";
      cargando = false;
      return;
    }
    prodId = id;

    try {
      // Cargar categorías
      const resCat = await fetch('https://eltragolocorest.runasp.net/api/Categoria');
      if (!resCat.ok) throw new Error("Error al obtener categorías");
      categorias = await resCat.json();
      categorias.forEach(cat => categoriaMap.set(cat.CAT_ID, cat.CAT_NOMBRE));

      // Cargar producto
      const resProd = await fetch(`https://eltragolocorest.runasp.net/api/Producto/${id}`);
      if (!resProd.ok) throw new Error("Producto no encontrado");
      const prod = await resProd.json();
      nombre = prod.PROD_NOMBRE;
      descripcion = prod.PROD_DESCRIPCION;
      precio = parseFloat(prod.PROD_PRECIO).toFixed(2);
      stock = prod.PROD_STOCK;
      img = prod.PROD_IMG;
      categoria = prod.CAT_ID;
    } catch (e) {
      error = e.message;
    }
    cargando = false;
  });

  // Formatea el precio a dos decimales al salir del input
  function formatearPrecio() {
    if (!isNaN(parseFloat(precio))) {
      precio = parseFloat(precio).toFixed(2);
    }
  }

  async function guardarCambios(e) {
    e.preventDefault();

    if (!categoria || !nombre.trim() || !descripcion.trim() || !img.trim() ||
        isNaN(parseFloat(precio)) || isNaN(parseInt(stock)) ||
        parseFloat(precio) < 0 || parseInt(stock) < 0) {
      alert("⚠️ Por favor complete todos los campos correctamente.");
      return;
    }

    // Se envía el nombre de la categoría, no el ID
    const producto = {
      PROD_ID: parseInt(prodId),
      PROD_NOMBRE: nombre.trim(),
      PROD_DESCRIPCION: descripcion.trim(),
      PROD_PRECIO: parseFloat(precio),
      PROD_STOCK: parseInt(stock),
      PROD_IMG: img.trim(),
      CAT_ID: categoriaMap.get(categoria)
    };

    try {
      const res = await fetch(`https://eltragolocorest.runasp.net/api/Producto/${producto.PROD_ID}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(producto)
      });
      if (res.status === 204 || res.status === 200) {
        alert("✅ Producto actualizado con éxito.");
        goto('/app/productos');
      } else {
        const text = await res.text();
        throw new Error(text);
      }
    } catch (err) {
      alert("❌ Error al actualizar el producto.\n" + err.message);
    }
  }
</script>

<svelte:head>
  <title>Editar Producto</title>
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet" />
  <link href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.5/font/bootstrap-icons.css" rel="stylesheet" />
  <link href="https://fonts.googleapis.com/css2?family=Raleway:wght@400;600&display=swap" rel="stylesheet" />
</svelte:head>

<div class="container mt-5 main-bg">
  <div class="text-center mb-4">
    <h1 class="fw-bold text-dorado">
      <i class="bi bi-pencil-square"></i> Editar Producto
    </h1>
  </div>

  <div class="card card-form shadow">
    {#if cargando}
      <div class="alert alert-info text-center">Cargando...</div>
    {:else if error}
      <div class="alert alert-danger text-center">{error}</div>
    {:else}
      <form on:submit|preventDefault={guardarCambios} autocomplete="off">
        <input type="hidden" id="prodId" value={prodId} />

        <div class="mb-3">
          <label for="ProdNombre" class="form-label">Nombre</label>
          <input id="ProdNombre" class="form-control" bind:value={nombre} required maxlength="100" />
        </div>

        <div class="mb-3">
          <label for="ProdDescripcion" class="form-label">Descripción</label>
          <textarea id="ProdDescripcion" class="form-control" bind:value={descripcion} required maxlength="255"></textarea>
        </div>

        <div class="mb-3">
          <label for="ProdPrecio" class="form-label">Precio ($)</label>
          <input
            id="ProdPrecio"
            type="number"
            class="form-control gold-input"
            bind:value={precio}
            required
            step="0.01"
            min="0"
            on:blur={formatearPrecio}
          />
        </div>

        <div class="mb-3">
          <label for="ProdStock" class="form-label">Stock</label>
          <input id="ProdStock" type="number" class="form-control" bind:value={stock} required min="0" />
        </div>

        <div class="mb-3">
          <label for="ProdImg" class="form-label">Imagen (URL)</label>
          <input id="ProdImg" type="url" class="form-control" bind:value={img} required />
        </div>

        <div class="mb-3">
          <label for="CatId" class="form-label">Categoría</label>
          <select id="CatId" class="form-select" bind:value={categoria} required>
            <option value="">Seleccione una categoría</option>
            {#each categorias as cat}
              <option value={cat.CAT_ID}>{cat.CAT_NOMBRE}</option>
            {/each}
          </select>
        </div>

        <div class="d-flex justify-content-between">
          <button type="submit" class="btn btn-warning">
            <i class="bi bi-check-circle-fill"></i> Guardar Cambios
          </button>
          <a href="/app/productos" class="btn btn-secondary">
            <i class="bi bi-arrow-left-circle-fill"></i> Cancelar
          </a>
        </div>
      </form>
    {/if}
  </div>
</div>

<style>
  :root {
    --dorado: #f0db7d;
    --dorado-claro: #ffe082;
    --gris-medio: rgb(35, 37, 43); /* Ploma oscuro */
    --gris-oscuro: #1a1b1f;
  }

  body, .main-bg {
    background: var(--gris-medio) !important; /* Fondo ploma oscuro */
    min-height: 100vh;
  }

  .container {
    background: transparent !important;
  }

  .card-form {
    max-width: 700px;
    margin: 3rem auto;
    background-color: var(--gris-medio); /* Fondo ploma oscuro para el formulario */
    padding: 2rem;
    border-radius: 1rem;
    box-shadow: 0 6px 18px rgba(0, 0, 0, 0.25); /* Sombra más oscura */
    color: var(--dorado);
  }

  .text-dorado {
    color: var(--dorado);
  }

  .form-label {
    color: var(--dorado);
    font-weight: 600;
    letter-spacing: 0.5px;
  }

  .gold-input,
  .form-control,
  .form-select,
  textarea {
    border: 2px solid var(--dorado) !important;
    background-color: var(--gris-medio) !important; /* Fondo ploma oscuro para inputs */
    color: #fff !important;
    border-radius: 0.5rem !important;
    font-size: 1.08rem;
    box-shadow: 0 0 0 0.08rem var(--dorado-claro, #f0db7d33);
    transition: border-color 0.2s, box-shadow 0.2s;
  }

  .gold-input:focus,
  .form-control:focus,
  .form-select:focus,
  textarea:focus {
    border-color: var(--dorado-claro) !important;
    box-shadow: 0 0 0 0.18rem var(--dorado-claro, #f0db7d55);
    background-color: var(--gris-medio) !important; /* Fondo ploma oscuro al enfocar */
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
    background: var(--gris-medio);
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

  .card-form .btn i {
    margin-right: 5px;
  }
</style>