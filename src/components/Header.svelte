<script>
  import { onMount } from 'svelte';

  let showMobileMenu = false;

  function toggleMenu() {
    showMobileMenu = !showMobileMenu;
  }

  function cerrarSesion() {
    localStorage.removeItem("isAdmin");
    localStorage.removeItem("adminCiRuc");
    window.location.href = "/";
  }

  // Cierra el menú si se hace clic fuera en móvil
  onMount(() => {
    function handleClick(e) {
      const menu = document.getElementById('mobileMenu');
      const toggle = document.getElementById('menuToggle');
      if (menu && !menu.contains(e.target) && toggle && !toggle.contains(e.target)) {
        showMobileMenu = false;
      }
    }
    document.addEventListener('mousedown', handleClick);
    return () => document.removeEventListener('mousedown', handleClick);
  });
</script>

<header class="header-placeholder">
  <div class="logo">
    <a href="/app/dashboard" class="logo-link" aria-label="Ir a la página principal">
      <img src="/imagenes/logo.png" alt="Logo Trago Loco" class="logo-img" />
      <span class="panel-title">Panel Admin</span>
    </a>
  </div>
  <div class="acciones-header">
    <button id="menuToggle" class="menu-toggle" aria-label="Mostrar menú" on:click={toggleMenu}>
      <span class="menu-icon-lines">
        <span></span>
        <span></span>
        <span></span>
      </span>
    </button>
    <button class="logout-btn" on:click={cerrarSesion}>🔒 Cerrar Sesión</button>
  </div>
  {#if showMobileMenu}
    <div id="mobileMenu" class="mobile-menu">
      <button class="logout-btn-mobile" on:click={cerrarSesion}>🔒 Cerrar Sesión</button>
    </div>
  {/if}
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
    position: relative;
    z-index: 100;
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

  .acciones-header {
    display: flex;
    align-items: center;
    gap: 1rem;
  }

  .menu-toggle {
    background: none;
    border: 2px solid #f0db7d;
    border-radius: 0.7rem;
    color: #f0db7d;
    cursor: pointer;
    width: 44px;
    height: 44px;
    display: none;
    align-items: center;
    justify-content: center;
    position: relative;
    transition: box-shadow 0.2s, background 0.2s;
    padding: 0;
  }
  .menu-toggle:focus,
  .menu-toggle:hover {
    box-shadow: 0 0 8px #f0db7d88;
    background: #35373c;
  }

  .menu-icon-lines {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    width: 28px;
    height: 28px;
    gap: 5px;
  }
  .menu-icon-lines span {
    display: block;
    width: 100%;
    height: 3.5px;
    background: #f0db7d;
    border-radius: 2px;
    transition: background 0.2s;
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

  .logout-btn-mobile {
    background: #23252b;
    border: 2px solid #f0db7d;
    color: #f0db7d;
    cursor: pointer;
    font-size: 1.15rem;
    font-weight: 700;
    border-radius: 1.5rem;
    padding: 14px 0;
    width: 180px;
    margin: 0 auto;
    display: block;
    margin-bottom: 8px;
    transition: all 0.2s;
    text-align: center;
    box-shadow: 0 2px 12px #00000033;
    letter-spacing: 0.5px;
  }
  .logout-btn-mobile:hover {
    background: #f0db7d;
    color: #23252b;
    border-color: #f0db7d;
    box-shadow: 0 0 8px #f0db7d88;
  }

  .mobile-menu {
    display: none;
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
    .acciones-header {
      width: 100%;
      justify-content: flex-end;
    }
    .menu-toggle {
      display: flex;
    }
    .logout-btn {
      display: none;
    }
    .mobile-menu {
      display: flex;
      flex-direction: column;
      align-items: center;
      position: absolute;
      top: 60px;
      right: 0;
      left: 0;
      margin: 0 auto;
      background: #23252b;
      border-radius: 1rem;
      box-shadow: 0 4px 24px #00000055;
      padding: 24px 0 16px 0;
      z-index: 200;
      animation: fadeInMenu 0.2s;
      border: 2px solid #f0db7d;
      width: 90vw;
      max-width: 320px;
    }
    @keyframes fadeInMenu {
      from { opacity: 0; transform: translateY(-10px);}
      to { opacity: 1; transform: translateY(0);}
    }
  }
</style>