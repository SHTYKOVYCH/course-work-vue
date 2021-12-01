<template>
  <section>
    <section>
      <Task
          v-bind="tasks"
          v-for="task in tasks"
          :key="task.taskId"
          v-bind:task="task"
          v-on:deleteTask="deleteTask"
      ></Task>
    </section>
    <section>
      <form v-on:submit="this.addTask">
        <input name="title" type="text">
        <button type="submit">Добавить</button>
      </form>
    </section>
  </section>
</template>

<script>
import Task from "@/components/Task";

export default {
  name: "TaskList",
  components: {
    Task
  },
  data: function () {
    return {
      tasks: [],
      updating: null,
    }
  },
  mounted: async function() {
    await this.setTasks();
  },
  methods: {
    setTasks: async function () {
      let tasks = (await (await fetch("http://test.site/api/").then(res => res.json()))).tasks;

      let indexes = Object.values(tasks).map(el => el[0]);

      this.tasks.forEach((el, index) => {
        if (indexes.findIndex(index => index == el.id) == -1) {
          this.tasks.splice(index, 1);
        }
      })

      for (let task in tasks) {
        let taskId = this.tasks.findIndex(el => el.id == tasks[task].id);

        let t_task = {};
        t_task["id"] = tasks[task].id;
        t_task["title"] = tasks[task].title;
        t_task["body"] = tasks[task].body ?? '';
        t_task["priority"] = tasks[task].priority;

        if (taskId == -1) {
          this.tasks.push(t_task);
        } else if (!this.tasks[taskId].updating) {
          this.tasks[taskId] = Object.assign(this.tasks[taskId], t_task);
        }
      }

      this.updating = setTimeout(this.setTasks.bind(this), 1500);
    },

    addTask: async function (e) {
        e.preventDefault()
        let task = {};
        task["title"] = e.target.elements["title"].value;
        task['priority'] = '';

        try {
          let task_data = await fetch("http://test.site/api/", {method: "POST", body: JSON.stringify({"data": task})})
          task["id"] = (await task_data.json())["taskId"];
          task["updating"] = false;

          this.tasks.push(task);

          e.target.elements["title"].value = "";
        } catch (e) {
          console.log(e.message);
        }

      },

      deleteTask: async function(data) {
        let taskId = data.taskId;

        try {
          await fetch("http://test.site/api/", {method: "DELETE", body: JSON.stringify({"id": taskId})});

          let taskIndex = this.tasks.findIndex((el) => el.id == taskId);

          this.tasks.splice(taskIndex, 1);
        } catch (e) {
          console.log(e.message);
        }
      }
    }
}
</script>

<style scoped>

</style>