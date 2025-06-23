<script>
  function toggleMenu() {
    const menu = document.getElementById('mobileMenu');
    if (menu) {
      menu.style.display = menu.style.display === 'none' ? 'block' : 'none';
    }
  }

  function cerrarSesion() {
    localStorage.removeItem("isAdmin");
    localStorage.removeItem("adminCiRuc");
    window.location.href = "/";
  }
</script>

<header class="header-placeholder">
  <div class="logo">
    <a href="/app/dashboard" class="logo-link" aria-label="Ir a la página principal">
      <img src="/imagenes/logo.png" alt="Logo Trago Loco" class="logo-img" />
      <span class="panel-title">Panel Admin</span>
    </a>
  </div>
  <div class="text-end">
    <button class="menu-toggle" aria-label="Mostrar menú" on:click={toggleMenu}>☰</button>
    <button class="logout-btn" on:click={cerrarSesion}>🔒 Cerrar Sesión</button>
  </div>
  <div id="mobileMenu" class="mobile-menu">
    <button class="logout-btn" on:click={cerrarSesion}>🔒 Cerrar Sesión</button>
  </div>
</header>

<style>
  .header-placeholder {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 14px 32px;
    background-color: #484b52;
    color: #f0db7d;
    border-radius: 0 0 2rem 2rem;
    box-shadow: 0 2px 8px #00000044;
    margin-bottom: 1.5rem;
  }

  .logo-link {
    display: flex;
    align-items: center;
    gap: 0.7rem;
    text-decoration: none;
  }

  .logo-img {
    height: 40px;
    margin-right: 0.5rem;
  }

  .panel-title {
    font-family: 'Raleway', Arial, sans-serif;
    font-size: 2rem;
    font-weight: 700;
    color: #f0db7d;
    letter-spacing: 1px;
    text-shadow: 0 1px 2px #00000055;
  }

  .text-end {
    display: flex;
    align-items: center;
    gap: 1rem;
  }

  .logout-btn {
    background: none;
    border: 2px solid #f0db7d;
    color: #f0db7d;
    cursor: pointer;
    font-size: 1.1rem;
    font-weight: 600;
    border-radius: 1.5rem;
    padding: 6px 18px;
    transition: all 0.2s;
    margin-left: 0.5rem;
  }
  .logout-btn:hover {
    background: #f0db7d;
    color: #484b52;
    border-color: #f0db7d;
    box-shadow: 0 0 8px #f0db7d88;
  }

  .menu-toggle {
    background: none;
    border: none;
    color: #f0db7d;
    cursor: pointer;
    font-size: 1.7rem;
    display: none;
  }

  .mobile-menu {
    display: none;
    position: absolute;
    top: 60px;
    right: 20px;
    background: #222;
    border-radius: 8px;
    box-shadow: 0 2px 8px #d4af37bb;
    padding: 10px 20px;
    z-index: 20;
  }

  @media (max-width: 600px) {
    .header-placeholder {
      flex-direction: column;
      align-items: stretch;
      padding: 10px 8px;
      border-radius: 0 0 1.2rem 1.2rem;
    }
    .panel-title {
      font-size: 1.3rem;
    }
    .menu-toggle {
      display: inline-block;
    }
    .logout-btn {
      display: none;
    }
    .mobile-menu {
      display: block;
    }
  }
</style>