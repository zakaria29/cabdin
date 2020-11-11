<template>
  <!-- Main content -->
  <section class="content">
    <div class="container-fluid">
      <div class="row mb-2">
        <div class="col-sm-12">
          <h3>Data Administrator</h3>
        </div>
      </div>

      <b-toast id="loadingToast" title="Processing Data" no-auto-hide>
        <b-spinner label="Spinning" variant="success"></b-spinner>
        <strong class="text-success">Loading...</strong>
      </b-toast>

      <!-- toast untuk tampilan message box -->
      <b-toast id="message" title="Message">
        <strong class="text-success">{{ message }}</strong>
      </b-toast>

      <div class="card mt-2">
        <div class="col-sm-12 card-header bg-success text-white">
          <div class="row">
            <div class="col-lg-10 col-sm-8">
              <h4 class="card-title">Daftar Administrator</h4>
            </div>
            <div class="col-lg-2 col-sm-4">
              <b-button @click="Add" variant="light" pill class="pull-right"
              type="button" size="sm" v-b-modal.modal_admin style="float:right">
                 <span class="fa fa-plus"></span> Tambah Data
              </b-button>
            </div>
          </div>

        </div>
        <div class="card-body" style="overflow:auto">
          <!-- <b-form-input v-model="find" placeholder="Pencarian..."
          class="mb-3" v-on:keyup.enter="get_admin"></b-form-input> -->
          <b-table hover :items="admin" :fields="fields">
            <template v-slot:cell(option)="data">
              <b-button variant="info" size="sm" v-on:click="Edit(data.item)"
              v-b-modal.modal_admin pill>
                  <span class="fa fa-edit"></span>
              </b-button>
              <b-button variant="danger" size="sm"
              v-on:click="Drop(data.item.id_admin)" pill>
                  <span class="fa fa-trash"></span>
              </b-button>
            </template>
          </b-table>
        </div>
        <!-- <div class="card-footer">
          <b-pagination
            v-model="pagination.currentPage"
            :per-page="pagination.perPage"
            :total-rows="pagination.rows"
            align="center"
            v-on:input="get_admin">
          </b-pagination>
        </div> -->
      </div>
    </div>

    <b-modal
      id="modal_admin"
      title="Form Administrator"
      header-bg-variant="light"
      border-variant="light"
      hide-footer>

      <b-container fluid>
        <form v-on:submit.prevent="Save">
          Nama
          <b-form-input v-model="formAdmin.nama_admin" class="mb-2" required>
          </b-form-input>
          Email
          <b-form-input type="email" v-model="formAdmin.email" class="mb-2" required>
          </b-form-input>
          Kontak
          <b-form-input v-model="formAdmin.kontak" class="mb-2" required>
          </b-form-input>
          Username
          <b-form-input v-model="formAdmin.username" class="mb-2" required>
          </b-form-input>
          <b-form-checkbox v-model="formAdmin.require_password"
          value="true" unchecked-value="false"
          v-if="formAdmin.action === 'update'">
             Change Password
          </b-form-checkbox>

          <div v-if="formAdmin.require_password">
            Password
            <b-form-input type="password" v-model="formAdmin.password" class="mb-2"
            :required="formAdmin.require_password">
            </b-form-input>
          </div>

           <b-button variant="primary" pill block class="" type="submit">
              <span class="fa fa-check"></span> Simpan
           </b-button>
        </form>
      </b-container>
    </b-modal>
  </section>
</template>
<script type="text/javascript">
  module.exports = {
    data : function(){
      return {
        formAdmin: {
          action: "",
          id_admin: "",
          nama_admin: "",
          email: "",
          kontak: "",
          username: "",
          password: "",
          require_password: true
        },
        pagination: {
          currentPage: 1,
          rows: 0,
          perPage: 10,
        },
        find: "",
        admin: [],
        fields: ["nama_admin","email","kontak","option"],
        message: "",
        key: "",
      }
    },

    methods: {
      get_admin : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        axios.get(base_url+"/admin", conf)
        .then(response => {
          this.admin = response.data;
        })
        .catch(error => {
          console.log(error);
        });
      },

      Add : function(){
        this.formAdmin.action = "insert";
        this.formAdmin.id_admin = "";
        this.formAdmin.nama_admin = "";
        this.formAdmin.email = "";
        this.formAdmin.kontak = "";
        this.formAdmin.username = "";
        this.formAdmin.password = "";
        this.formAdmin.require_password = true;
      },
      Edit : function(item){
        this.formAdmin.action = "update";
        this.formAdmin.id_admin = item.id_admin;
        this.formAdmin.nama_admin = item.nama_admin;
        this.formAdmin.email = item.email;
        this.formAdmin.kontak = item.kontak;
        this.formAdmin.require_password = false;
        this.formAdmin.username = item.username;
        this.formAdmin.password = "";
      },
      Save : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        this.$bvToast.show("loadingToast");
        this.$bvModal.hide("modal_admin");
        let form = new FormData();
        form.append("action", this.formAdmin.action);
        form.append("id_admin", this.formAdmin.id_admin);
        form.append("nama_admin", this.formAdmin.nama_admin);
        form.append("email", this.formAdmin.email);
        form.append("kontak", this.formAdmin.kontak);
        form.append("username", this.formAdmin.username);

        if (this.formAdmin.require_password) {
          form.append("password", this.formAdmin.password);
        }

        axios.post(base_url+"/admin", form, conf)
        .then(response => {
          this.$bvToast.hide("loadingToast");
          this.get_admin();
          this.message = response.data.message;
          this.$bvToast.show("message");
        })
        .catch(error => {
          console.log(error);
        });
      },
      Drop : function(id){
        let conf = { headers: { "Api-Token" : this.key} };
        if (confirm('Apakah Anda yakin ingin menghapus data ini?')) {
          this.$bvToast.show("loadingToast");
          axios.delete(base_url+"/admin/"+id, conf)
          .then(response => {
            this.get_admin();
            this.$bvToast.hide("loadingToast");
            this.message = response.data.message;
            this.$bvToast.show("message");
          })
          .catch(error => {
            console.log(error);
          });
        }
      },
      authenticate : function(){
        if (!this.$cookies.isKey("Api-Token")) {
          window.location = "login.html";
          console.log("token is gone");
        }else{
          let token = this.$cookies.get("Api-Token");
          this.key = token;
          this.get_admin();
        }
      },

    },

    mounted(){
      this.authenticate();

    }
  }
</script>
