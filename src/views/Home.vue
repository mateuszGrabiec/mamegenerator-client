<template>
  <div class="home">
    <!-- <HelloWorld msg="Welcome to Your Vue.js + TypeScript App" /> -->
    <h1 class="page-header" style="margin-top: 30px">Choose meme</h1>
    <a role="button" class="btn btn-danger" href="/custom">
        Hell no i wanna upload my template
    </a>
    <div v-if="memeList && !memeTemplate">
      <div style="text-align: center">
        <br />
        <div class="row p-3 ml-5 mr-5">
          <div class="col-lg-1 col-12 col-sm-2" v-for="meme in memeList"  v-bind:key="meme.id" @click.prevent="choose(meme)">
              <img style="height:50%; width:100%" :src="meme.url">
              <p>{{meme.name}}</p>
          </div>
        </div>
        <br />
      </div>
    </div>
    <div v-else-if="!memeList && memeTemplate">
      <h2>No memes :(</h2>
    </div>
    <div v-else>
      <editor :imgUrl="memeTemplate.url" :height="memeTemplate.height" :width="memeTemplate.width" :boxes="memeTemplate.box_count"></editor>
    </div>
  </div>
</template>

<script lang="ts">
import {Component, Vue} from "vue-property-decorator";
import HelloWorld from "@/components/HelloWorld.vue"; // @ is an alias to /src
import Editor from "@/components/Editor.vue"

@Component({
  components: {
    HelloWorld,
    Editor
  }

})


export default class Home extends Vue {
  private memeList: Array<object> = [];
  private memeTemplate:any = false;

  mounted() {    
    fetch('https://api.imgflip.com/get_memes')
    .then(res=>res.json())
    .then(res=>{      
      this.memeList=res.data.memes;
    });
    
  }
  choose(meme:any){
    
    let endpoint = process.env.VUE_APP_BACKEND || 'http://memegenerator-server-deploy-labproj13.apps.cp4apps.cloudpak.site';
    console.log(process.env.VUE_APP_BACKEND);
    

    fetch(endpoint+'/template', {
      method: 'POST', // *GET, POST, PUT, DELETE, etc.
      mode: 'cors', // no-cors, *cors, same-origin
      // cache: 'no-cache', // *default, no-cache, reload, force-cache, only-if-cached
      headers: {
        'Accept': 'application/json',
        'Content-Type': 'application/json'
      },
      // redirect: 'follow', // manual, *follow, error
      // referrerPolicy: 'no-referrer', // no-referrer, *no-referrer-when-downgrade, origin, origin-when-cross-origin, same-origin, strict-origin, strict-origin-when-cross-origin, unsafe-url
      body: JSON.stringify(meme) // body data type must match "Content-Type" header
    })
    .then(res => res.json())
    .then(res=>{
      this.memeTemplate=res;
    });
    
  }
  }
</script>
