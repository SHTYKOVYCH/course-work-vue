<template>
  <article class="task" v-bind:class="'_' + task.priority">
    <header class="task__header">
      <span>{{ task.id }}</span>
      <button v-on:click="$emit('deleteTask', {taskId: task.id})">Done</button>
      <span v-if="!title_input" v-on:click="setInput">{{ task.title }}</span>
      <input v-else type="text" v-model="task.title" v-on:blur="setInput" v-on:input="lockTask" v-on:change="updateTask" name="TITLE">
    </header>
    <section class="task__body">
        <textarea
            class="task__detail"
            name="BODY"
            cols="30"
            rows="10"
            style="resize: none"
            v-bind:value="task.body"
            v-on:input="lockTask"
            v-on:blur="unlockTask"
            v-on:change="updateTask">
        </textarea>
      <select name="priority" v-model="task.priority" v-on:change="updateTask">
        <option value="">Нет</option>
        <option value="low">Низкий</option>
        <option value="medium">Средний</option>
        <option value="hight">Высокий</option>
      </select>
    </section>
  </article>
</template>

<script>
export default {
  name: "Task",
  props: {
    task: Object
  },
  data: function () {
    return {
      timeout: null,
      title_input: false,
    };
  },
  methods: {
    lockTask: function () {
      console.log("task locked");
      this.task.updating = true;
    },

    unlockTask: function () {
      console.log("task unlocked");
      this.task.updating = false;
    },

    updateTask: function (e) {
      console.log("changed");
      this.task.updating = true;

      if (this.timeout) {
        clearTimeout(this.timeout);
      }

      this.timeout = setTimeout(() => {
        let field_name = e.target.name;
        let field_value = e.target.value;
        let data = {};

        data[field_name] = field_value;

        fetch("http://test.site/api/", {method: "PUT", body: JSON.stringify({"id": this.task.id, "data": data})});

        this.unlockTask();
      }, 500);
    },

    setInput: function () {
      this.title_input = !this.title_input;

      this.$nextTick(() => {
        let input = this.$el.querySelector("input");

        if (input) {
          input.focus();
        }
      });
    },
  }
}
</script>

<style lang="sass" scoped>
.task
  display: flex
  flex-direction: column

  border-left: 4px solid transparent
  padding-left: .5em

  &._low
    border-color: blue

  &._medium
    border-color: yellow

  &._hight
    border-color: red

  &__header
    display: flex

    margin-bottom: .5em

  &__body
    display: flex
    flex-direction: row

    align-content: stretch

</style>