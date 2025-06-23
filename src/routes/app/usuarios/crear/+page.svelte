<script>
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';

  // Redirección si no es admin
  onMount(() => {
    if (localStorage.getItem("isAdmin") !== "true") {
      window.location.href = "/";
    }
  });

  // Form fields
  let ciRuc = '';
  let nombre = '';
  let email = '';
  let telefono = '';
  let direccion = '';
  let ciudad = '';
  let sector = '';
  let genero = '';

  let sectores = [];

  // Sectores por ciudad
  const sectoresPorCiudad = {
    Guayaquil: ['Saucillo', 'Urdesa', 'Centro', 'Alborada'],
    Quito: ['Carapungo', 'Valle de los Chillos', 'Cumbayá', 'La Floresta'],
    Cuenca: ['Centro Histórico', 'Choloma', 'El Valle', 'Baños']
  };

  // Actualiza sectores cuando cambia ciudad
  $: if (ciudad) {
    sectores = sectoresPorCiudad[ciudad] || [];
    sector = '';
  } else {
    sectores = [];
    sector = '';
  }

  async function crearUsuario(e) {
    e.preventDefault();

    // Validaciones
    if (ciRuc.length < 10 || ciRuc.length > 13) {
      alert("⚠️ La cédula o RUC debe tener entre 10 y 13 caracteres.");
      return;
    }
    if (telefono.length < 10) {
      alert("⚠️ El teléfono debe tener al menos 10 caracteres.");
      return;
    }
    if (!nombre || !email || !direccion || !ciudad || !sector || !genero) {
      alert("⚠️ Por favor complete todos los campos.");
      return;
    }

    const usuarioDTO = {
      USU_CI_RUC: ciRuc,
      USU_NOMBRE: nombre,
      USU_EMAIL: email,
      USU_TELEFONO: telefono,
      USU_DIRECCION: direccion,
      USU_CIUDAD: ciudad,
      USU_SECTOR_ENTREGA: sector,
      USU_GENERO: genero,
      USU_FECHA_REGISTRO: new Date().toISOString(),
      USU_ESTADO: "ACT"
    };

    try {
      const response = await fetch('https://eltragolocorest.runasp.net/api/Usuario', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(usuarioDTO)
      });

      if (!response.ok) {
        const errorText = await response.text();
        throw new Error(errorText || 'Error al crear el usuario');
      }

      alert("✅ Usuario creado correctamente.");
      // Redirige a la lista de usuarios
      goto('/app/usuarios');
    } catch (error) {
      alert("❌ " + error.message);
    }
  }
</script>

<svelte:head>
  <title>Agregar Usuario</title>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" />
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.5/font/bootstrap-icons.css" />
  <link href="https://fonts.googleapis.com/css2?family=Raleway:wght@400;600&display=swap" rel="stylesheet" />
</svelte:head>

<div class="container mt-5">
  <div class="text-center mb-4">
    <h1 class="fw-bold text-warning">
      <i class="bi bi-person-plus-fill"></i> Agregar Nuevo Usuario
    </h1>
    <p class="text-warning">Complete los datos para registrar un nuevo cliente</p>
  </div>

  <div class="card shadow p-4 rounded dark-container mx-auto" style="max-width:700px;">
    <form on:submit|preventDefault={crearUsuario} autocomplete="off">
      <div class="mb-3">
        <label for="ciRuc" class="form-label fw-bold text-warning">Cédula / RUC</label>
        <input id="ciRuc" type="text" class="form-control bg-dark text-light border-warning" bind:value={ciRuc} maxlength="13" required />
      </div>

      <div class="mb-3">
        <label for="nombre" class="form-label fw-bold text-warning">Nombre Completo</label>
        <input id="nombre" type="text" class="form-control bg-dark text-light border-warning" bind:value={nombre} required />
      </div>

      <!-- Email -->
      <div class="mb-3">
        <label for="email" class="form-label fw-bold text-warning">Email</label>
        <input id="email" type="email" class="form-control bg-dark text-light border-warning" bind:value={email} required />
      </div>

      <div class="mb-3">
        <label for="telefono" class="form-label fw-bold text-warning">Teléfono</label>
        <input id="telefono" type="text" class="form-control bg-dark text-light border-warning" bind:value={telefono} maxlength="10" required />
      </div>

      <!-- Dirección -->
      <div class="mb-3">
        <label for="direccion" class="form-label fw-bold text-warning">Dirección</label>
        <input id="direccion" type="text" class="form-control bg-dark text-light border-warning" bind:value={direccion} required />
      </div>

      <div class="mb-3">
        <label for="ciudad" class="form-label fw-bold text-warning">Ciudad</label>
        <select id="ciudad" class="form-select bg-dark text-light border-warning" bind:value={ciudad} required>
          <option value="">Seleccione una ciudad</option>
          <option value="Guayaquil">Guayaquil</option>
          <option value="Quito">Quito</option>
          <option value="Cuenca">Cuenca</option>
        </select>
      </div>

      <div class="mb-3">
        <label for="sector" class="form-label fw-bold text-warning">Sector de Entrega</label>
        <select id="sector" class="form-select bg-dark text-light border-warning" bind:value={sector} disabled={!ciudad} required>
          <option value="">Seleccione un sector</option>
          {#each sectores as s}
            <option value={s}>{s}</option>
          {/each}
        </select>
      </div>

      <!-- Género -->
      <div class="mb-3">
        <label for="genero" class="form-label fw-bold text-warning">Género</label>
        <select id="genero" class="form-select bg-dark text-light border-warning" bind:value={genero} required>
          <option value="">Seleccione un género</option>
          <option value="Masculino">Masculino</option>
          <option value="Femenino">Femenino</option>
          <option value="Otro">Otro</option>
        </select>
      </div>

      <div class="d-grid gap-2">
        <button type="submit" class="btn btn-warning btn-lg fw-bold">
          <i class="bi bi-person-plus-fill"></i> Agregar Usuario
        </button>
      </div>
    </form>
  </div>
</div>

<style>
  .dark-container {
    background-color: #121212;
  }

  .form-label {
    color: #f8f9fa;
  }

  .bg-dark {
    background-color: #343a40 !important;
  }

  .text-light {
    color: #f8f9fa !important;
  }

  .border-warning {
    border-color: #ffc107 !important;
  }

  .btn-warning {
    background-color: #ffc107;
    border-color: #ffc107;
  }

  .btn-warning:hover {
    background-color: #e0a800;
    border-color: #d39e00;
  }
</style>