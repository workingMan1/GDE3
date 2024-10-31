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
                    </div>
                </div>
            </div>

                        <!-- Tabla para mostrar los datos según la duración del programa en meses -->
                        <div class="row layout-top-spacing">
                <div class="col-xl-12 col-lg-12 col-sm-12 layout-spacing">
                    <button type="button" @click="exportMonthsData" class="btn btn-success btn-lg mb-4">Export</button>
                    <div class="panel p-0">
                        <div class="custom-table">
                            <v-client-table 
                        :data="monthsData" 
                        :columns="['Program Length', 'Started', 'Graduated', 'Dropped', 'Active', 'Available for Placement', 'Initial Date', 'Final Date']"  
                        :options="table_option2">
                        
                        </v-client-table>
                        </div>
                    </div>
                </div>
            </div>

          
            <!-- Tabla para mostrar los datos de los programas -->
            <div class="row layout-top-spacing">
                <div class="col-xl-12 col-lg-12 col-sm-12 layout-spacing">
                    <button type="button" @click="exportItemsData" class="btn btn-success btn-lg mb-4">Export</button>
                    <div class="panel p-0">
                        <div class="custom-table">
                            <v-client-table :data="itemsData" :columns="columnsDataProgram" :options="table_option2"
                                >
                            </v-client-table>

                        </div>
                    </div>
                </div>
            </div>

       

            <!-- <vue3-json-excel class="btn btn-primary m-1" name="table.xls" :fields="excel_columns()" :json-data="excel_items()">Export</vue3-json-excel> -->


            <!-- Tabla para mostrar los datos del nuevo Excel -->
            <div class="row layout-top-spacing">
                <div class="col-xl-12 col-lg-12 col-sm-12 layout-spacing">
                    <button type="button" @click="exportItemsDataByDate" class="btn btn-success btn-lg mb-4">Export</button>
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

    
    const defaultFilePath = process.env.NODE_ENV === 'production'
  ? '/GDE3/datos.xlsx'  // Para producción en GitHub Pages (reemplaza GDE3 por el nombre de tu repo)
  : '/datos.xlsx';  // Para desarrollo local // Ruta relativa al archivo en el directorio 'public'
    fetch(defaultFilePath)
        .then(response => response.arrayBuffer())
        .then(data => {
            const workbook = XLSX.read(data, { type: 'array' });
            const worksheet = workbook.Sheets[workbook.SheetNames[0]];
            excelDataTemp.value = XLSX.utils.sheet_to_json(worksheet);
            excelDataOriginal.value = [...excelDataTemp.value];

            // Solo cuando la carga del Excel está completa, ejecutar la generación de datos:
            generateData();
        });

  
});
 


const generateData = () => {
    items.value = [];  // Limpiar los datos previos
    itemsData.value = [];  // Limpiar los datos previos de la tabla de programas
    monthsData.value = []; // Limpiar cualquier dato previo en monthsData
    const filteredItems = []; // Aquí guardaremos los estudiantes filtrados por programa

    // Usar todos los valores de options1 para iterar
    options1.value.forEach(month => {
        // Actualizar dinámicamente los programas según el valor de month
        const currentPrograms = store.getters.getProgramMonthsMap[month] || [];

        // Calcular Final Date en función del valor actual de month
        const reportDate = new Date(report_date.value);
        const monthsToSubtract = (month * 1.5 + 3);
        const fullMonthsToSubtract = Math.floor(monthsToSubtract);
        const fractionalMonth = monthsToSubtract - fullMonthsToSubtract;

        reportDate.setMonth(reportDate.getMonth() - fullMonthsToSubtract);
        const daysInCurrentMonth = new Date(reportDate.getFullYear(), reportDate.getMonth() + 1, 0).getDate();
        const daysToSubtract = Math.round(daysInCurrentMonth * fractionalMonth);
        reportDate.setDate(reportDate.getDate() - daysToSubtract);

        const calculatedFinalDate = new Date(reportDate.getFullYear(), reportDate.getMonth() + 1, 0);
        const finalDateStr = calculatedFinalDate.toISOString().split('T')[0];

        // Calcular Initial Date a partir de la Final Date calculada
        const initialDateObj = new Date(finalDateStr);
        initialDateObj.setMonth(initialDateObj.getMonth() - 11);
        initialDateObj.setDate(0);
        const initialDateStr = initialDateObj.toISOString().split('T')[0];

        // Añadir los datos a monthsData
        monthsData.value.push({
            'Program Length': month,
            'Initial Date': initialDateStr,
            'Final Date': finalDateStr
        });

        // Continuar con el procesamiento de estudiantes usando estas fechas
        const initialDateObjForFilter = new Date(initialDateStr);
        const finalDateObjForFilter = new Date(finalDateStr);

        if (excelDataTemp.value.length > 0) {
            currentPrograms.forEach(selectedProgram => {
                const programVersions = store.getters.getProgramsVersions[selectedProgram] || [];

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

                        if (startDateObj >= initialDateObjForFilter && startDateObj <= finalDateObjForFilter) {
                            const item = {
                                Student: row['Student'],
                                Program: row['Program'],
                                Status: row['Status'],
                                'Term Started': termStarted,
                                'Start Date': startDateStr,
                                
                            };

                            // Verificar si el programa del estudiante coincide con alguna de las versiones permitidas
                            if (
                                item.Status !== 'Not Enrolled' && 
                                item.Status !== 'No Start' && 
                                programVersions.some(version => version === item.Program)
                            ) {
                                const studentKey = `${item.Student}-${item.Program}-${item['Start Date']}`;

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
                        return row['Student'] === studentName && studentEnrollDate > finalDateObjForFilter && row['Status'] === 'Enroll';
                    });

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
                    Started: graduateCount + dropCount + enrolledCount,
                    Graduated: graduateCount,
                    Dropped: dropCount,
                    Active: enrolledCount,
                    Longitud: month,
                    'Available for Placement': placementCount  // Valor de Placement ya corregido
                });
            });
        }
    });

    // Numerar los estudiantes filtrados
    filteredItems.forEach((item, index) => {
        item['No'] = index + 1;
    });

    // Asignar el resultado acumulado de estudiantes filtrados
    items.value = filteredItems;

    // Calcular la suma de la columna "Started"
    const totalStarted = itemsData.value.reduce((sum, item) => sum + (item.Started || 0), 0);

    // Arreglo final con los datos agregados por longitud de programa
const aggregatedData = [];

// Objeto temporal para almacenar los totales por longitud
const lengthTotals = {};

// Recorrer cada elemento de itemsData para sumar valores por longitud
itemsData.value.forEach(item => {
    const length = item.Longitud; // Obtener la longitud de cada programa

    // Inicializar el objeto para esta longitud si aún no existe
    if (!lengthTotals[length]) {
        lengthTotals[length] = {
            Longitud: length,
            Started: 0,
            Graduated: 0,
            Dropped: 0,
            Active: 0,
            'Available for Placement': 0
        };
    }

    // Sumar los valores en los campos correspondientes
    lengthTotals[length].Started += item.Started || 0;
    lengthTotals[length].Graduated += item.Graduated || 0;
    lengthTotals[length].Dropped += item.Dropped || 0;
    lengthTotals[length].Active += item.Active || 0;
    lengthTotals[length]['Available for Placement'] += item['Available for Placement'] || 0;
});

// Convertir el objeto lengthTotals en un arreglo de resultados
for (const length in lengthTotals) {
    aggregatedData.push(lengthTotals[length]);
}

// Mostrar el arreglo resultante en consola
console.log(aggregatedData);

// Actualizar monthsData en función de aggregatedData
monthsData.value = monthsData.value.map(item => {
    const aggregateItem = aggregatedData.find(data => data.Longitud === item['Program Length']);
    
    if (aggregateItem) {
        return {
            ...item,  // Mantener las fechas de Initial Date y Final Date tal cual
            Started: aggregateItem.Started,
            Graduated: aggregateItem.Graduated,
            Dropped: aggregateItem.Dropped,
            Active: aggregateItem.Active,
            'Available for Placement': aggregateItem['Available for Placement']
        };
    }
    return item; // Mantener el elemento sin cambios si no hay coincidencia
});

const totalRow = {
    'Program Length': 'Total',
    Started: monthsData.value.reduce((sum, item) => sum + (item.Started || 0), 0),
    Graduated: monthsData.value.reduce((sum, item) => sum + (item.Graduated || 0), 0),
    Dropped: monthsData.value.reduce((sum, item) => sum + (item.Dropped || 0), 0),
    Active: monthsData.value.reduce((sum, item) => sum + (item.Active || 0), 0),
    'Available for Placement': monthsData.value.reduce((sum, item) => sum + (item['Available for Placement'] || 0), 0),
    'Initial Date': '',
    'Final Date': '',
};

// Añadir la fila total al final de monthsData
monthsData.value.push(totalRow);



};







const exportMonthsData = () => {
    const workbook = XLSX.utils.book_new();
    
    // Orden de las columnas para `monthsData`
    const worksheet = XLSX.utils.json_to_sheet(monthsData.value, { header: ['Program Length', 'Started', 'Graduated', 'Dropped', 'Active', 'Available for Placement', 'Initial Date', 'Final Date'] });
    
    XLSX.utils.book_append_sheet(workbook, worksheet, 'Months Data');
    XLSX.writeFile(workbook, 'months_data.xlsx');
};

const exportItemsData = () => {
    const workbook = XLSX.utils.book_new();
    
    // Orden de las columnas para `itemsData`
    const worksheet = XLSX.utils.json_to_sheet(itemsData.value, { header: ['Program', 'Started', 'Graduated', 'Dropped', 'Active', 'Available for Placement'] });
    
    XLSX.utils.book_append_sheet(workbook, worksheet, 'Items Data');
    XLSX.writeFile(workbook, 'items_data.xlsx');
};

const exportItemsDataByDate = () => {
    const workbook = XLSX.utils.book_new();
    
    // Orden de las columnas para `items`
    const worksheet = XLSX.utils.json_to_sheet(items.value, { header: ['No', 'Student', 'Program', 'Status', 'Term Started', 'Start Date'] });
    
    XLSX.utils.book_append_sheet(workbook, worksheet, 'Items Data By Date');
    XLSX.writeFile(workbook, 'items_data_by_date.xlsx');
};







// Encabezados de la tabla con la nueva columna de 'Start Date'
const columns = ref([
    'No',
    'Student',
    'Program',
    'Status',
    'Term Started',
    'Start Date' // Nueva columna
]);

const columnsDataProgram = ref([
    'Program',
    'Started',
    'Graduated',
    'Dropped',
    'Active',
    'Available for Placement',
      // Nueva columna para mostrar la longitud
]);


const columnsDataProgramByDate = ref([
    'Initial Date',
    'Started',
    'Graduated',
    'Dropped',
    'Active',
]);

const report_date = ref("2025-07-01"); // Fecha final seleccionada
const months = ref("");
const program = ref("All");
const excelDataTemp = ref([]); // Variable temporal para almacenar los datos del Excel
const items = ref([]);  // Datos finales que se mostrarán en la tabla
const itemsData = ref([]);  // Datos finales que se mostrarán en la tabla por programas

const monthsData = ref([]);


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
    perPage: 15,
    perPageValues: [5, 10, 20, 50],
    skin: 'table',
    pagination: { nav: 'scroll', chunk: 5 },
    sortable: ['Student', 'Program', 'Status',  'Start Date'],
    

});
const table_option2 = ref({
    perPage: 15,
    perPageValues: [5, 10, 20, 50],
    skin: 'table',
    pagination: { nav: 'scroll', chunk: 5 },
    sortable: [],
});

const excelDataOriginal = ref([]);  // Variable para almacenar una copia de todos los datos originales del Excel





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
<style>
    .total-row {
    font-size: 3.2em; /* Ajusta el tamaño según prefieras */
    font-weight: bold;
}
</style>
