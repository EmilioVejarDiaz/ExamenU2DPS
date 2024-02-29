<template>
    <div style="margin-left: auto">
      <div class="d-flex flex-column align-items-end">
        <div class="aside-container d-flex align-items-center" style="background-color: green;">
          <b-button style="background-color: green;" class="border-0" v-b-modal.modal-1>Agregar Libro</b-button>
        </div>
  
        <div class="aside-container d-flex align-items-center" style="background-color: blue;">
          <b-button style="background-color: blue;" class="border-0" v-b-modal.modal-1>Modificar Libro</b-button>
        </div>
  
        <div class="aside-container d-flex align-items-center" style="background-color: red;">
          <b-button style="background-color: red;" class="border-0" v-b-modal.modal-1>Eliminar Libro</b-button>
        </div>
      </div>
      <b-modal
        id="modal-2"
        title="Editar Libro"
        hide-footer
        @shown="clearErrors"
      >
        <template #modal-header="{ close }">
          <h5>Editar Libro</h5>
          <b-button size="sm" variant="outline-danger" @click="close()">
            X
          </b-button>
        </template>
        <form id="form" @submit.prevent="checkForm">
          <div v-if="errors.length">
            <b-alert show variant="danger">
              <b>Por favor, corrija el(los) siguiente(s) error(es):</b>
              <ul>
                <li v-for="error in errors" :key="error">{{ error }}</li>
              </ul>
            </b-alert>
          </div>
          
          
          <div class="fieldContainer">
            <div class="labelContainer">
              <label for="name">Nombre: </label>
            </div>
            <div class="inputContainer">
              <input id="name" v-model="name" type="text" name="name" />
            </div>
          </div>
  
          <div class="fieldContainer">
            <div class="labelContainer">
              <label for="autor">Autor: </label>
            </div>
            <div class="inputContainer">
              <input id="autor" v-model="autor" type="text" name="autor" />
            </div>
          </div>
  
          <div class="fieldContainer">
            <div class="labelContainer">
              <label for="fecha">Fecha de publicación: </label>
            </div>
            <div class="inputContainer">
              <input
                id="fecha"
                v-model="fecha"
                name="fecha"
                type="date"
              />
            </div>
          </div>
          <div class="buttonsContainer">
            <b-button variant="success" type="submit" id="botonEnviar">Enviar</b-button>
            <b-button type="button" variant="danger" @click="closeModal" id="botonCancelar">
              Cancelar
            </b-button>
          </div>
        </form>
      </b-modal>
    </div>
    
  </template>
  
  <script>
  export default {
    name: "Modal",
    data() {
      return {
        errors: [],
        name: null,
        autor: null,
        fecha: null,
      };
    },
    methods: {
      checkForm() {
        if (
          this.name &&
          this.autor &&
          this.fecha
        ) {
          this.sendForm();
          return true;
        }
        this.errors = [];
        if (!this.name) {
          this.errors.push("El nombre es obligatorio.");
        }
        if (!this.autor) {
          this.errors.push("El autor es obligatorio.");
        }
        if (!this.fecha) {
          this.errors.push("La fecha de publicación es obligatoria");
        }
      },
      async sendForm() {
  try {
    const response = await fetch("http://localhost:8080/api/library", {
      method: "PUT",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        name: this.name,
        autor: this.autor,
        fecha: this.fecha,
      }),
    });
    if (!response.ok) {
      throw new Error("Error al enviar el formulario.");
    }
    this.name = null;
    this.autor = null;
    this.fecha = null;
    this.errors = [];
    this.$emit("registroExitoso"); // Emitiendo el evento registroExitoso
    this.$swal({
      title: "Actualizado",
      text: "El libro ha sido actualizado con éxito",
      icon: "success",
    });
    this.$root.$emit("bv::hide::modal", "modal-1");
  } catch (error) {
    console.error(error);
  }
},
      closeModal() {
        this.$root.$emit("bv::hide::modal", "modal-1");
        this.name = null;
        this.fecha = null;
        this.autor = null;
      },
      clearErrors() {
        this.errors = [];
      },
    },
  };
  </script>
  
  <style>
  .aside-container {
    top: 0;
    right: 0;
    height: 100px;
    width: 100px;
    background-color: black;
  }
  </style>
  