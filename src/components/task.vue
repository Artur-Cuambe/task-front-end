<template>
  <v-container class="fill-height">
    <v-responsive class="align-center text-center fill-height">
      <v-container style="max-width: 800px">
        <v-text-field
          v-model="task.title"
          variant="solo"
          :label="
            task._id === ''
              ? `Forneça o titulo da tarefa que deseja adicionar.`
              : `Forneça o titulo da tarefa que deseja actualizar.`
          "
        >
        </v-text-field>
        <v-text-field
          v-model="newTask"
          :label="
            task._id === ''
              ? `Forneça detalhes sobre a tarefa que deseja adicionar.`
              : `Forneça detalhes sobre a tarefa que deseja actualizar.`
          "
          variant="solo"
          @keydown.enter="create"
        >
        </v-text-field>
        <!-- <v-fade-transition> -->
        <v-btn
          :disabled="!newTask"
          variant="outlined"
          @click="create"
          v-if="task._id === ''"
          >Adicionar</v-btn
        >
        <v-btn
          :disabled="!newTask"
          variant="outlined"
          color="primary"
          @click="edit(task)"
          v-else
          >Salvar</v-btn
        >
        <!-- </v-fade-transition> -->
        <v-divider class="mt-4"></v-divider>

        <h2 class="text-h4 text-primary">Gerenciamento de tarefas</h2>

        <v-divider class="mt-4"></v-divider>

        <v-row class="my-1" align="center">
          <strong class="mx-4 text-info-darken-2">
            Total:
            <v-fade-transition leave-absolute>
              <span :key="`tasks-${tasks.length}`">
                {{ tasks.length }}
              </span>
            </v-fade-transition>
          </strong>

          <v-divider vertical></v-divider>

          <strong class="mx-4 text-success-darken-2">
            Em falta: {{ remainingTasks }}
          </strong>
          <v-divider vertical></v-divider>

          <strong class="mx-4 text-success-darken-2">
            Feitas: {{ completedTasks }}
          </strong>

          <v-spacer></v-spacer>

          <v-progress-circular
            v-model="progress"
            class="me-2"
            color="primary"
          ></v-progress-circular>
        </v-row>

        <v-divider class="mb-4"></v-divider>

        <v-card v-if="tasks.length > 0">
          <v-slide-y-transition class="py-0" group tag="v-row">
            <template v-for="(t, i) in tasks" :key="`${i}-${t.description}`">
              <v-divider v-if="i !== 0" :key="`${i}-divider`"></v-divider>

              <v-row no-gutters>
                <v-col col="12" md="1">
                  <v-checkbox-btn
                    v-model="t.completed"
                    color="grey"
                    @change="edit(t)"
                  ></v-checkbox-btn
                ></v-col>
                <v-col col="12" md="2">
                  <span
                    :class="
                      t.completed
                        ? 'text-grey text-decoration-line-through font-weight-bold'
                        : 'text-primary font-weight-bold'
                    "
                    >{{ t.title }}</span
                  >
                </v-col>
                <v-col col="12" md="6">
                  <span
                    :class="
                      t.completed
                        ? 'text-grey text-decoration-line-through'
                        : 'text-primary'
                    "
                    >{{ t.description }}</span
                  >
                </v-col>
                <v-col col="12" md="3">
                  <v-row no-gutters class="ma-3">
                    <!-- <v-spacer></v-spacer> -->
                    <v-expand-x-transition>
                      <v-icon v-if="t.completed" color="success">
                        mdi-check
                      </v-icon>
                    </v-expand-x-transition>
                    <v-btn
                      density="compact"
                      icon
                      variant="tonal"
                      color="primary"
                      class="ml-1"
                      @click="select(t)"
                      :disabled="t.completed"
                    >
                      <v-icon size="small">mdi-pencil</v-icon>
                    </v-btn>
                    <v-btn
                      density="compact"
                      icon
                      variant="tonal"
                      color="error"
                      class="ml-1"
                      @click="remove(t._id)"
                      :disabled="t.completed"
                    >
                      <v-icon size="small">mdi-delete</v-icon>
                    </v-btn>
                  </v-row>
                </v-col>
              </v-row>
            </template>
          </v-slide-y-transition>
        </v-card>
      </v-container>
    </v-responsive>
  </v-container>
</template>
<script>
import axios from "axios";

export default {
  data: () => ({
    tasks: [],
    task: {
      _id: "",
      description: "",
      title: "",
      completed: false,
    },
    newTask: null,
    url: "http://localhost:4000/tasks",
  }),

  computed: {
    completedTasks() {
      return this.tasks.filter((task) => task.completed).length;
    },
    progress() {
      return (this.completedTasks / this.tasks.length) * 100;
    },
    remainingTasks() {
      return this.tasks.length - this.completedTasks;
    },
  },
  created() {
    this.fetchList();
  },
  methods: {
    async create() {
      try {
        const { data } = await axios({
          url: this.url,
          method: "post",
          data: {
            title: this.task.title,
            description: this.newTask,
          },
        });
        this.tasks.push(data);
        this.clean();
      } catch (error) {
        console.log(error);
      }
    },
    select(task) {
      this.newTask = task.description;
      this.task = task;
    },
    clean() {
      this.newTask = null;
      this.task = {
        _id: "",
        title: "",
        completed: false,
        description: "",
      };
    },
    async edit(task) {
      try {
        const id = task._id;
        if (this.newTask) {
          task.description = this.newTask;
        }
        const { data } = await axios({
          url: this.url + "/" + id,
          method: "put",
          data: {
            ...task,
          },
        });
        let index = this.tasks.findIndex((t) => t._id === id);
        this.tasks[index] = data;
        this.clean();
      } catch (error) {
        console.log(error);
      }
    },
    async remove(id) {
      try {
        await axios({
          url: this.url + "/" + id,
          method: "delete",
        });
        let index = this.tasks.findIndex((t) => t._id === id);
        this.tasks.splice(index, 1);
        this.clean();
      } catch (error) {
        console.log(error);
      }
    },
    async fetchList() {
      try {
        const { data } = await axios({
          url: this.url,
          method: "get",
        });
        this.tasks = data;
      } catch (error) {
        console.log(error);
      }
    },
  },
};
</script>
