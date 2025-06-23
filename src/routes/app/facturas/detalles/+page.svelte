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
    <h1 class="fw-bold text-info"><i class="bi bi-receipt-cutoff"></i> Detalle de Factura</h1>
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
          <table class="table table-dark table-bordered table-striped" id="productosTable">
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
        {#if factura.FAC_ESTADO === "PEN"}
          <button type="button" class="btn btn-danger fw-bold" on:click={anularFactura} disabled={anulando}>
            <i class="bi bi-x-octagon-fill"></i> Anular Factura
          </button>
        {/if}
        <a href="/app/facturas" class="btn btn-primary fw-bold">
          <i class="bi bi-arrow-left-circle-fill"></i> Volver al Panel
        </a>
      </div>
    {/if}
  </div>
</div>

<style>
  .card-factura {
    background-color: #23252b;
    color: #f0db7d;
    border-radius: 1rem;
    box-shadow: 0 6px 18px rgba(212, 175, 55, 0.25);
    margin-bottom: 2rem;
  }
  .text-info { color: #0dcaf0 !important; }
</style>