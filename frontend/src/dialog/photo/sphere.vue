<template>
  <v-dialog :value="show" fullscreen hide-overlay scrollable
            lazy persistent class="p-photo-sphere-dialog" @keydown.esc="close">
    <v-card color="application">
      <v-toolbar dark flat color="navigation" :dense="$vuetify.breakpoint.smAndDown">
        <v-btn icon dark class="action-close" @click.stop="close">
          <v-icon>close</v-icon>
        </v-btn>
        <v-toolbar-title>{{ title }}
          <v-icon v-if="isPrivate" title="Private">lock</v-icon>
        </v-toolbar-title>
        <v-spacer></v-spacer>
        <v-toolbar-items v-if="selection.length > 1">
          <v-btn icon :disabled="selected < 1" class="action-previous" @click.stop="prev">
            <v-icon v-if="!rtl">navigate_before</v-icon>
            <v-icon v-else>navigate_next</v-icon>
          </v-btn>

          <v-btn icon :disabled="selected >= selection.length - 1" class="action-next" @click.stop="next">
            <v-icon v-if="!rtl">navigate_next</v-icon>
            <v-icon v-else>navigate_before</v-icon>
          </v-btn>
        </v-toolbar-items>
      </v-toolbar>
      <div id="viewer" style="width: 100vw; height: 100vh;">
<!--        <v-img :src="imgUrl"></v-img>-->
        <p-photo-sphere :src="imgUrl"></p-photo-sphere>
      </div>
    </v-card>
  </v-dialog>
</template>
<script>
import Photo from "model/photo";
import Event from "pubsub-js";
import PPhotoSphere from "../share/photo-sphere.vue";

export default {
  name: 'PPhotoSphereDialog',
  components: {
    'p-photo-sphere': PPhotoSphere
  },
  props: {
    index: {
      type: Number,
      required: true
    },
    show: Boolean,
    selection: {
      type: Array,
      default: () => [],
    },
    album: {
      type: Object,
      default: () => {},
    },
  },
  data() {
    return {
      selected: 0,
      selectedId: "",
      model: new Photo,
      uid: "",
      loading: false,
      search: null,
      items: [],
      readonly: this.$config.get("readonly"),
      active: this.tab,
      rtl: this.$rtl,
      subscriptions: [],
    };
  },
  computed: {
    imgUrl: function() {
      return this.model.thumbnailUrl("fit_7680");
    },
    title () {
      if (this.model && this.model.Title) {
        return this.model.Title;
      }

      return this.$gettext("Photo Sphere");
    },
    isPrivate () {
      if (this.model && this.model.Private && this.$config.settings().features.private) {
        return this.model.Private;
      }

      return false;
    },
  },
  watch: {
    show: function (show) {
      if (show) {
        this.find(this.index);
      }
    }
  },
  created() {
    this.subscriptions.push(Event.subscribe("photos.updated", (ev, data) => this.onUpdate(ev, data)));
  },
  destroyed() {
    for (let i = 0; i < this.subscriptions.length; i++) {
      Event.unsubscribe(this.subscriptions[i]);
    }
  },
  methods: {
    onUpdate(ev, data) {
      if (!data || !data.entities || !Array.isArray(data.entities) || this.loading || !this.model || !this.model.UID) {
        return;
      }

      const type = ev.split('.')[1];

      switch (type) {
        case 'updated':
          for (let i = 0; i < data.entities.length; i++) {
            const values = data.entities[i];
            if (values.UID && values.Title && this.model.UID === values.UID) {
              this.model.setValues({Title: values.Title, Description: values.Description}, true);
            }
          }
          break;
      }
    },
    close() {
      this.$emit('close');
    },
    prev() {
      if (this.selected > 0) {
        this.find(this.selected - 1);
      }
    },
    next() {
      if (!this.selection) return;

      if (this.selected < this.selection.length) {
        this.find(this.selected + 1);
      }
    },
    find(index) {
      if (this.loading) {
        return;
      }

      if (!this.selection || !this.selection[index]) {
        this.$notify.error(this.$gettext("Invalid photo selected"));
        return;
      }

      this.loading = true;
      this.selected = index;
      this.selectedId = this.selection[index];

      this.model.find(this.selectedId).then(model => {
        model.refreshFileAttr();
        this.model = model;
        this.loading = false;
        this.uid = this.selectedId;
      }).catch(() => this.loading = false);
    },
  },
};
</script>
