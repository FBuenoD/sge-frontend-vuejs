<template>
  <v-data-table :headers="headers" :items="eventos" sort-by="calories" class="elevation-1">
    <template v-slot:top>
      <v-toolbar flat>
        <v-toolbar-title>{{ viewTitle }}</v-toolbar-title>
        <v-divider class="mx-4" inset vertical></v-divider>
        <v-spacer></v-spacer>

        <v-dialog v-model="dialog" max-width="500px">
          <template v-slot:activator="{ on, attrs }">
            <v-btn color="primary" dark class="mb-2" v-bind="attrs" v-on="on">Novo</v-btn>
          </template>
          <v-card>
            <v-card-title>
              <span class="headline">{{ formTitle }}</span>
            </v-card-title>

            <v-card-text>
              <v-container>
                <v-row>
                  <v-col cols="12">
                    <v-text-field v-model="editedItem.titulo" label="Título do Evento"></v-text-field>
                  </v-col>

                  <v-col cols="12" sm="6">
                    <v-menu
                      ref="menuDataInicial"
                      v-model="menuDataInicial"
                      :close-on-content-click="false"
                      :return-value.sync="dataInicial"
                      transition="scale-transition"
                      offset-y
                      min-width="290px"
                    >
                      <template v-slot:activator="{ on, attrs }">
                        <v-text-field
                          v-model="dataInicial"
                          label="Data inicial"
                          readonly
                          v-bind="attrs"
                          v-on="on"
                        ></v-text-field>
                      </template>
                      <v-date-picker v-model="dataInicial" no-title scrollable>
                        <v-spacer></v-spacer>
                        <v-btn text color="primary" @click="menuDataInicial = false">Cancel</v-btn>
                        <v-btn
                          text
                          color="primary"
                          @click="$refs.menuDataInicial.save(dataInicial)"
                        >OK</v-btn>
                      </v-date-picker>
                    </v-menu>
                  </v-col>

                  <v-col cols="12" sm="6">
                    <v-menu
                      ref="menuDataFinal"
                      v-model="menuDataFinal"
                      :close-on-content-click="false"
                      :return-value.sync="dataFinal"
                      transition="scale-transition"
                      offset-y
                      min-width="290px"
                    >
                      <template v-slot:activator="{ on, attrs }">
                        <v-text-field
                          v-model="dataFinal"
                          label="Data final"
                          readonly
                          v-bind="attrs"
                          v-on="on"
                        ></v-text-field>
                      </template>
                      <v-date-picker v-model="dataFinal" no-title scrollable>
                        <v-spacer></v-spacer>
                        <v-btn text color="primary" @click="menuDataFinal = false">Cancel</v-btn>
                        <v-btn text color="primary" @click="$refs.menuDataFinal.save(dataFinal)">OK</v-btn>
                      </v-date-picker>
                    </v-menu>
                  </v-col>

                  <v-col cols="12">
                    <v-text-field v-model="editedItem.descricao" label="Descrição do Evento"></v-text-field>
                  </v-col>
                </v-row>
              </v-container>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="close">Cancelar</v-btn>
              <v-btn color="blue darken-1" text @click="save">Salvar</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <v-dialog v-model="dialogExcluir" max-width="430px">
          <v-card>
            <v-card-title class="headline">Deseja mesmo remover este Item?</v-card-title>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="closeExcluir">Cancelar</v-btn>
              <v-btn color="blue darken-1" text @click="deleteItemComfirm">Sim</v-btn>
              <v-spacer></v-spacer>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-toolbar>
    </template>

    <template v-slot:item.actions="{ item }">
      <v-icon small class="mr-2" @click="editItem(item)">mdi-pencil</v-icon>
      <v-icon small @click="deleteItem(item)">mdi-delete</v-icon>
    </template>
    <template v-slot:no-data>
      <v-btn color="primary" @click="fetchRecords">Recarregar</v-btn>
    </template>
  </v-data-table>
</template>

<script>
import EventoService from "../service/domain/EventoService";
const service = EventoService.build();

const textos = {
  titulo: "Eventos",
  novo: "Novo Evento",
  edicao: "Edição de Evento",
  exclusao: "Deseja mesmo remover este Evento?",
};

export default {
  name: "Eventos",
  components: {},

  data: (vm) => ({
    dialog: false,
    dialogExcluir: false,
    headers: [
      { text: "ID", value: "id" },
      { text: "Título", align: "start", value: "titulo" },
      { text: "Data de início", align: "center", value: "inicio" },
      { text: "Data de término", align: "center", value: "fim" },
      { text: "Ações", align: "end", value: "actions", sortable: false },
    ],

    dataInicial: new Date().toISOString(),
    dataFinal: new Date().toISOString().substr(0, 10),

    dataInicialFormatted: vm.formatDate(new Date().toISOString().substr(0, 10)),
    dataFinalFormatted: vm.formatDate(new Date().toISOString().substr(0, 10)),

    menuDataInicial: false,
    menuDataFinal: false,

    eventos: [],
    editedIndex: -1,
    editedItem: {},
    defaultItem: {},
  }),

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? textos.novo : textos.edicao;
    },

    viewTitle() {
      return textos.titulo;
    },
  },

  watch: {
    dialog(val) {
      val || this.close();
    },
    dialogExcluir(val) {
      val || this.closeExcluir();
    },

    dataInicial() {
      // this.dataInicialFormatted = this.formatDate(this.dataInicial);
      this.editedItem.inicio = `${this.dataInicial} 00:00`;
    },

    dataFinal() {
      // this.dataFinalFormatted = this.formatDate(this.dataFinal);
      this.editedItem.fim = `${this.dataFinal} 00:00`;
    },
  },

  created() {
    this.fetchRecords();
  },

  methods: {
    fetchRecords() {
      service.search({}).then(this.fetchRecodsSuccess);
    },

    fetchRecodsSuccess(response) {
      if (Array.isArray(response.rows)) {
        this.eventos = response.rows;
        return;
      }
      this.eventos = [];
    },

    formatDate(date) {
      if (!date) return null;
      const [year, month, day] = date.split("-");
      return `${day}/${month}/${year}`;
    },

    parseDate(date) {
      console.log(date);
      if (!date) return null;
      const [month, day, year] = date.split("/");
      return `${year}-${month.padStart(2, "0")}-${day.padStart(2, "0")}`;
    },

    editItem(item) {
      this.editedIndex = this.eventos.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },

    deleteItem(item) {
      this.editedIndex = this.eventos.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialogExcluir = true;
    },

    deleteItemComfirm() {
      service
        .destroy(this.editedItem)
        .then(this.eventos.splice(this.editedIndex, 1));
      this.closeExcluir();
    },

    closeExcluir() {
      this.dialogExcluir = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
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
        service
          .update(this.editedItem)
          .then(Object.assign(this.eventos[this.editedIndex], this.editedItem));
      } else {
        service
          .create(this.editedItem)
          .then((response) => this.eventos.push(response));
      }
      this.close();
    },
  },
};
</script>
