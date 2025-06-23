<!-- filepath: /admintrago-svelte/admintrago-svelte/src/pages/Facturas.svelte -->
<script>
  import { onMount } from 'svelte';

  let facturas = [];
  let filtroCedula = '';
  let cargando = true;
  let error = '';

  // Redirección si no es admin
  onMount(() => {
    if (localStorage.getItem("isAdmin") !== "true") {
      window.location.href = "/";
    }
    cargarFacturas();
  });

  async function cargarFacturas() {
    cargando = true;
    error = '';
    try {
      const res = await fetch("https://eltragolocorest.runasp.net/api/Factura");
      if (!res.ok) throw new Error("Error al obtener las facturas");
      facturas = await res.json();
    } catch (err) {
      error = err.message;
      facturas = [];
    }
    cargando = false;
  }

  $: facturasFiltradas = facturas.filter(f =>
    f.USU_CI_RUC?.toLowerCase().includes(filtroCedula.toLowerCase())
  );

  async function anularFactura(facNumero) {
    if (!confirm(`⚠️ ¿Estás seguro de anular la factura ${facNumero}? Esta acción no se puede deshacer.`)) return;
    try {
      const res = await fetch(`https://eltragolocorest.runasp.net/api/Factura?facNumero=${encodeURIComponent(facNumero)}`, {
        method: 'DELETE'
      });
      if (!res.ok) throw new Error('Error al anular la factura');
      alert('✅ Factura anulada.');
      cargarFacturas();
    } catch {
      alert('❌ Error al intentar anular la factura');
    }
  }

  function verDetalle(facNumero) {
    // Redirige a la página de detalles (ajusta la ruta según tu estructura)
    window.location.href = `/app/facturas/detalles?facNumero=${facNumero}`;
  }
</script>

<svelte:head>
  <title>Lista de Facturas</title>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" />
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.5/font/bootstrap-icons.css" />
  <link href="https://fonts.googleapis.com/css2?family=Raleway:wght@400;600&display=swap" rel="stylesheet" />
</svelte:head>

<main class="container mt-5">
  <div class="text-center mb-4">
    <h1 class="fw-bold text-warning">
      <i class="bi bi-file-earmark-text-fill"></i> Lista de Facturas
    </h1>
    <p class="text-light">Administra las facturas emitidas por la tienda</p>
  </div>

  <div class="mb-3 d-flex gap-3 flex-wrap align-items-center">
    <button type="button" class="btn btn-warning fw-bold" disabled
      style="cursor: default; user-select: none; min-width: 130px;">
      Buscar por Cédula:
    </button>
    <input
      type="search"
      bind:value={filtroCedula}
      class="form-control w-auto"
      placeholder="Ingrese cédula o RUC"
      aria-label="Buscar factura por cédula"
    />
  </div>

  <div class="card shadow-lg p-4 rounded-4 bg-secondary text-light card-lista">
    <div class="d-flex justify-content-end mb-3">
      <a href="/app/dashboard" class="btn btn-outline-light fw-bold">
        <i class="bi bi-arrow-left-circle-fill"></i> Volver al Panel
      </a>
    </div>

    {#if cargando}
      <div class="alert alert-info text-center">Cargando facturas...</div>
    {:else if error}
      <div class="alert alert-danger text-center">{error}</div>
    {:else if facturasFiltradas.length === 0}
      <div class="alert alert-info text-center">📭 No hay facturas registradas.</div>
    {:else}
      <div class="table-responsive">
        <table class="table table-dark table-hover table-bordered text-center">
          <thead class="table-light text-dark">
            <tr>
              <th>Número</th>
              <th>Cliente</th>
              <th>Fecha</th>
              <th>Total</th>
              <th>Estado</th>
              <th>Acciones</th>
            </tr>
          </thead>
          <tbody>
            {#each facturasFiltradas as factura}
              <tr>
                <td>{factura.FAC_NUMERO}</td>
                <td>{factura.USU_CI_RUC}</td>
                <td>
                  {#if factura.FAC_FECHA}
                    {#if !isNaN(new Date(factura.FAC_FECHA).getTime())}
                      {new Date(factura.FAC_FECHA).toLocaleDateString()}
                    {:else}
                      Fecha no válida
                    {/if}
                  {:else}
                    Sin fecha
                  {/if}
                </td>
                <td>${factura.FAC_TOTAL?.toFixed(2)}</td>
                <td>
                  {#if factura.FAC_ESTADO === "PEN"}
                    <span class="badge bg-warning text-dark">Pendiente</span>
                  {:else if factura.FAC_ESTADO === "ANU"}
                    <span class="badge bg-danger">Anulada</span>
                  {:else}
                    <span class="badge bg-success">Pagada</span>
                  {/if}
                </td>
                <td>
                  <div class="acciones-container">
                    <button class="btn btn-info btn-sm" aria-label="Ver detalle" on:click={() => verDetalle(factura.FAC_NUMERO)}>
                      <i class="bi bi-eye-fill"></i> Ver
                    </button>
                    {#if factura.FAC_ESTADO === "PEN"}
                      <button class="btn btn-danger btn-sm" aria-label="Anular factura" on:click={() => anularFactura(factura.FAC_NUMERO)}>
                        <i class="bi bi-x-circle-fill"></i> Anular
                      </button>
                    {/if}
                  </div>
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
  .card-lista {
    background-color: #2c2f36;
    color: #f0db7d;
    border-radius: 10px;
    box-shadow: 0 0 15px #d4af37cc;
  }
  .acciones-container .btn {
    margin-right: 0.3rem;
  }
  .acciones-container .btn:last-child {
    margin-right: 0;
  }
</style>