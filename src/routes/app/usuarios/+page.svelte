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
      <input type="text" bind:value={filtroNombre} class="form-control w-auto" placeholder="Filtrar por nombre" aria-label="Filtrar por nombre" />
      <input type="text" bind:value={filtroCI} class="form-control w-auto" placeholder="Filtrar por CI/RUC" aria-label="Filtrar por CI/RUC" />
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
  .card-usuarios {
    background-color: #2c2f36;
    color: #f0db7d;
    border-radius: 10px;
    box-shadow: 0 0 15px #d4af37cc;
  }
  .btn-icon-group .btn {
    margin-right: 0.3rem;
  }
  .btn-icon-group .btn:last-child {
    margin-right: 0;
  }
</style>