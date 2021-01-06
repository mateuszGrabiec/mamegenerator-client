<template>
  <div>
    <div>
      <button id="save" class="btn btn-danger">Save as image</button>
      <button id="gohome" class="btn btn-success">
        <a class="not-active" href="/">Go to list</a>
      </button>
      <input id="load-image" type="file" class="btn btn-success" />
      <button id="add-txt" class="btn btn-success" @click.prevent="addTr">Add text box</button>
    </div>
    <div>
      <v-stage :config="configKonva" ref="stage">
        <v-layer>
          <v-image
            v-if="image"
            :config="{
              image: image,
            }"
          />
        </v-layer>
        <v-layer ref="layer">
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
  private configKonva: any = {
    width: window.innerWidth,
    height: window.innerHeight,
  };
  mounted() {
    const image = new window.Image();
    image.src = this.imgUrl;
    image.onload = () => {
      // set image only when it is loaded
      this.image = image;
    };
  }
  addTr(){
    console.log('Add TR')
  }
}
</script>
