<template>
  <div>
    <div>
      <button id="save" class="btn btn-danger">Save as image</button>
      <button id="gohome" class="btn btn-success">
        <a class="not-active" href="/">Go to list</a>
      </button>
      <button
        id="save-server"
        class="btn btn-success"
        @click.prevent="saveOnServer"
      >
        Add meme
      </button>
      <input id="load-image" type="file" class="btn btn-success" />
      <button id="add-txt" class="btn btn-success" @click.prevent="addTr">
        Add text box
      </button>
    </div>
    <div
      :style="{width: width + 2 + 'px', height: height + 2 + 'px'}"
      class="mx-auto my-2 border border-dark"
    >
    <div id="container"></div>
      <v-stage :config="configKonva" ref="stage" @mousedown="handleStageMouseDown"
    @touchstart="handleStageMouseDown">
        <v-layer ref="layer">
          <v-image
            v-if="image"
            :config="{
              image: image,
            }"
          />
        </v-layer>
        <v-layer ref="textLayer">
          <v-text v-for="item in textCollection" :key="item._id"
            :config="item"
            @transformend="handleTransformEnd"
            @dblclick="editText(item)"
          > 
          </v-text>
          <v-transformer ref="transformer"/>
        </v-layer>
        <v-layer>
          <textarea/>
        </v-layer>
      </v-stage>
    </div>
  </div>
</template>

<script lang="ts">
import {Component, Prop, Vue} from "vue-property-decorator";
import Konva from "konva";

@Component
export default class Editor extends Vue {
  @Prop() private imgUrl!: string;
  @Prop() private height!: number;
  @Prop() private width!: number;
  @Prop() private boxes!: number;
  private image: any = false;
  private textCollection: Array<any> = [];
  private vueCanvas: any;
  private selectedShapeId = -1;
  private editableText= "";

  private configKonva: any = {
    container: 'container',
    width: this.width,
    height: this.height,
  };

  mounted() {
    const image = new window.Image();
    image.src = this.imgUrl;
    image.crossOrigin = "Anonymous";
    image.onload = () => {
      // set image only when it is loaded
      this.image = image;
    };
    this.vueCanvas = this.$refs.stage;
  }

  addTr() {
    const textBox = new Konva.Text({
      text: "Some text here",
      x: 50,
      y: 50,
      fontSize: 20,
      scaleX: 1,
      scaleY: 1,
      name: "text"+(this.textCollection.length + 1),
      draggable:true,
      rotation: 0
    });
    console.log(textBox);
    this.textCollection.push(textBox);
  }

  editText(item: any){
    //TODO SHOW EDIT BOX
    console.log(item);
    const textPosition = item.getAbsolutePosition();

        (this.$refs.textLayer as any).getNode().draw()

        // then lets find position of stage container on the page:
        const stageBox = (this.$refs.stage as any).getStage().container().getBoundingClientRect();

        console.log(stageBox);
        

        // so position of textarea will be the sum of positions above:
        const areaPosition = {
          x: stageBox.left + textPosition.x,
          y: stageBox.top + textPosition.y,
        };

        // create textarea and style it
        const textarea = document.createElement('textarea');
        document.body.appendChild(textarea);

        textarea.value = item.text();
        textarea.style.position = 'absolute';
        textarea.style.top = areaPosition.y + 'px';
        textarea.style.left = areaPosition.x + 'px';
        textarea.style.width = item.width()+'px';

        textarea.focus();

        //TODO change IMPL TO update specific fielc

        const holder = ()=> this.addTr();
        
        console.log('\n\n\nLayer');        

        textarea.addEventListener('keydown', function (e) {          
          // hide on enter
          if (e.keyCode === 13) {
            item.text(textarea.value);
            holder();
            document.body.removeChild(textarea);
          }
        });
    
  }

  handleTransformEnd(e: any) {
    // shape is transformed, let us save new attrs back to the node
    // find element in our state
    
    const tr = this.textCollection.find((r) => r.attrs.name === this.selectedShapeId);
    
    // update the state
    tr.x = e.target.x();
    tr.y = e.target.y();
    tr.rotation = e.target.rotation();
    tr.scaleX = e.target.scaleX();
    tr.scaleY = e.target.scaleY();

    // change fill
    // tr.fill = Konva.Util.getRandomColor();
  }

  handleStageMouseDown(e: any) {
    // clicked on stage - clear selection
    if (e.target === e.target.getStage()) {
      this.selectedShapeId = -1;
      this.updateTransformer();
      return;
    }

    // clicked on transformer - do nothing
    const clickedOnTransformer = e.target.getParent().className === "Transformer";
    if (clickedOnTransformer) {
      return;
    }

    // find clicked text by its id
    
    const targetId = e.target.name();
    const rect = this.textCollection.find((r) => r.attrs.name === targetId);
       

    if (rect) {
      this.selectedShapeId = targetId;     
    } else {
      this.selectedShapeId = -1;
    }
    this.updateTransformer();
  }

  updateTransformer() {
    // here we need to manually attach or detach Transformer node
    const transformerNode = (this.$refs.transformer as any).getNode();
    
    const stage = transformerNode.getStage();
    
    const {selectedShapeId} = this;  

    const selectedNode = stage.findOne("."+selectedShapeId);
    // do nothing if selected node is already attached
    if (selectedNode === transformerNode.node()) {
      return;
    }

    if (selectedNode) {
      // attach to another node
      transformerNode.nodes([selectedNode]);
    } else {
      // remove transformer
      transformerNode.nodes([]);
    }
    transformerNode.getLayer().batchDraw();
  }

  async saveOnServer() {
    let ctx = this.vueCanvas;
    ctx = ctx.getStage();
    const image = ctx.toDataURL();
    console.log(image);
    //TODO check is en mus starts from VUE_APP
    fetch(process.env.VUE_APP_URL + "/save", {
      method: "POST", // *GET, POST, PUT, DELETE, etc.
      mode: "cors", // no-cors, *cors, same-origin
      // cache: 'no-cache', // *default, no-cache, reload, force-cache, only-if-cached
      headers: {
        Accept: "application/json",
        "Content-Type": "application/json",
      },
      // redirect: 'follow', // manual, *follow, error
      // referrerPolicy: 'no-referrer', // no-referrer, *no-referrer-when-downgrade, origin, origin-when-cross-origin, same-origin, strict-origin, strict-origin-when-cross-origin, unsafe-url
      body: JSON.stringify({meme: image}),
    }).then((res) => console.log(res));
  }
}
</script>

<style lang="sass" scoped>

</style>
