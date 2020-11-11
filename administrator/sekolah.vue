<template>
  <!-- Main content -->
  <section class="content">
    <div class="container-fluid">
      <div class="row mb-2">
        <div class="col-sm-12">
          <h3>Data Sekolah</h3>
        </div>
      </div>
      <div class="card mt-2">
        <div class="col-sm-12 card-header bg-success text-white">
          <div class="row">
            <div class="col-lg-10 col-sm-8">
              <h4 class="card-title">Daftar Sekolah</h4>
            </div>
            <div class="col-lg-2 col-sm-4">
              <b-button @click="Add" variant="light" pill class="pull-right"
              type="button" size="sm" v-b-modal.modal_sekolah style="float:right">
                 <span class="fa fa-plus"></span> Tambah Data
              </b-button>
            </div>
          </div>

        </div>
        <div class="card-body" style="overflow:auto">
          <b-form-input v-model="find" placeholder="Pencarian..."
          class="mb-3" v-on:keyup.enter="get_sekolah"></b-form-input>
          <ul class="list-group">
            <li class="list-group-item mb-2" v-for="item in sekolah">
              <b-row>
                <b-col cols="10">
                  <h5 class="text-info">{{ item.nama_sekolah }}</h5>
                  <h6>Alamat: {{ item.alamat_sekolah }}</h6>
                  <h6>Jenjang: {{ item.jenjang }}</h6>
                </b-col>
                <b-col>
                  <b-button :variant="item.admin_sekolah !== null ? 'secondary' : 'success'" 
                  size="sm" v-on:click="Admin(item)"
                  v-b-modal.modal_admin block>
                      <span class="fa fa-user" v-if="item.admin_sekolah !== null"></span>
                      <span class="fa fa-plus" v-else></span>
                       Admin
                  </b-button>
                  <b-button variant="info" size="sm" v-on:click="Edit(item)"
                  v-b-modal.modal_sekolah block>
                      <span class="fa fa-edit"></span> Edit
                  </b-button>
                  <b-button variant="danger" size="sm"
                  v-on:click="Drop(item.id_sekolah)" block>
                      <span class="fa fa-trash"></span> Hapus
                  </b-button>
                </b-col>
              </b-row>
            </li>
          </ul>
        </div>
        <div class="card-footer">
          <b-pagination
            v-model="pagination.currentPage"
            :per-page="pagination.perPage"
            :total-rows="pagination.rows"
            align="center"
            v-on:input="get_sekolah">
          </b-pagination>
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
    </div>

    <b-modal
      id="modal_sekolah"
      title="Form Sekolah"
      header-bg-variant="light"
      border-variant="light"
      hide-footer>

      <b-container fluid>
        <form v-on:submit.prevent="Save">
          Nama Sekolah
          <b-form-input v-model="formSekolah.nama_sekolah" class="mb-2" required>
          </b-form-input>
          Alamat Sekolah
          <b-form-input v-model="formSekolah.alamat_sekolah" class="mb-2" required>
          </b-form-input>
          Jenjang
          <b-form-select v-model="formSekolah.jenjang" class="mb-2" required>
            <option value="SD">SD</option>
            <option value="SMP">SMP</option>
            <option value="SMA/SMK">SMA/SMK</option>
          </b-form-select>

           <b-button variant="primary" pill block class="pull-right" type="submit">
              <span class="fa fa-check"></span> Simpan
           </b-button>
        </form>
      </b-container>
    </b-modal>

    <b-modal
      id="modal_admin"
      title="Form Admin Sekolah"
      header-bg-variant="light"
      border-variant="light"
      hide-footer>

      <b-container fluid>
        <form v-on:submit.prevent="SaveAdmin">
          Nama Admin
          <b-form-input v-model="formAdmin.nama_admin_sekolah" class="mb-2" required>
          </b-form-input>
          Kontak
          <b-form-input v-model="formAdmin.kontak" class="mb-2" required>
          </b-form-input>
          Email
          <b-form-input v-model="formAdmin.email" class="mb-2" required>
          </b-form-input>
          Sekolah
          <b-form-input v-model="formAdmin.nama_sekolah" class="mb-2" readonly>
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


           <b-button variant="primary" pill block class="pull-right" type="submit">
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
        formSekolah: {
          action: "",
          id_sekolah: "",
          nama_sekolah: "",
          alamat_sekolah: "",
          jenjang: ""
        },
        formAdmin: {
          action: "",
          id_admin_sekolah: "",
          nama_admin_sekolah: "",
          kontak: "",
          email: "",
          id_sekolah: "",
          nama_sekolah: "",
          username: "",
          password: "",
          require_password: true,
        },
        pagination: {
          currentPage: 1,
          rows: 0,
          perPage: 10,
        },
        find: "",
        sekolah: [],
        fields: ["nama_sekolah","alamat_sekolah","jenjang","option"],
        message: "",
        key: ""
      }
    },

    methods: {
      get_sekolah : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        let offset = (this.pagination.currentPage - 1) * this.pagination.perPage;
        this.$bvToast.show("loadingToast");
        if(this.find === ""){
          axios.get(base_url+"/sekolah/"+this.pagination.perPage+"/"+offset, conf)
          .then(response => {
            this.$bvToast.hide("loadingToast");
            this.sekolah = response.data.sekolah;
            this.pagination.rows = response.data.count;
          })
          .catch(error => {
            console.log(error);
          });
        }else{
          let form = new FormData();
          form.append("find", this.find);
          axios.post(base_url+"/sekolah/find/"+this.pagination.perPage+"/"+offset, form, conf)
          .then(response => {
            this.$bvToast.hide("loadingToast");
            this.sekolah = response.data.sekolah;
            this.pagination.rows = response.data.count;
          })
          .catch(error => {
            console.log(error);
          });
        }
      },
      Add : function(){
        this.formSekolah.action = "insert";
        this.formSekolah.id_sekolah = "";
        this.formSekolah.nama_sekolah = "";
        this.formSekolah.alamat_sekolah = "";
        this.formSekolah.jenjang = "";
      },
      Edit : function(item){
        this.formSekolah.action = "update";
        this.formSekolah.id_sekolah = item.id_sekolah;
        this.formSekolah.nama_sekolah = item.nama_sekolah;
        this.formSekolah.alamat_sekolah = item.alamat_sekolah;
        this.formSekolah.jenjang = item.jenjang;
      },
      Save : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        this.$bvToast.show("loadingToast");
        this.$bvModal.hide("modal_sekolah");
        let form = new FormData();
        form.append("action", this.formSekolah.action);
        form.append("id_sekolah", this.formSekolah.id_sekolah);
        form.append("nama_sekolah", this.formSekolah.nama_sekolah);
        form.append("alamat_sekolah", this.formSekolah.alamat_sekolah);
        form.append("jenjang", this.formSekolah.jenjang);

        axios.post(base_url+"/sekolah", form, conf)
        .then(response => {
          this.$bvToast.hide("loadingToast");
          this.get_sekolah();
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
          axios.delete(base_url+"/sekolah/"+id, conf)
          .then(response => {
            this.get_sekolah();
            this.$bvToast.hide("loadingToast");
            this.message = response.data.message;
            this.$bvToast.show("message");
          })
          .catch(error => {
            console.log(error);
          });
        }
      },

      Admin : function(item){
        this.formAdmin.action = (item.admin_sekolah == null) ? "insert" : "update";
        this.formAdmin.id_admin_sekolah = (item.admin_sekolah == null) ? "" : item.admin_sekolah.id_admin_sekolah;
        this.formAdmin.nama_admin_sekolah = (item.admin_sekolah == null) ? "" : item.admin_sekolah.nama_admin_sekolah;
        this.formAdmin.kontak = (item.admin_sekolah == null) ? "" : item.admin_sekolah.kontak;
        this.formAdmin.email = (item.admin_sekolah == null) ? "" : item.admin_sekolah.email;
        this.formAdmin.username = (item.admin_sekolah == null) ? "" : item.admin_sekolah.username;
        this.formAdmin.id_sekolah = item.id_sekolah;
        this.formAdmin.nama_sekolah = item.nama_sekolah;
        this.formAdmin.require_password = (item.admin_sekolah == null) ? true : false;
        this.formAdmin.password = "";
      },

      SaveAdmin : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        this.$bvToast.show("loadingToast");
        this.$bvModal.hide("modal_admin");
        let form = new FormData();
        form.append("action", this.formAdmin.action);
        form.append("id_admin_sekolah", this.formAdmin.id_admin_sekolah);
        form.append("nama_admin_sekolah", this.formAdmin.nama_admin_sekolah);
        form.append("id_sekolah", this.formAdmin.id_sekolah);
        form.append("email", this.formAdmin.email);
        form.append("kontak", this.formAdmin.kontak);
        form.append("username", this.formAdmin.username);

        if (this.formAdmin.require_password) {
          form.append("password", this.formAdmin.password);
        }

        axios.post(base_url+"/admin-sekolah", form, conf)
        .then(response => {
          this.$bvToast.hide("loadingToast");
          this.get_sekolah();
          this.message = response.data.message;
          this.$bvToast.show("message");
        })
        .catch(error => {
          console.log(error);
        });
      },

      authenticate : function(){
        if (!this.$cookies.isKey("Api-Token")) {
          window.location = "login.html";
          console.log("token is gone");
        }else{
          let token = this.$cookies.get("Api-Token");
          this.key = token;
          this.get_sekolah();
        }
      },
    },

    mounted(){
      this.authenticate();

    }
  }
</script>
