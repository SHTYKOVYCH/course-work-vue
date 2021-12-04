<template>
  <div class="app">
    <TaskList></TaskList>
  </div>
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue'
import TaskList from "@/components/TaskList";

export default {
  name: 'App',
  components: {
    TaskList
  },
  data: function () {
    return {}
  }
}
</script>

<style lang="sass">
body
  background-color: #77767b

.app
  display: flex
  width: 100%

  max-width: 1920px

  justify-content: center

</style>
