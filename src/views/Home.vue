<template>
  <div class="home">
    <Demo msg="Welcome to Your Vue.js App"/>
  </div>
</template>

<script>
// @ is an alias to /src
import Demo from '@/components/Demo.vue'

export default {
  name: 'home',
  components: {
    Demo
  }
}
</script>
