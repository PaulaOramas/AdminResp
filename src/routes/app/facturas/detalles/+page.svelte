<script>
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';

  let facNumero = '';
  let factura = null;
  let usuario = null;
  let detalles = [];
  let cargando = true;
  let error = '';
  let anulando = false;

  onMount(async () => {
    // Redirección si no es admin
    if (localStorage.getItem("isAdmin") !== "true") {
      goto('/');
      return;
    }

    facNumero = new URLSearchParams(window.location.search).get("facNumero");
    if (!facNumero) {
      error = "❌ No se encontró el número de factura.";
      cargando = false;
      return;
    }

    try {
      // Obtener factura
      const resFactura = await fetch(`https://eltragolocorest.runasp.net/api/Factura/${facNumero}`);
      if (!resFactura.ok) throw new Error("No se pudo obtener la factura.");
      factura = await resFactura.json();

      // Obtener usuario
      const resUsuario = await fetch(`https://eltragolocorest.runasp.net/api/Usuario?ciRuc=${factura.USU_CI_RUC}`);
      if (!resUsuario.ok) throw new Error("No se pudo obtener la información del usuario.");
      usuario = await resUsuario.json();

      // Obtener detalles
      const resDetalle = await fetch(`https://eltragolocorest.runasp.net/api/DetalleFactura/${facNumero}`);
      if (!resDetalle.ok) throw new Error("No se pudo obtener el detalle de la factura.");
      detalles = await resDetalle.json();

      // Obtener nombres de productos
      for (const item of detalles) {
        const resProducto = await fetch(`https://eltragolocorest.runasp.net/api/Producto/${item.PROD_ID}`);
        if (resProducto.ok) {
          const producto = await resProducto.json();
          item.PROD_NOMBRE = producto.PROD_NOMBRE;
        } else {
          item.PROD_NOMBRE = `ID ${item.PROD_ID}`;
        }
      }
    } catch (e) {
      error = e.message;
    }
    cargando = false;
  });

  async function anularFactura() {
    if (!confirm("⚠️ ¿Está seguro que desea anular esta factura? Esta acción no se puede deshacer.")) return;
    anulando = true;
    try {
      const res = await fetch(`https://eltragolocorest.runasp.net/api/Factura?facNumero=${encodeURIComponent(facNumero)}`, {
        method: 'DELETE'
      });
      if (!res.ok) throw new Error('Error al anular la factura');
      alert('✅ Factura anulada.');
      goto('/app/facturas');
    } catch {
      alert('❌ Error al intentar anular la factura');
    }
    anulando = false;
  }
</script>

<svelte:head>
  <title>Detalle de Factura</title>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" />
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.5/font/bootstrap-icons.css" />
  <link href="https://fonts.googleapis.com/css2?family=Raleway:wght@400;600&display=swap" rel="stylesheet" />
</svelte:head>

<div class="container mt-5">
  <div class="text-center mb-4">
    <h1 class="fw-bold titulo-factura text-dorado">
      <i class="bi bi-journal-text"></i> Detalle de Factura
    </h1>
  </div>

  <div class="card-factura p-4 rounded shadow-lg bg-dark text-light" style="max-width:700px; margin:auto;">
    {#if cargando}
      <div class="alert alert-info text-center">Cargando...</div>
    {:else if error}
      <div class="alert alert-danger text-center">{error}</div>
    {:else if factura}
      <div class="mb-3">
        <label class="form-label" for="facNumero">Número de Factura</label>
        <input type="text" class="form-control" id="facNumero" value={factura.FAC_NUMERO} disabled />
      </div>

      <div class="row">
        <div class="col-md-6 mb-3">
          <label class="form-label" for="fecha">Fecha</label>
          <input type="text" class="form-control" id="fecha" value={factura.FAC_FECHA ? new Date(factura.FAC_FECHA).toLocaleDateString() : ''} disabled />
        </div>
        <div class="col-md-6 mb-3">
          <label class="form-label" for="estado">Estado</label>
          <input type="text" class="form-control" id="estado"
            value={
              factura.FAC_ESTADO === "PEN" ? "Pendiente" :
              factura.FAC_ESTADO === "PAG" ? "Pagada" :
              factura.FAC_ESTADO === "ANU" ? "Anulada" : "Desconocido"
            }
            disabled
          />
        </div>
      </div>

      <div class="row">
        <div class="col-md-6 mb-3">
          <label class="form-label" for="cliente">Cliente</label>
          <input type="text" class="form-control" id="cliente" value={usuario?.USU_NOMBRE || factura.USU_CI_RUC} disabled />
        </div>
        <div class="col-md-6 mb-3">
          <label class="form-label" for="cedula">Cédula/RUC</label>
          <input type="text" class="form-control" id="cedula" value={factura.USU_CI_RUC} disabled />
        </div>
      </div>

      <div class="mb-4">
        <label class="form-label" for="productosTable">Productos Comprados</label>
        <div class="table-responsive">
          <table class="table table-productos" id="productosTable">
            <thead>
              <tr>
                <th>Nombre</th>
                <th>Precio</th>
                <th>Cantidad</th>
                <th>Total</th>
              </tr>
            </thead>
            <tbody>
              {#each detalles as item}
                <tr>
                  <td>{item.PROD_NOMBRE}</td>
                  <td>${item.DXF_PRECIO_UNIT?.toFixed(2)}</td>
                  <td>{item.DXF_CANTIDAD}</td>
                  <td>${(item.DXF_CANTIDAD * item.DXF_PRECIO_UNIT).toFixed(2)}</td>
                </tr>
              {/each}
            </tbody>
          </table>
        </div>
      </div>

      <div class="row">
        <div class="col-md-4 mb-3">
          <label class="form-label" for="subtotal">Subtotal</label>
          <input type="text" class="form-control" id="subtotal" value={`$${factura.FAC_SUBTOTAL?.toFixed(2)}`} disabled />
        </div>
        <div class="col-md-4 mb-3">
          <label class="form-label" for="iva">IVA</label>
          <input type="text" class="form-control" id="iva" value={`$${factura.FAC_IVA?.toFixed(2)}`} disabled />
        </div>
        <div class="col-md-4 mb-3">
          <label class="form-label" for="total">Total</label>
          <input type="text" class="form-control" id="total" value={`$${factura.FAC_TOTAL?.toFixed(2)}`} disabled />
        </div>
      </div>

      <div class="d-flex justify-content-between">
        {#if factura.FAC_ESTADO === "PEN" || factura.FAC_ESTADO === "PAG"}
          <button
            type="button"
            class="btn btn-anular fw-bold"
            on:click={anularFactura}
            disabled={anulando}
          >
            <i class="bi bi-x-octagon-fill"></i> Anular Factura
          </button>
        {:else}
          <span></span>
        {/if}
        <a href="/app/facturas" class="btn btn-dorado fw-bold">
          <i class="bi bi-arrow-left-circle-fill"></i> Volver al Panel
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
    --rojo: #c0392b;
  }
  :global(body) {
    background: var(--gris-oscuro) !important;
    min-height: 100vh;
  }
  .card-factura {
    background-color: var(--gris-medio);
    color: #fff;
    border-radius: 1.2rem;
    box-shadow: 0 6px 24px rgba(212, 175, 55, 0.18);
    margin-bottom: 2rem;
    border: 3px solid var(--dorado);
  }
  .form-label {
    color: var(--dorado);
    font-weight: 600;
  }
  .form-control {
    border: 2px solid var(--dorado) !important;
    background-color: var(--gris-oscuro) !important;
    color: #fff !important;
    border-radius: 0.5rem !important;
    font-size: 1.08rem;
    box-shadow: 0 0 0 0.08rem var(--dorado-claro, #f0db7d33);
    transition: border-color 0.2s, box-shadow 0.2s;
  }
  .form-control:disabled {
    background-color: var(--gris-medio) !important;
    color: #fff !important;
    opacity: 1;
  }
  .form-control::placeholder {
    color: #f0db7d99 !important;
    opacity: 1;
  }
  .table {
    border: 2px solid var(--dorado);
    margin-bottom: 0;
  }
  .table th, .table td {
    border: 2px solid var(--dorado) !important;
    color: #fff !important;
    background-color: var(--gris-oscuro) !important;
    vertical-align: middle;
  }
  .table th {
    color: var(--dorado);
    font-weight: 700;
    background-color: var(--gris-medio) !important;
  }
  .titulo-factura {
    color: var(--dorado) !important;
    font-weight: 800;
    letter-spacing: 1px;
  }
  .table-productos {
    background: #fff;
    color: #23252b;
    border: 2px solid var(--dorado);
    margin-bottom: 0;
  }
  .table-productos th {
    background: var(--dorado);
    color: #23252b;
    font-weight: 700;
    border: 2px solid var(--dorado) !important;
  }
  .table-productos td {
    background: var(--gris-oscuro) !important;
    color: blanchedalmond!important;
    font-weight: 700;
    border: 2px solid var(--dorado) !important;
    font-size: 1.05rem;
  }
  .btn-dorado {
    background: var(--dorado);
    color: #23252b;
    font-weight: 700;
    border: none;
    border-radius: 0.5rem;
    min-width: 170px;
    box-shadow: 0 2px 8px #f0db7d33;
    transition: background 0.2s, color 0.2s;
  }
  .btn-dorado:hover, .btn-dorado:focus {
    background: var(--dorado-claro);
    color: #23252b;
  }
  .btn-anular {
    background: var(--rojo);
    color: #fff;
    font-weight: 700;
    border: none;
    border-radius: 0.5rem;
    min-width: 170px;
    box-shadow: 0 2px 8px #c0392b33;
    transition: background 0.2s, color 0.2s;
  }
  .btn-anular:hover, .btn-anular:focus {
    background: #e74c3c;
    color: #fff;
  }
  .d-flex.justify-content-between {
    gap: 1rem;
  }
  .text-dorado {
    color: var(--dorado) !important;
  }
</style>