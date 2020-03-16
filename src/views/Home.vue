<template>
  <div class="home">
    <img alt="Vue logo" src="../assets/logo.png" />
    <HelloWorld msg="Welcome to Your Vue.js App" />
    <q-input color="purple-12" v-model="text" label="Label">
      <template v-slot:prepend>
        <q-icon name="event" />
      </template>
    </q-input>
  </div>
</template>

<script>
// @ is an alias to /src

export default {
  name: "Home",
  components: {}
};
</script>
