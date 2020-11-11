<template>
  <!-- Main content -->
  <section class="content">
    <div class="container-fluid">
      <div class="row mb-2">
        <div class="col-sm-12">
          <h3>Data Jurnal</h3>
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
              <h4 class="card-title">Daftar Jurnal</h4>
            </div>
            <div class="col-lg-2 col-sm-4">
              <b-button @click="Add" variant="light" pill class="pull-right"
              type="button" size="sm" v-b-modal.modal_jurnal style="float:right">
                 <span class="fa fa-plus"></span> Tambah Data
              </b-button>
            </div>
          </div>

        </div>
        <div class="card-body" style="overflow:auto">
          <b-form-input v-model="find" placeholder="Pencarian..."
          class="mb-3" v-on:keyup.enter="get_jurnal"></b-form-input>
          <ul class="list-group">
            <li class="list-group-item mb-2" v-for="item in jurnal">
              <b-row>
                <b-col cols="10">
                  <h5 class="text-info">{{ item.judul_jurnal }}</h5>
                  <h6>Penulis: {{ item.penulis }}</h6>
                  <h6>Sekolah: {{ item.sekolah.nama_sekolah }}</h6>
                  <h6>Kategori: {{ item.kategori }}</h6>
                  <h6 v-if="item.file_jurnal === null">
                    Link: <a :href="item.link" target="_blank">{{ item.link }}</a>
                  </h6>
                </b-col>
                <b-col>
                  <b-button variant="success" class="btn-sm" v-on:click="previewFile(item)"
                  v-b-modal.modal_preview block v-if="item.file_jurnal !== null">
                      <span class="fa fa-eye"></span> View File
                  </b-button>
                  <b-button variant="info" class="btn-sm" v-on:click="Edit(item)"
                  v-b-modal.modal_jurnal block>
                      <span class="fa fa-edit"></span> Edit
                  </b-button>
                  <b-button variant="danger" class="btn-sm"
                  v-on:click="Drop(item.id_jurnal)" block>
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
            v-on:input="get_jurnal">
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
      id="modal_jurnal"
      title="Form Jurnal"
      header-bg-variant="light"
      border-variant="light"
      hide-footer>

      <b-container fluid>
        <form v-on:submit.prevent="Save">
          Judul Jurnal <sup class="text-danger">*</sup>
          <textarea v-model="formJurnal.judul_jurnal" rows="3" class="form-control" required>
          </textarea>
          Penulis <sup class="text-danger">*</sup>
          <b-form-input v-model="formJurnal.penulis" class="mb-2" required>
          </b-form-input>
          Kategori <sup class="text-danger">*</sup>
          <b-form-input v-model="formJurnal.kategori" class="mb-2" required>
          </b-form-input>
          Sekolah <sup class="text-danger">*</sup>
          <b-form-select v-model="formJurnal.id_sekolah" class="mb-2" required>
            <option v-for="item in sekolah" :value="item.id_sekolah">
              {{ item.nama_sekolah }}
            </option>
          </b-form-select>
          <b-form-group label="Option Attach File">
            <b-row>
              <b-col>
                <b-form-radio v-model="formJurnal.option" name="radios" value="0">Attach Link</b-form-radio>
              </b-col>
              <b-col>
                <b-form-radio v-model="formJurnal.option" name="radios" value="1">Upload File</b-form-radio>
              </b-col>
            </b-row>
          </b-form-group>

          <div v-if="formJurnal.option === '0'">
            URL File
            <b-form-input type="url" v-model="formJurnal.link" class="mb-2"
            :required="formJurnal.option === '0'">
            </b-form-input>
          </div>
          <!-- <b-form-checkbox v-model="formJurnal.require_file" value="true" unchecked-value="false"
          v-if="formJurnal.action === 'update'">
             Re-upload File
          </b-form-checkbox> -->

          <div v-if="formJurnal.option === '1'">
            <b-form-checkbox v-model="formJurnal.require_file" value="true" unchecked-value="false"
            v-if="formJurnal.action === 'update'">
               Re-upload File
            </b-form-checkbox>
            File Jurnal
            <b-form-file v-model="formJurnal.file_jurnal" class="mb-2"
            :required="formJurnal.require_file" accept="application/pdf"
            v-if="formJurnal.require_file">
            </b-form-file>
          </div>

           <b-button variant="primary" block pill class="pull-right" type="submit">
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
        formJurnal: {
          option: "",
          action: "",
          id_jurnal: "",
          judul_jurnal: "",
          penulis: "",
          kategori: "",
          file_jurnal: null,
          id_sekolah: "",
          require_file: true,
          link: "",
        },
        pagination: {
          currentPage: 1,
          rows: 0,
          perPage: 10,
        },
        find: "",
        jurnal: [],
        sekolah: [],
        message: "",
        key: "",
        source_file: "",
        file_title: "",
      }
    },

    methods: {
      get_sekolah : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        axios.get(base_url+"/sekolah", conf)
        .then(response => {
          this.sekolah = response.data.sekolah;
        })
        .catch(error => {
          console.log(error);
        });
      },
      get_jurnal : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        let offset = (this.pagination.currentPage - 1) * this.pagination.perPage;
        this.$bvToast.show("loadingToast");
        if(this.find === ""){
          axios.get(base_url+"/jurnal/"+this.pagination.perPage+"/"+offset, conf)
          .then(response => {
            this.$bvToast.hide("loadingToast");
            this.jurnal = response.data.jurnal;
            this.pagination.rows = response.data.count;
          })
          .catch(error => {
            console.log(error);
          });
        }else{
          let form = new FormData();
          form.append("find", this.find);
          axios.post(base_url+"/jurnal/find/"+this.pagination.perPage+"/"+offset, form, conf)
          .then(response => {
            this.$bvToast.hide("loadingToast");
            this.jurnal = response.data.jurnal;
            this.pagination.rows = response.data.count;
          })
          .catch(error => {
            console.log(error);
          });
        }
      },
      Add : function(){
        this.formJurnal.action = "insert";
        this.formJurnal.id_jurnal = "";
        this.formJurnal.judul_jurnal = "";
        this.formJurnal.penulis = "";
        this.formJurnal.kategori = "";
        this.formJurnal.id_sekolah = "";
        this.formJurnal.link = "";
        this.formJurnal.option = "";
        this.formJurnal.file_jurnal = null;
        this.formJurnal.require_file = true;
      },
      Edit : function(item){
        this.formJurnal.action = "update";
        this.formJurnal.id_jurnal = item.id_jurnal;
        this.formJurnal.judul_jurnal = item.judul_jurnal;
        this.formJurnal.penulis = item.penulis;
        this.formJurnal.kategori = item.kategori;
        this.formJurnal.id_sekolah = item.id_sekolah;
        this.formJurnal.require_file = false;
        this.formJurnal.file_jurnal = null;
        if (item.file_jurnal === null) {
          this.formJurnal.link = item.link;
          this.formJurnal.option = "0";
        }else{
          this.formJurnal.link = "";
          this.formJurnal.option = "1";
        }
      },
      Save : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        this.$bvToast.show("loadingToast");
        this.$bvModal.hide("modal_jurnal");
        let form = new FormData();
        form.append("action", this.formJurnal.action);
        form.append("id_jurnal", this.formJurnal.id_jurnal);
        form.append("judul_jurnal", this.formJurnal.judul_jurnal);
        form.append("penulis", this.formJurnal.penulis);
        form.append("kategori", this.formJurnal.kategori);
        form.append("id_sekolah", this.formJurnal.id_sekolah);
        if (this.formJurnal.option == "0") {
          form.append("link", this.formJurnal.link);
        } else {
          if (this.formJurnal.require_file) {
            form.append("file", this.formJurnal.file_jurnal);
          }
        }
        // if (this.formJurnal.require_file) {
        //   form.append("file", this.formJurnal.file_jurnal);
        // }

        axios.post(base_url+"/jurnal", form, conf)
        .then(response => {
          this.$bvToast.hide("loadingToast");
          this.get_jurnal();
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
          axios.delete(base_url+"/jurnal/"+id, conf)
          .then(response => {
            this.get_jurnal();
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
          this.get_sekolah();
          this.get_jurnal();
        }
      },

      previewFile: function(item){
        this.file_title = item.file_jurnal;
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
