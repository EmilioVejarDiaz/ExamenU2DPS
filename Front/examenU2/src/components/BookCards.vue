<template>
  <div class="container" style="height: 100vh">
    <Carousel v-show="showElement"></Carousel>
    <div style="display: flex; margin-top: 50px; padding-bottom: 50 px">
      <h1 style="margin-bottom: 0">Libros</h1>
      <Modal @registroExitoso="actualizarLibros"></Modal>
    </div>

    <div>

    </div>

      <div style="margin-bottom: 50px; width: 50%">
      <div>
        <b-button @click="ordenarLibrosDesc" class="button" variant="success">Ordenar por fecha</b-button>
        <b-button @click="ordenarLibrosAutor" class="button" variant="success">Ordenar por autor</b-button>
       </div>
    </div>
    

    <div class="text-center" id="loaderSpinner" v-if="loading">
      <b-spinner class="spinner" type="grow"></b-spinner>
      <b-spinner class="spinner" type="grow"></b-spinner>
      <b-spinner class="spinner" type="grow"></b-spinner>
      <b-spinner class="spinner" type="grow"></b-spinner>
      <b-spinner class="spinner" type="grow"></b-spinner>
    </div>

        <div v-if="books">
      <TransitionGroup name="zoomDown" tag="div" class="row">
        <div class="col-4 bookCards" v-for="book in books" :key="book.id" >
          <div>
            <b-card :title="book.name" img-src="https://picsum.photos/600/300/?image=20" img-alt="Image" img-top
              tag="article" style="max-width: 20rem" class="mb-2" draggable @dragstart="onDragStart(book)">
              <b-card-text class="description">
                <h2>Autor: {{ book.autor }}</h2>
              </b-card-text>
              <div>
                <p><b>Fecha de publicación: </b>{{ dateFormat(book.fecha) }}</p>
              </div>
              <div style="width: 100%; display: flex">
                <div style="display: flex; width: 100%; justify-content: space-around">
                 
                    

                </div>
                
              </div>
            </b-card>
          </div>
        </div>
      </TransitionGroup>
    </div>

    <div v-else>
      <h3 class="text-center">Sin resultados</h3>
    </div>


      <b-col>

        <b-row>
          <div id="putDrop">
          <b-row>

            <b-col>
              <div class="drop-zone, mt-3" @drop="onDropEdit($event)" @dragover.prevent @dragenter.prevent>
          <b-card id="dropZoneModificar" title="Suelta aquí los libros que quieras modificar"></b-card>
        </div>
            </b-col>

          </b-row>
          <b-row>
            <b-col>
              <div class="drop-zone, mt-3" @drop="onDropDelete($event)" @dragover.prevent @dragenter.prevent>
                <b-card id="dropZoneEliminar" title="Suelta aquí los libros que quieras eliminar"></b-card>
              </div>
            </b-col>
          </b-row>
        </div>
        </b-row>

      </b-col>
      
    
   
    
  </div>
</template>

<script>
import Modal from "./Modal.vue";
import Carousel from "./Carousel.vue";
import axios from "axios";

export default {
  name: "MovieCards",
  components: {
    Modal,

    Carousel
  },
  data() {
    return {
      books: [],
      loading: null,
      search: null,
      endDate: null,
      startDate: null,
      lastScrollPosition: 0,
      scrollDistance: 0,
      showElement: true,
      loaded: false
    };
  },
  methods: {
    onDragStart(book) {
      event.dataTransfer.setData("text/plain", JSON.stringify(book));
    },

    openEditModal(book) {
      this.selectedBook = book;
      this.$refs.editModal.show();
    },

    onDropEdit(event) {
      event.preventDefault();
      const book = JSON.parse(event.dataTransfer.getData("text/plain"));
      this.openEditModal(book);
    },

    onDropDelete(event) {
      event.preventDefault();
      const book = JSON.parse(event.dataTransfer.getData("text/plain"));
      this.deleteLibro(book.id);
    },
   
    actualizarLibros() {
      this.loading = true;
      this.fetchBooks("http://localhost:8080/api/library");
    },
    ordenarLibrosDesc() {
      this.loading = true;
      this.fetchBooks("http://localhost:8080/api/library/latest");
    },
    ordenarLibrosAutor() {
      this.loading = true;
      this.fetchBooks("http://localhost:8080/api/library/autorAsc");
    },
    fetchBooks(url) {
      axios
        .get(url)
        .then((response) => {
          this.books = response.data;
        })
        .catch((e) => {
          console.error("Error en la peticion: ", e);
        })
        .finally(() => {
          this.loading = false;
        });
    },
    deleteLibro(id) {
      this.$swal({
        title: "¿Estas seguro?",
        text: "No podrás revertir este cambio",
        icon: "warning",
        showCancelButton: true,
        confirmButtonColor: "#3085d6",
        cancelButtonColor: "#d33",
        cancelButtonText: "cancelar",
        confirmButtonText: "Si, eliminar"
      }).then((result) => {
        if (result.isConfirmed) {
          axios
            .delete(`http://localhost:8080/api/library/${id}`)
            .then((response) => {
              this.$swal({
                title: "Eliminada",
                text: "El libro ha sido eliminado con éxito",
                icon: "success"
              });
              this.actualizarLibros();
            })
            .catch((error) => {
              console.error(error);
            });
        }
      });
    },
    dateFormat(date) {
      let dateParts = date.split("-");
      let year = dateParts[0];
      let month = dateParts[1];
      let day = dateParts[2];
      let newDate = day + "-" + month + "-" + year;
      return newDate;
    },
    onScroll() {

      // Obtiene la posición actual del scroll
      const currentScrollPosition = window.pageYOffset || document.documentElement.scrollTop;
      console.log(currentScrollPosition);

      // La función abs para tener valores absolutos y se delimita con un offset o bien llamado 
      // margen para que el valor de la posición sea después de desplazarce y no desde que uno se desplaza
      if (Math.abs(currentScrollPosition - this.lastScrollPosition) < 60) {
        return;
      }
      // aqui determinamos si la posición es mayor a la posición anterior. Entonces, si lo es, mostramos el elemento.
      this.showElement = currentScrollPosition < this.lastScrollPosition;
      //  
      this.lastScrollPosition = currentScrollPosition;
    },
  },
  mounted() {
    this.loading = true;
    window.addEventListener("scroll", this.onScroll);

    axios
      .get("http://localhost:8080/api/library")
      .then((response) => {
        this.books = response.data;
      })
      .catch((e) => {
        console.error("Error en la peticion: ", e);
      })
      .finally(() => {
        this.loading = false;
      });
  },
  beforeDestroy() {
    window.removeEventListener("scroll", this.onScroll);
  }
};
</script>

<style scoped>
.bookCards {
  display: flex;
  justify-content: center;
}

#loaderSpinner {
  height: 75%;
  display: flex;
  justify-content: center;
  align-items: center;
}

.spinner {
  margin: 7px;
}


.button {
  padding: 10px;
  border-radius: 8px;
  width: 50%;
  margin-top: 20px;
}

.hidden {
  display: none;
}
</style>
