<template>
  <article
      class="task"
      v-bind:class="taskClasses">
    <header class="task__header">
      <a
          href="#"
          class="task__delete"
          tabindex="1"
          v-on:click="$emit('deleteTask', {taskId: task.id})"></a>
      <a
          href="#"
          class="task__title-text"
          tabindex="1"
          v-if="!title_input"
          v-on:click="setInput">
        {{ normalizedString }}
      </a>
      <input
          class="task__title-input"
          type="text"
          name="title"
          tabindex="-1"
          v-else
          v-model.trim="task.title"
          v-on:blur="setInput"
          v-on:input="lockTask"
          v-on:change="updateTask">
      <a
          href="#"
          class="btn task__open"
          tabindex="1"
          v-on:click="openTask"></a>
    </header>
    <section class="task__additional">
      <section class="task__additional-section _left">
        <span class="task__additional-title">Notes</span>
        <textarea
            class="task__body"
            name="body"
            rows="10"
            style="resize: none"
            tabindex="-1"
            v-bind:value="task.body"
            v-on:input="lockTask"
            v-on:blur="unlockTask"
            v-on:change="updateTask">
        </textarea>
      </section>
      <section class="task__additional-section">
        <span class="task__additional-title">Priority</span>
        <select
            class="task__priority"
            name="priority"
            tabindex="-1"
            v-model="task.priority"
            v-on:change="updateTask($event), $emit('updateTask')">
          <option value="">None</option>
          <option value="low">Low</option>
          <option value="medium">Medium</option>
          <option value="hight">Hight</option>
        </select>
      </section>
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
      opened: false,
    };
  },
  methods: {
    setRowsForTextArea: function rows(res = 0) {
      if (this.title - res * 25 > 0) {
        return rows(res + 1);
      }

      return res;
    },

    lockTask: function () {
      this.task.updating = true;
    },

    openTask: function (e) {
      this.opened = !this.opened;

      e.target.classList.toggle("_opened");
    },

    unlockTask: function () {
      this.task.updating = false;
    },

    updateTask: function (e) {
      this.lockTask();

      if (this.timeout) {
        clearTimeout(this.timeout);
      }

      this.timeout = setTimeout(() => {
        let field_name = e.target.name;
        let field_value = e.target.value;
        let data = {};

        data[field_name] = field_value;

        try {
          fetch("http://v9260926.beget.tech/api/", {
            method: "PUT",
            body: JSON.stringify({"id": this.task.id, "data": data})
          }).then(res => {
            if (res.status !== 200) throw new Error(res.json().message)
          });
        } catch (e) {
          console.log(e.message);
        }

        this.unlockTask();
      }, 500);
    },

    setInput: function () {
      this.title_input = !this.title_input;

      this.$nextTick(() => {
        let input = this.$el.querySelector('[name="title"]');

        if (input) {
          input.focus();
        }
      });
    },
  },
  computed: {
    taskClasses: function () {
      return {
        _opened: this.opened,
        _low: this.task.priority === 'low',
        _medium: this.task.priority === 'medium',
        _hight: this.task.priority === 'hight'
      };
    },

    normalizedString: function() {
      return this.task.title.length > 75 ? this.task.title.substr(0, 72) + '...' : this.task.title
    }
  }
}
</script>

<style lang="sass" scoped>
.task
  display: flex
  flex-direction: column

  background-color: #272727

  border-radius: .2em

  border-left: .5em solid transparent

  padding: .5em .5em

  margin-bottom: .5em

  color: #e2e2e2

  max-height: 2.1em
  overflow: hidden

  transition: max-height ease .75s

  &._low
    border-left-color: #3465a4

  &._medium
    border-left-color: #f57900

  &._hight
    border-left-color: #cc0000

  &._opened
    max-height: 15em

  &__title-input
    padding: 0
    border: 0
    outline: 0
    appearance: none
    box-shadow: none

    background-color: #272727
    color: #e2e2e2

  &__title-text,
  &__title-input
    display: block
    width: 95%
    flex: 1 1 95%

    margin-left: .2em

    font-size: 1em
    font-family: Arial
    font-weight: bold
    vertical-align: bottom

    color: #e2e2e2
    text-decoration: none

    border-radius: .2em
    padding: .5em .5em

    &:hover
      outline: .1em solid black

    &:focus
      outline: .1em solid orange

  &__header
    display: flex

    padding-bottom: .5em
    margin-bottom: .5em

    align-items: center

    border-bottom: .1em solid rgba(0, 0, 0, .15)

  &__additional
    display: flex
    flex-direction: row

    align-items: start
    justify-content: space-between


  &__additional-title
    display: block

    font-size: .9em
    font-family: Arial
    color: #e2e2e2

    margin-bottom: .2em

  &__additional-section
    display: flex
    flex-direction: column
    width: 40%
    flex: 1 1 40%

    justify-content: stretch

    &._left
      flex: 3 2 100%

      margin-right: 1em

    &._right
      flex: 2 3 100%

  &__body
    box-sizing: border-box

    width: 100%
    flex: 1 1 100%

    padding: .5em

    background-color: #272727
    border: none
    outline: .1em solid black

    color: #e2e2e2

    &:focus
      outline-color: orange

  &__priority
    box-sizing: border-box

    appearance: none

    outline: 0
    border: 0

    width: 100%
    flex: 1 1 100%

    background-color: #373737
    color: #e2e2e2
    padding: .5em

    font-size: 1em

    &:hover
      outline: .1em solid black

    &:active, &:focus
      outline: .1em solid orange


  &__delete
    display: block

    flex: 0 0 2em

    width: 2em
    height: 2em

    background-color: #393939

    position: relative

    &:hover
      outline: .1em solid black

    &:active, &:focus
      outline: .1em solid orange

    &::before,
    &::after
      content: ' '
      position: absolute
      background-color: #e2e2e2
      border: none

      height: .25em

      top: 50%
      left: 47%

      border-radius: .2em

    &::before
      width: .6em

      transform: rotate(-135deg) translate(40%, -200%)

    &::after
      width: 1em

      transform: rotate(-45deg) translate(-15%, -80%)

  &__open
    width: 2em
    height: 2em

    flex: 0 0 2em

    position: relative

    transition: background-color ease .2s

    &::before,
    &::after
      content: ' '

      width: 1em
      height: .2em

      position: absolute
      top: 50%
      left: 50%

      background-color: #e2e2e2
      border-radius: .3em

      transition: all ease .2s

    &::before
      transform: translate(-79%, 0) rotate(45deg)


    &::after
      transform: translate(-16%, 0) rotate(-45deg)

    &:hover
      background-color: #393939

    &:active, &:focus
      outline: .1em solid orange

    &._opened

      &::before
        transform: translate(-79%, 0) rotate(-45deg)

      &::after
        transform: translate(-16%, 0) rotate(45deg)

</style>