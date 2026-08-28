
<template>

  <link rel="stylesheet" href="./style.css">

  <div class="pag">

    <header class="encabezado">

      <div class="fila">

        <div class="img">
          <img src="./assets/logo.png" alt="">
        </div>

        <div class="texto">
          <h1>Barberia Don Ramiro</h1>
          <p>Registro diario de servicios</p>
        </div>

      </div>

      <div class="divisor"></div>

    </header>

    <main>

      <div
        v-if="servicios.length === 0"
        class="estado-vacio"
      >
        <p>Toca el botón + para agregar el primer servicio.</p>
      </div>

      <div
        class="cuadricula"
        v-else
      >

        <div
          class="tarjeta"
          v-for="servicio in servicios"
          :key="servicio.id"
        >

          <div class="tarjeta-encabezado">

            <div>
              <p class="cliente">{{ servicio.cliente }}</p>
              <p class="etiqueta-servicio">{{ servicio.tipo }}</p>
            </div>

            <div class="precio mono">
              {{ formatoDinero(servicio.precio) }}
            </div>

          </div>

          <div class="fila-meta">

            <span>💈 {{ servicio.barbero }}</span>
            <span>🕒 {{ servicio.fecha }}</span>

          </div>

          <div class="tarjeta-acciones">

            <button
              class="boton boton-eliminar"
              @click="eliminarServicio(servicio.id)"
            >
              Eliminar
            </button>

          </div>

        </div>

      </div>

    </main>

    <button
      class="boton-flotante"
      @click="mostrarFormulario = true"
    >
      +
    </button>

    <div
      class="superposicion"
      v-if="mostrarFormulario"
      @click.self="cerrarFormulario"
    >

      <div class="modal">

        <h2>Nuevo servicio</h2>

        <p class="subtitulo">
          Completa los datos del corte realizado.
        </p>

        <form @submit.prevent="agregarServicio">

          <div class="campo">

            <label>Nombre del cliente</label>

            <input
              type="text"
              v-model="formulario.cliente"
              placeholder="Ej: Carlos Pérez"
            >

          </div>

          <div class="campo">

            <label>Tipo de servicio</label>

            <select v-model="formulario.tipo">

              <option value="">Seleccionar...</option>

              <option
                v-for="tipo in tiposServicio"
                :key="tipo"
                :value="tipo"
              >
                {{ tipo }}
              </option>

            </select>

          </div>

          <div class="campo">

            <label>Barbero</label>

            <select v-model="formulario.barbero">

              <option value="">Seleccionar...</option>

              <option
                v-for="barbero in barberos"
                :key="barbero"
                :value="barbero"
              >
                {{ barbero }}
              </option>

            </select>

          </div>

          <div class="campo">

            <label>Precio</label>

            <input
              type="number"
              v-model.number="formulario.precio"
              placeholder="25000"
              min="0"
            >

          </div>

          <div class="modal-acciones">

            <button
              type="button"
              class="boton boton-cancelar"
              @click="cerrarFormulario"
            >
              Cancelar
            </button>

            <button
              type="submit"
              class="boton boton-guardar"
            >
              Guardar servicio
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

const barberos = [
  'Don Ramiro',
  'Julian',
  'Kevin'
]

const tiposServicio = [
  'Corte',
  'Barba',
  'Cejas',
  'Tinte',
  'Corte niño',
  'Retoque'
]

const servicios = ref(
  JSON.parse(
    localStorage.getItem('barberia') || '[]'
  )
)

const mostrarFormulario = ref(false)

const formulario = ref({
  cliente: '',
  tipo: '',
  barbero: '',
  precio: null
})

function agregarServicio() {

  if (
    !formulario.value.cliente ||
    !formulario.value.tipo ||
    !formulario.value.barbero ||
    !formulario.value.precio
  ) {
    alert('Completa todos los campos')
    return
  }

  const nuevoServicio = {
    id: Date.now(),
    cliente: formulario.value.cliente,
    tipo: formulario.value.tipo,
    barbero: formulario.value.barbero,
    precio: formulario.value.precio,
    fecha: new Date().toLocaleString('es-CO')
  }

  servicios.value.push(nuevoServicio)

  guardarLocalStorage()

  formulario.value = {
    cliente: '',
    tipo: '',
    barbero: '',
    precio: null
  }

  mostrarFormulario.value = false
}

function eliminarServicio(id) {

  servicios.value = servicios.value.filter(
    servicio => servicio.id !== id
  )

  guardarLocalStorage()
}

function guardarLocalStorage() {

  localStorage.setItem(
    'barberia',
    JSON.stringify(servicios.value)
  )

}

function cerrarFormulario() {

  mostrarFormulario.value = false

}

function formatoDinero(valor) {

  return '$' +
    Number(valor).toLocaleString('es-CO')

}

</script>
