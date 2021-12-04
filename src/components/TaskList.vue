<template>
  <section class="task-list">
    <section class="task-list__list">
      <Task
          v-for="task in tasks"
          :key="task.taskId"
          v-bind:task="task"
          v-on:deleteTask="deleteTask"
          v-on:updateTask="sortTasks"
      ></Task>
    </section>
    <section class="task-list__form-wrapper">
      <form class="task-list__form" v-on:submit="this.addTask">
        <input
            tabindex="2"
            class="task-list__input" name="title" type="text">
        <a
            href="#"
            class="task-list__submit"
            tabindex="3"
            v-on:click="addTask">
        </a>
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
  mounted: async function () {
    await this.syncTasks();
  },
  methods: {
    syncTasks: async function () {
      try {
        fetch("http://v9260926.beget.tech/api/")
            .then((async function (res) {
              if (res.status !== 200) {
                throw new Error("Failed to fetch data from source");
              }

              let tasks = (await res.json()).tasks;

              if (Object.values(tasks).length === 0) {
                return;
              }

              let indexes = Object.values(tasks).map(el => el[0]);

              let changes = 0;

              this.tasks.forEach((el, index) => {
                if (indexes.findIndex(index => index === el.id) === -1) {
                  this.tasks.splice(index, 1);
                  changes += 1;
                }
              });

              for (let task in tasks) {
                let taskId = this.tasks.findIndex(el => el.id === tasks[task].id);

                let t_task = {};
                t_task["id"] = tasks[task].id;
                t_task["title"] = tasks[task].title;
                t_task["body"] = tasks[task].body ?? '';
                t_task["priority"] = tasks[task].priority;

                if (taskId === -1) {
                  this.tasks.push(t_task);
                  changes += 1;
                } else if (!this.tasks[taskId].updating && !this.compareObjects(this.tasks[taskId], t_task)) {
                  this.tasks[taskId] = Object.assign(this.tasks[taskId], t_task);
                  changes += 1;
                  console.log("updated")
                }
              }

              if (changes !== 0) {
                this.sortTasks();
              }
            }).bind(this));
      } catch (e) {
        console.log(e.message);
      }

      this.updating = setTimeout(this.syncTasks.bind(this), 3000);
    },

    addTask: async function (e) {
      e.preventDefault();

      let form = document.forms[0];

      if (!form.elements["title"].value) {
        console.log("Пустой input");
        return;
      }

      let task = {};

      task["title"] = form.elements["title"].value;
      task['priority'] = '';

      try {
        let task_data = await fetch("http://v9260926.beget.tech/api/", {method: "POST", body: JSON.stringify({"data": task})})
        task["id"] = (await task_data.json())["taskId"];
        task["updating"] = false;

        this.tasks.push(task);

        form.elements["title"].value = "";
      } catch (e) {
        console.log(e.message);
      }

    },

    deleteTask: async function (data) {
      let taskId = data.taskId;

      try {
        await fetch("http://v9260926.beget.tech/api/", {method: "DELETE", body: JSON.stringify({"id": taskId})});

        let taskIndex = this.tasks.findIndex((el) => el.id === taskId);

        this.tasks.splice(taskIndex, 1);
      } catch (e) {
        console.log(e.message);
      }
    },

    sortTasks: function() {
      console.log("sorting");
      this.tasks = this.tasks.sort((a, b) => {
        let aPriority = a.priority === 'low' ? 1 : a.priority === "medium" ? 2 : a.priority === "hight" ? 3 : 0;
        let bPriority = b.priority === 'low' ? 1 : b.priority === "medium" ? 2 : b.priority === "hight" ? 3 : 0;

        return -(aPriority - bPriority);
      });
    },

    compareObjects: function compareObjects(a, b) {
      for (let property in a) {
        if (a[property] !== b[property]) {
          return false;
        }
      }
      return true;
    }
  },
}
</script>

<style scoped lang="sass">
.task-list
  width: 40%
  flex: 0 0 40%

  @media (max-width: 960px)
    width: 95%
    flex: 0 0 95%

  &__form-wrapper
    display: flex
    width: 100%

  &__form
    display: flex
    width: 100%
    justify-content: stretch
    position: relative

  &__input
    appearance: none
    outline: 0
    border: 0

    flex: 1 1 100%
    height: 2.1em

    padding: .5em 3.75em

    font-size: 1em

    background-color: #272727
    color: #e2e2e2

    &:focus
      outline: .1em solid orange
      border: 0

  &__submit
    display: block
    width: 2em
    height: 2em

    position: absolute
    left: 1em
    top: 50%

    transform: translate(0, -50%)

    &::before,
    &::after
      content: ' '

      display: block

      width: 1.5em
      height: .3em

      background-color: #e2e2e2

      border-radius: .5em

      position: absolute

      left: 50%
      top: 50%

      transform: translate(-50%, -50%)

    &::after
      transform: translate(-50%, -50%) rotate(90deg)

    &:focus,
    &:hover
      background-color: #303030

</style>