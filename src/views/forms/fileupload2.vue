<template>
    <div class="container select2">
        <div class="container">
            <div v-scroll-spy class="row layout-top-spacing">
                <div id="fuSingleFile" class="col-lg-12 layout-spacing">
                    <div class="statbox panel box box-shadow">
                        <div class="panel-heading">
                            <div class="row">
                                <div class="col-xl-12 col-md-12 col-sm-12 col-12">
                                    <h4>Graduation and Employment Data</h4>
                                </div>
                            </div>
                        </div>
                        <div class="panel-body">
                            <div class="custom-file-container" data-upload-id="myFirstImage">
                                <label>Upload (Single File) <a href="javascript:void(0)"
                                        class="custom-file-container__image-clear" title="Clear Image">x</a></label>
                                <label class="custom-file-container__custom-file">
                                    <input type="file" class="custom-file-container__custom-file__custom-file-input"
                                        accept=".xlsx, .xls" @change="handleFileUpload" />
                                    <input type="hidden" name="MAX_FILE_SIZE" value="10485760" />
                                    <span class="custom-file-container__custom-file__custom-file-control"></span>
                                </label>
                                <div class="custom-file-container__image-preview" style="display: none;"></div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>

            <div class="row">
                <div class="col-md-3">
                    <div class="form-group mb-4">
                        <label for="report_date">Report Date</label>
                        <flat-pickr v-model="report_date" class="form-control form-control-sm flatpickr active"
                            placeholder="Report Date"></flat-pickr>
                    </div>
                </div>
                <div class="col-md-3 panel-body statbox  box box-shadow">
                    <div class="form-group mb-4 panel-body">
                        <label for="months">Program Length Months</label>
                        <multiselect v-model="months" :options="options1" :searchable="true" :preselect-first="true"
                            selected-label="" select-label="" deselect-label=""></multiselect>

                    </div>
                </div>
                <div class="col-md-3">
                    <div class="form-group mb-4">
                        <label for="initial_date">Initial Date</label>
                        <flat-pickr v-model="initial_date" :disabled="true"
                            class="form-control form-control-sm flatpickr active" placeholder="Initial Date"></flat-pickr>
                    </div>
                </div>

                <div class="col-md-3">
                    <div class="form-group mb-4">
                        <label for="final_date">Final Date</label>
                        <flat-pickr v-model="final_date" :disabled="true"
                            class="form-control form-control-sm flatpickr active" placeholder="Final Date"></flat-pickr>
                    </div>
                </div>
            </div>
            <div class="row">
                <div class="col-md-6 panel-body statbox  box box-shadow">
                    <div class="form-group mb-4 panel-body">
                        <label for="programs">Programs</label>
                        <multiselect v-model="program" :options="programs" :searchable="true" :preselect-first="true"
                            selected-label="" select-label="" deselect-label=""></multiselect>

                    </div>
                </div>
            </div>
            <div class="row">
                <div class="col-md-12 panel-body statbox box box-shadow d-flex justify-content-end">

                    <button  type="button"  @click="exportToExcel"  class="btn btn-success btn-lg mb-4 me-2">Export </button>
                    <button @click="generateData" type="button" class="btn btn-danger btn-lg mb-4 me-2">Generate</button>
                </div>
            </div>

            <!-- Tabla para mostrar los datos de los programas -->
            <div class="row layout-top-spacing">
                <div class="col-xl-12 col-lg-12 col-sm-12 layout-spacing">
                    <div class="panel p-0">
                        <div class="custom-table">
                            <v-client-table :data="itemsData" :columns="columnsDataProgram" :options="table_option2"
                                >
                            </v-client-table>

                        </div>
                    </div>
                </div>
            </div>

            <!-- Tabla para mostrar los datos de los programas por fecha -->
            <div class="row layout-top-spacing">
                <div class="col-xl-12 col-lg-12 col-sm-12 layout-spacing">
                    <div class="panel p-0">
                        <div class="custom-table">
                            <v-client-table :data="itemsDataByDate" :columns="columnsDataProgramByDate" :options="table_option2"
                                >
                            </v-client-table>

                        </div>
                    </div>
                </div>
            </div>

            <!-- Tabla para mostrar los datos del nuevo Excel -->
            <div class="row layout-top-spacing">
                <div class="col-xl-12 col-lg-12 col-sm-12 layout-spacing">
                    <div class="panel p-0">
                        <div class="custom-table">
                            <v-client-table :data="items" :columns="columns" :options="table_option">
                                <!-- Personalizar la columna de Status con una clase dinámica -->
                                <template #status="props">
                                    <span :class="getStatusClass(props.row.Status)" class="badge">{{ props.row.Status
                                    }}</span>
                                </template>
                                <!-- Mostrar la fecha de inicio asociada al término -->
                                <template #startDate="props">
                                    <span>{{ props.row['Start Date'] }}</span>
                                </template>
                            </v-client-table>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted, computed, watch } from 'vue';
import '@/assets/sass/forms/file-upload-with-preview.min.css';
import * as XLSX from 'xlsx';
import FileUploadWithPreview from 'file-upload-with-preview';

// flatpickr
import flatPickr from 'vue-flatpickr-component';
import 'flatpickr/dist/flatpickr.css';
import '@/assets/sass/forms/custom-flatpickr.css';

//multiselect
import Multiselect from '@suadelabs/vue3-multiselect';
import '@suadelabs/vue3-multiselect/dist/vue3-multiselect.css';
import '@/assets/sass/scrollspyNav.scss';
import { useStore } from 'vuex';

const store = useStore();

onMounted(() => {
    new FileUploadWithPreview('myFirstImage');
});

// Encabezados de la tabla con la nueva columna de 'Start Date'
const columns = ref([
    'Student',
    'Program',
    'Status',
    'Term Started',
    'Start Date' // Nueva columna
]);

const columnsDataProgram = ref([
    'Program',
    'Total',
    'Graduate',
    'Dropped',
    'Enrolled',
    'Placement'
]);
const columnsDataProgramByDate = ref([
    'Initial Date',
    'Total',
    'Graduated',
    'Dropped',
    'Enrolled',
]);

const report_date = ref("2025-07-01"); // Fecha final seleccionada
const months = ref("");
const program = ref("All");
const excelDataTemp = ref([]); // Variable temporal para almacenar los datos del Excel
const items = ref([]);  // Datos finales que se mostrarán en la tabla
const itemsData = ref([]);  // Datos finales que se mostrarán en la tabla por programas
const itemsDataByDate = ref([]);  // Datos finales que se mostrarán en la tabla por programas por fecha


const options1 = ref(["41", "24", "20", "15", "14", "12", "8", "6", "5", "4"]);

const programs = computed(() => {
    if (!months.value) return ["All"];

    // Accedemos al objeto del store que contiene los programas por meses
    const programsForMonths = store.getters.getProgramMonthsMap[months.value] || [];

    // Agregar "All" al inicio de la lista de programas
    return ["All", ...programsForMonths];
});

const final_date = computed(() => {
    if (!report_date.value || !months.value) return "";

    // Convertir la fecha de reporte a un objeto Date
    const reportDate = new Date(report_date.value);

    // Aplicar la fórmula: restar los meses según la lógica: (S8*1.5+3)-1
    const monthsToSubtract = (months.value * 1.5 + 3);
    const fullMonthsToSubtract = Math.floor(monthsToSubtract);
    const fractionalMonth = monthsToSubtract - fullMonthsToSubtract;

    reportDate.setMonth(reportDate.getMonth() - fullMonthsToSubtract);
    const daysInCurrentMonth = new Date(reportDate.getFullYear(), reportDate.getMonth() + 1, 0).getDate();
    const daysToSubtract = Math.round(daysInCurrentMonth * fractionalMonth);

    reportDate.setDate(reportDate.getDate() - daysToSubtract);
    const finalDate = new Date(reportDate.getFullYear(), reportDate.getMonth() + 1, 0);
    return finalDate.toISOString().split('T')[0];
});

const initial_date = computed(() => {
    if (!final_date.value) return "";

    const finalDate = new Date(final_date.value);
    finalDate.setMonth(finalDate.getMonth() - 11);
    finalDate.setDate(0);
    return finalDate.toISOString().split('T')[0];
});

// Opciones de la tabla
const table_option = ref({
    perPage: 5,
    perPageValues: [5, 10, 20, 50],
    skin: 'table',
    pagination: { nav: 'scroll', chunk: 5 },
    sortable: ['Student', 'Program', 'Status', 'Term Started'],
});
const table_option2 = ref({
    perPage: 5,
    perPageValues: [5, 10, 20, 50],
    skin: 'table',
    pagination: { nav: 'scroll', chunk: 5 },

});

const excelDataOriginal = ref([]);  // Variable para almacenar una copia de todos los datos originales del Excel

// Función para manejar la carga de archivos y leer el Excel
const handleFileUpload = (event) => {
    const file = event.target.files[0];
    const reader = new FileReader();

    reader.onload = (e) => {
        const data = new Uint8Array(e.target.result);
        const workbook = XLSX.read(data, { type: 'array' });
        const worksheet = workbook.Sheets[workbook.SheetNames[0]];

        // Guardar los datos del Excel en una variable temporal
        excelDataTemp.value = XLSX.utils.sheet_to_json(worksheet);

        // Hacer una copia de los datos originales del Excel para mantenerlos sin modificar
        excelDataOriginal.value = [...excelDataTemp.value];  // Hacer una copia profunda de los datos
    };

    reader.readAsArrayBuffer(file);
};

const generateData = () => {
    items.value = [];  // Limpiar los datos previos
    itemsData.value = [];  // Limpiar los datos previos de la tabla de programas
    const filteredItems = []; // Aquí guardaremos los estudiantes filtrados por programa

    const initialDateObj = new Date(initial_date.value);
    const finalDateObj = new Date(final_date.value);

    // Obtener todas las fechas del mapa de términos
    const allTermDates = Object.values(store.getters.getTermDateMap).map(termDateStr => new Date(termDateStr));

    // Filtrar solo las fechas que están dentro del intervalo
    const filteredDates = allTermDates.filter(date => {
        return date >= initialDateObj && date <= finalDateObj;
    });

    // Ordenar las fechas filtradas
    filteredDates.sort((a, b) => a - b);

    // Llenar la tabla con las fechas filtradas
    filteredDates.forEach(date => {
        itemsDataByDate.value.push({
            'Initial Date': date.toISOString().split('T')[0], // Convertir la fecha al formato YYYY-MM-DD
            'Total': 0, // Valores por defecto para empezar
            'Graduated': 0,
            'Dropped': 0,
            'Enrolled': 0
        });
    });

    // Imprimir los datos de la tabla en consola para verificar
    console.log('Datos para la tabla por fechas:', itemsDataByDate.value);

    let selectedPrograms = [];
    if (program.value === "All") {
        selectedPrograms = programs.value.filter(p => p !== "All");
    } else {
        selectedPrograms = [program.value];
    }

    if (excelDataTemp.value.length > 0) {
        selectedPrograms.forEach(selectedProgram => {
            let graduateCount = 0;
            let dropCount = 0;
            let placementCount = 0;  // Inicializar el contador de Placement para cada programa
            let enrolledCount = 0;

            // Limpiar el arreglo de graduados al empezar a procesar un nuevo programa
            const graduatedStudents = {};  // Reinicializar para cada programa

            const studentStatusMap = {};

            // Primer paso: recorrer todos los estudiantes en el rango de fechas seleccionado
            excelDataTemp.value.forEach(row => {
                const termStarted = row['Term Started'];
                const startDateStr = store.getters.getTermDateMap[termStarted] || '';

                if (startDateStr) {
                    const startDateObj = new Date(startDateStr);

                    if (startDateObj >= initialDateObj && startDateObj <= finalDateObj) {
                        const item = {
                            Student: row['Student'],
                            Program: row['Program'],
                            Status: row['Status'],
                            'Term Started': termStarted,
                            'Start Date': startDateStr
                        };

                        const programVersions = store.getters.getProgramsVersions[selectedProgram] || [];
                        if (item.Status !== 'Not Enrolled' && item.Status !== 'No Start' && programVersions.includes(item.Program)) {
                            const studentKey = `${item.Student}-${item.Program}-${item['Start Date']}`; // Agregamos la fecha como parte de la clave

                            if (!studentStatusMap[studentKey]) {
                                studentStatusMap[studentKey] = {
                                    item: item,
                                    Graduated: false,
                                    Dropped: false,
                                    TransferredOut: false,
                                    Enroll: false
                                };
                            }

                            // Marcar los estados
                            if (item.Status === 'Graduated') {
                                studentStatusMap[studentKey].Graduated = true;
                                if (!graduatedStudents[item.Student]) {
                                    graduatedStudents[item.Student] = [];
                                }
                                graduatedStudents[item.Student].push(item);
                            }
                            if (item.Status === 'Dropped') {
                                studentStatusMap[studentKey].Dropped = true;
                            }
                            if (item.Status === 'Transferred Out') {
                                studentStatusMap[studentKey].TransferredOut = true;
                            }
                            if (item.Status === 'Enroll') {
                                studentStatusMap[studentKey].Enroll = true;
                            }
                        }
                    }
                }
            });

            // Segunda pasada: verificar si los estudiantes graduados tienen estado "Enroll" después de la fecha final
            Object.keys(graduatedStudents).forEach(studentName => {
                const studentRecords = graduatedStudents[studentName];

                // Aquí verificamos en excelDataOriginal (datos sin filtrar) si tienen un estado Enroll después de la fecha final
                const hasFutureEnroll = excelDataOriginal.value.some(row => {
                    const studentEnrollDate = new Date(row['Start Date']);
                    return row['Student'] === studentName && studentEnrollDate > finalDateObj && row['Status'] === 'Enroll';
                });

                if (hasFutureEnroll) {
                    console.log(`Estudiante que no necesita Placement: ${studentName}`);
                }

                studentRecords.forEach(record => {
                    if (!hasFutureEnroll) {
                        placementCount++;  // Incrementar solo si el estudiante necesita Placement
                    }
                });
            });

            // Aplicar las reglas para cada estudiante por programa
            Object.keys(studentStatusMap).forEach(studentKey => {
                const status = studentStatusMap[studentKey];

                if (status.Graduated && status.TransferredOut) {
                    graduateCount++;
                    filteredItems.push({ ...status.item, Status: 'Graduated' });
                } else if (status.Dropped && status.TransferredOut) {
                    dropCount++;
                    filteredItems.push({ ...status.item, Status: 'Dropped' });
                } else if (status.TransferredOut && status.Enroll) {
                    enrolledCount++;
                    filteredItems.push({ ...status.item, Status: 'Enroll' });
                } else if (status.TransferredOut) {
                    dropCount++;
                    filteredItems.push({ ...status.item, Status: 'Dropped' });
                } else if (status.Graduated) {
                    graduateCount++;
                    filteredItems.push({ ...status.item, Status: 'Graduated' });
                } else if (status.Dropped) {
                    dropCount++;
                    filteredItems.push({ ...status.item, Status: 'Dropped' });
                } else if (status.Enroll) {
                    enrolledCount++;
                    filteredItems.push({ ...status.item, Status: 'Enroll' });
                }
            });

       

            // Añadir los datos finales del programa a la tabla
            itemsData.value.push({
                Program: selectedProgram,
                Total: graduateCount + dropCount + enrolledCount,
                Graduate: graduateCount,
                Dropped: dropCount,
                Enrolled: enrolledCount,
                Placement: placementCount  // Valor de Placement ya corregido
            });
        });

        // Mostrar los estudiantes filtrados
        items.value = filteredItems;

        // Imprimir los datos en la consola
        console.log('Todos los estudiantes (sin filtros):', excelDataOriginal.value);
        console.log(filteredItems);
        
      
      
        itemsDataByDate.value = [];  // Limpiar los datos previos de la tabla de programas por fechas
    
    const dateGroupedData = [];  // Arreglo de objetos para agrupar por fechas

    // Recorrer los items filtrados
    filteredItems.forEach(item => {
        const startDate = item['Start Date'];

        // Buscar si ya existe un objeto con la misma fecha
        const existingDateEntry = dateGroupedData.find(entry => entry['Initial Date'] === startDate);

        if (existingDateEntry) {
            // Si la fecha ya existe, actualizamos los contadores
            existingDateEntry.Total++;
            if (item.Status === 'Graduated') {
                existingDateEntry.Graduated++;
            } else if (item.Status === 'Dropped') {
                existingDateEntry.Dropped++;
            } else if (item.Status === 'Enroll') {
                existingDateEntry.Enrolled++;
            }
        } else {
            // Si la fecha no existe, añadimos un nuevo objeto con esa fecha
            dateGroupedData.push({
                'Initial Date': startDate,
                'Total': 1, // Empezamos con 1 porque es el primer estudiante en esa fecha
                'Graduated': item.Status === 'Graduated' ? 1 : 0,
                'Dropped': item.Status === 'Dropped' ? 1 : 0,
                'Enrolled': item.Status === 'Enroll' ? 1 : 0
            });
        }
    });

    // Asignar los datos agrupados a la tabla
    itemsDataByDate.value = dateGroupedData;

    // Mostrar los datos agrupados por fechas en la consola
    console.log('Datos agrupados por fecha:', itemsDataByDate.value);

// Mostrar los datos actualizados agrupados por fecha
console.log('Datos agrupados por fecha:', itemsDataByDate.value);


    }
};






const exportToExcel = () => {
    // Crear un nuevo libro de trabajo y una hoja
    const wb = XLSX.utils.book_new();
    
    // Valores dinámicos que vamos a usar
    const selectedProgram = program.value;  // Asumo que 'program.value' es el programa seleccionado
    const totals = itemsData.value.reduce((totals, item) => {
        totals.total += item.Total;
        totals.graduate += item.Graduate;
        totals.dropped += item.Dropped;
        totals.enrolled += item.Enrolled;
        totals.placement += item.Placement;
        return totals;
    }, { total: 0, graduate: 0, dropped: 0, enrolled: 0, placement: 0 });

    const ws = XLSX.utils.aoa_to_sheet([
        [], // Fila vacía
        [], // Fila vacía
        [], // Fila vacía
        ['Date Report:', report_date.value],  // C4: Date Report
        [], // Fila vacía
        ['Program Length in Months:', months.value], // C5: Program Length in Months
        [], // Fila vacía
        ['Beginning Date:', initial_date.value, '', '', 'Ending Date:', final_date.value], // C6: Beginning Date en C6, Ending Date en F6
        [], // Fila vacía
        ['Program:', selectedProgram], // Mostrar el programa seleccionado
        ['', '', '', 'Total', 'Graduate', 'Dropped', 'Enrolled', 'Placement'], // Encabezados desde E9 en adelante
        ['', '', '', totals.total, totals.graduate, totals.dropped, totals.enrolled, totals.placement] // Totales debajo de los encabezados
    ]);

    // Añadir la hoja al libro de trabajo
    XLSX.utils.book_append_sheet(wb, ws, "GraduationData");

    // Descargar el archivo
    XLSX.writeFile(wb, "GraduationData.xlsx");
};










// Función para asignar clases según el estado
const getStatusClass = (status) => {
    if (status === 'Graduated') return 'badge-success';
    if (status === 'Dropped') return 'badge-danger';
    if (status === 'No Start') return 'badge-warning';
    return 'badge-info'; // Clase por defecto
};

watch(months, (newValue) => {
    program.value = "All";  // Cambiar automáticamente el valor de program a "All"
});

</script>
