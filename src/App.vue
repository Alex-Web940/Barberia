<template>
  <div class="pag">

    <header class="encabezado">
      <div class="fila-encabezado">
        <div class="marca">

          <div class="icono-marca">
            <img src="./assets/logo.png" alt="Barbería" class="img-marca">
          </div>

          <div>
            <h1>Barbería Don Ramiro</h1>
            <p>Registro diario de servicios</p>
          </div>
        </div>
      </div>

      <div class="tira-resumen" v-if="servicios.length > 0">
        <div class="ticket">
          <span class="ticket-etiqueta">Hoy</span>
          <span class="ticket-valor mono">{{ formatoDinero(totalHoy()) }}</span>
        </div>
        <div class="ticket">
          <span class="ticket-etiqueta">Esta semana</span>
          <span class="ticket-valor mono">{{ formatoDinero(totalSemana()) }}</span>
        </div>
        <div class="ticket" :class="{ 'ticket-alerta': cantidadPendientes() > 0 }">
          <span class="ticket-etiqueta">Por cobrar</span>
          <span class="ticket-valor mono">{{ cantidadPendientes() }}</span>
        </div>
        <div class="ticket">
          <span class="ticket-etiqueta">Más pedido</span>
          <span class="ticket-valor">{{ servicioMasPedido() }}</span>
        </div>
      </div>
    </header>

    <div class="filtros" v-if="servicios.length > 0">
      <button class="chip" :class="{ activo: filtro === 'todos' }" @click="filtro = 'todos'">Todos</button>
      <button class="chip" :class="{ activo: filtro === 'pendientes' }" @click="filtro = 'pendientes'">Sin
        pagar</button>
      <button class="chip" :class="{ activo: filtro === 'pagados' }" @click="filtro = 'pagados'">Pagados</button>
    </div>

    <main>

      <div v-if="servicios.length === 0" class="estado-vacio">
        <span class="material-symbols-outlined estado-vacio-icono">content_cut</span>
        <h3>Aún no hay servicios registrados</h3>
        <p>Toca el botón + para registrar el primer corte del día.</p>
      </div>

      <div v-else-if="serviciosFiltrados().length === 0" class="estado-vacio">
        <span class="material-symbols-outlined estado-vacio-icono">manage_search</span>
        <h3>No hay servicios en este filtro</h3>
        <p>Prueba con otra categoría.</p>
      </div>

      <div class="lista" v-else>

        <div class="tarjeta" :class="claseTarjeta(servicio)" v-for="servicio in serviciosFiltrados()"
          :key="servicio.id">

          <div class="tarjeta-cuerpo">

            <div class="tarjeta-encabezado">
              <div>
                <p class="cliente">{{ servicio.cliente }}</p>
                <p class="etiqueta-servicio">{{ servicio.tipos.join(' + ') }}</p>
              </div>
              <div class="precio mono">{{ formatoDinero(servicio.precio) }}</div>
            </div>

            <div class="fila-meta">
              <span class="meta-item">
                <span class="material-symbols-outlined meta-icono">content_cut</span>
                {{ servicio.barbero }}
              </span>
              <span class="meta-item">
                <span class="material-symbols-outlined meta-icono">schedule</span>
                {{ formatoFecha(servicio.fechaHora) }}
              </span>
              <span class="meta-item">
                <span class="material-symbols-outlined meta-icono">{{ iconoPago(servicio.metodoPago) }}</span>
                {{ servicio.metodoPago }}
              </span>
            </div>

            <div class="fila-meta">
              <span class="insignia" :class="'insignia-' + servicio.estadoPago">
                {{ etiquetaEstado(servicio.estadoPago) }}
              </span>
            </div>

            <div class="fila-calificacion">
              <span class="calificacion-etiqueta">Calificación:</span>
              <span class="estrellas-mostrar"
                :class="{ 'calificacion-baja': servicio.calificacion > 0 && servicio.calificacion <= 2 }">
                <span class="estrella-clic" v-for="n in 5" :key="n" :class="{ activa: n <= servicio.calificacion }"
                  @click="calificarServicio(servicio, n)">★</span>
              </span>
              <span v-if="servicio.calificacion === 0" class="sin-calificar">Sin calificar aún</span>
              <span v-else-if="servicio.calificacion <= 2" class="aviso-baja">Cliente insatisfecho</span>
            </div>

            <div class="observaciones-zona" v-if="editandoObservacionId !== servicio.id">
              <p class="observaciones" v-if="servicio.observaciones">
                <span class="material-symbols-outlined">
                  edit_note
                </span>
                {{ servicio.observaciones }}
              </p>
              <button class="enlace-observacion" @click="abrirEdicionObservacion(servicio.id)">
                <span class="material-symbols-outlined" v-if="servicio.observaciones"></span>
                <span class="material-symbols-outlined" v-else>add_circle</span>
                {{ servicio.observaciones ? 'Editar observación' : 'Agregar observación' }}
              </button>
            </div>

            <div class="editor-observacion" v-else>
              <textarea v-model="servicio.observaciones" rows="2"
                placeholder="Ej: Pidió el mismo estilo de la última vez"></textarea>
              <button class="boton boton-guardar boton-pequeno" @click="cerrarEdicionObservacion()">Guardar</button>
            </div>

          </div>

          <div class="tarjeta-acciones" v-if="confirmarId !== servicio.id">
            <button class="boton boton-editar" @click="editarServicio(servicio)">Editar</button>
            <button class="boton boton-eliminar" @click="pedirEliminar(servicio.id)">Eliminar</button>
          </div>

          <div class="tarjeta-acciones tarjeta-acciones-confirmar" v-else>
            <span class="texto-confirmar">¿Eliminar este registro?</span>
            <button class="boton boton-cancelar" @click="cancelarEliminar()">Cancelar</button>
            <button class="boton boton-eliminar" @click="confirmarEliminar(servicio.id)">Sí, eliminar</button>
          </div>

        </div>

      </div>

    </main>

    <button class="boton-flotante" :class="{ girado: mostrarFormulario }" @click="abrirFormulario()"
      aria-label="Agregar servicio">+</button>

    <div class="superposicion" v-if="mostrarFormulario">
      <div class="modal">

        <h2>{{ editandoId ? 'Editar servicio' : 'Nuevo servicio' }}</h2>
        <p class="subtitulo">Completa los datos del corte realizado.</p>

        <form @submit.prevent="guardarServicio">

          <div class="campo">
            <label>Nombre del cliente</label>
            <input type="text" v-model="formulario.cliente" placeholder="Ej: Carlos Pérez">
          </div>

          <div class="campo">
            <label>Tipo de servicio</label>
            <button type="button" class="selector-desplegable" @click="alternarTiposMenu()">
              <span class="selector-desplegable-texto" :class="{ vacio: formulario.tipos.length === 0 }">
                {{ resumenTipos() }}
              </span>
              <span class="selector-desplegable-flecha" :class="{ girada: mostrarTiposMenu }">

                <span class="material-symbols-outlined">expand_more</span>
              </span>
            </button>
            <div class="opciones-tipo" v-show="mostrarTiposMenu">
              <label class="opcion-tipo" v-for="tipo in tiposServicio" :key="tipo.nombre">
                <input type="checkbox" v-model="formulario.tipos" :value="tipo.nombre">
                <span>{{ tipo.nombre }}</span>
                <span class="mono opcion-tipo-precio">{{ formatoDinero(tipo.precio) }}</span>
              </label>
            </div>
          </div>

          <div class="campo">
            <label>Barbero</label>
            <select v-model="formulario.barbero">
              <option value="">Seleccionar...</option>
              <option v-for="barbero in barberos" :key="barbero" :value="barbero">{{ barbero }}</option>
            </select>
          </div>

          <div class="fila-campos">
            <div class="campo">
              <label>Fecha</label>
              <input type="date" v-model="formulario.fecha" :min="fechaMinima()">
            </div>
            <div class="campo">
              <label>Hora</label>
              <input type="time" v-model="formulario.hora" min="08:00" max="20:00">
            </div>
          </div>

          <div class="campo">
            <label>Precio total</label>
            <input type="text" :value="formatoDinero(calcularPrecio())" disabled class="mono campo-calculado">
          </div>

          <div class="campo">
            <label>Método de pago</label>
            <select v-model="formulario.metodoPago">
              <option value="">Seleccionar...</option>
              <option v-for="metodo in metodosPago" :key="metodo.valor" :value="metodo.valor">
                {{ metodo.etiqueta }}
              </option>
            </select>
          </div>

          <div class="campo">
            <label>Estado del pago</label>
            <select v-model="formulario.estadoPago">
              <option value="">Seleccionar...</option>
              <option value="pagado">Pagado</option>
              <option value="pendiente">Pendiente</option>
            </select>
          </div>

          <p class="mensaje-error" v-if="errorFormulario">{{ errorFormulario }}</p>

          <div class="modal-acciones">
            <button type="button" class="boton boton-cancelar" :disabled="guardando"
              @click="cerrarFormulario">Cancelar</button>
            <button type="submit" class="boton boton-guardar" :disabled="guardando">
              <span class="spinner" v-if="guardando"></span>
              {{ guardando ? 'Guardando...' : 'Guardar servicio' }}
            </button>
          </div>

        </form>

      </div>
    </div>

  </div>
</template>

<script setup>
import { ref } from 'vue'
import { useLocalStorage } from '@vueuse/core'

const barberos = ['Don Ramiro', 'Julián', 'Kevin']

const tiposServicio = [
  { nombre: 'Corte clásico', precio: 20000 },
  { nombre: 'Corte moderno', precio: 25000 },
  { nombre: 'Barba', precio: 15000 },
  { nombre: 'Cejas', precio: 8000 },
  { nombre: 'Tinte', precio: 40000 },
  { nombre: 'Corte niño', precio: 18000 },
  { nombre: 'Retoque', precio: 10000 }
]

const metodosPago = [
  { valor: 'efectivo', etiqueta: 'Efectivo' },
  { valor: 'transferencia', etiqueta: 'Transferencia' },
  { valor: 'tarjeta', etiqueta: 'Tarjeta' }
]

const servicios = useLocalStorage('barberia-servicios', [])

const mostrarFormulario = ref(false)
const editandoId = ref(null)
const errorFormulario = ref('')
const filtro = ref('todos')
const confirmarId = ref(null)
const mostrarTiposMenu = ref(false)
const editandoObservacionId = ref(null)
const guardando = ref(false)

function formularioVacio() {
  return {
    cliente: '',
    tipos: [],
    barbero: '',
    fecha: '',
    hora: '',
    metodoPago: '',
    estadoPago: ''
  }
}

const formulario = ref(formularioVacio())

function calcularPrecio() {
  return tiposServicio
    .filter(t => formulario.value.tipos.includes(t.nombre))
    .reduce((total, t) => total + t.precio, 0)
}

function resumenTipos() {
  if (formulario.value.tipos.length === 0) return 'Seleccionar servicios...'
  return formulario.value.tipos.join(' - ')
}

function alternarTiposMenu() {
  mostrarTiposMenu.value = !mostrarTiposMenu.value
}

function serviciosFiltrados() {
  let lista = [...servicios.value]
  if (filtro.value === 'pendientes') {
    lista = lista.filter(s => s.estadoPago !== 'pagado')
  } else if (filtro.value === 'pagados') {
    lista = lista.filter(s => s.estadoPago === 'pagado')
  }
  return lista.sort((a, b) => new Date(b.fechaHora) - new Date(a.fechaHora))
}

function esHoy(fechaHora) {
  const ahora = new Date()
  const año = ahora.getFullYear()
  const mes = String(ahora.getMonth() + 1).padStart(2, '0')
  const dia = String(ahora.getDate()).padStart(2, '0')
  const hoy = `${año}-${mes}-${dia}`
  return fechaHora.startsWith(hoy)
}

function fechaMinima() {
  const ahora = new Date()
  const año = ahora.getFullYear()
  const mes = String(ahora.getMonth() + 1).padStart(2, '0')
  const dia = String(ahora.getDate()).padStart(2, '0')
  return `${año}-${mes}-${dia}`
}

function totalHoy() {
  return servicios.value
    .filter(s => esHoy(s.fechaHora))
    .reduce((total, s) => total + s.precio, 0)
}

function inicioSemana() {
  const ahora = new Date()
  const dia = ahora.getDay()
  const diferencia = dia === 0 ? 6 : dia - 1
  const lunes = new Date(ahora)
  lunes.setDate(ahora.getDate() - diferencia)
  lunes.setHours(0, 0, 0, 0)
  return lunes
}

function totalSemana() {
  const lunes = inicioSemana()
  return servicios.value
    .filter(s => new Date(s.fechaHora) >= lunes)
    .reduce((total, s) => total + s.precio, 0)
}

function cantidadPendientes() {
  return servicios.value.filter(s => s.estadoPago !== 'pagado').length
}

function servicioMasPedido() {
  if (servicios.value.length === 0) return '—'
  const conteo = {}
  servicios.value.forEach(s => {
    s.tipos.forEach(t => {
      conteo[t] = (conteo[t] || 0) + 1
    })
  })
  let masPedido = '—'
  let maximo = 0
  for (const tipo in conteo) {
    if (conteo[tipo] > maximo) {
      maximo = conteo[tipo]
      masPedido = tipo
    }
  }
  return masPedido
}

function abrirFormulario() {
  editandoId.value = null
  errorFormulario.value = ''
  mostrarTiposMenu.value = false
  formulario.value = formularioVacio()
  mostrarFormulario.value = true
}

function editarServicio(servicio) {
  editandoId.value = servicio.id
  errorFormulario.value = ''
  mostrarTiposMenu.value = false

  const partes = servicio.fechaHora.split('T')

  formulario.value = {
    cliente: servicio.cliente,
    tipos: [...servicio.tipos],
    barbero: servicio.barbero,
    fecha: partes[0],
    hora: partes[1],
    metodoPago: servicio.metodoPago,
    estadoPago: servicio.estadoPago
  }

  mostrarFormulario.value = true
}

function cerrarFormulario() {
  mostrarFormulario.value = false
  errorFormulario.value = ''
  mostrarTiposMenu.value = false
}

function calificarServicio(servicio, n) {
  servicio.calificacion = n
}

function abrirEdicionObservacion(id) {
  editandoObservacionId.value = id
}

function cerrarEdicionObservacion() {
  editandoObservacionId.value = null
}

function validarFormulario() {
  const f = formulario.value
  if (!f.cliente || f.cliente.trim().length < 2) {
    return 'Escribe el nombre del cliente (mínimo 2 letras).'
  }
  if (f.tipos.length === 0) {
    return 'Selecciona al menos un tipo de servicio.'
  }
  if (!f.barbero) {
    return 'Selecciona quién atendió al cliente.'
  }
  if (!f.fecha) {
    return 'Selecciona la fecha del servicio.'
  }
  if (!f.fecha < fechaMinima()) {
    return 'no se puede agendar citas antes de hoy '
  }
  if (!f.hora) {
    return 'Selecciona la hora del servicio.'
  }

  if (f.hora < '08:00' || f.hora > '20.00') {
    return 'la hora debe de estar entre las 08:00 A.M y 8:00 P.M'
  }
  if (!f.metodoPago) {
    return 'Selecciona el método de pago.'
  }
  if (!f.estadoPago) {
    return 'Selecciona el estado del pago.'
  }
  return ''
}

function esperar(ms) {
  return new Promise(resolve => setTimeout(resolve, ms))
}

async function guardarServicio() {
  const error = validarFormulario()
  if (error) {
    errorFormulario.value = error
    return
  }
  errorFormulario.value = ''
  guardando.value = true

  await esperar(1200)

  const fechaHora = formulario.value.fecha + 'T' + formulario.value.hora

  const datos = {
    cliente: formulario.value.cliente.trim(),
    tipos: formulario.value.tipos,
    barbero: formulario.value.barbero,
    fechaHora: fechaHora,
    precio: calcularPrecio(),
    metodoPago: formulario.value.metodoPago,
    estadoPago: formulario.value.estadoPago
  }

  if (editandoId.value) {
    const posicion = servicios.value.findIndex(s => s.id === editandoId.value)
    const anterior = servicios.value[posicion]
    servicios.value[posicion] = {
      ...datos,
      id: editandoId.value,
      calificacion: anterior.calificacion,
      observaciones: anterior.observaciones
    }
  } else {
    servicios.value.push({
      ...datos,
      id: Date.now(),
      calificacion: 0,
      observaciones: ''
    })
  }

  guardando.value = false
  mostrarFormulario.value = false
}

function pedirEliminar(id) {
  confirmarId.value = id
}

function cancelarEliminar() {
  confirmarId.value = null
}

function confirmarEliminar(id) {
  servicios.value = servicios.value.filter(servicio => servicio.id !== id)
  confirmarId.value = null
}

function formatoDinero(valor) {
  return '$' + Number(valor).toLocaleString('es-CO')
}

function formatoFecha(fechaHora) {
  return new Date(fechaHora).toLocaleString('es-CO', {
    day: '2-digit',
    month: 'short',
    hour: '2-digit',
    minute: '2-digit'
  })
}

function iconoPago(metodo) {
  if (metodo === 'efectivo') return 'payments'
  if (metodo === 'transferencia') return 'account_balance'
  if (metodo === 'tarjeta') return 'credit_card'
  return 'help'
}

function etiquetaEstado(estado) {
  if (estado === 'pagado') return 'Pagado'
  if (estado === 'pendiente') return 'Pendiente'
  return estado
}

function claseTarjeta(servicio) {
  return {
    'tarjeta-pagado': servicio.estadoPago === 'pagado',
    'tarjeta-pendiente': servicio.estadoPago === 'pendiente'
  }
}
</script>
