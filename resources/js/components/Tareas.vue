<template>
    <div class="container container-task">
        <div class="row">
            <div class="col-md-6">
                <h2>Lista de tareas</h2>
                <table class="table text-center"><!--Creamos una tabla que mostrará todas las tareas-->
                        <thead>
                            <tr>
                                <th scope="col">Nombre</th>
                                <th scope="col">Descripción</th>
                                <th scope="col">Contenido</th>
                                <th scope="col">Acciones</th>
                            </tr>
                        </thead>
                        <tbody>
                            <!--Recorremos el array y cargamos nuestra tabla-->
                            <tr v-for="task in arrayTasks" :key="task.id">
                                <td v-text="task.name"></td>
                                <td v-text="task.description"></td>
                                <td v-text="task.content"></td>
                                <td>
                                    <!--Botón modificar, que carga los datos del formulario con la tarea seleccionada-->
                                    <button class="btn btn-success" @click="loadFieldsUpdate(task)" title="Editar">E</button>
                                    <!--Botón que borra la tarea que seleccionemos-->
                                    <button class="btn btn-danger" @click="deleteTask(task)" title="Borrar">D</button>
                                </td>
                            </tr>
                        </tbody>
                    </table>
            </div>
            <div class="col-md-6">
                <!-- Formulario para la creación o modificación de nuestras tareas-->
                <div class="form-group">
                    <label>Nombre</label>
                    <input v-model="name" type="text" class="form-control">

                    <label>Descripción</label>
                    <input v-model="description" type="text" class="form-control">

                    <label>Contenido</label>
                    <input v-model="content" type="text" class="form-control">
                </div>
                <div class="container-buttons">
                    <!-- Botón que añade los datos del formulario, solo se muestra si la variable update es igual a 0-->
                    <button v-if="update == 0" @click="saveTasks()" class="btn btn-success">Añadir</button>
                    <!-- Botón que modifica la tarea que anteriormente hemos seleccionado, solo se muestra si la variable update es diferente a 0-->
                    <button v-if="update != 0" @click="updateTasks()" class="btn btn-warning">Actualizar</button>
                    <!-- Botón que limpia el formulario y inicializa la variable a 0, solo se muestra si la variable update es diferente a 0-->
                    <button v-if="update != 0" @click="clearFields()" class="btn">Atrás</button>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        data(){
            return{
                name:"",
                description:"",
                content:"",
                update:0,
                arrayTasks:[],
            }
        },
        methods:{
            getTasks(){
                let me =this;
                let url = '/api/tareas'
                axios.get(url).then(function (response) {
                    me.arrayTasks = response.data;
                })
                .catch(function (error) {
                    // handle error
                    console.log(error);
                });
            },
            saveTasks(){
                let me =this;
                let url = '/api/tareas/guardar'
                axios.post(url,{
                    'name':this.name,
                    'description':this.description,
                    'content':this.content,
                }).then(function (response) {
                    me.getTasks();
                    me.clearFields();
                })
                .catch(function (error) {
                    console.log(error);
                });

            },
            /*
             * Esta funcion, es igual que la anterior, solo que tambien envia la variable update
             * que contiene el id de la tarea que queremos modificar
             */
            updateTasks(){
                let me = this;
                axios.put('/api/tareas/actualizar',{
                    'id':this.update,
                    'name':this.name,
                    'description':this.description,
                    'content':this.content,
                }).then(function (response) {
                   me.getTasks();
                   me.clearFields();
                })
                .catch(function (error) {
                    console.log(error);
                });
            },
            /*
             * Esta función rellena los campos y la variable update,
             * con la tarea que queremos modificar
            */
            loadFieldsUpdate(data){
                this.update = data.id
                let me =this;
                let url = '/api/tareas/buscar?id='+this.update;
                axios.get(url).then(function (response) {
                    me.name= response.data.name;
                    me.description= response.data.description;
                    me.content= response.data.content;
                })
                .catch(function (error) {
                    // handle error
                    console.log(error);
                });
            },
            /* Esta nos abrirá un alert de javascript y si aceptamos
             * borrará la tarea que hemos elegido
            */
            deleteTask(data){
                let me =this;
                let task_id = data.id
                if (confirm('¿Seguro que deseas borrar esta tarea?')) {
                    axios.delete('/api/tareas/borrar/'+task_id
                    ).then(function (response) {
                        me.getTasks();
                    })
                    .catch(function (error) {
                        console.log(error);
                    });
                }
            },
            /*
             * Limpia los campos e inicializa la variable update a 0
            */
            clearFields(){
                this.name = "";
                this.description = "";
                this.content = "";
                this.update = 0;
            }
        },
        mounted() {
           this.getTasks();
        }
    }
</script>
