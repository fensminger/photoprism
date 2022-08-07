<template>
  <div>
    <span style="display: none">src {{src}}</span>
  </div>
</template>
<script>
import 'photo-sphere-viewer/dist/photo-sphere-viewer.css';
import {Viewer} from 'photo-sphere-viewer';

export default {
  name: 'PPhotoSphere',
  props: {
    src: {
      type: String,
      required: true
    }
  },
  data(){
    return {
      viewer: undefined
    };
  },
  mounted() {
    if (this.viewer) {
      this.viewer.needsUpdate();
    }
  },
  updated(){
    if (!this.viewer) {
      this.viewer = new Viewer({
        container:document.querySelector('#viewer'),
        panorama:this.src,
        size:{
          width: '100vw',
          height: '100vh',
        },
      });
    } else {
      this.viewer.setPanorama(this.src);
    }
  },
  unmounted() {
    if (this.viewer) {
      this.viewer.destroy();
      this.viewer = undefined;
    }
  }
};
</script>