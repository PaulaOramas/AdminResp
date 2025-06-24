<script>
  import { onMount } from 'svelte';

  let usuarios = [];
  let filtroNombre = '';
  let filtroCI = '';
  let cargando = true;
  let error = '';

  // Redirección si no es admin
  onMount(() => {
    if (localStorage.getItem("isAdmin") !== "true") {
      window.location.href = "/";
    }
    cargarUsuarios();
  });

  async function cargarUsuarios() {
    cargando = true;
    error = '';
    try {
      const res = await fetch('https://eltragolocorest.runasp.net/api/Usuario');
      if (!res.ok) throw new Error("Error al cargar usuarios");
      usuarios = await res.json();
    } catch (err) {
      error = err.message;
      usuarios = [];
    }
    cargando = false;
  }

  $: usuariosFiltrados = usuarios.filter(u => {
    const coincideNombre = u.USU_NOMBRE?.toLowerCase().includes(filtroNombre.toLowerCase());
    const coincideCI = u.USU_CI_RUC?.toLowerCase().includes(filtroCI.toLowerCase());
    return coincideNombre && coincideCI;
  });

  async function eliminarUsuario(ciRuc, nombre) {
    if (confirm(`⚠️ ¿Estás seguro que deseas eliminar el usuario "${nombre}"? Esta acción no se puede deshacer.`)) {
      try {
        const res = await fetch(`https://eltragolocorest.runasp.net/api/Usuario?ciRuc=${encodeURIComponent(ciRuc)}`, {
          method: 'DELETE'
        });
        if (!res.ok) throw new Error("Error al eliminar usuario");
        alert("✅ Usuario eliminado.");
        cargarUsuarios();
      } catch (err) {
        alert("❌ Error al intentar eliminar el usuario.");
      }
    }
  }
</script>

<svelte:head>
  <title>Lista de Usuarios</title>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" />
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.5/font/bootstrap-icons.css" />
  <link href="https://fonts.googleapis.com/css2?family=Raleway:wght@400;600&display=swap" rel="stylesheet" />
</svelte:head>

<main class="container mt-5">
  <div class="text-center mb-4">
    <h1 class="fw-bold text-warning">
      <i class="bi bi-people-fill"></i> Lista de Usuarios
    </h1>
    <p class="text-light">Gestiona los clientes registrados en el sistema</p>
  </div>

  <div class="card shadow-lg p-4 rounded-4 bg-secondary text-light card-usuarios">
    <div class="d-flex justify-content-between mb-3 flex-wrap gap-2">
      <a href="/app/usuarios/crear" class="btn btn-warning fw-bold" aria-label="Agregar nuevo usuario">
        <i class="bi bi-person-plus-fill"></i> Agregar Nuevo Usuario
      </a>
      <a href="/app/dashboard" class="btn btn-outline-light fw-bold">
        <i class="bi bi-arrow-left-circle-fill"></i> Volver al Panel
      </a>
    </div>

    <!-- Filtros -->
    <div class="mb-3 d-flex gap-3 flex-wrap">
      <input type="text" bind:value={filtroNombre} class="form-control gold-input w-auto" placeholder="Filtrar por nombre" aria-label="Filtrar por nombre" />
      <input type="text" bind:value={filtroCI} class="form-control gold-input w-auto" placeholder="Filtrar por CI/RUC" aria-label="Filtrar por CI/RUC" />
    </div>

    <div id="usuariosTableContainer">
      {#if cargando}
        <div class="alert alert-info text-center m-0">Cargando usuarios...</div>
      {:else if error}
        <div class="alert alert-danger text-center m-0">{error}</div>
      {:else if usuariosFiltrados.length === 0}
        <div class="alert alert-info text-center m-0">📭 No hay usuarios que coincidan con el filtro.</div>
      {:else}
        <div class="table-responsive">
          <table class="table table-dark table-hover table-bordered text-center align-middle">
            <thead class="table-light text-dark">
              <tr>
                <th>CI / RUC</th>
                <th>Nombre</th>
                <th>Email</th>
                <th>Teléfono</th>
                <th>Acciones</th>
              </tr>
            </thead>
            <tbody>
              {#each usuariosFiltrados as u}
                <tr>
                  <td>{u.USU_CI_RUC}</td>
                  <td class="text-start">{u.USU_NOMBRE}</td>
                  <td>{u.USU_EMAIL}</td>
                  <td>{u.USU_TELEFONO}</td>
                  <td class="btn-icon-group">
                    <a
                      href={`/app/usuarios/detalles?id=${u.USU_CI_RUC}`}
                      class="btn btn-sm btn-info text-white"
                      title="Ver Detalle"
                      aria-label="Ver detalle de usuario"
                    >
                      <i class="bi bi-eye-fill"></i>
                    </a>
                    <a
                      href={`/app/usuarios/editar?id=${u.USU_CI_RUC}`}
                      class="btn btn-sm btn-warning"
                      title="Editar"
                      aria-label="Editar usuario"
                    >
                      <i class="bi bi-pencil-fill"></i>
                    </a>
                    <button
                      class="btn btn-sm btn-danger"
                      title="Eliminar"
                      aria-label="Eliminar usuario"
                      on:click={() => eliminarUsuario(u.USU_CI_RUC, u.USU_NOMBRE)}
                    >
                      <i class="bi bi-trash-fill"></i>
                    </button>
                  </td>
                </tr>
              {/each}
            </tbody>
          </table>
        </div>
      {/if}
    </div>
  </div>
</main>

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
  .card-usuarios {
    background-color: var(--gris-medio);
    color: #fff; /* Letras blancas */
    border-radius: 1rem;
    box-shadow: 0 0 15px #d4af37cc;
    padding: 2rem;
  }
  .text-warning, .text-dorado {
    color: var(--dorado) !important;
  }
  .gold-input,
  .form-control,
  .form-select {
    border: 2px solid var(--dorado) !important;
    background-color: var(--gris-oscuro) !important;
    color: #fff !important; /* Letras blancas en inputs */
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
  .form-control::placeholder {
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
  .btn-outline-light {
    border: 2px solid var(--dorado) !important;
    color: var(--dorado) !important;
    background: transparent !important;
    font-weight: 600;
    transition: background 0.2s, color 0.2s;
  }
  .btn-outline-light:hover, .btn-outline-light:focus {
    background: var(--dorado) !important;
    color: #23252b !important;
  }
  .btn-icon-group .btn {
    margin-right: 0.3rem;
  }
  .btn-icon-group .btn:last-child {
    margin-right: 0;
  }
  .table-bordered th,
  .table-bordered td {
    border-color: var(--dorado) !important;
    border-width: 2px !important;
  }
  .table-dark {
    background-color: var(--gris-oscuro) !important;
    color: #fff !important; /* Letras blancas en tabla */
  }
  .table-dark th, .table-dark td {
    background-color: var(--gris-oscuro) !important;
    color: #fff !important; /* Letras blancas en tabla */
  }
  .table-light {
    background-color: var(--dorado-claro) !important;
    color: #23252b !important;
  }
</style>