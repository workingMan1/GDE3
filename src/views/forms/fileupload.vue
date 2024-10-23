

<template>
    <div class="container">
        <div class="container">
            <div v-scroll-spy class="row layout-top-spacing">
                <div id="fuSingleFile" class="col-lg-12 layout-spacing">
                    <div class="statbox panel box box-shadow">
                        <div class="panel-heading">
                            <div class="row">
                                <div class="col-xl-12 col-md-12 col-sm-12 col-12">
                                    <h4>Single File Upload</h4>
                                </div>
                            </div>
                        </div>
                        <div class="panel-body">
                            <div class="custom-file-container" data-upload-id="myFirstImage">
                                <label>Upload (Single File) <a href="javascript:void(0)" class="custom-file-container__image-clear" title="Clear Image">x</a></label>
                                <label class="custom-file-container__custom-file">
                                    <input type="file" class="custom-file-container__custom-file__custom-file-input" accept=".xlsx, .xls" @change="handleFileUpload" />
                                    <input type="hidden" name="MAX_FILE_SIZE" value="10485760" />
                                    <span class="custom-file-container__custom-file__custom-file-control"></span>
                                </label>
                                <div class="custom-file-container__image-preview" style="display: none;"></div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Primera Tabla (Con columna "Semestre") -->
            <div class="row layout-top-spacing">
                <div class="col-xl-12 col-lg-12 col-sm-12 layout-spacing">
                    <div class="panel p-0">
                        <div class="custom-table">
                            <v-client-table :data="items" :columns="columns" :options="table_option" @row-click="handleRowClick">
                                <template #semestre="props">
                                    <!-- Mostrar letras A, B, C, D con colores dinámicos -->
                                    <span :class="getTermClass(props.row.terminos.A)" class="badge inv-status me-1">A</span>
                                    <span :class="getTermClass(props.row.terminos.B)" class="badge inv-status me-1">B</span>
                                    <span :class="getTermClass(props.row.terminos.C)" class="badge inv-status me-1">C</span>
                                    <span :class="getTermClass(props.row.terminos.D)" class="badge inv-status">D</span>
                                </template>
                            </v-client-table>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Segunda Tabla (Sin nombre del estudiante) -->
            <div class="row layout-top-spacing">
                <div class="col-xl-12 col-lg-12 col-sm-12 layout-spacing">
                    <div class="panel p-0">
                        <div class="custom-table">
                            <v-client-table :data="secondItems" :columns="columnsWithoutName" :options="table_option">
                                <template #status="props">
                                    <span class="badge inv-status">{{ props.row['Course Description'] }}</span>
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
import { ref,onMounted } from 'vue';
import '@/assets/sass/forms/file-upload-with-preview.min.css';
import '@/assets/sass/scrollspyNav.scss';
import FileUploadWithPreview from 'file-upload-with-preview';
import * as XLSX from 'xlsx';
import { useMeta } from '@/composables/use-meta';
useMeta({ title: 'File Upload' });

onMounted(() => {
    initTooltip();
    new FileUploadWithPreview('myFirstImage', {
        images: {
            baseImage: require('@/assets/images/file-preview.png'),
            backgroundImage: '',
        },
    });
});

const initTooltip = () => {
    setTimeout(() => {
        var tooltipTriggerList = [].slice.call(document.querySelectorAll('[data-bs-toggle="tooltip"]'));
        tooltipTriggerList.map((tooltipTriggerEl) => {
            return new window.bootstrap.Tooltip(tooltipTriggerEl);
        });
    });
};

// Definir los términos y sus fechas
const terminos = {
    A: { start: '2024-05-06', end: '2024-06-02' },
    B: { start: '2024-06-03', end: '2024-06-30' },
    C: { start: '2024-07-01', end: '2024-07-28' },
    D: { start: '2024-07-29', end: '2024-08-25' }
};

// Encabezados de la primera tabla (Con columna "Semestre")
const columns = ref([
    'Student Full Name', 
    'Term Code', 
    'Class Section Start', 
    'Class Section End',
    'semestre' // Nueva columna para las letras A, B, C, D
]);

// Encabezados de la segunda tabla (Sin nombre del estudiante)
const columnsWithoutName = ref([
    'Course Description', 
    'Term Code', 
    'Class Section Start', 
    'Class Section End'
]);

// Datos iniciales para la primera tabla
const items = ref([]);

// Datos iniciales para la segunda tabla
const secondItems = ref([]);

// Objeto para almacenar los cursos agrupados por estudiante
const lista = ref({});

// Opciones de las tablas
const table_option = ref({
    perPage: 5,
    perPageValues: [5, 10, 20, 50],
    skin: 'table',
    pagination: { nav: 'scroll', chunk: 5 },
    sortable: ['Course Description', 'Term Code'],
});

// Función para obtener la clase CSS para un término
const getTermClass = (termStatus) => {
    return termStatus ? 'badge-success' : 'badge-danger'; // Verde si ocupado, rojo si libre
};

// Función para manejar la carga de archivos y leer el Excel
const handleFileUpload = (event) => {
    const file = event.target.files[0];
    const reader = new FileReader();

    reader.onload = (e) => {
        const data = new Uint8Array(e.target.result);
        const workbook = XLSX.read(data, { type: 'array' });
        const worksheet = workbook.Sheets[workbook.SheetNames[0]];

        // Convertir la hoja del Excel a JSON
        const excelData = XLSX.utils.sheet_to_json(worksheet);

        // Limpiar el objeto lista
        lista.value = {};

        // Crear objeto para agrupar cursos por nombre del estudiante
        excelData.forEach(row => {
            const studentName = row['Student.Full Name'];
            const course = {
                'Course Description': row['Course.Description'],
                'Term Code': row['Term.Code'],
                'Class Section Start': formatExcelDate(row['Class Section.Class/Section Start']),
                'Class Section End': formatExcelDate(row['Class Section.Class/Section End']),
            };

            if (lista.value[studentName]) {
                lista.value[studentName].push(course);
            } else {
                lista.value[studentName] = [course];
            }
        });

        // Actualizar la primera tabla (Con columna "Semestre")
        items.value = Object.keys(lista.value).map(name => ({
            'Student Full Name': name,
            'Term Code': 'R24SD-1M', // Valor fijo
            'Class Section Start': '2024-07-28', // Valor fijo
            'Class Section End': '2024-08-24', // Valor fijo
            'terminos': checkTermOccupancy(lista.value[name]) // Checar qué términos están ocupados
        }));

        console.log(lista.value);
    };

    reader.readAsArrayBuffer(file);
};

// Función para verificar qué términos están ocupados
const checkTermOccupancy = (courses) => {
    const occupiedTerms = { A: false, B: false, C: false, D: false };

    courses.forEach(course => {
        const courseStart = course['Class Section Start'];
        const courseEnd = course['Class Section End'];

        // Verificar cada término
        if (courseStart <= terminos.A.end && courseEnd >= terminos.A.start) {
            occupiedTerms.A = true;
        }
        if (courseStart <= terminos.B.end && courseEnd >= terminos.B.start) {
            occupiedTerms.B = true;
        }
        if (courseStart <= terminos.C.end && courseEnd >= terminos.C.start) {
            occupiedTerms.C = true;
        }
        if (courseStart <= terminos.D.end && courseEnd >= terminos.D.start) {
            occupiedTerms.D = true;
        }
    });

    return occupiedTerms;
};

// Función para formatear fechas de Excel
const formatExcelDate = (excelDate) => {
    if (!excelDate) return '';
    if (typeof excelDate === 'number') {
        const date = new Date((excelDate - (25567 + 2)) * 86400 * 1000);
        return date.toISOString().split('T')[0];
    }
    return new Date(excelDate).toISOString().split('T')[0];
};

// Función para manejar el clic en una fila
const handleRowClick = (row) => {
    const studentName = row.row['Student Full Name'];
    let cursos_asociados = [];

    if (lista.value[studentName]) {
        cursos_asociados = lista.value[studentName];
    }

    secondItems.value = cursos_asociados;
    console.log('Cursos asociados:', cursos_asociados);
};
</script>
