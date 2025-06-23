<script>
  import { onMount } from 'svelte';
  import { page } from '$app/stores';
  import { goto } from '$app/navigation';

  let ciRuc = '';
  let usuario = null;
  let login = null;
  let cargando = true;
  let error = '';

  onMount(async () => {
    // Redirección si no es admin
    if (localStorage.getItem("isAdmin") !== "true") {
      goto('/');
      return;
    }

    ciRuc = new URLSearchParams(window.location.search).get("id");
    if (!ciRuc) {
      error = "❌ No se proporcionó el identificador del usuario.";
      cargando = false;
      return;
    }

    try {
      // Datos generales
      const resUsuario = await fetch(`https://eltragolocorest.runasp.net/api/Usuario?ciRuc=${ciRuc}`);
      if (!resUsuario.ok) throw new Error("Usuario no encontrado");
      usuario = await resUsuario.json();

      // Datos de login
      const resLogin = await fetch(`https://eltragolocorest.runasp.net/api/Login/Usuario/${ciRuc}`);
      if (!resLogin.ok) throw new Error("Datos de login no encontrados");
      login = await resLogin.json();
    } catch (e) {
      error = e.message;
    }
    cargando = false;
  });

  function volver() {
    goto('/app/usuarios');
  }
</script>

<svelte:head>
  <title>Detalle de Usuario</title>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" />
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.5/font/bootstrap-icons.css" />
  <link href="https://fonts.googleapis.com/css2?family=Raleway:wght@400;600&display=swap" rel="stylesheet" />
</svelte:head>

<div class="container mt-5">
  <div class="text-center mb-4">
    <h1 class="fw-bold text-warning">
      <i class="bi bi-person-vcard-fill"></i> Detalle de Usuario
    </h1>
    <p class="text-warning">Consulta los datos registrados de un cliente</p>
  </div>

  <div class="card shadow p-4 rounded dark-container mx-auto" style="max-width:700px;">
    {#if cargando}
      <div class="alert alert-info text-center">Cargando...</div>
    {:else if error}
      <div class="alert alert-danger text-center">{error}</div>
    {:else if usuario}
      <div class="mb-3">
        <label for="usuCiRuc" class="form-label fw-bold text-warning">Cédula / RUC</label>
        <input id="usuCiRuc" type="text" class="form-control bg-dark text-light border-warning" value={usuario.USU_CI_RUC || "N/D"} disabled />
      </div>
      <div class="mb-3">
        <label for="usuNombre" class="form-label fw-bold text-warning">Nombre Completo</label>
        <input id="usuNombre" type="text" class="form-control bg-dark text-light border-warning" value={usuario.USU_NOMBRE || "N/D"} disabled />
      </div>
      <div class="mb-3">
        <label for="usuEmail" class="form-label fw-bold text-warning">Email</label>
        <input id="usuEmail" type="text" class="form-control bg-dark text-light border-warning" value={usuario.USU_EMAIL || "N/D"} disabled />
      </div>
      <div class="mb-3">
        <label for="usuTelefono" class="form-label fw-bold text-warning">Teléfono</label>
        <input id="usuTelefono" type="text" class="form-control bg-dark text-light border-warning" value={usuario.USU_TELEFONO || "N/D"} disabled />
      </div>
      <div class="mb-3">
        <label for="usuDireccion" class="form-label fw-bold text-warning">Dirección</label>
        <input id="usuDireccion" type="text" class="form-control bg-dark text-light border-warning" value={usuario.USU_DIRECCION || "N/D"} disabled />
      </div>
      <div class="mb-3">
        <label for="usuCiudad" class="form-label fw-bold text-warning">Ciudad</label>
        <input id="usuCiudad" type="text" class="form-control bg-dark text-light border-warning" value={usuario.USU_CIUDAD || "N/D"} disabled />
      </div>
      <div class="mb-3">
        <label for="usuSectorEntrega" class="form-label fw-bold text-warning">Sector de Entrega</label>
        <input id="usuSectorEntrega" type="text" class="form-control bg-dark text-light border-warning" value={usuario.USU_SECTOR_ENTREGA || "N/D"} disabled />
      </div>
      <div class="mb-3">
        <label for="usuGenero" class="form-label fw-bold text-warning">Género</label>
        <input id="usuGenero" type="text" class="form-control bg-dark text-light border-warning" value={usuario.USU_GENERO || "N/D"} disabled />
      </div>
      <div class="mb-3">
        <label for="usuFechaRegistro" class="form-label fw-bold text-warning">Fecha de Registro</label>
        <input id="usuFechaRegistro" type="text" class="form-control bg-dark text-light border-warning"
          value={usuario.USU_FECHA_REGISTRO
            ? (isNaN(new Date(usuario.USU_FECHA_REGISTRO)) ? "Fecha no disponible" : new Date(usuario.USU_FECHA_REGISTRO).toLocaleDateString() + " " + new Date(usuario.USU_FECHA_REGISTRO).toLocaleTimeString())
            : "N/D"} disabled />
      </div>
      <div class="mb-3">
        <label for="usuPassword" class="form-label fw-bold text-warning">Contraseña</label>
        <input id="usuPassword" type="password" class="form-control bg-dark text-light border-warning" value={login?.PASSWORD || "N/D"} disabled />
      </div>
      <div class="mb-3">
        <label for="usuRol" class="form-label fw-bold text-warning">Rol</label>
        <input id="usuRol" type="text" class="form-control bg-dark text-light border-warning" value={login?.LOG_ROL || "N/D"} disabled />
      </div>
      <div class="d-flex justify-content-end">
        <a href="/app/usuarios" class="btn btn-outline-warning fw-bold">
          <i class="bi bi-arrow-left-circle-fill"></i> Volver
        </a>
      </div>
    {/if}
  </div>
</div>

<style>
  .dark-container {
    background-color: #23252b;
    color: #f0db7d;
    border-radius: 1rem;
    box-shadow: 0 6px 20px rgba(0,0,0,0.2);
  }
</style>