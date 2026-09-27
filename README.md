<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Plataforma Comercial CM</title>
  <script src="https://js.stripe.com/v3/"></script>
  <style>
    :root {
      --bg-main: #121214;
      --card-bg: #202024;
      --text-color: #e1e1e6;
      --accent-color: #FFC107;
      --accent-text: #121214;
      --border-color: #323238;
      --overlay-bg: rgba(0, 0, 0, 0.88);
      --error-color: #ff5c5c;
    }

    * { box-sizing: border-box; margin: 0; padding: 0; font-family: 'Segoe UI', sans-serif; }
    body { background-color: var(--bg-main); color: var(--text-color); min-height: 100vh; display: flex; flex-direction: column; }

    /* ENCABEZADO */
    header {
      background-color: rgba(32, 32, 36, 0.95);
      border-bottom: 1px solid var(--border-color);
      padding: 0.8rem 2rem;
      display: flex;
      justify-content: space-between;
      align-items: center;
      gap: 15px;
      flex-wrap: wrap;
      position: sticky; top: 0; z-index: 100;
    }

    .brand-container { display: flex; align-items: center; gap: 12px; }
    .header-logo-svg { height: 45px; width: auto; border-radius: 8px; }
    .search-input { width: 100%; max-width: 400px; padding: 8px 16px; border-radius: 20px; border: 1px solid var(--border-color); background: #121214; color: #fff; }

    /* BOTONES */
    .btn-primario { background: var(--accent-color); color: var(--accent-text); border: none; padding: 10px 18px; border-radius: 6px; font-weight: bold; cursor: pointer; }
    .btn-primario:disabled { opacity: 0.6; cursor: not-allowed; }
    .btn-secundario { background: transparent; border: 1px solid var(--border-color); color: var(--text-color); padding: 9px 16px; border-radius: 6px; cursor: pointer; }

    /* MODALES Y FORMULARIOS */
    .modal { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: var(--overlay-bg); display: flex; justify-content: center; align-items: center; z-index: 500; }
    .modal-contenido { background: var(--card-bg); padding: 2rem; border-radius: 10px; width: 100%; max-width: 450px; border: 1px solid var(--border-color); }
    .campo { margin-bottom: 1rem; display: flex; flex-direction: column; gap: 6px; }
    .campo input, .campo select, .campo textarea { background: #121214; border: 1px solid var(--border-color); color: #fff; padding: 10px; border-radius: 6px; }

    .mensaje-error { background: rgba(255,92,92,0.12); border: 1px solid var(--error-color); color: var(--error-color); padding: 8px 12px; border-radius: 6px; font-size: 0.85rem; margin-bottom: 12px; display: none; }
    .toggle-auth { font-size: 0.8rem; color: #a8a8b3; margin-top: 10px; text-align: center; cursor: pointer; }

    /* GRID PRODUCTOS */
    main { padding: 2rem; max-width: 1200px; margin: 0 auto; width: 100%; }
    .grid-productos { display: grid; grid-template-columns: repeat(auto-fill, minmax(280px, 1fr)); gap: 1.5rem; margin-top: 1.5rem; }
    .tarjeta-producto { background: var(--card-bg); border: 1px solid var(--border-color); border-radius: 10px; padding: 1rem; display: flex; flex-direction: column; gap: 10px; }
    .img-producto { width: 100%; height: 180px; object-fit: cover; border-radius: 6px; background: #121214; }
    .badge-categoria { align-self: flex-start; background: #121214; border: 1px solid var(--border-color); color: #a8a8b3; font-size: 0.7rem; padding: 3px 8px; border-radius: 12px; }

    .desglose-precio { background: #121214; padding: 10px; border-radius: 6px; font-size: 0.85rem; margin-bottom: 12px; }
    .desglose-item { display: flex; justify-content: space-between; margin-bottom: 4px; }
    .vacio { text-align: center; color: #6b6b76; padding: 3rem 0; grid-column: 1 / -1; }
  </style>
</head>
<body>

  <!-- ENCABEZADO -->
  <header>
    <div class="brand-container">
      <svg class="header-logo-svg" viewBox="0 0 300 300">
        <rect width="300" height="300" rx="40" fill="#FFC107" />
        <path d="M115 65 h50 l12 35 h-58 z" fill="none" stroke="#121214" stroke-width="9" stroke-linejoin="round"/>
        <path d="M100 50 h18 l10 25" fill="none" stroke="#121214" stroke-width="9" stroke-linecap="round"/>
        <circle cx="127" cy="115" r="7.5" fill="#121214"/>
        <circle cx="167" cy="115" r="7.5" fill="#121214"/>
        <text x="150" y="235" font-family="sans-serif" font-size="125" font-weight="900" fill="#121214" text-anchor="middle">CM</text>
      </svg>
      <h2>Marketplace</h2>
    </div>

    <input type="text" class="search-input" id="buscador" placeholder="🔍 Buscar productos..." oninput="buscarProductos()">

    <div>
      <button class="btn-secundario" id="btnUser" onclick="abrirModal('modalAuth')">Iniciar Sesión</button>
      <button class="btn-primario" id="btnVender" onclick="abrirModal('modalVender')" style="display:none;">+ Publicar</button>
    </div>
  </header>

  <main>
    <h2>Catálogo General</h2>
    <div id="contenedorProductos" class="grid-productos"></div>
  </main>

  <!-- MODAL: INICIO DE SESIÓN / REGISTRO -->
  <div id="modalAuth" class="modal" style="display:none;">
    <div class="modal-contenido">
      <span style="float:right; cursor:pointer;" onclick="cerrarModal('modalAuth')">&times;</span>
      <h3 id="tituloAuth">Iniciar Sesión</h3>
      <div class="mensaje-error" id="errorAuth"></div>
      <form onsubmit="enviarAuth(event)">
        <div class="campo">
          <label>Correo Electrónico:</label>
          <input type="email" id="emailUser" placeholder="tu@correo.com" required>
        </div>
        <div class="campo">
          <label>Contraseña:</label>
          <input type="password" id="passUser" minlength="6" required>
        </div>
        <div class="campo" id="campoTipoCuenta" style="display:none;">
          <label>Tipo de Cuenta:</label>
          <select id="tipoCuenta">
            <option value="comprador">Comprador</option>
            <option value="vendedor">Vendedor / Tienda</option>
          </select>
        </div>
        <button type="submit" class="btn-primario" style="width: 100%;" id="btnSubmitAuth">Ingresar</button>
      </form>
      <div class="toggle-auth" onclick="alternarModoAuth()" id="toggleAuthTexto">¿No tienes cuenta? Regístrate</div>
    </div>
  </div>

  <!-- MODAL: VERIFICACIÓN DE CÉDULA Y EDAD -->
  <div id="modalVerificar" class="modal" style="display:none;">
    <div class="modal-contenido">
      <h3>Verificación de Identidad</h3>
      <p style="font-size:0.85rem; color:#a8a8b3; margin: 10px 0;">Debes ser mayor de edad para operar en la plataforma.</p>
      <form onsubmit="guardarVerificacion(event)">
        <div class="campo">
          <label>Número de Cédula / DNI:</label>
          <input type="text" id="numCedula" placeholder="10 dígitos" required pattern="[0-9]{10}">
        </div>
        <div class="campo" style="flex-direction:row; align-items:center; gap:8px;">
          <input type="checkbox" id="checkEdad" required>
          <label for="checkEdad" style="font-size:0.85rem;">Confirmo que tengo 18 años o más.</label>
        </div>
        <button type="submit" class="btn-primario" style="width: 100%;">Confirmar e Ingresar</button>
      </form>
    </div>
  </div>

  <!-- MODAL: PUBLICAR PRODUCTO -->
  <div id="modalVender" class="modal" style="display:none;">
    <div class="modal-contenido">
      <span style="float:right; cursor:pointer;" onclick="cerrarModal('modalVender')">&times;</span>
      <h3>Publicar Producto</h3>
      <div class="mensaje-error" id="errorVender"></div>
      <form onsubmit="publicarProducto(event)">
        <div class="campo">
          <label>Título:</label>
          <input type="text" id="tituloProd" maxlength="80" required>
        </div>
        <div class="campo">
          <label>Categoría:</label>
          <input type="text" id="categoriaProd" maxlength="40" placeholder="Electrónica, Hogar, Ropa...">
        </div>
        <div class="campo">
          <label>Precio de Venta ($ USD):</label>
          <input type="number" id="precioProd" step="0.01" min="0.01" required placeholder="0.00">
          <small style="color:#FFC107;">* La plataforma retendrá un 5% de comisión sobre cada venta.</small>
        </div>
        <div class="campo">
          <label>Foto (Galería):</label>
          <input type="file" id="fotoProd" accept="image/*" required>
          <small style="color:#a8a8b3;">Máximo 2MB por imagen.</small>
        </div>
        <button type="submit" class="btn-primario" style="width:100%;">Publicar</button>
      </form>
    </div>
  </div>

  <!-- MODAL: PAGO CON COMISIÓN DEL 5% -->
  <div id="modalPago" class="modal" style="display:none;">
    <div class="modal-contenido">
      <span style="float:right; cursor:pointer;" onclick="cerrarModal('modalPago')">&times;</span>
      <h3>Detalle de la Compra</h3>

      <div class="desglose-precio" id="desgloseContenedor"></div>

      <button class="btn-primario" style="width:100%;" onclick="confirmarPagoFinal()">Pagar Ahora con Tarjeta</button>
    </div>
  </div>

  <script>
    /*
      NOTA IMPORTANTE DE SEGURIDAD:
      Esta versión sigue guardando todo en localStorage, es decir, SOLO EN ESTE NAVEGADOR.
      Cada persona que abra esta página en su propio navegador tiene su propio catálogo,
      sus propios usuarios y sus propias ventas — no se comparte nada entre visitantes.
      El "hash" de contraseña de aquí abajo es una ofuscación simple, NO es seguridad real.
      Para un catálogo compartido de verdad y contraseñas realmente protegidas, se necesita
      un servidor (backend) — si more adelante quieres eso, puedo dártelo por separado.
    */

    let usuarioActual = null;
    let productoSeleccionado = null;
    let modoRegistro = false;
    const COMISION_PORCENTAJE = 0.05; // 5% de comisión
    const TAM_MAX_IMAGEN = 2 * 1024 * 1024; // 2MB

    const productosBase = [
      { id: 1, titulo: 'Monitor 22 Pulgadas Full HD', precio: 120.00, categoria: 'Electrónica', vendedor: 'TechStore', imagen: 'https://images.unsplash.com/photo-1527443224154-c4a3942d3acf?w=400' }
    ];

    // Evita que texto de usuario (título, vendedor, categoría) inyecte HTML/JS
    function escaparHTML(texto) {
      const div = document.createElement('div');
      div.textContent = String(texto ?? '');
      return div.innerHTML;
    }

    // Hash simple (NO criptográfico) solo para no guardar contraseñas en texto plano.
    // Para seguridad real se necesita un backend con bcrypt.
    function hashSimple(texto) {
      let hash = 0;
      for (let i = 0; i < texto.length; i++) {
        hash = (hash << 5) - hash + texto.charCodeAt(i);
        hash |= 0;
      }
      return String(hash);
    }

    function leerUsuarios() { return JSON.parse(localStorage.getItem('usuarios_cm')) || []; }
    function guardarUsuarios(lista) { localStorage.setItem('usuarios_cm', JSON.stringify(lista)); }

    window.onload = () => {
      if (!localStorage.getItem('prods_cm')) {
        localStorage.setItem('prods_cm', JSON.stringify(productosBase));
      }

      // Restaurar sesión si ya había una activa (antes se perdía al recargar)
      const sesionGuardada = JSON.parse(localStorage.getItem('sesion_cm') || 'null');
      if (sesionGuardada) {
        const usuarios = leerUsuarios();
        const existe = usuarios.find(u => u.email === sesionGuardada.email);
        if (existe) actualizarUIUsuario(existe);
      }

      renderizar();
    };

    function actualizarUIUsuario(usuario) {
      usuarioActual = usuario;
      localStorage.setItem('sesion_cm', JSON.stringify({ email: usuario.email }));
      document.getElementById('btnUser').innerText = usuario.email;
      document.getElementById('btnUser').onclick = cerrarSesion;
      document.getElementById('btnVender').style.display = usuario.tipo === 'vendedor' ? 'inline-block' : 'none';

      // La verificación de edad ahora queda ligada a ESTA cuenta, no al navegador entero
      if (!usuario.verificado) {
        document.getElementById('modalVerificar').style.display = 'flex';
      }
    }

    function cerrarSesion() {
      usuarioActual = null;
      localStorage.removeItem('sesion_cm');
      document.getElementById('btnUser').innerText = 'Iniciar Sesión';
      document.getElementById('btnUser').onclick = () => abrirModal('modalAuth');
      document.getElementById('btnVender').style.display = 'none';
    }

    function alternarModoAuth() {
      modoRegistro = !modoRegistro;
      document.getElementById('tituloAuth').innerText = modoRegistro ? 'Crear Cuenta' : 'Iniciar Sesión';
      document.getElementById('btnSubmitAuth').innerText = modoRegistro ? 'Registrarme' : 'Ingresar';
      document.getElementById('campoTipoCuenta').style.display = modoRegistro ? 'flex' : 'none';
      document.getElementById('toggleAuthTexto').innerText = modoRegistro ? '¿Ya tienes cuenta? Inicia sesión' : '¿No tienes cuenta? Regístrate';
      document.getElementById('errorAuth').style.display = 'none';
    }

    function enviarAuth(e) {
      e.preventDefault();
      const email = document.getElementById('emailUser').value.trim().toLowerCase();
      const password = document.getElementById('passUser').value;
      const tipo = document.getElementById('tipoCuenta').value;
      const errorBox = document.getElementById('errorAuth');
      errorBox.style.display = 'none';

      const usuarios = leerUsuarios();
      const existente = usuarios.find(u => u.email === email);

      if (modoRegistro) {
        if (existente) {
          errorBox.innerText = 'Ese correo ya está registrado. Intenta iniciar sesión.';
          errorBox.style.display = 'block';
          return;
        }
        const nuevo = { email, passHash: hashSimple(password), tipo, verificado: false, cedula: null };
        usuarios.push(nuevo);
        guardarUsuarios(usuarios);
        actualizarUIUsuario(nuevo);
      } else {
        if (!existente || existente.passHash !== hashSimple(password)) {
          errorBox.innerText = 'Correo o contraseña incorrectos.';
          errorBox.style.display = 'block';
          return;
        }
        actualizarUIUsuario(existente);
      }
      cerrarModal('modalAuth');
      e.target.reset();
    }

    function guardarVerificacion(e) {
      e.preventDefault();
      const cedula = document.getElementById('numCedula').value;
      const usuarios = leerUsuarios();
      const usuario = usuarios.find(u => u.email === usuarioActual.email);
      usuario.cedula = cedula;
      usuario.verificado = true;
      guardarUsuarios(usuarios);
      usuarioActual.verificado = true;
      cerrarModal('modalVerificar');
    }

    function publicarProducto(e) {
      e.preventDefault();
      const titulo = document.getElementById('tituloProd').value.trim();
      const categoria = document.getElementById('categoriaProd').value.trim() || 'General';
      const precio = parseFloat(document.getElementById('precioProd').value);
      const file = document.getElementById('fotoProd').files[0];
      const errorBox = document.getElementById('errorVender');
      errorBox.style.display = 'none';

      if (!file) {
        errorBox.innerText = 'Selecciona una foto.';
        errorBox.style.display = 'block';
        return;
      }
      if (file.size > TAM_MAX_IMAGEN) {
        errorBox.innerText = 'La imagen pesa demasiado (máximo 2MB). Elige una más liviana.';
        errorBox.style.display = 'block';
        return;
      }

      const reader = new FileReader();
      reader.onload = (event) => {
        const lista = JSON.parse(localStorage.getItem('prods_cm')) || [];
        lista.unshift({
          id: Date.now(),
          titulo,
          categoria,
          precio,
          vendedor: usuarioActual.email,
          imagen: event.target.result
        });
        try {
          localStorage.setItem('prods_cm', JSON.stringify(lista));
        } catch (err) {
          errorBox.innerText = 'No se pudo guardar: se acabó el espacio disponible en este navegador.';
          errorBox.style.display = 'block';
          return;
        }
        cerrarModal('modalVender');
        e.target.reset();
        renderizar();
      };
      reader.readAsDataURL(file);
    }

    function abrirPago(id) {
      if (!usuarioActual) { abrirModal('modalAuth'); return; }
      const lista = JSON.parse(localStorage.getItem('prods_cm'));
      productoSeleccionado = lista.find(p => p.id === id);

      const precioBase = productoSeleccionado.precio;
      const comision = precioBase * COMISION_PORCENTAJE;
      const pagoVendedor = precioBase - comision;

      document.getElementById('desgloseContenedor').innerHTML = `
        <div class="desglose-item"><span>Producto:</span> <strong>${escaparHTML(productoSeleccionado.titulo)}</strong></div>
        <div class="desglose-item"><span>Precio Total:</span> <strong>$${precioBase.toFixed(2)} USD</strong></div>
        <hr style="border-color:var(--border-color); margin:8px 0;">
        <div class="desglose-item" style="color:#FFC107;"><span>Comisión Plataforma (5%):</span> <span>$${comision.toFixed(2)} USD</span></div>
        <div class="desglose-item" style="color:#a8a8b3;"><span>Ganancia Vendedor:</span> <span>$${pagoVendedor.toFixed(2)} USD</span></div>
      `;

      abrirModal('modalPago');
    }

    function confirmarPagoFinal() {
      const comision = productoSeleccionado.precio * COMISION_PORCENTAJE;
      const paraVendedor = productoSeleccionado.precio - comision;

      alert(`¡Pago Exitoso!\n\n- Pago al Vendedor (${productoSeleccionado.vendedor}): $${paraVendedor.toFixed(2)} USD\n- Comisión retenida para ti (5%): $${comision.toFixed(2)} USD`);
      cerrarModal('modalPago');
    }

    function renderizar(filtro = '') {
      const contenedor = document.getElementById('contenedorProductos');
      const lista = JSON.parse(localStorage.getItem('prods_cm')) || [];

      const filtrados = lista.filter(p => p.titulo.toLowerCase().includes(filtro.toLowerCase()));

      if (filtrados.length === 0) {
        contenedor.innerHTML = `<div class="vacio">No se encontraron productos.</div>`;
        return;
      }

      contenedor.innerHTML = filtrados.map(p => `
        <div class="tarjeta-producto">
          <img src="${p.imagen}" class="img-producto" loading="lazy">
          <span class="badge-categoria">${escaparHTML(p.categoria || 'General')}</span>
          <h3>${escaparHTML(p.titulo)}</h3>
          <p style="color:#a8a8b3; font-size:0.8rem;">Vendedor: ${escaparHTML(p.vendedor)}</p>
          <div style="font-size:1.2rem; font-weight:bold; color:var(--accent-color);">$${p.precio.toFixed(2)} USD</div>
          <button class="btn-primario" onclick="abrirPago(${p.id})">Comprar</button>
        </div>
      `).join('');
    }

    let temporizadorBusqueda;
    function buscarProductos() {
      clearTimeout(temporizadorBusqueda);
      temporizadorBusqueda = setTimeout(() => renderizar(document.getElementById('buscador').value), 250);
    }

    function abrirModal(id) { document.getElementById(id).style.display = 'flex'; }
    function cerrarModal(id) { document.getElementById(id).style.display = 'none'; }
  </script>
</body>
</html>
