<template xmlns:v-clipboard="http://www.w3.org/1999/xhtml">
  <div class="card">
    <div class="card-image">
      <gmap-map
        ref="gmap"
        :center="mapCenter"
        :zoom="zoom"
        style="width: 100%; height: 520px"
      >
        <gmap-marker
          :position="markerPosition"
          :clickable="true"
          :draggable="true"
          @dragend="dragEnd"
        />
      </gmap-map>
    </div>
    <div class="card-content">
      <div class="columns">
        <div class="column">
          <div class="control">
            <label class="label">Latitude</label>
            <div class="field has-addons">
              <div class="control">
                <a class="button is-static">Y</a>
              </div>
              <div class="control is-expanded">
                <label>
                  <input
                    type="text"
                    class="input"
                    placeholder="Latitude"
                    :value="markerPosition.lat"
                  />
                </label>
              </div>
              <div class="control">
                <button
                  class="button is-light"
                  v-clipboard="() => markerPosition.lat"
                  v-clipboard:success="clipboardSuccessHandler"
                >
                  <span class="icon">
                    <i class="far fa-copy"></i>
                  </span>
                </button>
              </div>
            </div>
          </div>
        </div>
        <div class="column">
          <label class="label">Longitude</label>
          <div class="control">
            <div class="field has-addons">
              <div class="control">
                <a class="button is-static">X</a>
              </div>
              <div class="control is-expanded">
                <label>
                  <input
                    type="text"
                    class="input"
                    placeholder="Longitude"
                    :value="markerPosition.lng"
                  />
                </label>
              </div>
              <div class="control">
                <button
                  class="button is-light"
                  v-clipboard="() => markerPosition.lng"
                  v-clipboard:success="clipboardSuccessHandler"
                >
                  <span class="icon">
                    <i class="far fa-copy"></i>
                  </span>
                </button>
              </div>
            </div>
          </div>
        </div>
        <div class="column">
          <label for="hash" class="label">Geohash</label>
          <div class="control">
            <div class="field has-addons">
              <div class="control is-expanded">
                <input
                  id="hash"
                  type="text"
                  class="input is-fullwidth"
                  placeholder="Geohash"
                  :value="hash"
                />
              </div>
              <div class="control">
                <button
                  class="button is-light"
                  v-clipboard="() => hash"
                  v-clipboard:success="clipboardSuccessHandler"
                >
                  <span class="icon">
                    <i class="far fa-copy"></i>
                  </span>
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div class="field has-addons" v-if="address">
        <div class="control">
          <a class="button is-static">Address</a>
        </div>
        <div class="control is-expanded">
          <input type="text" class="input is-fullwidth" v-model="address" />
        </div>
        <div class="control">
          <button
            class="button is-light"
            v-clipboard="() => address"
            v-clipboard:success="clipboardSuccessHandler"
          >
            <span class="icon">
              <i class="far fa-copy"></i>
            </span>
          </button>
        </div>
      </div>
    </div>
    <footer class="card-footer">
      <a class="card-footer-item" @click="toggleSearchAddress">
        <span class="icon">
          <i class="fas fa-search"></i>
        </span>
        <span>SEARCH ADDRESS</span>
      </a>
      <a class="card-footer-item" @click="updatePosition">
        <span class="icon">
          <i class="far fa-compass"></i>
        </span>
        <span>USE GEOLOCATION</span>
      </a>
      <a class="card-footer-item" @click="dropMarker">
        <span class="icon">
          <i class="fas fa-map-marker-alt"></i>
        </span>
        <span>DROP MARKER</span>
      </a>
    </footer>
  </div>
</template>

<script>
var geohash = require("ngeohash");

export default {
  data() {
    return {
      map: null
    };
  },
  computed: {
    zoom() {
      return this.$store.getters.zoom;
    },
    address() {
      return this.$store.getters.address;
    },
    mapCenter() {
      return this.$store.getters.latLng;
    },
    markerPosition() {
      return this.$store.getters.latLng;
    },
    hash() {
      return geohash.encode(this.markerPosition.lat, this.markerPosition.lng);
    }
  },
  methods: {
    updatePosition() {
      navigator.geolocation.getCurrentPosition(
        position => {
          const latLng = {
            lat: position.coords.latitude,
            lng: position.coords.longitude
          };
          this.$store.dispatch("setLatLng", latLng);
          this.$store.dispatch("setZoom", 15);
        },
        error => {
          console.error(error);
          // TODO: handle geolocation errors
        }
      );
    },
    dropMarker() {
      if (!this.map) {
        return;
      }

      const center = this.map.getCenter();
      const latLng = {
        lat: center.lat(),
        lng: center.lng()
      };
      this.$store.dispatch("setLatLng", latLng);
    },
    dragEnd(evt) {
      const latLng = { lat: evt.latLng.lat(), lng: evt.latLng.lng() };
      this.$store.dispatch("setLatLng", latLng);
    },
    toggleSearchAddress() {
      this.$store.dispatch("toggleSearchAddress");
    },
    clipboardSuccessHandler() {
      this.$toasted.show("Copied to clipboard.");
    }
  },
  mounted() {
    var self = this;
    self.$refs.gmap.$mapPromise.then(() => {
      self.map = self.$refs.gmap.$mapObject;
    });
  }
};
</script>
