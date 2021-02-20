<template>
  <div class="container"><br><br>
    <v-card-title>
      Data Absen
      <v-spacer></v-spacer>
      <v-text-field
        v-model="username"
        append-icon="mdi-magnify"
        label="Search Username"
        single-line
        hide-details
      ></v-text-field>
    </v-card-title>
    <v-data-table
      :headers="headers"
      :items="absen"
      :search="username"
      :hide-default-header="hideHeaders"
      :show-select="showSelect"
      :loading="isLoading"
      item-key="name"
      class="elevation-1 table-bordered"
    >
      <template
        v-if="isEnabled('body')"
        v-slot:body="{ items }"
      >
        <thead>
          <tr>
            <th>Username</th>
            <th>Jam Absen</th>
            <th>Lat</th>
            <th>Lng</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="item in items"
            :key="item.name"
          >
            <td>{{ item.username }}</td>
            <td>{{ item.createdAt }}</td>
            <td>{{ item.lat }}</td>
            <td>{{ item.lng }}</td>
            <td>
              <v-icon class="mdi mdi-map-marker large" @click="editAbsen(item.id)"></v-icon>
            <v-icon class="large" @click="deleteAbsen(item.id), snackbar = true">mdi-delete</v-icon>
            </td>
          </tr>
        </tbody>
      </template>
    </v-data-table>
  </div>
</template>

<script>
import AbsenDataService from "../services/AbsenDataService";

  export default {
    data () {
      return {
        absen: [],
      currentAbsen: null,
      currentIndex: -1,
      username: "",
        enabled: "body",
        search: null,
        slots: [
          'body'
        ],
        headers: [
        { text: "Username", value: "username", sortable: false },
        { text: "Absensi", value: "absensi", sortable: false },
        { text: "Jam Absen", value: "createdAt", sortable: false },
        { text: "Latitude", value: "lat", sortable: false },
        { text: "Longitude", value: "lng", sortable: false },
        ],
      }
    },
    methods: {
   retrieveAbsen() {
      AbsenDataService.getAll()
        .then(response => {
          this.absen = response.data;
          console.log(response.data);
        })
        .catch(e => {
          console.log(e);
        });
    },

    refreshList() {
      this.retrieveAbsen();
      this.currentAbsen = null;
      this.currentIndex = -1;
    },

    editAbsen(id) {
      this.$router.push({ name: "map", params: { id: id } });
    },


    removeAllAbsen() {
      AbsenDataService.deleteAll()
        .then(response => {
          console.log(response.data);
          this.refreshList();
        })
        .catch(e => {
          console.log(e);
        });
    },
    
    searchUsername() {
      AbsenDataService.findByUsername(this.username)
        .then(response => {
          this.absen = response.data;
          console.log(response.data);
        })
        .catch(e => {
          console.log(e);
        });
    },
    deleteAbsen(id) {
      AbsenDataService.delete(id)
        .then(() => {
          this.refreshList();
        })
        .catch((e) => {
          console.log(e);
        });
    },
    isEnabled (slot) {
        return this.enabled === slot
      }
  },
    computed: {
      showSelect () {
        return this.isEnabled('header.data-table-select') || this
.isEnabled('item.data-table-select')
      },
      hideHeaders () {
        return !this.showSelect
      },
      isLoading () {
        return this.isEnabled('progress')
      }
    },

    watch: {
      enabled (slot) {
        if (slot === 'no-data') {
          this.items = []
        } else if (slot === 'no-results') {
          this.search = '...'
        } else {
          this.search = null
          this.items = []
        }
      }
    },
    mounted() {
    this.retrieveAbsen();
    }
  }
</script>