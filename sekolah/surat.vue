<template>
  <!-- Main content -->
  <section class="content">
    <div class="container-fluid">
      <div class="row mb-2">
        <div class="col-sm-12">
          <h3>Data Surat</h3>
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
              <h4 class="card-title">Daftar Surat</h4>
            </div>

          </div>

        </div>
        <div class="card-body" style="overflow:auto">
          <b-form-input v-model="find" placeholder="Pencarian..."
          class="mb-3" v-on:keyup.enter="get_surat"></b-form-input>
          <ul class="list-group">
            <li class="list-group-item mb-2" v-for="item in surat">
              <b-row>
                <b-col cols="10">
                  <h5 class="text-info">{{ item.prihal }}</h5>
                  <h6>Nomor: {{ item.nomor_surat }}</h6>
                  <h6>Sekolah: {{ item.sekolah.nama_sekolah }}</h6>
                  <h6>Waktu: {{ item.waktu }}</h6>
                </b-col>
                <b-col>
                  <b-button variant="info" size="sm" v-on:click="previewFile(item)"
                  v-b-modal.modal_preview block class="mb-2">
                      <span class="fa fa-eye"></span> View File
                  </b-button>
                  <a :href="item.link" target="_blank" download>
                    <b-button variant="success" size="sm" block>
                        <span class="fa fa-download"></span> Download File
                    </b-button>
                  </a>
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
            v-on:input="get_surat">
          </b-pagination>
        </div>
      </div>
    </div>

    <b-modal id="modal_preview"
    size="xl"
    :title="file_title"
    header-bg-variant="info"
    header-text-variant="white"
    border-variant="info" hide-footer>

    <iframe :src="source_file" width="100%" height="500"></iframe>
    <!-- <template v-slot:modal-header>
      <b-button variant="secondary" @click="$bvModal.hide('modal_preview_file')">
          Close
      </b-button>
    </template> -->
  </b-modal>

    <b-modal
      id="modal_surat"
      title="Form Surat"
      header-bg-variant="light"
      border-variant="light"
      hide-footer>

      <b-container fluid>
        <form v-on:submit.prevent="Save">
          Prihal
          <textarea v-model="formSurat.prihal" rows="3" class="form-control" required>
          </textarea>
          Nomor Surat
          <b-form-input v-model="formSurat.nomor_surat" class="mb-2" required>
          </b-form-input>
          Sekolah
          <b-form-select v-model="formSurat.id_sekolah" class="mb-2" required>
            <option v-for="item in sekolah" :value="item.id_sekolah">
              {{ item.nama_sekolah }}
            </option>
          </b-form-select>
          <b-form-checkbox v-model="formSurat.require_file" value="true" unchecked-value="false"
          v-if="formSurat.action === 'update'">
             Re-upload File
          </b-form-checkbox>

          <div v-if="formSurat.require_file">
            File Surat
            <b-form-file v-model="formSurat.file_surat" class="mb-2"
            :required="formSurat.require_file" accept="application/pdf,image/jpeg,image/x-png">
            </b-form-file>
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
        formSurat: {
          action: "",
          id_surat: "",
          prihal: "",
          nomor_surat: "",
          file_surat: null,
          id_sekolah: "",
          require_file: true
        },
        pagination: {
          currentPage: 1,
          rows: 0,
          perPage: 10,
        },
        find: "",
        surat: [],
        sekolah: [],
        message: "",
        key: "",
        source_file: "",
        file_title: "",
      }
    },

    methods: {

      get_surat : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        let offset = (this.pagination.currentPage - 1) * this.pagination.perPage;
        this.$bvToast.show("loadingToast");
        if(this.find === ""){
          axios.get(base_url+"/surat-sekolah/"+this.formSurat.id_sekolah+"/"+this.pagination.perPage+"/"+offset, conf)
          .then(response => {
            this.$bvToast.hide("loadingToast");
            this.surat = response.data.surat;
            this.pagination.rows = response.data.count;
          })
          .catch(error => {
            console.log(error);
          });
        }else{
          let form = new FormData();
          form.append("find", this.find);
          axios.post(base_url+"/surat-sekolah/find/"+this.formSurat.id_sekolah+"/"+this.pagination.perPage+"/"+offset, form, conf)
          .then(response => {
            this.$bvToast.hide("loadingToast");
            this.surat = response.data.surat;
            this.pagination.rows = response.data.count;
          })
          .catch(error => {
            console.log(error);
          });
        }
      },
      Add : function(){
        this.formSurat.action = "insert";
        this.formSurat.id_surat = "";
        this.formSurat.prihal = "";
        this.formSurat.nomor_surat = "";
        this.formSurat.id_sekolah = "";
        this.formSurat.file_surat = null;
        this.formSurat.require_file = true;
      },
      Edit : function(item){
        this.formSurat.action = "update";
        this.formSurat.id_surat = item.id_surat;
        this.formSurat.prihal = item.prihal;
        this.formSurat.nomor_surat = item.nomor_surat;
        this.formSurat.id_sekolah = item.id_sekolah;
        this.formSurat.require_file = false;
        this.formSurat.file_surat = null;
      },
      Save : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        this.$bvToast.show("loadingToast");
        this.$bvModal.hide("modal_surat");
        let form = new FormData();
        form.append("action", this.formSurat.action);
        form.append("id_surat", this.formSurat.id_surat);
        form.append("prihal", this.formSurat.prihal);
        form.append("nomor_surat", this.formSurat.nomor_surat);
        form.append("id_sekolah", this.formSurat.id_sekolah);

        if (this.formSurat.require_file) {
          form.append("file", this.formSurat.file_surat);
        }

        axios.post(base_url+"/surat", form, conf)
        .then(response => {
          this.$bvToast.hide("loadingToast");
          this.get_surat();
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
          axios.delete(base_url+"/surat/"+id, conf)
          .then(response => {
            this.get_surat();
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
          window.location = "../login.html";
          console.log("token is gone");
        }else{
          let token = this.$cookies.get("Api-Token");
          this.key = token;

          let form = new FormData();
          form.append("token", token);
          axios.post(base_url + "/admin-sekolah/check", form)
          .then(response => {
            console.log(response.data);
            this.formSurat.id_sekolah = response.data.admin_sekolah.id_sekolah;
            this.get_surat();
          })
          .catch(error => {
            console.log(error);
          })

        }
      },

      previewFile: function(item){
        this.file_title = item.file_surat;
        let url = new URL(item.link);
        let id = url.searchParams.get("id");
        this.source_file = "https://drive.google.com/file/d/"+id+"/preview";
      },
    },

    mounted(){
      this.authenticate();

    }
  }
</script>
