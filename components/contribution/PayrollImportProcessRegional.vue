<template>
    <v-container fluid>
        <v-dialog
            v-model="dialog"
            fullscreen
            hide-overlay
            transition="dialog-bottom-transition"
            persistent
        >
            <v-card>
                <v-toolbar dark color="primary">
                    <v-btn icon dark @click="close()">
                        <v-icon>mdi-close</v-icon>
                    </v-btn>
                    <v-toolbar-title>IMPORTACIÓN {{item_import.name}}</v-toolbar-title>
                </v-toolbar>
                
                <template v-if="exist_processes_imcomplete">
                    <v-row justify="center" class="mt-5">
                        <v-col cols="8">
                            <!-- ALERTA -->
                            <v-alert
                                prominent
                                type="error"
                                border="left"
                                class="mb-4"
                            >
                                Se tiene procesos pendientes de importación.
                                Debe eliminar estos procesos pendientes para realizar una nueva importación.
                            </v-alert>
                            <!-- CARD DE FECHAS -->
                            <v-card
                                outlined
                                elevation="1"
                            >
                                <v-card-title class="subtitle-1 font-weight-bold">
                                    Procesos pendientes
                                </v-card-title>
                                <v-divider></v-divider>
                                <v-card-text>
                                    <v-row
                                        v-for="(process, index) in incomplete_processes"
                                        :key="process.date_import"
                                        align="center"
                                        justify="space-between"
                                        class="mx-2"
                                    >
                                        <div>
                                            <strong>Fecha:</strong>
                                            {{ process.date_import }} |
                                            <strong>PlanillaAux</strong>
                                            {{ process.count_data_aux }} |
                                            <strong>Planilla</strong>
                                            {{ process.count_data_payroll_regionals }} |
                                            <strong>Contribuciones</strong>
                                            {{ process.count_data_contribution_passives }}
                                        </div>
                                        <v-btn
                                        :disabled="process.count_data_contribution_passives > 0"                                        
                                        icon
                                        small
                                        color="error"
                                        @click="dialog_delete_process = true; date_process_selected = process.date_import"
                                        >
                                            <v-icon>mdi-delete</v-icon>
                                        </v-btn>
                                    </v-row>
                                </v-card-text>
                            </v-card>
                        </v-col>
                    </v-row>
                </template>
                <template v-if="progress.percentage == 100">
                    <v-row justify="center" class="mt-5">
                        <v-col cols="8">
                            <v-alert
                                prominent
                                type="info"
                                border="left"
                                class="mb-4"
                            >
                                El proceso de hoy ya fue completado. No se puede realizar una nueva importación.
                            </v-alert>
                        </v-col>
                    </v-row>
                </template>
                <template v-if="!exist_processes_imcomplete && progress.percentage !== 100">
                    <v-row justify="center" class="mt-5">
                        <v-col cols="8">
                            <v-toolbar-title class="pb-5">
                                <center><b>GESTIÓN {{year_selected}}</b></center>
                                <div class="text-right">
                                    <Information :title="information.title" :parameters="information.parameters"/>
                                </div>
                            </v-toolbar-title>

                            <v-text-field                        
                                v-model="current_date"
                                label="Fecha de importación actual"
                                outlined
                                readonly
                            />                        
                            <v-stepper v-model="e1" editable>
                                <!-- C A B E C E R Á S   P A S O S -->
                                <v-stepper-header>
                                    <v-stepper-step :complete="e1 > 1" step="1">
                                        Subir archivo
                                    </v-stepper-step>
                                    <v-divider></v-divider>
                                    <v-stepper-step :complete="e1 > 2" step="2">
                                        Validar datos
                                    </v-stepper-step>
                                    <v-divider></v-divider>
                                    <v-stepper-step :complete="e1 > 3" step="3">
                                        Procesar planilla
                                    </v-stepper-step>
                                    <v-divider></v-divider>
                                    <v-stepper-step :complete="e1 > 4" step="4">
                                        Importar aportes
                                    </v-stepper-step>
                                </v-stepper-header>
                                <v-stepper-items>
                                    <!-- B A R R A   D E   P O R C E N T A J E -->
                                    <v-progress-linear
                                        color="info"
                                        height="20"
                                        :value="progress.percentage"
                                        striped
                                    >
                                        <strong>Porcentaje de Importación: {{progress.percentage}}%</strong>
                                    </v-progress-linear>
                                    <!-- P R I M E R   P A S O -->
                                    <v-stepper-content step="1">
                                        <v-card class="mb-12">
                                            <v-card-text>
                                                <v-card color="white" class="pa-2">
                                                    <v-form ref="forStep1">
                                                        <v-col cols="12" md="6">
                                                            <v-file-input
                                                                v-model="import_export.file"
                                                                counter
                                                                show-size
                                                                truncate-length="30"
                                                                outlined
                                                                small-chips
                                                                label="Cargar Archivo"
                                                                :disabled="progress.query_step_1"
                                                                :rules="[
                                                                    $rules.soloArchivoCSV()
                                                                ]"
                                                            ></v-file-input>
                                                        </v-col>
                                                        <v-col cols="12" md="6" v-if="progress.query_step_1">
                                                            <strong>Nombre del archivo: </strong>{{progress.file_exists ? progress.file_name : import_export.file.name}}<br>
                                                            <strong>Total de registros copiados:</strong>{{$filters.thousands(data_count.num_total_data_copy)}}<br>
                                                        </v-col>
                                                    </v-form>
                                                </v-card>
                                            </v-card-text>
                                        </v-card>
                                        <v-btn color="primary" @click="validateForm1()" :loading="btn_update_file" :disabled="progress.query_step_1">
                                            Subir archivo
                                        </v-btn>
                                        <v-btn color="secundary" @click="nextStep(1)" :disabled="!progress.query_step_1">
                                            Siguiente
                                        </v-btn>
                                    </v-stepper-content>
                                    <!-- S E G U N D O   P A S O -->
                                    <v-stepper-content step="2">
                                        <v-card class="mb-12" color="grey lighten-1">
                                            <v-card-text>
                                                <v-card color="white" class="pa-2">
                                                    <v-row>
                                                        <v-col cols="12" md="6">
                                                            <strong>Total de registros copiados: </strong>{{$filters.thousands(data_count.num_total_data_copy)}}<br>
                                                            <strong class="sucess--text">Total de registros con error: </strong>{{$filters.thousands(data_count.count_data_error || 0)}}<br>
                                                            <strong class="sucess--text">Total de registros no identificados: </strong>{{$filters.thousands(data_count.count_data_unidentified || 0)}}<br>
                                                        </v-col>
                                                        <v-col cols="12" md="6">
                                                            <strong class="success--text">Total de registros validados: </strong>{{$filters.thousands(data_count.count_data_automatic_link || 0)}}<br>
                                                        </v-col>
                                                    </v-row>
                                                </v-card>
                                            </v-card-text>
                                        </v-card>
                                        <v-btn color="error" @click="redo=true">
                                            Rehacer
                                        </v-btn>
                                        <v-btn color="primary" @click="show=true" :disabled="btn_validate">
                                            Validar archivo
                                        </v-btn>
                                        <v-btn color="secundary" @click="nextStep(2)" :disabled="!btn_next">
                                            Siguiente
                                        </v-btn>
                                    </v-stepper-content>
                                    <!-- T E R C E R   P A S O -->
                                    <v-stepper-content step="3">
                                        <v-card class="mb-12" color="grey lighten-1">
                                            <v-card-text>
                                                <v-card color="white" class="pa-2">
                                                    <v-row>
                                                        <v-col cols="12" md="6">
                                                            <strong>Total de registros copiados: </strong>{{$filters.thousands(data_count.num_total_data_copy)}}<br>
                                                        </v-col>
                                                        <v-col cols="12" md="6">
                                                            <strong class="success--text">Total de registros validados para procesar: </strong>{{$filters.thousands(data_count.count_data_automatic_link || 0)}}
                                                        </v-col>
                                                    </v-row>
                                                </v-card>
                                            </v-card-text>
                                        </v-card>
                                        <v-btn color="error" @click="redo=true" :disabled="btn_redo">
                                            Rehacer
                                        </v-btn>
                                        <v-btn color="primary" @click="show=true" :disabled="btn_import">
                                            Procesar
                                        </v-btn>
                                        <v-btn color="secundary" @click="nextStep(3)" :disabled="!btn_next">
                                            Siguiente
                                        </v-btn>
                                    </v-stepper-content>
                                    <!-- C U A R T O   P A S O -->
                                    <v-stepper-content step="4">
                                        <v-card class="mb-12" color="grey lighten-1">
                                            <v-card-text>
                                                <v-card color="white" class="pa-2">
                                                    <v-row>
                                                        <v-col cols="12" md="6">
                                                            <strong>Número de aportes importados: </strong>{{$filters.thousands(data_count.num_total_data_contribution || 0)}}<br>
                                                        </v-col>
                                                    </v-row>
                                                </v-card>
                                            </v-card-text>
                                        </v-card>
                                        <v-btn color="secundary" @click="show=true" >
                                            Importar
                                        </v-btn>
                                    </v-stepper-content>
                                </v-stepper-items>
                            </v-stepper>
                        </v-col>
                    </v-row>
                </template>
            </v-card>
        <div class="overlay" v-if="loading_circular">
            <GlobalLoading />
        </div>
        </v-dialog>
        <!-- D I Á L O G O   P A R A   R E H A C E R -->
        <v-dialog
            v-model="redo"
            max-width="600"
            persistent
        >
            <v-card>
                <v-card-title>
                    <center>¿Está seguro que quiere rehacer el proceso?</center>
                    <br>
                    <br> <small class='caption'>Al rehacer se borraran todos los datos ingresados</small>
                </v-card-title>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn
                        color="error"
                        text
                        @click="redo=false"
                    >
                    Cancelar
                    </v-btn>
                    <v-btn
                        color="success"
                        text
                        @click="rollbackContribution()"
                    >
                    Aceptar
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
        <!-- D I Á L O G O   P A R A   L O S   P A S O S -->
        <v-dialog
            v-model="show"
            max-width="600"
            persistent
        >
            <v-card>
                <v-card-title>
                    {{title_dialog}}
                    <br v-if="small_dialog !== ''">
                    <br> <small class='caption'>{{small_dialog}}</small>
                </v-card-title>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn
                        color="error"
                        text
                        @click="show=false"
                    >
                    Cancelar
                    </v-btn>
                    <v-btn
                        color="success"
                        text
                        @click="action()"
                    >
                    Aceptar
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
                <!-- D I Á L O G O   P A R A   E L I M I N A R -->
        <v-dialog
            v-model="dialog_delete_process"
            max-width="600"
            persistent
        >
            <v-card>
                <v-card-title>
                    <center>¿Está seguro que quiere eliminar el proceso?</center>
                    <br>
                    <br> <small class='caption'>Al eliminar el proceso se borraran todos los registros importados del periodo.</small>
                </v-card-title>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn
                        color="error"
                        text
                        @click="dialog_delete_process=false"
                    >
                    Cancelar
                    </v-btn>
                    <v-btn
                        color="success"
                        text
                        @click="deleteProcess(process_selected)"
                    >
                    Aceptar
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
    </v-container>
</template>action


<script>
import Information from '@/components/contribution/Information.vue'
import GlobalLoading from '@/components/common/GlobalLoading.vue'
export default {
    name: "PayrollImportProcessRegional",
    components: {
        Information,
        GlobalLoading
    },
    data: () => ({
        import_export: {},
        e1: 1, // current_step
        //month_selected: null,
        progress: {
            file_exists: false,
            file_name: null,
            percentage: 0,
            query_step_1: false,
            query_step_2: false,
            query_step_3: false,
            query_step_4: false
        },
        data_count: {
            num_total_data_copy: 0,
            count_data_automatic_link: 0,
            count_data_creation: 0,
            count_data_revision: 0,
            num_total_data_payroll: 0,
            num_total_data_contribution: 0
        },
        btn_update_file: false,
        btn_rollback: false,
        btn_validate: false,
        btn_import: false,
        btn_redo: false,
        btn_next: false,

        loading_circular: false,
        information: {
            title: "IMPORTACIÓN",
            parameters: [
                {
                    title: "FORMATO ARCHIVO",
                    body: "tipo CSV delimitado por \"dos puntos\" (:)"
                },
                {
                    title: "NOMBRE ARCHIVO",
                    body: "tipo-mes-año.csv Ejm. regional-2026-02-26.csv"
                },
                {
                    title: "MONTOS",
                    body: "Los montos deben utilizar como separador de decimales punto(.)."
                },
            ]
        },

        // Dialog dinámico
        show: false,
        title_dialog: '',
        small_dialog: '',
        redo: false,
        dialog_delete_process: false,

        block_select: false,
        exist_processes_imcomplete: false,
        incomplete_processes: [],
        date_process_selected: null
    }),
    props: {
        dialog: {
            type: Boolean,
            required: true,
            default: false
        },
        item_import: {
            type: Object,
            required: true,
            default: {}
        },
        year_selected: null,
        current_date: {
            type: Date,
            required: false,
            default: new Date().toLocaleDateString('es-BO'),
        }
    },

    mounted() {
        
    },
    computed: {
        dateFormat() {
            return this.current_date
        }
    },
    watch: {
        dialog: function() { 
            this.incompleteProcesses()
            this.importProgressBar()
        },
        show: function() {
            if(this.e1 == 2) {
                this.title_dialog = "¿Está seguro de realizar la validación?"
            } else if(this.e1 == 3) {
                this.title_dialog = "¿Está seguro de importar la plantilla?"
            } else if(this.e1 >= 4) {
                this.title_dialog = "¿Está seguro de realizar la importación de aportes?"
            }
        },
    },
    methods: {
        //paso 1
        async uploadFile() {
            this.btn_update_file = true
            let formData = new FormData()
            formData.append("file", this.import_export.file)
            formData.append("date_import", this.dateFormat)
            try {               
                    let res = await this.$axios.post(`${this.item_import.route_upload_file}`,
                        formData
                    )
                    if(res.payload.successfully) {
                        this.data_count.num_total_data_copy = res.payload.data_count.num_total_data_copy
                        this.$toast.success(res.message)
                        this.progress.query_step_1 = true
                    } else {
                        if(res.payload.route !== '' && res.payload.route.length !== 0) {
                            this.downloadFile(res.payload.route, res.payload.route_file_name);
                        }
                        this.$toast.error(res.payload.error)
                    }
                    this.btn_update_file = false
            } catch(e) {
                console.log(e)
                this.btn_update_file = false
                this.$toast.error("Hubo un error")
            }
        },
        async importProgressBar() {
            try {
                let res = await this.$axios.post(`${this.item_import.route_import_progressBar}`,{
                    date_import: this.dateFormat
                });
                this.progress = res.payload.import_progress_bar
                this.data_count = res.payload.data_count

                const steps = Object.entries(this.progress).reduce((accumulator, [key, value]) => {
                    if(key.includes('query_step')) accumulator.push(key)
                    return accumulator
                },[])

                const values = Object.entries(this.progress).reduce((accumulator, [key, value]) => {
                    if(steps.includes(key)) accumulator.push(value)
                    return accumulator;
                },[])

                this.e1 = values.indexOf(true) + 1

                if(this.e1 === 0) { // si es el primer paso
                    this.e1 = 1
                }
                this.btn_next = false

                if(this.e1 > 1) {
                    this.block_select = true
                }

            } catch(e) {
                console.log(e)
                this.$toast.error("Hubo un error")
            }
        },
        async rollbackContribution() {
            this.btn_rollback = true
            try {
                let res = await this.$axios.post(`${this.item_import.route_rollback_contribution}`,{
                    date_import: this.dateFormat,
                });
                if (res.payload.valid_rollback) {
                    this.$toast.info(res.message + ". Se ha realizado el borrado de datos")
                    this.clearData()
                    this.redo = false
                } else {
                    this.redo = false
                    this.$toast.error(res.message)
                }
                this.btn_rollback = false
            } catch (e) {
                this.redo = false
                this.btn_rollback = false
                console.log(e)
                this.$toast.error("Hubo un error")
            }
        },
        close() {
            this.block_select = false
            this.clearData()
            this.$emit('open-close-regional', !this.dialog)
        },
        nextStep(n) {
            this.btn_next = false
            if (n == 1) {
                this.block_select = true
                this.progress.percentage = this.progress.percentage + 25
            }
            if (n == 2) {
                this.progress.percentage = this.progress.percentage + 25
            }
            if (n == 3) {
                this.progress.percentage = this.progress.percentage + 25
            }
            if (n == 4) {
                this.progress.percentage = this.progress.percentage + 25
            }
            this.e1 = n + 1
            if(this.e1 > 1) {
                this.block_select = true
            }
        },
        validateForm1() {
            if(this.$refs.forStep1) {
                if(this.$refs.forStep1.validate()) {
                    this.uploadFile()
                } else {
                    this.$toast.error('Ingrese los datos necesarios')
                }
            }
        },
        //2do Paso
        async validateData() {
            this.show = false
            this.loading_circular = true
            try {
                let res  = await this.$axios.post(`${this.item_import.route_validate_data}`, {
                    date_import: this.dateFormat
                })
                if(res.payload.successfully) {
                    this.btn_validate = true
                    this.btn_next = true
                    this.$toast.success("Se ha realizado la validación de los afiliados")
                }
                if(res.payload.route !== '' && res.payload.route !== 0) {
                    this.downloadFile(res.payload.route, res.payload.route_file_name)
                }
                this.data_count = res.payload.data_count
                this.loading_circular = false
            } catch(e) {
                console.log(e)
                this.show = false
                this.loading_circular = false
                this.$toast.error("Hubo un error")
            }
        },
        //3er paso
        async importPayroll() {
            try {
                this.loading_circular = true
                this.show = false
                let res = await this.$axios.post(`${this.item_import.route_import_payroll}`, {
                    date_import: this.dateFormat
                })
                if(res.payload.successfully) {
                    this.data_count = res.payload.data_count
                    this.$toast.success(res.message)
                    this.btn_import = true
                    this.btn_redo = true
                    this.btn_next = true
                } else {
                    this.$toast.error(res.message)
                }
                this.loading_circular = false
            } catch (e) {
                this.show = false
                this.loading_circular = false
                this.$toast.error("Hubo un error")
                console.log(e)
            }
        },
        async downloadFile(route, name) {
            try {
                let res = await this.$axios.post(`${route}`, {
                    date_import: this.dateFormat,
                },
                { 'Accept': 'application/vnd.ms-excel' },
                { 'responseType':  'blob' }
                )
                const url = window.URL.createObjectURL(new Blob([res]))
                const link = document.createElement("a")
                link.href = url
                link.setAttribute("download", `${name}`)
                document.body.appendChild(link)
                link.click()
            } catch(e) {
                console.log(e)
                this.$toast.error("Hubo un error")
            }
        },
        clearData() {
            this.e1 = 1
            this.import_export = {}
            this.progress.file_exists = false
            this.progress.file_name = null
            this.progress.percentage = 0
            this.progress.query_step_1 = false
            this.progress.query_step_2 = false
            this.progress.query_step_3 = false
            this.progress.query_step_4 = false
            this.btn_validate = false
            this.btn_next = false
        },
        async action() {
            if(this.e1 == 2) {
                this.validateData()
            } else if(this.e1 == 3) {
                this.importPayroll()
            } else if(this.e1 == 4) {
                this.importContribution()
            }
        },
        // 4to paso
        async importContribution() {
            try {
                this.loading_circular = true
                this.show = false
                let res = await this.$axios.post(`${this.item_import.route_import_contribution}`, {
                    date_import: this.dateFormat
                })
                if(res.payload.successfully) {
                    this.data_count.num_total_data_contribution = res.payload.num_total_data_contribution
                    this.$toast.success(`La importación ${this.dateFormat} fue realizada correctamente.` )
                    this.clearData()
                    this.close()
                    this.nextStep(4)
                } else {
                    this.$toast.error(res.message)
                }
                this.loading_circular = false
            } catch(e) {
                this.loading_circular = false
                this.$toast.error("Hubo un error")
                console.log(e)
            }
        },
        async incompleteProcesses(){
            try {
                let res = await this.$axios.get('/contribution/list_incomplete_processes')
                this.exist_processes_imcomplete = res.error_process
                this.incomplete_processes = res.incomplete_processes

            } catch(e) {
                console.log(e)
                this.$toast.error("Hubo un error")
            }
        },
        async deleteProcess(process_selected) {
            try {
                let res = await this.$axios.post(`/contribution/delete_incomplete_processes`, {
                    date_import: this.date_process_selected
                })
                if(res.successfully) {
                    
                    this.$toast.success(res.message)
                } else {
                    this.$toast.error(res.message)
                }
                this.dialog_delete_process = false
                this.incompleteProcesses()
            } catch(e) {
                console.log(e)
                this.$toast.error("Hubo un error")
                this.dialog_delete_process = false
            }
        }
    },
}
</script>
<style scoped>
.overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.3);
  z-index: 1;
}
</style>