<template>
  <v-container fluid>
    <v-card flat>
      <v-card-title>
        <v-toolbar dense color="tertiary" class="caption">
          <GlobalBreadCrumb />
          <div class="flex-grow-1"></div>

          <!-- S E L E C C I O N A R   G E S T I Ó N -->
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

          <!-- B O T Ó N   A G R E G A R   G E S T I Ó N -->
          <v-tooltip top v-if="permissionSimpleSelected.includes(item_import.permissions_create)">
            <template v-slot:activator="{ on }">
              <v-btn
                fab
                dark
                x-small
                :color="'success'"
                bottom
                right
                v-on="on"
                class="mx-2"
                @click="openDialog()"
              >
                <v-icon>mdi-plus</v-icon>
              </v-btn>
            </template>
            <div>
              <span>Añadir Periodo</span>
            </div>
          </v-tooltip>
        </v-toolbar>
      </v-card-title>
      <v-card-text> </v-card-text>
    </v-card>

    <div v-if="loading_circular">
      <GlobalLoading />
    </div>

    <!-- C O N T E N I D O   D E   L O S   C A R D S -->

    <v-row justify="center" class="py-0 mt-2" v-if="!loading_circular">
       <v-card
        class="headline font-weight-bold ma-2"
        max-width="250px"
        v-for="(item, i) in list_months"
        :key="i"
      >       
        <template>
          <v-card-title class="accent">
            <v-row justify="center" v-if="item_import.name != 'REGIONAL'">
              <h3 class="white--text">{{ item.period_month_name }}</h3></v-row
            >
            <v-row justify="center" v-else>
              <h3 class="white--text">{{ item.period_month_name }} {{item.period_day}} </h3></v-row
            >
          </v-card-title>
          <v-divider inset></v-divider>
          <v-card-text class="backgroundCard">
            <v-row>
              <v-col cols="12" md="12" class="py-0">
                <b>{{ item_import.name }} <v-icon small>mdi-home-analytics</v-icon></b>
              </v-col>
              <v-divider inset></v-divider>
              <v-col cols="12" md="12" class="py-0">

                <template v-if="item_import.name == 'SENASIR'">
                  <span class="info--text">N° reg. copiados: </span><strong>{{$filters.thousands(item.data_count.num_total_data_copy)}}</strong><br>
                  <span class="info--text">N° reg. considerados: </span><strong>{{$filters.thousands(item.data_count.num_data_considered)}}</strong><br>
                  <span class="error--text">N° reg. no considerados: </span><strong>{{$filters.thousands(item.data_count.num_data_not_considered)}}</strong><br>
                  <span class="info--text">N° reg. validados: </span><strong>{{$filters.thousands(item.data_count.num_data_validated)}}</strong><br>
                </template>
                <template v-if="item_import.name == 'COMANDO'">
                  <span class="info--text">N° reg. copiados: </span><strong>{{$filters.thousands(item.data_count.num_total_data_copy)}}</strong><br>
                  <span class="info--text">N° reg. nuevos: </span><strong>{{$filters.thousands(item.data_count.num_data_new)}}</strong><br>
                  <span class="info--text">N° reg. regulares: </span><strong>{{$filters.thousands(item.data_count.num_data_regular)}}</strong><br>
                </template>
                <template v-if="item_import.name == 'TRANSCRIPCIÓN'">
                  <span class="info--text"></span>
                  <span class="info--text">N° reg. copiados: </span><strong>{{$filters.thousands(item.data_count.num_total_data_copy)}}</strong><br>
                  <span class="info--text">N° reg. enlazados: </span><strong>{{$filters.thousands(item.data_count.count_data_automatic_link)}}</strong><br>
                  <span class="info--text">N° afiliados creados. </span><strong>{{$filters.thousands(item.data_count.count_data_creation)}}</strong><br>
                  <span class="info--text">N° total datos planilla: </span><strong>{{$filters.thousands(item.data_count.num_total_data_payroll)}}</strong><br>
                  <span class="info--text">N° total de aportes: </span><strong>{{$filters.thousands(item.data_count.num_total_data_contribution)}}</strong><br>
                </template>
                <template v-if="item_import.name == 'DISPONIBILIDAD'">
                  <span class="info--text"></span>
                  <span class="info--text">N° reg. copiados: </span><strong>{{$filters.thousands(item.data_count.num_total_data_copy)}}</strong><br>
                  <span class="info--text">N° afiliados actualizados: </span><strong>{{$filters.thousands(item.data_count.num_of_affiliates_updated)}}</strong><br>
                  <span class="info--text">N° afiliados no actualizados. </span><strong>{{$filters.thousands(item.data_count.num_of_affiliates_not_updated)}}</strong><br>
                </template>
                <template v-if="item_import.name == 'REGIONAL'">
                  <span class="info--text"></span>
                  <span class="info--text">N° reg. importados: </span><strong>{{$filters.thousands(item.data_count.num_total_data_contribution)}}</strong><br>
                </template>
                  <div class="text-right pb-1" v-if="permissionSimpleSelected.includes(item_import.permissions_download)">
                    <v-tooltip top class="my-0">
                      <template v-slot:activator="{ on }">
                        <v-btn
                          small
                          :color="'primary'"
                          fab
                          v-on="on"
                          :loading="loading_rep_state && i == loading_pos_index"
                          @click.stop="loading_pos_index = i; reportPayroll(item.period_month,item.period_day, 'mensual')"
                        >
                          <v-icon>mdi-file-document</v-icon>
                        </v-btn>
                      </template>
                      <div>
                        <span>Detalle de Importación de registros válidos</span>
                      </div>
                    </v-tooltip>
                  </div>
              </v-col>
            </v-row>
          </v-card-text>
        </template>
      </v-card>
      <!--REINTEGROS-->
      <template v-if="item_import.name == 'COMANDO'">
        <v-card
          class="headline font-weight-bold ma-2"
          max-width="250px"
          v-for="(item, i) in list_months_re"
          :key="i"
        >
          <template>
            <v-card-title class="secondary">
              <v-row justify="center">
                <h3 class="white--text">Reintegro {{ item.period_month_name }}</h3></v-row
              >
            </v-card-title>
            <v-divider inset></v-divider>
            <v-card-text class="backgroundCard">
              <v-row>
                <v-col cols="12" md="12" class="py-0">
                  <b>{{ item_import.name }} <v-icon small>mdi-home-analytics</v-icon></b>
                </v-col>
                <v-divider inset></v-divider>
                <v-col cols="12" md="12" class="py-0">
                  <span class="info--text">N° reg. copiados: </span><strong>{{$filters.thousands(item.data_count.num_total_data_copy)}}</strong><br>
                  <span class="info--text">N° reg. nuevos: </span><strong>{{$filters.thousands(item.data_count.num_data_new)}}</strong><br>
                  <span class="info--text">N° reg. regulares: </span><strong>{{$filters.thousands(item.data_count.num_data_regular)}}</strong><br>

                    <div class="text-right pb-1" v-if="permissionSimpleSelected.includes(item_import.permissions_download)">
                      <v-tooltip top class="my-0">
                        <template v-slot:activator="{ on }">
                          <v-btn
                            small
                            :color="'primary'"
                            fab
                            v-on="on"
                            :loading="loading_rep_state_re && i == loading_pos_index_re"
                            @click.stop="loading_pos_index_re = i; reportPayroll(item.period_month,item.period_day ? item.period_day : '01','reintegro')"
                          >
                            <v-icon>mdi-file-document</v-icon>
                          </v-btn>
                        </template>
                        <div>
                          <span>Detalle de Importación de registros válidos</span>
                        </div>
                      </v-tooltip>
                    </div>
                </v-col>
              </v-row>
            </v-card-text>
          </template>
        </v-card>
        <!--ADICIONALES-->
        <v-card
          class="headline font-weight-bold ma-2"
          max-width="250px"
          v-for="(item, k) in list_months_ad"
          :key="'ad'+k"
        >
          <template>
            <v-card-title class="info">
              <v-row justify="center">
                <h3 class="white--text">Adicional {{ item.period_month_name }}</h3></v-row
              >
            </v-card-title>
            <v-divider inset></v-divider>
            <v-card-text class="backgroundCard">
              <v-row>
                <v-col cols="12" md="12" class="py-0">
                  <b>{{ item_import.name }} <v-icon small>mdi-home-analytics</v-icon></b>
                </v-col>
                <v-divider inset></v-divider>
                <v-col cols="12" md="12" class="py-0">
                  <span class="info--text">N° reg. copiados: </span><strong>{{$filters.thousands(item.data_count.num_total_data_copy)}}</strong><br>
                  <span class="info--text">N° reg. nuevos: </span><strong>{{$filters.thousands(item.data_count.num_data_new)}}</strong><br>
                  <span class="info--text">N° reg. regulares: </span><strong>{{$filters.thousands(item.data_count.num_data_regular)}}</strong><br>

                    <div class="text-right pb-1" v-if="permissionSimpleSelected.includes(item_import.permissions_download)">
                      <v-tooltip top class="my-0">
                        <template v-slot:activator="{ on }">
                          <v-btn
                            small
                            :color="'primary'"
                            fab
                            v-on="on"
                            :loading="loading_rep_state_ad && k == loading_pos_index_ad"
                            @click.stop="loading_pos_index_ad = k; reportPayroll(item.period_month,item.period_day ? item.period_day : '01','adicional')"
                          >
                            <v-icon>mdi-file-document</v-icon>
                          </v-btn>
                        </template>
                        <div>
                          <span>Detalle de Importación de registros válidos</span>
                        </div>
                      </v-tooltip>
                    </div>
                </v-col>
              </v-row>
            </v-card-text>
          </template>
        </v-card>
      </template>
    </v-row>
    <PayrollImportProcess 
      :dialog="dialog" 
      :item_import="item_import" 
      :year_selected="year_selected" 
      :list_months_not_import.sync="list_months_not_import" 
      :list_months_not_import_re.sync="list_months_not_import_re"
      :list_months_not_import_ad.sync="list_months_not_import_ad"
      @open-close="openClose()"
    />
    <PayrollImportProcessTranscript 
      :dialog="dialog_transcript" 
      :item_import="item_import" 
      :year_selected="year_selected" 
      :list_months_not_import.sync="list_months_not_import" 
      @open-close-transcript="openCloseTranscript()"
    />
    <ImportProcessAvailability 
      :dialog="dialog_availability" 
      :item_import="item_import" 
      :year_selected="year_selected"  
      :list_months_not_import.sync="list_months_not_import" 
      @open-close-availability="openCloseAvailability()"
    />
    <PayrrollImportProcessRegional
      :current_date="current_date"
      :dialog="dialog_regional" 
      :item_import="item_import" 
      :year_selected="year_selected"  
      @open-close-regional="openCloseRegional()"
    />

  </v-container>
</template>

<script>
import GlobalBreadCrumb from "@/components/common/GlobalBreadCrumb.vue";
import GlobalLoading from "@/components/common/GlobalLoading.vue";
import PayrollImportProcess from "@/components/contribution/PayrollImportProcess.vue";
import PayrollImportProcessTranscript from "@/components/contribution/PayrollImportProcessTranscript.vue";
import ImportProcessAvailability from "@/components/affiliate/ImportProcessAvailability.vue";
import PayrrollImportProcessRegional from "@/components/contribution/PayrollImportProcessRegional.vue"
export default {
  name: "MainImportation",
  components: {
    GlobalBreadCrumb,
    GlobalLoading,
    PayrollImportProcess,
    PayrollImportProcessTranscript,
    ImportProcessAvailability,
    PayrrollImportProcessRegional
  },
  props: {
    item_import:{
      type: Object,
      required: true,
    }
  },
  data: () => ({
    years: [],
    loading: false,
    year_selected: null,
    list_months: [],
    list_months_not_import: [],
    list_months_not_import_re: [],
    list_months_not_import_ad: [],
    list_months_re: [],
    list_months_ad: [],
    list_dates: [],
    dialog: false,
    dialog_transcript: false,
    dialog_availability: false,
    dialog_regional: false,
    btn_import_contributions: false,
    loading_circular:false,
    loading_pos_index: -1,
    loading_pos_index_re: -1,
    loading_pos_index_ad: -1,
    loading_rep_state: false,
    loading_rep_state_re: false,
    loading_rep_state_ad: false,
    items_import: [],
    cancelToken: null,
    current_date: null,
  }),
  created() {
     this.getYears();
  },
  computed: {
    //permisos del selector global por rol
    permissionSimpleSelected () {
      return this.$store.getters.permissionSimpleSelected
    },
  },

  watch: {
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
        //let import_type_name = this.item_import.import_type_name
        let res = await this.$axios.get(`/contribution/list_years/${this.item_import.import_type_name}`, undefined);
        this.years = res.payload.list_years;
        this.year_selected = this.years[0];
        this.loading = false;
      } catch (e) {
        this.loading = false;
        console.log(e);
      }
    },
    async getMonths() {
      this.loading_circular = true
      try {
          let res = await this.$axios.post(`${this.item_import.route_get_months}`,{
            period_year: this.year_selected,
            with_data_count: true
          },
        );
        if(this.item_import.name == 'REGIONAL'){
          this.list_months = res.payload.list_dates;
          this.current_date = res.payload.current_date;
        }else{
          this.list_months = res.payload.list_months;
          this.list_months_not_import = res.payload.list_months_not_import;
          if(this.item_import.name == 'COMANDO'){
            this.list_months_re = res.payload.list_months_re;
            this.list_months_not_import_re = res.payload.list_months_not_import_re;
            this.list_months_ad = res.payload.list_months_ad;
            this.list_months_not_import_ad = res.payload.list_months_not_import_ad;
          }
        }
        this.loading_circular = false
      } catch (e) {
        console.log(e);
        this.loading_circular = false
      }
    },
    openDialog() {
      this.dialog = false;
      this.dialog_transcript = false;
      this.dialog_availability = false;
      this.dialog_regional = false;
      if(this.item_import.name == 'COMANDO' || this.item_import.name == 'SENASIR') 
        this.dialog = true;
      else if(this.item_import.name == 'TRANSCRIPCIÓN') 
        this.dialog_transcript = true;
      else if(this.item_import.name == 'DISPONIBILIDAD')  
        this.dialog_availability = true;
      else if(this.item_import.name == 'REGIONAL')
        this.dialog_regional = true;
      else
      this.month_selected= null
    },
    async reportPayroll(month_selected,day_selected,var_type_payroll){
      this.month_selected = month_selected
      if(var_type_payroll == 'reintegro') this.loading_rep_state_re=true;
      else if(var_type_payroll == 'adicional') this.loading_rep_state_ad=true;
      else this.loading_rep_state=true;

      try {
        let params ={}
        if(this.item_import.name != 'DISPONIBILIDAD' && this.item_import.name != 'REGIONAL'){
          params.date_payroll= this.dateFormat(this.year_selected, day_selected, this.month_selected)
        }
        if(this.item_import.name == 'DISPONIBILIDAD' && this.item_import.name != 'REGIONAL'){
          params.date_import= this.dateFormat(this.year_selected, day_selected, this.month_selected)
        }
        if (this.item_import.name == 'COMANDO') {
          params.type_payroll = var_type_payroll ? var_type_payroll : 'mensual';
        }
        if(this.item_import.name == 'REGIONAL'){
          params.date_import= this.dateFormat(this.year_selected, day_selected, this.month_selected)
        }
        let res = await this.$axios.post(`${this.item_import.route_report}`,params,
          {'Accept': 'application/vnd.ms-excel' },
          {'responseType': 'blob'}
        );
        const url = window.URL.createObjectURL(new Blob([res]))
        const link = document.createElement("a")
        link.href = url;
        link.setAttribute("download", this.item_import.name_report_file)
        document.body.appendChild(link)
        link.click()
      } catch (e) {
        console.log(e);
      } finally {
        this.loading_rep_state=false;
        this.loading_rep_state_re=false;
        this.loading_rep_state_ad=false;
        this.loading_pos_index=-1;
        this.loading_pos_index_re=-1;
        this.loading_pos_index_ad=-1;
      }
    },
    openClose(newValue) {
      this.dialog = newValue
      this.getMonths()
    },
    openCloseTranscript(newValue) {
      this.dialog_transcript = newValue
      this.getMonths()
    },
    openCloseAvailability(newValue) {
      this.dialog_availability = newValue
      this.getMonths()
    },
    openCloseRegional(newValue) {
      this.dialog_regional = newValue
      this.getMonths()
    },
    dateFormat(year_selected, day_selected, month_selected) {
      const month = String(month_selected).padStart(2, '0')
      const day = String(day_selected).padStart(2, '0')
      return `${year_selected}-${month}-${day}`
    }
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