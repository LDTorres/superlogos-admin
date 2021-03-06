<template class="currencies">
<div>
  <v-toolbar flat color="white" class="sticky-nav">
      <v-toolbar-title hidden-md-and-down class="text-capitalize">{{ viewNameESP }}</v-toolbar-title>
      <v-spacer></v-spacer>
      <v-btn :to="'/trashed?m='+viewName" color="error" flat class="mb-2">PAPELERA</v-btn>
      <v-dialog v-model="dialog" max-width="500px">
        <v-btn slot="activator" color="primary" flat class="mb-2">Nuevas {{ viewNameESP }}</v-btn>
        <v-card>
          <v-card-title>
            <span class="headline">{{ formTitle }}</span>
          </v-card-title>

          <v-card-text>
            <v-container grid-list-md>
              <v-layout wrap>
                <v-flex xs12>
                  <v-text-field type="text" name="name" v-validate="'required'" v-model="editedItem.name" label="Nombre"></v-text-field>                   
                  <span v-show="errors.has('name')">{{ errors.first('name') }}</span>
                </v-flex>
                <v-flex xs12>
                  <v-text-field type="text" name="Iso" v-validate="'required|max:3'" v-model="editedItem.iso" label="Iso"></v-text-field>
                  <span v-show="errors.has('Iso')">{{ errors.first('Iso') }}</span>
                </v-flex>
                <v-flex xs12>
                  <v-text-field type="text" name="Simbolo" v-validate="'required|max:3'" v-model="editedItem.symbol" label="Simbolo"></v-text-field>
                  <span v-show="errors.has('Simbolo')">{{ errors.first('Simbolo') }}</span>
                </v-flex>

                <v-flex xs12>
                  <v-select
                  v-model="editedItem.display"
                  :items="status"
                  item-text="text"
                  :error-messages="selectErrors"
                  label="Posición del simbolo"
                  required
                  name="Posicion" 
                  v-validate="'required'"
                  ></v-select>
                  <span v-show="errors.has('Posicion')">{{ errors.first('Posicion') }}</span>
                </v-flex>
              </v-layout>
            </v-container>
          </v-card-text>

          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="error" flat  @click.native="close">Cancelar</v-btn>
            <v-btn color="primary" flat  @click.native="save" :disabled="errors.count() > 0">Guardar</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-toolbar>
  <v-container fluid>
    <v-card>
      <v-card-title>
        <v-layout row wrap>
          <v-flex offset-xs8 xs4>
              <v-text-field
                v-model="search"
                append-icon="search"
                label="Buscar:"
                single-line
                hide-details
              ></v-text-field>
          </v-flex>
        </v-layout>
      </v-card-title>
      <v-data-table
        :headers="headers"
        :items="list"
        class="elevation-1"
        no-data-text="No hubo resultados"
        :search="search"
      >
      <template slot="items" slot-scope="props">
        <td>{{ props.item.id }}</td>
        <td >{{ props.item.name }}</td>
        <td >{{ props.item.iso }}</td>
        <td >{{ props.item.symbol }}</td>
        <td >
          <span v-for="country in props.item.countries" :key="country.name">{{ country.name }} </span>
        </td>
        <td class="justify-center layout px-0">
          <v-icon
            title
            class="mr-2" color="primary"
            @click="editItem(props.item)"
          >
            edit
          </v-icon>
          <v-icon
            title
            @click="deleteItem(props.item)" color="error"
          >
            delete
          </v-icon>
        </td>
      </template>
    </v-data-table>
    </v-card>
  </v-container>
</div>
</template>

<script lang="js">
  export default {
    name: 'currencies',
    props: ['search'],
    created () {
      this.$store.dispatch('getAll', {state: this.viewName})
    },
    mounted () {
    },
    data () {
      return {
        status: [
          { text: 'Izquierda', value: 'left' },
          { text: 'Derecha', value: 'right' }
        ],
        selectErrors: [],
        pagination: {},
        dialog: false,
        editedIndex: -1,
        viewNameESP: 'Monedas'
      }
    },
    methods: {
      editItem (item) {
        this.editedIndex = this.list.indexOf(item)
        this.editedItem = Object.assign({}, item)
        this.dialog = true
      },
      deleteItem (item) {
        item.index = this.list.indexOf(item)
        let params = {
          state: this.viewName,
          item: item
        }

        confirm('Esta seguro que desea eliminar este elemento?') && this.$store.dispatch('deleteOne', params)
      },
      close () {
        this.dialog = false
        setTimeout(() => {
          this.editedItem = Object.assign({}, this.defaultItem)
          this.editedIndex = -1
        }, 1500)
      },
      save () {
        this.$validator.validate().then(result => {
          if (!result) {
            alert('Llene los campos correctamente.')
          } else {
            let params = {
              state: this.viewName,
              item: this.editedItem
            }

            if (this.editedIndex > -1) {
              this.$store.dispatch('updateOne', params)
            } else {
              this.$store.dispatch('create', params)
            }
            this.close()
          }
        })
      }
    },
    watch: {
      dialog (val) {
        val || this.close()
      }
    },
    computed: {
      headers () {
        return this.$store.state[this.viewName].struct
      },
      list () {
        return this.$store.getters.getAll('currencies')
      },
      pages () {
        if (this.pagination.rowsPerPage == null ||
          this.pagination.totalItems == null
        ) return 0

        return Math.ceil(this.pagination.totalItems / this.pagination.rowsPerPage)
      },
      viewName () {
        return this.$route.name
      },
      formTitle () {
        var title = this.editedIndex === -1 ? 'Nuevo ' : 'Editar '
        return title + this.viewNameESP
      },
      defaultItem () {
        return this.$store.state[this.viewName].defaultItem
      },
      editedItem: {
        get () {
          return this.$store.state[this.viewName].editedItem
        },
        set (value) {
          this.$store.state[this.viewName].editedItem = value
          return this.$store.state[this.viewName].editedItem
        }
      }
    }
}
</script>