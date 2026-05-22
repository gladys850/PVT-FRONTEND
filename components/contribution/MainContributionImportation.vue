<template>
  <!--2DO PROCESO "IMPORTACIÓN DE APORTES" COMANDO Y SENASIR --->
  <v-container fluid>
    <v-card flat>
      <v-card-title>
        <v-toolbar dense color="tertiary" class="caption">
          <GlobalBreadCrumb />
          <div class="flex-grow-1"></div>
          <v-btn-toggle
            v-model="active"
            active-class="secondary white--text"
            mandatory
          >
            <span  v-for="item in items_import" :key="item.name">
              <v-btn
                v-if="permissionSimpleSelected.includes(item.permissions_create)"
                :value="item.name"
              > {{item.name}}</v-btn>
            </span>
          </v-btn-toggle>
          <v-divider class="mx-2" inset vertical></v-divider>
          <v-select
            :items="years"
            :loading="loading"
            label="Gestión"
            v-model="year_selected"
            dense
            outlined
            class="select-year"
            close-on-click
          ></v-select>
        </v-toolbar>
      </v-card-title>

    </v-card>
    <!--contenido-->

      <div v-if="loading_circular">
        <GlobalLoading />
      </div>

    <v-row justify="center" class="py-0 mt-2" v-if="!loading_circular">
      <v-card
        :class="item.state_importation ? 'headline font-weight-bold ma-2 backgroundCard' : 'headline font-weight-bold ma-2'"
        max-width="250px"
        v-for="(item, i) in list_months"
        :key="'month-' + item.period_month"
      >
        <template v-if="item.state_validated_payroll">
          <v-card-title :class="item.state_importation ? 'accent' : 'normal'">
            <v-row justify="center">
              <h3 class="white--text">{{ item.period_month_name }}</h3>
            </v-row>
          </v-card-title>
          <v-divider inset></v-divider>
          <v-card-text>
            <v-row>

              <v-col cols="12" md="12" class="py-0">
                <span class="info--text">N° reg. validados: </span
                ><strong>{{$filters.thousands(item.data_count.num_data_validated) }}</strong
                >

                <v-tooltip top>
                    <template v-slot:activator="{ on }">
                      <v-btn 
                        class="ma-2 accent white--text btn-period" 
                        v-on="on"
                        @click="confirmImportContribution(item.period_month, true, 'mensual')"
                        :disabled="item.state_importation"
                      >
                        <v-icon dark left small>mdi-arrow-down</v-icon>Imp. Aportes
                      </v-btn>
                    </template>
                    <div>
                      <span>Importar Aportes</span>
                    </div>
                </v-tooltip>
  
                <div v-show="item.state_importation">
                  <template v-if="type_import.name == 'SENASIR'">
                    <span class="info--text">N° reg. importados: </span><strong>{{$filters.thousands(item.data_count.num_total_data_contribution_passives)}}</strong><br>
                    <span class="info--text">Total aportes Bs.: </span><strong>{{$filters.money(item.data_count.sum_amount_total_contribution_passives)}}</strong><br>
                   </template>
                  <template v-if="type_import.name == 'COMANDO'">
                    <span class="info--text">N° reg. importados: </span><strong>{{$filters.thousands(item.data_count.num_total_data_contributions)}}</strong><br>
                    <span class="info--text">Total aportes Bs.: </span><strong>{{$filters.money(item.data_count.sum_amount_total_contributions)}}</strong><br>
                  </template>
                  <div class="text-right pb-1" >
                    <v-tooltip top class="my-0" v-if="permissionSimpleSelected.includes(type_import.permissions_download)">
                      <template v-slot:activator="{ on }">
                        <v-btn
                          small
                          :color="'primary'"
                          fab
                          v-on="on"
                          :loading="loading_rep_state && i == loading_pos_index"
                          @click.stop="loading_pos_index = i; reportImportContribution(item.period_month)"
                        >
                          <v-icon>mdi-file-document</v-icon>
                        </v-btn>
                      </template>
                      <div>
                        <span>Detalle de Aportes</span>
                      </div>
                    </v-tooltip>
                  </div>
                </div>

              </v-col>
            </v-row>
          </v-card-text>
        </template>
      </v-card>
      <template v-if="type_import.name == 'COMANDO'">
      <v-card
        :class="item.state_importation ? 'headline font-weight-bold ma-2 backgroundCard' : 'headline font-weight-bold ma-2'"
        max-width="250px"
        v-for="(item, i) in list_months_re"
        :key="'re-' + item.period_month"
      >
        <template v-if="item.state_validated_payroll">
          <v-card-title :class="item.state_importation ? 'accent' : 'normal'">
            <v-row justify="center">
              <h3 class="white--text">Reintegro {{ item.period_month_name }}</h3>
            </v-row>
          </v-card-title>
          <v-divider inset></v-divider>
          <v-card-text>
            <v-row>

              <v-col cols="12" md="12" class="py-0">
                <span class="info--text">N° reg. validados: </span
                ><strong>{{$filters.thousands(item.data_count.num_data_validated) }}</strong
                >

                <v-tooltip top>
                    <template v-slot:activator="{ on }">
                      <v-btn 
                        class="ma-2 accent white--text btn-period" 
                        v-on="on"
                        @click="confirmImportContribution(item.period_month, true, 'reintegro')"
                        :disabled="item.state_importation"
                      >
                        <v-icon dark left small>mdi-arrow-down</v-icon>Imp. Aportes
                      </v-btn>
                    </template>
                    <div>
                      <span>Importar Aportes</span>
                    </div>
                </v-tooltip>
  
                <div v-show="item.state_importation">
                    <span class="info--text">N° reg. importados: </span><strong>{{$filters.thousands(item.data_count.num_total_data_contributions)}}</strong><br>
                    <span class="info--text">Total aportes Bs.: </span><strong>{{$filters.money(item.data_count.sum_amount_total_contributions)}}</strong><br>
                  <div class="text-right pb-1" >
                    <v-tooltip top class="my-0" v-if="permissionSimpleSelected.includes(type_import.permissions_download)">
                      <template v-slot:activator="{ on }">
                        <v-btn
                          small
                          :color="'Secondary'"
                          fab
                          v-on="on"
                          :loading="loading_rep_state_re && i == loading_pos_index_re"
                          @click.stop="loading_pos_index_re = i; reportImportReimbursement(item.period_month, 'reintegro')"
                        >
                          <v-icon>mdi-file-document</v-icon>
                        </v-btn>
                      </template>
                      <div>
                        <span>Detalle de Aportes</span>
                      </div>
                    </v-tooltip>
                  </div>
                </div>

              </v-col>
            </v-row>
          </v-card-text>
        </template>
      </v-card>
      <!--ADICIONALES-->
      <v-card
        :class="item.state_importation ? 'headline font-weight-bold ma-2 backgroundCard' : 'headline font-weight-bold ma-2'"
        max-width="250px"
        v-for="(item, i) in list_months_ad"
        :key="'ad-' + item.period_month"
      >
        <template v-if="item.state_validated_payroll">
          <v-card-title :class="item.state_importation ? 'accent' : 'normal'">
            <v-row justify="center">
              <h3 class="white--text">Adicional {{ item.period_month_name }}</h3>
            </v-row>
          </v-card-title>
          <v-divider inset></v-divider>
          <v-card-text>
            <v-row>

              <v-col cols="12" md="12" class="py-0">
                <span class="info--text">N° reg. validados: </span
                ><strong>{{$filters.thousands(item.data_count.num_data_validated) }}</strong
                >

                <v-tooltip top>
                    <template v-slot:activator="{ on }">
                      <v-btn 
                        class="ma-2 accent white--text btn-period" 
                        v-on="on"
                        @click="confirmImportContribution(item.period_month, true, 'adicional')"
                        :disabled="item.state_importation"
                      >
                        <v-icon dark left small>mdi-arrow-down</v-icon>Imp. Aportes
                      </v-btn>
                    </template>
                    <div>
                      <span>Importar Aportes</span>
                    </div>
                </v-tooltip>
  
                <div v-show="item.state_importation">
                    <span class="info--text">N° reg. importados: </span><strong>{{$filters.thousands(item.data_count.num_total_data_contributions)}}</strong><br>
                    <span class="info--text">Total aportes Bs.: </span><strong>{{$filters.money(item.data_count.sum_amount_total_contributions)}}</strong><br>
                  <div class="text-right pb-1" >
                    <v-tooltip top class="my-0" v-if="permissionSimpleSelected.includes(type_import.permissions_download)">
                      <template v-slot:activator="{ on }">
                        <v-btn
                          small
                          :color="'info'"
                          fab
                          v-on="on"
                          :loading="loading_rep_state_ad && i == loading_pos_index_ad"
                          @click.stop="loading_pos_index_ad = i; reportImportReimbursement(item.period_month, 'adicional')"
                        >
                          <v-icon>mdi-file-document</v-icon>
                        </v-btn>
                      </template>
                      <div>
                        <span>Detalle de Aportes</span>
                      </div>
                    </v-tooltip>
                  </div>
                </div>

              </v-col>
            </v-row>
          </v-card-text>
        </template>
      </v-card>
      </template>
    </v-row>
    <!--fin contenido-->

    <v-dialog 
      v-model="dialog_confirm_import_contribution" 
      max-width="500"
      persistent
    >
      <v-card>
        <v-card-title> Esta seguro de Importar los aportes? </v-card-title>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn 
            color="error" 
            text 
            @click="dialog_confirm_import_contribution = false"
            :disabled="btn_import_contributions"
          >
            Cerrar
          </v-btn>
          <v-btn
            color="sucess"
            text
            @click="ImportContributions()"
            :loading="btn_import_contributions"
          >
            Aceptar
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
import GlobalBreadCrumb from "@/components/common/GlobalBreadCrumb.vue"
import GlobalLoading from "@/components/common/GlobalLoading.vue"
export default {
  name: "MainContributionImportation",
  components: {
    GlobalBreadCrumb,
    GlobalLoading
  },
  data: () => ({
    active: "SENASIR",
    years: [],
    loading: false,
    year_selected: null,
    list_months: [],
    list_months_re: [],
    list_months_ad: [],
    dialog: false,
    month_selected: null,
    data_count:{
      num_data_validated: 0,  //senasir , command
      sum_amount_total_contribution_passives: 0, //senasir
      num_total_data_contribution_passives: 0, //senasir
      num_total_data_contributions: 0, //command
      sum_amount_total_contributions: 0 //command
    },
    btn_import_contributions: false,
    dialog_confirm_import_contribution: false,
    loading_circular: false,
    loading_pos_index: -1,
    loading_pos_index_re: -1,
    loading_pos_index_ad: -1,
    loading_rep_state: false,
    loading_rep_state_re: false,
    loading_rep_state_ad: false,
    items_import: [],
    type_import:{},
    import_type_name: {
      SENASIR: 'senasir',
      COMANDO: 'command',
    },
    type_payroll: 'mensual'
  }),
  created() {
     this.items_import= [
      {
        id: 1,
        name: 'SENASIR',
        permissions_create: 'create-import-senasir',
        permissions_download: 'download-report-senasir',
        route_get_months: '/contribution/list_months_import_contribution_senasir',
        route_import_contribution: '/contribution/import_create_or_update_contribution_period_senasir', //Creacion de aportes
        route_download: '/contribution/report_import_contribution_senasir',
        name_download_file: "ReporteDetalleAportesSenasir.xls"
      },
      {
        id: 2,
        name: 'COMANDO',
        permissions_create: 'create-import-command',
        permissions_download: 'download-report-command',
        route_get_months: '/contribution/list_months_import_contribution_command',
        route_import_contribution: '/contribution/import_contribution_command', //Creacion de aportes
        route_download: '/contribution/report_import_contribution_command',
        name_download_file: "ReporteDetalleAportesComando.xls"
      }
    ],
    this.getYears();
    this.type_import = this.items_import[0]
     //seleccionar el mes en caso de null
    if(this.year_selected){
      this.getMonths()
    }else{
      this.year_selected = new Date().getFullYear()
      this.getMonths()
    }
   // this.getMonths()
  },
  computed: {
    //permisos del selector global por rol
    permissionSimpleSelected () {
      return this.$store.getters.permissionSimpleSelected
    },
    dateFormat() {
      if (this.month_selected < 10)
        return (
          this.year_selected + "-" + "0" + this.month_selected + "-" + "01"
        );
      else return this.year_selected + "-" + this.month_selected + "-" + "01";
    },
  },
  watch: {
    active(newVal, oldVal) {
     if (newVal != oldVal) {
       for(let i=0; i < this.items_import.length; i++){
         if(this.active == this.items_import[i].name){
           this.type_import = this.items_import[i]
         }
       }
       this.getYears()
       this.getMonths()
     }
   },

    year_selected(newVal, oldVal) {
      if (newVal != oldVal) {
        this.getMonths();
      }
    },
  },
  methods: {
    async getYears() {
      try {
        this.loading = true;
        let import_type_name = this.import_type_name[this.type_import.name]
        let res = await this.$axios.get(`/contribution/list_years/${import_type_name}`)
        this.years = res.payload.list_years;
        this.year_selected = this.years[0];
        this.loading = false;
      } catch (e) {
        console.log(e);
      }
    },
    async getMonths() {
      this.loading_circular = true
      try {
        let res = await this.$axios.post(`${this.type_import.route_get_months}`,{
            period_year: this.year_selected,
          }
        );
        this.list_months = res.payload.list_months
        if(this.type_import.name == 'COMANDO'){
          this.list_months_re = res.payload.list_months_re
          this.list_months_ad = res.payload.list_months_ad
        }
        this.loading_circular = false
        console.log(this.year_selected);
      } catch (e) {
        console.log(e);
        this.loading_circular = false
      }
    },
    close() {
      this.dialog = false;
      this.clearData();
    },
    //PASO3
    async ImportContributions() {
      this.btn_import_contributions = true;
      try {
        let params ={
          period_contribution: this.dateFormat
        }
        if(this.type_import.name == 'COMANDO'){
          params.type_payroll = this.type_payroll;
        }
        let res = await this.$axios.post(`${this.type_import.route_import_contribution}`,params);
        if (res.payload.successfully) {
          this.$toast.success("Total de registros importados: "+ res.payload.num_created)
          this.dialog_confirm_import_contribution = false
          this.getMonths();
        } else {
          this.$toast.error(res.message);
        }
        this.btn_import_contributions = false
      } catch (e) {
        console.log(e);
        this.$toast.error(e.message || 'Error al realizar la importación')
        this.btn_import_contributions = false
        this.dialog_confirm_import_contribution= false
      }
    },
    confirmImportContribution(month_selected, dialog, type_payroll){
      this.month_selected = month_selected
      this.dialog_confirm_import_contribution = dialog
      this.type_payroll = type_payroll
      console.log( month_selected)
    },

    async reportImportContribution(month_selected){
      this.month_selected = month_selected
      this.loading_rep_state=true;
      try {
        let res = await this.$axios.post(`${this.type_import.route_download}`,{
            date_contribution: this.dateFormat
          },
          {'Accept': 'application/vnd.ms-excel' },
          {'responseType': 'blob'}
        );
        const url = window.URL.createObjectURL(new Blob([res]))
        const link = document.createElement("a")
        link.href = url;
        link.setAttribute("download", `${this.type_import.name_download_file}`)
        document.body.appendChild(link)
        link.click()
      } catch (e) {
        console.log(e);
      } finally {
        this.loading_rep_state=false;
        this.loading_pos_index=-1;
      }
    },
    async reportImportReimbursement(month_selected, type_payroll){
      this.month_selected = month_selected
      if(type_payroll == 'reintegro') this.loading_rep_state_re=true;
      else this.loading_rep_state_ad=true;

      try {
        let res = await this.$axios.post(`/contribution/report_import_reimbursement_command`,{
            date_contribution: this.dateFormat,
            type_payroll: type_payroll
          },
          {'Accept': 'application/vnd.ms-excel' },
          {'responseType': 'blob'}
        );
        const url = window.URL.createObjectURL(new Blob([res]))
        const link = document.createElement("a")
        link.href = url;
        let file_name = type_payroll == 'reintegro' ? 'ReporteDetalleReintegrosComando.xls' : 'ReporteDetalleAdicionalesComando.xls';
        link.setAttribute("download", file_name)
        document.body.appendChild(link)
        link.click()
      } catch (e) {
        console.log(e);
      } finally {
        this.loading_rep_state_re=false;
        this.loading_rep_state_ad=false;
        this.loading_pos_index_re=-1;
        this.loading_pos_index_ad=-1;
      }
    },
  },
};
</script>
<style scoped>
.select-year {
  max-width: 100px;
  margin-bottom: -30px;
}
.btn-period.v-btn:not(.v-btn--round).v-size--default {
  min-width: 160px;
  height: 25px;
}
</style>