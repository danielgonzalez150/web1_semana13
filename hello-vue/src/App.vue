<template>
  <h1>Gestor de tareas</h1>
  <div class="container">
    <input v-model="nuevaTareaTitle" placeholder="Escribe la nueva tarea a realizar" @keyup.enter="agregarTarea" />
    <button @click="agregarTarea"> Agregar </button>
  </div>
  <hr>
  <div class="tareas" v-if="tareasPorHacer.length > 0 || tareasEnProceso.length > 0 || tareasRealizadas.length > 0">
    <h2>Tabla de tareas</h2>
    <div class="tablero">
    <section class="tareas-column todo">
      <h3>Tareas por hacer</h3>
      <ul>
      <li v-for="tarea in tareasPorHacer" :key="tarea.id">
          {{ tarea.title }}
          <button @click="porHacerAProceso(tarea.id)">→</button>
        </li>
      </ul>
    </section>
    <section class="tareas-column inprogress">
      <h3>Tareas en proceso</h3>
      <ul>
        <li v-for="tarea in tareasEnProceso" :key="tarea.id">
          {{ tarea.title }}
          <button @click="procesoARealizada(tarea.id)">→</button>
        </li>
     </ul>
    </section>
    <section class="tareas-column done">
      <h3>Tareas realizadas</h3>
      <ul>
        <li v-for="tarea in tareasRealizadas" :key="tarea.id">
          {{ tarea.title }}
        </li>
     </ul>
    </section>
    </div>
  </div>
  <div v-else>
    <p>No hay tareas agregadas</p>
  </div>
</template>
<script>
import { ref, onMounted, computed } from "vue"; 

const API_URL = 'http://localhost:3000/tasks'; 

export default {
  name: 'App',
  setup() {

    const tasks = ref([]); 
    const nuevaTareaTitle = ref(''); 

    const fetchTasks = async () => {
      try {
        const response = await fetch(API_URL);
        if (response.ok) {
            const data = await response.json();
            tasks.value = Array.isArray(data) ? data : [];
        } else {
             console.error("Error en la respuesta del servidor:", response.status, response.statusText);
        }
      } catch (error) {
        console.error("Error de conexión al obtener tareas. ¿Está JSON Server encendido en http://localhost:3000?", error);
      }
    };

    const agregarTarea = async () => {
      const title = nuevaTareaTitle.value.trim(); 
      if (title !== '') {
        const nueva = {
          title: title,

          status: 'todo' 
        };
        try {
          const response = await fetch(API_URL, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(nueva)
          });
          const data = await response.json();
          tasks.value.push(data); 
          nuevaTareaTitle.value = '';
        } catch (error) {
          console.error("Error al agregar la tarea:", error);
        }
      }
    };

    const actualizarEstadoTarea = async (id, nuevoEstado) => {
      try {
        const response = await fetch(`${API_URL}/${id}`, {

          method: 'PATCH', 
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ status: nuevoEstado }) 
        });

        if (response.ok) {
          const index = tasks.value.findIndex(t => t.id == id); 
          if (index !== -1) {
            tasks.value[index].status = nuevoEstado;
          }
        } else {
             console.error("Error al actualizar la tarea:", response.statusText);
        }
      } catch (error) {
        console.error("Error de conexión al actualizar el estado:", error);
      }
    };

    const porHacerAProceso = (id) => {
      actualizarEstadoTarea(id, 'doing'); 
    };

    const procesoARealizada = (id) => {
      actualizarEstadoTarea(id, 'done');
    };
    const tareasPorHacer = computed(() => tasks.value.filter(tarea => tarea.status === 'todo'));
    const tareasEnProceso = computed(() => tasks.value.filter(tarea => tarea.status === 'doing'));
    const tareasRealizadas = computed(() => tasks.value.filter(tarea => tarea.status === 'done'));
    onMounted(fetchTasks); 

    return {
      nuevaTareaTitle,
      tareasPorHacer,
      tareasEnProceso,
      tareasRealizadas, 
      agregarTarea,
      porHacerAProceso,
      procesoARealizada,
    };
  }
};
</script>
<style>
.container {
  display: flex;
  justify-content: center;
  gap: 10px;
  margin-bottom: 20px;
}
input {
  padding: 8px;
  width: 70%;
  border: 1px solid #ccc;
  border-radius: 4px;
}
.todo button {
  padding: 8px 12px;
  margin-left: 8px;
  border: none;
  background-color: #28a745;
  color: white;
  border-radius: 4px;
  cursor: pointer;
}
.inprogress button {
  padding: 8px 12px;
  margin-left: 8px;
  border: none;
  background-color: #007bff;
  color: white;
  border-radius: 4px;
  cursor: pointer;
}
ul {
  list-style: none;
  padding: 0;
  margin: 0;
}
li {
  margin-bottom: 10px;
  background: white;
  padding: 6px 10px;
  border-radius: 4px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-color: darkslateblue;
  border-style: solid;
  color: black;
}
.tablero {
  display: flex; 
  gap: 20px; 
  padding: 10px;
  justify-content: center; 
  align-items: flex-start; 
}

.tareas-column {
  flex: 1; 
  min-width: 0;
  padding: 15px;
  border-radius: 10px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.05);
  background: #fdfdfd;
  max-height: 400px; 
  overflow-y: auto; 
}
body {
  color:black;
}
</style>