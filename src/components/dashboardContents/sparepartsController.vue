<template>
  <v-container>
    <v-card>
      <v-container grid-list-md mb-0>
        <h2 class="text-md-center">Data Spareparts</h2>
        <v-layout row wrap style="margin:10px">
          <v-flex xs6>
            <v-btn
              depressed
              dark
              rounded
              style="text-transform: none !important;"
              color="green accent-3"
              @click="dialog = true"
            >
              <v-icon size="18" class="mr-2">mdi-pencil-plus</v-icon>Tambah Sparepart
            </v-btn>
          </v-flex>
          <v-flex xs6 class="text-right">
            <v-text-field v-model="keyword" append-icon="mdi-search" label="Search" hide-details></v-text-field>
          </v-flex>
        </v-layout>

        <v-data-table :headers="headers" :items="spareparts" :search="keyword" :loading="load">
          <template v-slot:body="{ items }">
            <tbody>
              <tr v-for="(item,index) in items" :key="item.id">
                <td>{{ index + 1 }}</td>
                <td>{{ item.name }}</td>
                <td>{{ item.merk}}</td>
                <td>{{ item.amount }}</td>
                <td>{{ item.created_at }}</td>
                <td class="text-center">
                  <v-btn icon color="indigo" light @click="editHandler(item)">
                    <v-icon>mdi-pencil</v-icon>
                  </v-btn>
                  <v-btn icon color="error" light @click="deleteData(item.id)">
                    <v-icon>mdi-delete</v-icon>
                  </v-btn>
                </td>
              </tr>
            </tbody>
          </template>
        </v-data-table>
      </v-container>
    </v-card>
    <v-dialog v-model="dialog" persistent max-width="600px">
      <v-card>
        <v-card-title>
          <span class="headline">Sparepart Data</span>
        </v-card-title>
        <v-card-text>
          <v-container>
            <v-row>
              <v-col cols="12">
                <v-text-field label="Name*" v-model="form.name" required></v-text-field>
              </v-col>
              <v-col cols="12">
                <v-select label="Merk*" v-model="form.merk" :items="item" required></v-select>
              </v-col>
              <v-col cols="12">
                <v-text-field label="Amount*" v-model="form.amount" type="number" required></v-text-field>
              </v-col>
            </v-row>
          </v-container>
          <small>*indicates required field</small>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="blue darken-1" text @click="dialog = false">Close</v-btn>
          <v-btn color="blue darken-1" text @click="setForm()">Save</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <v-snackbar v-model="snackbar" :color="color" :multi-line="true" :timeout="3000">
      {{ text }}
      <v-btn dark text @click="snackbar = false">Close</v-btn>
    </v-snackbar>
  </v-container>
</template> 

<script>
export default {
  data() {
    return {
      dialog: false,
      keyword: "",
      item: ["Yamaha", "Honda", "Suzuki"],
      headers: [
        {
          text: "No",
          value: "no"
        },

        {
          text: "Name",
          value: "name"
        },

        {
          text: "Merk",
          value: "merk"
        },

        {
          text: "Amount",
          value: "amount"
        },

        {
          text: "Created At",
          value: null
        },

        {
          text: "Aksi",
          value: null
        }
      ],
      spareparts: [],
      snackbar: false,
      color: null,
      text: "",
      load: false,
      form: {
        name: "",
        merk: "",
        amount: ""
      },
      sparepart: new FormData(),
      typeInput: "new",
      errors: "",
      updatedId: ""
    };
  },

  methods: {
    getData() {
      var uri = this.$apiUrl + "/sparepart";
      this.$http.get(uri).then(response => {
        this.spareparts = response.data.message;
      });
    },

    sendData() {
      this.sparepart.append("name", this.form.name);
      this.sparepart.append("merk", this.form.merk);
      this.sparepart.append("amount", this.form.amount);
      var uri = this.$apiUrl + "/sparepart";
      this.load = true;
      this.$http
        .post(uri, this.sparepart)
        .then(response => {
          this.snackbar = true; //mengaktifkan snackbar
          this.color = "green"; //memberi warna snackbar
          this.text = response.data.message; //memasukkan pesan ke snackbar
          this.load = false;
          this.dialog = false;
          this.getData(); //mengambil data user
          this.resetForm();
        })
        .catch(error => {
          this.errors = error;
          this.snackbar = true;
          this.text = "Try Again";
          this.color = "red";
          this.load = false;
        });
    },

    updateData() {
      this.sparepart.append("name", this.form.name);
      this.sparepart.append("merk", this.form.merk);
      this.sparepart.append("amount", this.form.amount);
      var uri = this.$apiUrl + "/sparepart/" + this.updatedId;
      this.load = true;
      this.$http
        .post(uri, this.sparepart)
        .then(response => {
          this.snackbar = true; //mengaktifkan snackbar
          this.color = "green"; //memberi warna snackbar
          this.text = response.data.message; //memasukkan pesan ke snackbar
          this.load = false;
          this.dialog = false;
          this.getData(); //mengambil data user
          this.resetForm();
          this.typeInput = "new";
        })
        .catch(error => {
          this.errors = error;
          this.snackbar = true;
          this.text = "Try Again";
          this.color = "red";
          this.load = false;
          this.typeInput = "new";
        });
    },

    editHandler(item) {
      this.typeInput = "edit";
      this.dialog = true;
      this.form.name = item.name;
      this.form.merk = item.merk;
      this.form.amount = item.amount;
      this.updatedId = item.id;
    },

    deleteData(deleteId) {
      //mengahapus data
      var uri = this.$apiUrl + "/sparepart/" + deleteId; //data dihapus berdasarkan id
      this.$http
        .delete(uri)
        .then(response => {
          this.snackbar = true;
          this.text = response.data.message;
          this.color = "green";
          this.deleteDialog = false;
          this.getData();
        })
        .catch(error => {
          this.errors = error;
          this.snackbar = true;
          this.text = "Try Again";
          this.color = "red";
        });
    },

    setForm() {
      if (this.typeInput === "new") {
        this.sendData();
      } else {
        console.log("dddd");
        this.updateData();
      }
    },

    resetForm() {
      this.form = {
        name: "",
        merk: "",
        amount: ""
      };
    }
  },

  mounted() {
    this.getData();
  }
};
</script> 
                