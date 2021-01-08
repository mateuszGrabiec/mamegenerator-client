<template>
  <div>
    <div>
      <button id="save" class="btn btn-danger">Save as image</button>
      <button id="gohome" class="btn btn-success">
        <a class="not-active" href="/">Go to list</a>
      </button>
      <button id="save-server" class="btn btn-success" @click.prevent="saveOnServer">
        Add meme
      </button>
      <input id="load-image" type="file" class="btn btn-success" />
      <button id="add-txt" class="btn btn-success" @click.prevent="addTr">Add text box</button>
    </div>
    <div>
      <v-stage :config="configKonva" ref="stage">
        <v-layer ref="layer">
          <v-image
            v-if="image"
            :config="{
              image: image,
            }"
          />
        </v-layer>
        <v-layer>
          <v-rect v-for="item in transformers"
            :key="item.id"
            :config="item">
          </v-rect>
        </v-layer>
      </v-stage>
    </div>
  </div>
</template>

<script lang="ts">
import {Component, Prop, Vue} from "vue-property-decorator";

@Component
export default class Editor extends Vue {
  @Prop() private imgUrl!: string;
  private image: any = false;
  private transformers:Array<any>=[];
  private vueCanvas:any;

  private configKonva: any = {
    width: window.innerWidth,
    height: window.innerHeight,
  };
  mounted() {
    const image = new window.Image();
    image.src = this.imgUrl;
    image.crossOrigin = 'Anonymous';
    image.onload = () => {
      // set image only when it is loaded
      this.image = image;
    };
    this.vueCanvas=this.$refs.stage;
  }
  addTr(){
    console.log('Add TR')
  }
  saveOnServer(){
    let ctx  = this.vueCanvas;
    ctx = ctx.getStage();
    const image = ctx.toDataURL();
    console.log(image);
    
    fetch(process.env.VUE_APP_URL+'/save', {
      method: 'POST', // *GET, POST, PUT, DELETE, etc.
      mode: 'no-cors', // no-cors, *cors, same-origin
      // cache: 'no-cache', // *default, no-cache, reload, force-cache, only-if-cached
      headers: {
        'Accept': 'application/json',
        'Content-Type': 'application/json'
      },
      // redirect: 'follow', // manual, *follow, error
      // referrerPolicy: 'no-referrer', // no-referrer, *no-referrer-when-downgrade, origin, origin-when-cross-origin, same-origin, strict-origin, strict-origin-when-cross-origin, unsafe-url
      body: image
    }).then(res=>console.log(res));
    
    

  }
}
</script>
