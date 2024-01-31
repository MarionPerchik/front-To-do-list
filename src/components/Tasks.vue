<template>
  <div class="container">
    <div class="row">
      <div class="col-sm-10">
        <h1>To-do-list</h1>
        <hr><br><br>
        <button type="button" class="btn btn-success btn-sm" @click="openAddTaskForm">Добавить задачу</button>
        <br><br>
        <table class="table table-hover">
          <thead>
            <tr>
              <th scope="col">Название</th>
              <th scope="col">Описание</th>
              <th scope="col">Дедлайн от</th>
              <th scope="col">Дедлайн до</th>
              <th></th>
            </tr>
          </thead>
          <tbody>
            <tr v-if="showAddTaskForm" :key="tasks.length">
              <td><input v-model="newTask.title" /></td>
              <td><input v-model="newTask.description" /></td>
              <td><input v-model="newTask.deadline1" type="date" /></td>
              <td><input v-model="newTask.deadline2" type="date" /></td>
              <td>
                <button type="button" class="btn btn-primary btn-sm mx-1" @click="saveNewTask">Сохранить</button>
                <button type="button" class="btn btn-danger btn-sm mx-1" @click="closeAddTaskForm">Отмена</button>
              </td>
            </tr>
            <tr v-for="(task, index) in tasks" :key="index">
              <td>
                <span v-if="editedTaskIndex !== index">{{ task.title }}</span>
                <input v-else v-model="task.title">

              </td>
              <td>
                <span v-if="editedTaskIndex !== index">{{ task.description }}</span>
                <input v-else v-model="task.description">
              </td>
              <td>
                <span v-if="editedTaskIndex !== index">{{ task.deadline1 }}</span>
                <input v-else v-model="task.deadline1" type="date">
              </td>
              <td>
                <span v-if="editedTaskIndex !== index">{{ task.deadline2 }}</span>
                <input v-else v-model="task.deadline2" type="date">
              </td>
              <td>
                <button v-if="editedTaskIndex !== index" type="button" class="btn btn-warning btn-sm mx-1" @click="openEditModal(index)">Изменить</button>
                <button v-else-if="!showAddTaskForm" type="button" class="btn btn-primary btn-sm mx-1" @click="saveChanges(index)">Сохранить</button>
                <button type="button" class="btn btn-danger btn-sm mx-1" @click="deleteTask(index)">Удалить</button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      tasks: [], // инициализация пустого массива
      editedTaskIndex: null,
      path: "http://localhost:5000/tasks", //адрес для запросов
      showAddTaskForm: false, //заглушка для открытия формочки создания тасков
      newTask: { //пустой JSON таск для будущего заполнения
        title: '', 
        description: '',
        deadline1: '',
        deadline2: '',
      },
    };
  },

  methods: {
    openEditModal(index) {
      this.editedTaskIndex = index;
    },

    openAddTaskForm() {
    this.showAddTaskForm = true;
    },

    closeAddTaskForm() {
      this.showAddTaskForm = false;
      // Очистка данных формы после закрытия
      this.newTask = {
        title: '',
        description: '',
        deadline1: '',
        deadline2: '',
      };
    },

    saveNewTask() {
      axios.post('http://localhost:5000/tasks', this.newTask)
        .then(response => {
          console.log(response.data.message);
          this.closeAddTaskForm();
          this.getTasks();
        })
        .catch(error => {
          console.error(error);
        });
    },

    saveChanges(index) {
      const updatedTask = this.tasks[index]; // запоминаем индекс редактируемой задачи
      axios.patch(`${this.path}/${index}`, updatedTask) // закидываем изменения
        .then((res) => {
          console.log(res.data)
          this.getTasks(); // обновляем список тасков
          this.editedTaskIndex = null;
        })
        .catch(error => {
          console.log(error)
        })
    },

    getTasks() {
      axios.get(this.path)
        .then((res) => {
          this.tasks = res.data.tasks;
        })
        .catch((error) => {
          console.error(error);
        });
    },
    
    deleteTask(index) {
    axios.delete(`${this.path}/${index}`)
      .then((response) => {
        console.log(response.data);
        this.getTasks();
      })
      .catch((error) => {
        console.error(error);
      });
    },
  },

  created() {
    this.getTasks();
  },
};
</script>
