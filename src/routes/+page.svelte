<script>
  let cedula = '';
  let contrasena = '';

  async function validarYEnviar(e) {
    e.preventDefault();

    if (cedula.length < 10 || cedula.length > 13) {
      alert("⚠️ La cédula o RUC debe tener entre 10 y 13 caracteres.");
      return;
    }
    if (contrasena.length < 6) {
      alert("⚠️ La contraseña debe tener al menos 6 caracteres.");
      return;
    }

    try {
      const res = await fetch(`https://eltragolocorest.runasp.net/api/Login/Usuario/${cedula}`);
      if (!res.ok) {
        alert("❌ Usuario no encontrado.");
        contrasena = '';
        return;
      }
      const data = await res.json();
      if (data.PASSWORD === contrasena && data.LOG_ROL === "ADM") {
        localStorage.setItem("isAdmin", "true");
        localStorage.setItem("adminCiRuc", cedula);
        window.location.href = "/app/dashboard"; // <--- aquí el cambio
      } else {
        alert("❌ Usuario, contraseña o rol incorrectos.");
        contrasena = '';
      }
    } catch (err) {
      alert("❌ Error al autenticar. Verifique sus credenciales o conexión.");
      contrasena = '';
    }
  }
</script>

<svelte:head>
  <title>Login Administrador</title>
  <!-- Bootstrap CSS y Bootstrap Icons -->
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" />
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.5/font/bootstrap-icons.css" />
  <!-- Fuente Raleway -->
  <link href="https://fonts.googleapis.com/css2?family=Raleway:wght@400;600&display=swap" rel="stylesheet" />
</svelte:head>

<div class="login-bg">
  <div class="login-card shadow-sm">
    <h3 class="text-center mb-4 fw-bold text-dark">
      <i class="bi bi-shield-lock-fill text-primary"></i> Login Administrador
    </h3>

    <form autocomplete="off" novalidate on:submit={validarYEnviar}>
      <div class="mb-3 input-group">
        <i class="bi bi-person-fill form-icon"></i>
        <input type="text" bind:value={cedula} class="form-control" placeholder="Cédula / RUC" maxlength="13" required aria-label="Cédula o RUC" />
      </div>

      <div class="mb-3 input-group">
        <i class="bi bi-key-fill form-icon"></i>
        <input type="password" bind:value={contrasena} class="form-control" placeholder="Contraseña" required aria-label="Contraseña" />
      </div>

      <button type="submit" class="btn btn-primary w-100 fw-bold mt-3">
        <i class="bi bi-box-arrow-in-right"></i> Entrar
      </button>
    </form>
  </div>
</div>

<style>
  :global(body), :global(html) {
    height: 100%;
    margin: 0;
    background-color: #2c2f36;
    font-family: 'Raleway', sans-serif;
    display: flex;
    justify-content: center;
    align-items: center;
    color: #f0db7d;
  }

  .login-bg {
    min-height: 100vh;
    width: 100vw;
    display: flex;
    align-items: center;
    justify-content: center;
    background: #2c2f36;
  }

  .login-card {
    background: whitesmoke;
    padding: 2rem;
    border-radius: 1rem;
    max-width: 400px;
    width: 100%;
    box-shadow: 0 6px 20px rgba(0,0,0,0.2);
    color: #f0db7d;
  }

  .form-icon {
    position: absolute;
    left: 12px;
    top: 50%;
    transform: translateY(-50%);
    color: #c0c0c0;
  }

  .input-group {
    position: relative;
  }

  .input-group input {
    padding-left: 2.5rem;
    background-color: whitesmoke;
    color: black;
    border: 1px solid #666;
  }

  .input-group input::placeholder {
    color: #cccccc;
  }

  .input-group input:focus {
    border-color: #d4af37;
    box-shadow: 0 0 5px #d4af37;
    outline: none;
  }

  .btn-primary {
    background-color: #d4af37;
    border: none;
    color: #2c2f36;
    font-weight: 700;
  }

  .btn-primary:hover,
  .btn-primary:focus {
    background-color: #f0db7d;
    border-color: #f0db7d;
    color: #2c2f36;
    box-shadow: 0 0 0 0.25rem rgba(244, 208, 63, 0.4);
  }
</style>

