<template>
    <div >
      <div class="container mt-3">
          <div class="row">
            <div class="col-lg-12 grid-margin stretch-card">
              <div class="card">
                <div class="card-body">
                  <p class="card-title float-left"><b>Data Poin Siswa</b></p>
                  <div class="table-responsive">
                    <b-form-input type="text" v-model="search" placeholder="Pencarian..." class="mb-3" v-on:keyup.enter="searching"></b-form-input>
                    <b-table striped hover :items="data_poin" :fields="fields">
                      <template v-slot:cell(total_poin)="data">
                        <h5><b-badge variant="warning">{{ data.item.total_poin }}</b-badge></h5>
                      </template>
                      <template v-slot:cell(Aksi)="data">
                        <b-button size="sm" variant="warning" v-on:click="Detail(data.item)" v-b-modal.modalTatib><i class="mdi mdi-file-document btn-icon-prepend"></i> Detail</b-button>&nbsp;
                      </template>
                  </b-table>
                  <b-pagination
                    v-model="currentPage"
                    :per-page="perPage"
                    :total-rows="rows"
                    align="center"
                    v-on:input="pagination">
                  </b-pagination>

                  <!-- <b-toast id="loadingToast" title="Processing Data" no-auto-hide>
                  <b-spinner label="Spinning" variant="success"></b-spinner>
                  <strong class="text-success">Loading...</strong>
                  </b-toast> -->

                <!-- toast untuk tampilan message box -->
                <!-- <b-toast id="message" title="Message">
                  <strong class="text-success">{{ message }}</strong>
                </b-toast> -->
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>

        <b-modal 
      id="modalTatib"
      @ok="Save"
    >
    <template v-slot:modal-title>
      Detail Poin Siswa
    </template>
    <b-table striped hover :items="detail_poin" :fields="fields_details">
      <template v-slot:cell(poin)="data">
    <h5><b-badge variant="primary">{{ data.item.poin }}</b-badge></h5>
      </template>
    </b-table>
    </b-modal>
      </div>
</template>

<script>
module.exports = {
  data : function(){
    return {
      search: "",
      id: "",
      nis: "",
      id_siswa: "",
      nama_siswa: "", 
      kelas: "",
      total_poin: "",
      poin: "",
      tanggal: "",
      nama_pelanggaran:"",
      kategori:"",
      keterangan:"",
      action: "",
      message: "",
      currentPage: 1,
      rows: 0,
      perPage: 10,
      key: "",
      fields: ["nis", "nama_siswa", "kelas", "total_poin", "Aksi"],
      fields_detail: ["tanggal", "nama_pelanggaran", "kategori", "keterangan", "poin"],
      data_poin: [],
      detail_poin: [],
    }
  },

methods: {
getData : function(){
      let conf = { headers: { "Authorization" : 'Bearer ' + this.key } };
      let offset = (this.currentPage - 1) * this.perPage;
      this.$bvToast.show("loadingToast");
      this.axios.get("/poin_siswa/" + this.perPage + "/" + offset, conf)
      .then(response => {
        if(response.data.status == 1){
          this.$bvToast.hide("loadingToast");
          this.data_poin = response.data.poin;
          this.rows = response.data.count;
        } else {
          this.$bvToast.hide("loadingToast");
          this.message = "Gagal menampilkan data poin semua siswa."
          this.$bvToast.show("message");
          this.$router.push({name: "login"})
        }
        
      })
      .catch(error => {
        console.log(error);
      });
    },
    Detail: function(item){
      let conf = { headers: {"Authorization" : 'Bearer' + this.key}};
      this.axios.get("/poin_siswa/" + item.id, conf)
      .then(response => {
        this.detail_poin = response.data.detail
      })
    },
    searching : function(){
      let conf = { headers: { "Authorization" : 'Bearer ' + this.key } };
      let offset = (this.currentPage - 1) * this.perPage;
      this.$bvToast.show("loadingToast");
      let form = new FormData();
      form.append("find", this.search);
      this.axios.post("/poin_siswa/" + this.perPage + "/" + offset, form, conf)
      .then(response => {
        if(response.data.status == 1){
          this.$bvToast.hide("loadingToast");
          this.data_poin = response.data.poin;
          this.rows = response.data.count;
        }
      })
      .catch(error => {
          console.log(error);
      });
    },
    
   pagination : function(){
      if(this.search == ""){
        this.getData();
      } else {
        this.searching();
      }
    },
  },
  mounted(){
    this.key = localStorage.getItem("Authorization");
    this.getData();
  }
}
</script>