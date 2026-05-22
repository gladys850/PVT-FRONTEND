<template>
  <v-container fluid>

    <v-card-title mb-4>
      <v-toolbar dense color="tertiary" class="caption">
        <GlobalBreadCrumb />
      </v-toolbar>
    </v-card-title>

    <v-card-text>
      <!-- Opciones del tab -->
      <v-tabs left background-color="backgroundTab" v-model="tab">
        <v-tab v-for="item in actions" :key="item.nameTab">{{item.nameTab}}</v-tab>
      </v-tabs>
      <!-- Fin de las opciones del tab -->

      <!-- Contenido de los tabs -->
      <v-tabs-items v-model="tab">
        <v-tab-item v-for="item in actions" :key="item.nameTab">
          <div class="mt-3 pt-3 pb-8 mx-0 px-0 backgroundCard">
            <v-row>
              <v-col cols="6" md="6" lg="4">
                <v-card class="mx-5 px-5 py-6 elevation-0">

                  <!-- Sub título  -->
                  <v-toolbar-title>
                    <b>Seleccione un reporte</b>
                  </v-toolbar-title>
                  <v-progress-linear class="mb-5"></v-progress-linear>
                  <!-- Fin subtitulo -->

                  <!-- Sección del select -->
                  <v-select
                    v-model="report_selected"
                    :items="computedReportsItems"
                    item-text="name"
                    return-object
                    label="Seleccione un reporte"
                    outlined dense>
                  </v-select>
                  <template v-if="report_selected && report_selected.name == 'Reporte de Notificaciones'">
                    <CriterionForm v-if="report_selected && report_inputs"
                      :report_inputs="report_inputs"
                      :report_selected="report_selected"
                      :types="type_notifications"
                    />
                  </template>
                  <template v-else-if="report_selected && report_selected.name == 'Reporte de Pagos y Derechohabientes'">
                    <CriterionForm v-if="report_selected && report_inputs"
                      :report_inputs="report_inputs"
                      :report_selected="report_selected"
                      :types="type_of_benefits"
                    />
                  </template>
                  <template v-else >
                    <CriterionForm v-if="report_selected && report_inputs"
                      :report_inputs="report_inputs"
                      :report_selected="report_selected"
                    />
                  </template>
                </v-card>
              </v-col>
            </v-row>
          </div>

        </v-tab-item>
      </v-tabs-items>
    </v-card-text>

  </v-container>
</template>

<script>
import GlobalBreadCrumb from "@/components/common/GlobalBreadCrumb.vue";
import CellBalance from "@/components/notification/CellBalance";
import CriterionForm from './Criterion.vue';

export default {
  name: "ReportNotification",
  components: {
    GlobalBreadCrumb,
    CellBalance,
    CriterionForm
  },
  data: () => ({
    menu: false,
    menu2: false,
    current_date: null,
    type_notifications: null,
    type_of_benefits: [
      'Fondo de Retiro Policial',
      'Cuota Mortuoria',
      'Auxilio Mortuorio'
    ],
    loading: false,
    actions: [
      {nameTab: "Reportes de Notificaciones", value: "notificaciones"},
      {nameTab: "Reportes de certificaciones", value: "certificaciones"},
      {nameTab: "Otros Reportes", value: "otros"},
    ],
    tab: null,
    reports_items: [],
    report_selected: null,
    visible: false,
    report_inputs: {
      type_report: null,
      start_date: new Date(Date.now() - new Date().getTimezoneOffset() * 60000)
      .toISOString()
      .substr(0, 10),
      end_date: new Date(Date.now() - new Date().getTimezoneOffset() * 60000)
      .toISOString()
      .substr(0, 10),
    }
  }),
  created() {
    this.reports_items = [
      {
        id: 0,
        name: "Reporte de Notificaciones Complemento Económico",
        tab: 0,
        criterios: ["start_date", "end_date"],
        method:"post",
        service: "/notification/report_notifications",
        type: "xls",
        permissions: "download-report-notification",
        name_download:"ReporteCE"
      },
      {
        id: 1,
        name: "Reporte de Notificaciones",
        tab: 0,
        criterios: ["start_date", "end_date", "type_report"],
        method:"post",
        service: "/notification/report",
        type: "xls",
        permissions: "download-report-notification",
        name_download:"ReporteNotificaciones"
      },
      {
        id: 2,
        name: "Reporte de certificación",
        tab: 1,
        criterios: ["start_date", "end_date"],
        method:"post",
        service: "/contribution/get_report_certificate",
        type: "xls",
        permissions: "download-certifications",
        name_download:"ReporteCertificaciones",
      },
      {
        id: 3,
        name: "Reporte de información sobre Afiliados y Cónyuges",
        tab: 2,
        criterios: [],
        method:"get",
        service: "/report/report_affiliates_spouses",
        type: "xls",
        permissions: "sin-permiso",
        name_download:"Reporte_Afiliados_Conyuges"
      },
      {
        id: 4,
        name: "Reporte de Clasificadores Cuentas Individuales FR",
        tab: 2,
        criterios: ["start_date", "end_date"],
        method:"post",
        service: "/report/report_retirement_funds",
        type: "xls",
        permissions: "sin-permiso",
        name_download:"ReporteClasificadoresFR"
      },
      {
        id: 5,
        name: "Reporte de Pagos y Derechohabientes",
        tab: 2,
        criterios: ["start_date", "end_date", "type_report"],
        method:"post",
        service: "/report/report_payments_beneficiaries",
        type: "xls",
        permissions: "sin-permiso",
        name_download:"ReportePagosDerechohabientesFR"
      },
      {
        id: 6,
        name: "Reporte de calificacion de servicios",
        tab: 2,
        criterios: ["start_date", "end_date"],
        method:"post",
        service: "/admin/get_qualification_report",
        type: "xls",
        permissions: "service-calification",
        name_download:"Reporte_calificacion_de_servicios"
      },
      {
        id: 7,
        name: "Reporte de pagos en demasía",
        tab: 2,
        criterios: [],
        method: "get",
        service: "/economic_complement/report_eco_com_movement",
        type: "xls",
        permissions: "sin-permiso",
        name_download: "Reporte_pagos_demasía"
      },
      {
        id: 8,
        name: "Reporte de posibles casos duplicados",
        tab: 2,
        criterios: [],
        method: "get",
        service: "/report/report_affiliates_similar",
        type: "xls",
        permissions: "sin-permiso",
        name_download: "Reporte_posibles_duplicados"
      },
      {
        id: 9,
        name: "Reporte de tramites de FRCAM",
        tab: 2,
        criterios: ["start_date", "end_date"],
        method: "post",
        service: "/report/report_procedures_frcam",
        type: "xls",
        permissions: "sin-permiso",
        name_download: "Reporte_tramites_FRCAM"
      }
    ]
  },
  mounted() {
    this.getTypeNotification()
    this.current_date = this.formater()
  },
  watch: {
    tab: function(newVal, oldVal) {
      if(newVal != oldVal) {
        this.visible = false
        this.clearInputs()
      } else {
        this.visible = true
      }
    },
    report_selected: {
      deep: true,
      handler(val) {
        this.visible = true
      }
    }
  },
  methods: {
    async getTypeNotification() {
        try {
            let response = await this.$axios.get(
                "/notification/get_type_notification",
                undefined
            )
            this.type_notifications = response.type_notifications
            this.type_notifications.unshift({ id: 3, name: "Todos" })
        } catch(e) {
            console.log(e)
        }
    },
    formater() {
      let current_date = new Date();
      let month = current_date.getMonth() + 1;
      let day = current_date.getDate();
      let year = current_date.getFullYear();

      if(day < 10) {
        day = '0' + day + '';
      }
      let date = year + "-" + month + "-" + day;
      return date;
    },
    clearInputs() {
      this.report_selected = null
      this.report_inputs.start_date = new Date(Date.now() - new Date().getTimezoneOffset() * 60000)
      .toISOString()
      .substr(0, 10)
      this.report_inputs.end_date = new Date(Date.now() - new Date().getTimezoneOffset() * 60000)
      .toISOString()
      .substr(0, 10)
      this.report_inputs.type_report = null
    }
  },
  computed: {
    permissionSimpleSelected() {
      return this.$store.getters.permissionSimpleSelected
    },
    computedReportsItems() {
      let reports_items = []
      let reports_items_notification = []
      let reports_items_certification = []
      let reports_items_others = []
      if(this.permissionSimpleSelected.includes("download-certifications")) {
        reports_items_certification = this.reports_items.filter((item) => item.permissions == 'download-certifications')
      }
      console.log(this.permissionSimpleSelected)
      if(this.permissionSimpleSelected.includes("download-report-notification")) {
        reports_items_notification = this.reports_items.filter((item) => item.permissions == 'download-report-notification')
      }
      if(this.reports_items.filter((item) => item.permissions == 'sin-permiso')) {
        reports_items_others = this.reports_items.filter((item) => item.permissions == 'sin-permiso')
        console.log(reports_items_others)
      }
      if(this.reports_items.filter((item) => item.permissions == 'service-calification')) {
        reports_items_others = this.reports_items.concat(this.reports_items.filter((item) => item.permissions == 'service-calification'))
      }
      reports_items = reports_items_others.concat(reports_items_certification.concat(reports_items_notification))
      reports_items = reports_items.filter((item) => item.tab == this.tab)
      return reports_items
    }
  }
};
</script>
