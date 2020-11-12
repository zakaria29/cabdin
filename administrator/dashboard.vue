<template>
  <!-- Main content -->
  <section class="content">
    <div class="container-fluid">
      <div class="row mb-2">
        <div class="col-sm-6">
          <h3>Dashboard</h3>
        </div>
      </div>
      <div class="row mb-2">
        <div class="col-sm-6">
          <div class="card card-widget widget-user shadow">
              <!-- Add the bg color to the header using any of the bg-* classes -->
              <div class="widget-user-header bg-success">
                <h3 class="widget-user-username">Cabang Dinas Pendidikan</h3>
                <h5 class="widget-user-desc">Wilayah Malang-Batu</h5>
              </div>
              <div class="widget-user-image">
                <img class="img-circle elevation-2" src="../assets/cabdin.png"
                alt="Cabdin Malang">
              </div>
              <div class="card-footer">
                <div class="row">
                  <div class="col-sm-4 border-right">
                    <div class="description-block">
                      <h5 class="description-header">{{ summary.sd }}</h5>
                      <span class="description-text">SD</span>
                    </div>
                    <!-- /.description-block -->
                  </div>
                  <!-- /.col -->
                  <div class="col-sm-4 border-right">
                    <div class="description-block">
                      <h5 class="description-header">{{ summary.smp }}</h5>
                      <span class="description-text">SMP</span>
                    </div>
                    <!-- /.description-block -->
                  </div>
                  <!-- /.col -->
                  <div class="col-sm-4">
                    <div class="description-block">
                      <h5 class="description-header">{{ summary.sma }}</h5>
                      <span class="description-text">SMA/SMK</span>
                    </div>
                    <!-- /.description-block -->
                  </div>
                  <!-- /.col -->
                </div>
                <!-- /.row -->
              </div>
            </div>
        </div>
        <div class="col-sm-6">
          <!-- small box -->
          <div class="small-box bg-primary">
            <div class="inner">
              <h3>{{ countSurat }}</h3>

              <p>Surat</p>
            </div>
            <div class="icon">
              <i class="fa fa-envelope"></i>
            </div>
            <!-- <a href="#" class="small-box-footer">More info <i class="fas fa-arrow-circle-right"></i></a> -->
          </div>

          <div class="small-box bg-warning">
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
  </section>
</template>
<script type="text/javascript">
  module.exports = {
    data : function(){
      return {
        key: "",
        countSurat: 0,
        countJurnal: 0,
        summary: {
          sd: 0, smp: 0, sma: 0
        }
      }
    },

    methods: {
      get_jurnal : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        axios.get(base_url+"/jurnal", conf)
        .then(response => {
          this.countJurnal = response.data.count;
        })
        .catch(error => {
          console.log(error);
        });
      },

      get_surat : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        axios.get(base_url+"/surat", conf)
        .then(response => {
          this.countSurat = response.data.count;
        })
        .catch(error => {
          console.log(error);
        });
      },

      get_summary : function(){
        let conf = { headers: { "Api-Token" : this.key} };
        axios.get(base_url+"/sekolah-summary", conf)
        .then(response => {
          this.summary.sd = response.data.sd;
          this.summary.smp = response.data.smp;
          this.summary.sma = response.data.sma;
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
          this.get_surat();
          this.get_jurnal();
          this.get_summary();
        }
      },
    },

    mounted(){
      this.authenticate();
    }
  }
</script>
