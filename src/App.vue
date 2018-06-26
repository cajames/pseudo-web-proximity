<script>
import { Vue, Component, Prop } from "vue-property-decorator";
import AppHeader from "@/components/Header.vue";

@Component({
  components: {
    AppHeader
  }
})
export default class App extends Vue {}
</script>

<template>
  <div id="app" class="font-sans leading-normal tracking-normal bg-blue-dark">
    <app-header></app-header>
    <router-view/>
  </div>
</template>
