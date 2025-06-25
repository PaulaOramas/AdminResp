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
</svelte:head>

<div class="container mt-5">
  <div class="text-center mb-4">
    <h1 class="fw-bold text-dorado">
      <i class="bi bi-person-plus-fill"></i> Agregar Nuevo Usuario
    </h1>
    <p class="text-dorado">Complete los datos para registrar un nuevo cliente</p>
  </div>

  <div class="card card-form dark-container shadow mx-auto">
    <form on:submit|preventDefault={crearUsuario} autocomplete="off">
      <div class="mb-3">
        <label for="ciRuc" class="form-label">Cédula / RUC</label>
        <input id="ciRuc" type="text" class="form-control gold-input" bind:value={ciRuc} maxlength="13" required />
      </div>
      <div class="mb-3">
        <label for="nombre" class="form-label">Nombre Completo</label>
        <input id="nombre" type="text" class="form-control gold-input" bind:value={nombre} required />
      </div>
      <div class="mb-3">
        <label for="email" class="form-label">Email</label>
        <input id="email" type="email" class="form-control gold-input" bind:value={email} required />
      </div>
      <div class="mb-3">
        <label for="telefono" class="form-label">Teléfono</label>
        <input id="telefono" type="text" class="form-control gold-input" bind:value={telefono} maxlength="10" required />
      </div>
      <div class="mb-3">
        <label for="direccion" class="form-label">Dirección</label>
        <input id="direccion" type="text" class="form-control gold-input" bind:value={direccion} required />
      </div>
      <div class="mb-3">
        <label for="ciudad" class="form-label">Ciudad</label>
        <select id="ciudad" class="form-select gold-input" bind:value={ciudad} required>
          <option value="">Seleccione una ciudad</option>
          <option value="Guayaquil">Guayaquil</option>
          <option value="Quito">Quito</option>
          <option value="Cuenca">Cuenca</option>
        </select>
      </div>
      <div class="mb-3">
        <label for="sector" class="form-label">Sector de Entrega</label>
        <select id="sector" class="form-select gold-input" bind:value={sector} disabled={!ciudad} required>
          <option value="">Seleccione un sector</option>
          {#each sectores as s}
            <option value={s}>{s}</option>
          {/each}
        </select>
      </div>
      <div class="mb-3">
        <label for="genero" class="form-label">Género</label>
        <select id="genero" class="form-select gold-input" bind:value={genero} required>
          <option value="">Seleccione un género</option>
          <option value="Masculino">Masculino</option>
          <option value="Femenino">Femenino</option>
          <option value="Otro">Otro</option>
        </select>
      </div>
      <div class="d-flex justify-content-between gap-2 mt-4">
        <button type="submit" class="btn btn-warning fw-bold flex-fill">
          <i class="bi bi-person-plus-fill"></i> Guardar
        </button>
        <a href="/app/usuarios" class="btn btn-outline-light fw-bold flex-fill">
          <i class="bi bi-arrow-left-circle"></i> Cancelar
        </a>
      </div>
    </form>
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
    color: var(--dorado) !important;
  }
  .card-form.dark-container {
    background-color: var(--gris-medio);
    color: #fff;
    border-radius: 1rem;
    box-shadow: 0 6px 20px rgba(212,175,55,0.18);
    max-width: 700px;
    margin: auto;
    padding: 2rem;
  }
  .form-label {
    color: var(--dorado);
    font-weight: 600;
  }
  .gold-input,
  .form-control,
  .form-select {
    border: 2px solid var(--dorado) !important;
    background-color: var(--gris-oscuro) !important;
    color: #fff !important;
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
</style>
