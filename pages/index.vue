<template>
  <!--
    🧭 Contenedor principal con alineación centrada
    🧭 Main container with centered alignment
  -->
  <div class="text-center">
    <!--
      📌 Título principal en español e inglés
      📌 Main title in Spanish and English
    -->
    <h1 class="fw-bold mb-3">Sube tu archivo ZIP / Upload your ZIP file</h1>

    <!--
      📖 Descripción de la funcionalidad de la herramienta
      📖 Description of what the tool does
    -->
    <p class="mb-4 text-muted">
      Esta herramienta te permite unir el contenido de múltiples archivos de
      texto plano (.txt o sin extensión) contenidos dentro de un archivo ZIP y
      visualizarlo con los saltos de línea preservados.<br />
      <strong>Importante:</strong> el ZIP debe contener una carpeta con
      cualquier nombre que incluya los archivos de texto. Ejemplo:
      <code>archivo.zip → carpeta → archivos.txt</code> o archivos sin
      extensión.<br /><br />
      This tool allows you to merge the content of multiple plain text files
      (.txt or without extension) inside a ZIP and display the result with
      preserved line breaks.<br />
      <strong>Important:</strong> the ZIP must contain a folder (any name) that
      holds the text files. Example:
      <code>file.zip → folder → .txt files</code>.
    </p>

    <!--
      📦 Componente de carga de archivos con soporte drag & drop
      📦 File upload component with drag & drop support
    -->
    <FileDrop @file-selected="handleFileSelected" :selectedFile="selectedFile" @clear-file="clearFile" />

    <!--
      ⚙️ Botón para iniciar el procesamiento del archivo
      ⚙️ Button to trigger ZIP file processing
    -->
    <div class="my-3">
      <button class="btn btn-primary px-4" :disabled="!selectedFile" @click="processFile">
        <i class="fas fa-cogs"></i> Procesar / Process
      </button>
    </div>

    <!--
      ⏳ Componente de carga que aparece mientras se consulta la API
      ⏳ Loading modal shown while waiting for API response
    -->
    <LoadingModal v-if="loading" />

    <!--
  📄 Visor del resultado en texto plano si ya hay respuesta
  📄 Displays the resulting plain text if available
  -->
    <JsonViewer v-if="resultJson" :text="resultJson" />
  </div>
</template>

<script setup>
// 📦 Importación de componentes del proyecto
// 📦 Importing project components
import FileDrop from "@/components/FileDrop.vue";
import LoadingModal from "@/components/LoadingModal.vue";
import JsonViewer from "@/components/JsonViewer.vue";

// 🔁 Composición de Vue y librería para alertas
// 🔁 Vue Composition API and SweetAlert for alerts
import { ref } from "vue";
import Swal from "sweetalert2";

// 🌐 Servicio API para subir archivos ZIP
// 🌐 API service to upload ZIP files
import { useApiService } from "@/services/api";

// 📁 Archivo actualmente seleccionado por el usuario
// 📁 Currently selected ZIP file
const selectedFile = ref(null);

// 📊 Resultado JSON que será mostrado después de procesar
// 📊 JSON result to be displayed after processing
const resultJson = ref(null);

// ⏳ Estado de carga para mostrar el modal de loading
// ⏳ Loading state for showing the loading modal
const loading = ref(false);

// 📥 Manejador cuando el usuario selecciona un archivo
// 📥 Handler when user selects a ZIP file
const handleFileSelected = (file) => {
  selectedFile.value = file;
};

// 🗑 Manejador para limpiar archivo y resultado
// 🗑 Handler to clear selected file and JSON result
const clearFile = () => {
  selectedFile.value = null;
  resultJson.value = null;
};

// 🚀 Procesa el archivo cargado llamando al backend FastAPI
// 🚀 Processes the uploaded ZIP file by calling the FastAPI backend
const processFile = async () => {
  if (!selectedFile.value) return; // 🛑 Verificación: no hay archivo

  loading.value = true; // ⏳ Activar modal de carga
  const api = useApiService(); // 🔗 Instanciar el servicio API

  try {
    // 📡 Enviar el archivo ZIP al backend y obtener la respuesta
    const result = await api.uploadZip(selectedFile.value);
    resultJson.value = result.text;
  } catch (err) {
    // ❌ Mostrar error si falla la petición
    Swal.fire("Error", err.message, "error");
  } finally {
    // ✅ Ocultar modal de carga
    loading.value = false;
  }
};
</script>
