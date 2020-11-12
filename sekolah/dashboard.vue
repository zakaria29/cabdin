<template>
  <!-- Main content -->
  <section class="content">
    <div class="container-fluid">
      <div class="row mb-2">
        <div class="col-sm-6">
          <h3>Dashboard</h3>
        </div>
      </div>

      <div class="row">
        <div class="col-lg-6 col-sm-12">
          <div class="card">
            <!-- <div class="card-header">
              Data Sekolah
            </div> -->
            <div class="card-body">
              <strong>Data Sekolah</strong> <hr />
              <b-row>
                <b-col cols="3">Nama</b-col>
                <b-col cols="9">: {{ (this.admin.sekolah === null) ? "" : this.admin.sekolah.nama_sekolah }}</b-col>
                <b-col cols="3">Alamat</b-col>
                <b-col cols="9">: {{ (this.admin.sekolah === null) ? "" : this.admin.sekolah.alamat_sekolah }}</b-col>
              </b-row>
              <br />
              <strong>Admin Sekolah</strong>
              <a class="text-info ml-3" v-b-modal.modal_admin>
                <span class="fa fa-edit"></span>
              </a>
              <hr />
              <b-row>
                <b-col cols="4">Nama</b-col>
                <b-col cols="8">: {{ admin.nama_admin_sekolah }}</b-col>
                <b-col cols="4">Kontak</b-col>
                <b-col cols="8">: {{ admin.kontak }}</b-col>
                <b-col cols="4">Email</b-col>
                <b-col cols="8">: {{ admin.email }}</b-col>
              </b-row>
            </div>
          </div>
        </div>

        <div class="col-lg-3 col-sm-6">
          <!-- small box -->
          <div class="small-box bg-info">
            <div class="inner">
              <h3>{{ countSurat }}</h3>

              <p>Surat</p>
            </div>
            <div class="icon">
              <i class="fa fa-envelope"></i>
            </div>
            <!-- <a href="#" class="small-box-footer">More info <i class="fas fa-arrow-circle-right"></i></a> -->
          </div>
        </div>

        <div class="col-lg-3 col-sm-6">
          <!-- small box -->
          <div class="small-box bg-success">
            <div class="inner">
              <h3>{{ countJurnal }}</h3>

              <p>Jurnal</p>
            </div>
            <div class="icon">
              <i class="fa fa-book"></i>
            </div>
            <!-- <a href="#" class="small-box-footer">More info <i class="fas fa-arrow-circle-right"></i></a> -->
          </div>
        </div>
      </div>
    </div>

    <b-modal
      id="modal_admin"
      title="Form Admin Sekolah"
      header-bg-variant="light"
      border-variant="light"
      hide-footer>

      <b-container fluid>
        <form v-on:submit.prevent="SaveAdmin">
          Nama Admin
          <b-form-input v-model="admin.nama_admin_sekolah" class="mb-2" required>
          </b-form-input>
          Kontak
          <b-form-input v-model="admin.kontak" class="mb-2" required>
          </b-form-input>
          Email
          <b-form-input v-model="admin.email" class="mb-2" required>
          </b-form-input>
          Username
          <b-form-input v-model="admin.username" class="mb-2" required>
          </b-form-input>
          <b-form-checkbox v-model="admin.require_password"
          value="true" unchecked-value="false">
             Change Password
          </b-form-checkbox>

          <div v-if="require_password">
            Password
            <b-form-input type="password" v-model="admin.password" class="mb-2"
            :required="require_password">
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
        countSurat: 0,
        countJurnal: 0,
        key: "",
        message: "",
        sekolah: {
          id_sekolah: "",
          nama_sekolah: "",
          alamat: "",
        },
        admin: {
          id_admin_sekolah: "",
          nama_admin_sekolah: "",
          kontak: "",
          email: "",
          id_sekolah: "",
          nama_sekolah: "",
          username: "",
          password: "",
        },
        require_password: false,
      }
    },

    methods: {
      getAdmin : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        axios.get(base_url + "/admin-sekolah-id/" + this.admin.id_admin_sekolah, conf)
        .then(response => {
          this.admin = response.data;
        })
        .catch(error => {
          console.log(error);
        })
      },
      SaveAdmin : function(){
        let conf = { headers: { "Api-Token" : this.key} };

        this.$bvModal.hide("modal_admin");
        let form = new FormData();
        form.append("action", "update");
        form.append("id_admin_sekolah", this.admin.id_admin_sekolah);
        form.append("nama_admin_sekolah", this.admin.nama_admin_sekolah);
        form.append("id_sekolah", this.admin.id_sekolah);
        form.append("email", this.admin.email);
        form.append("kontak", this.admin.kontak);
        form.append("username", this.admin.username);

        if (this.require_password) {
          form.append("password", this.admin.password);
        }

        axios.post(base_url+"/admin-sekolah", form, conf)
        .then(response => {
          this.$bvToast.hide("loadingToast");
          this.getAdmin();
          this.message = response.data.message;
          this.$bvToast.show("message");
        })
        .catch(error => {
          console.log(error);
        });
      },

      get_jurnal : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        axios.get(base_url+"/jurnal-sekolah/"+this.admin.id_sekolah, conf)
        .then(response => {
          this.countJurnal = response.data.count;
        })
        .catch(error => {
          console.log(error);
        });
      },

      get_surat : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        axios.get(base_url+"/surat-sekolah/"+this.admin.id_sekolah, conf)
        .then(response => {
          this.countSurat = response.data.count;
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

          let form = new FormData();
          form.append("token", token);
          axios.post(base_url + "/admin-sekolah/check", form)
          .then(response => {
            this.admin = response.data.admin_sekolah;
            this.get_jurnal();
            this.get_surat();
            this.getAdmin();
          })
          .catch(error => {
            console.log(error);
          })
        }
      },
    },

    mounted(){
      this.authenticate();
    }
  }
</script>
