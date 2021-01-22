<template>
  <div class="container">
    <div v-if="loading" class="d-flex flex-column text-center">
      Memes are loading ;)
      <div class="spinner-border text-success mx-auto mt-3" style="width: 5rem; height: 5rem;" role="status">
        <span class="sr-only">Loading...</span>
      </div>
    </div>
    <div v-else v-for="meme in memeList" :key="meme._id">
      {{meme.name}}
      <img :src="meme.img">
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";

@Component
export default class Memes extends Vue {
  private memeList: Array<object> = [];
  private loading = true;
  async beforeCreate() {
    try{
    const response = await (await fetch(process.env.VUE_APP_URL+'/memes')).json();
    console.log(response);
    
    this.memeList = response;
    this.loading=false;
    }catch(err){
      console.log(err);
    }
    
  }
}
</script>