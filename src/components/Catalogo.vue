<template>
  <v-card class="mx-auto" width="1200">
    <v-text-field color="purple darken-4" loading disabled></v-text-field>
    <v-data-table
      :headers="headers"
      :items="desserts"
      sort-by="nombre"
      class="elevation-1"
      :search="search"
    >
      <template v-slot:top>
        <v-toolbar flat color="black">
          <v-toolbar-title>Catalogo</v-toolbar-title>

          <v-divider class="mx-4" inset vertical></v-divider>
          <v-spacer></v-spacer>
          <v-text-field
            v-model="search"
            label="Buscar (POR NOMBRE)"
            class="mx-2"
          ></v-text-field>

          <v-spacer></v-spacer>
          <v-dialog v-model="dialog" max-width="500px">
            <template v-slot:activator="{ on }">
              <v-btn color="primary" dark class="mb-2" v-on="on"
                >Nuevo Item</v-btn
              >
            </template>
            <v-card>
              <v-card-title>
                <span class="headline">{{ formTitle }}</span>
              </v-card-title>

              <v-card-text>
                <v-container>
                  <v-row>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field
                        v-model="editedItem.nombre"
                        label="Nombre"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field
                        v-model="editedItem.descripcion"
                        label="Descripcion"
                      ></v-text-field>
                    </v-col>
                  </v-row>
                </v-container>
              </v-card-text>

              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="close"
                  >Cancelar</v-btn
                >
                <v-btn color="blue darken-1" text @click="save">Guardar</v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-toolbar>
      </template>
      <template v-slot:item.actions="{ item }">
        <v-icon small class="mr-2" @click="editItem(item)">
          view_comfy
        </v-icon>
        <v-icon small @click="deleteItem(item)">
          not_interested
        </v-icon>
      </template>
      <template v-slot:no-data>
        <v-btn color="primary" @click="initialize">Reset</v-btn>
      </template>
    </v-data-table>
  </v-card>
</template>

<script>
export default {
  data: () => ({
    idd: "",
    nombree: "",
    descripcionn: "",
    dialog: false,
    search: "",
    headers: [
      {
        text: "Id",
        align: "start",
        sortable: false,
        value: "id",
      },

      { text: "Nombre", value: "nombre" },
      { text: "Descripcion", value: "descripcion" },
      { text: "Actions", value: "actions", sortable: false },
    ],
    desserts: [],
    editedIndex: -1,
    editedItem: {
      id: "",
      nombre: "",
      descripcion: "",
    },
    defaultItem: {
      id: "",
      nombre: "",
      descripcion: "",
    },
  }),

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "Nuevo Item" : "Edita Item";
    },
  },

  watch: {
    dialog(val) {
      val || this.close();
    },
  },

  created() {
    this.initialize();
  },

  methods: {
    initialize() {
      var requestOptions = {
        method: "GET",

        redirect: "follow",
      };
      //
      fetch(
        "http://jorgeperalta-001-site6.itempurl.com/catalogo.php",
        requestOptions
      )
        .then((response) => response.json())
        .then((result) => (this.desserts = result))
        .catch((error) => console.log("error", error));
    },

    editItem(item) {
      this.idd = item.id;
      this.nombree = item.nombre;
      this.descripcionn = item.descripcion;
      this.editedIndex = this.desserts.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },

    deleteItem(item) {
      const index = this.desserts.indexOf(item);
      confirm("Esta seguro de eliminar este item?") &&
        this.desserts.splice(index, 1);

      var requestOptions = {
        method: "DELETE",
      };

      fetch(
        "http://jorgeperalta-001-site6.itempurl.com/catalogo.php?id=" + item.id,
        requestOptions
      )
        .then((response) => response.text())
        .then((result) => console.log(result))
        .catch((error) => console.log("error", error));
    },

    close() {
      this.dialog = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },

    save() {
      if (this.editedIndex > -1) {
      

        var requestOptions = {
          method: "PUT",
         
          redirect: "follow",
        };

        fetch(
          "http://jorgeperalta-001-site6.itempurl.com/catalogo.php?id=" +
            this.editedItem.id +
            "&nombre=" +
            this.editedItem.nombre +
            "&descripcion=" +
            this.editedItem.descripcion,
          requestOptions
        )
          .then((response) => response.text())
          .then((result) => console.log(result));
        this.initialize();
        //  Object.assign(this.desserts[this.editedIndex], this.editedItem);
      } else {
        var formdata = new FormData();

        formdata.append("id", this.editedItem.id);
        formdata.append("nombre", this.editedItem.nombre);
        formdata.append("descripcion", this.editedItem.descripcion);

        var requestOptions = {
          method: "POST",
          body: formdata,
          redirect: "follow",
        };

        fetch(
          "http://jorgeperalta-001-site6.itempurl.com/catalogo.php",
          requestOptions
        )
          .then((response) => response.text())
          .then((result) => console.log(result));
        // this.desserts.push(this.editedItem);
        this.initialize();
      }
      this.close();
    },
  },
};
</script>
