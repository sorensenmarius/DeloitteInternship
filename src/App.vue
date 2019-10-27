<template>
  <v-content>
    <div>
      <v-text-field
        dark 
        solo
        outlined
        v-model="query"
        label="Press enter to search" 
        class="searchbar"
        @input="gifs = []"
        @keyup.enter.native="getGifs(true)" />
    </div>
    <div>
    <div 
      class="centered-text"
      v-if="gifs.length == 0">
      <h2 id="errorMsg">
        {{ errorMsg }}
      </h2>
    </div>
    <Waterfall
      v-if="gifs.length > 0"
      :resizable="true"
      class="pb-2">
        <WaterfallItem
          :width="250"
          :key="gif.id"
          v-for="gif in gifs"
          class="waterfallItem meet">
          <v-img 
            :src="`https://media.giphy.com/media/${gif.id}/200w_d.gif`" 
            @click='gotoLink(gif)'/>
        </WaterfallItem>
    </Waterfall>
    </div>
    <div 
      class="centered-text"
      v-if="end">
      <h2 id="errorMsg">
        {{ "Showing all " + this.gifs.length + " gifs for the keyword " + query}}
      </h2>
    </div>
  </v-content>
</template>

<script> 
import axios from 'axios'
import {Waterfall, WaterfallItem} from 'vue2-waterfall';

export default {
  name: 'GiphySearch',
  components: {
    Waterfall,
    WaterfallItem
  },
  data() {
    return {
      query: '',
      gifs: [],
      limit: 50,
      loading: false,
      end: false
    }
  },
  computed: {
    offset: function() {
      return this.gifs.length;
    },
    errorMsg: {
      get: function() {
        if(this.loading) {
          return ''
        } else if(!this.query) {
          return "Search for a cool gif above!"
        }
        return ''
      },
      set: function(input) {
        this.errorMsg = input
      }
    }
  },
  methods: {
    getGifs(reset) {
      this.loading = true;
      let offset = reset ? 0 : this.gifs.length
      axios.get(`https://api.giphy.com/v1/gifs/search?api_key=w38EcMTY1UOtAYqQF6JSZMaQjWiSWpzH&q=${this.query}&limit=${this.limit}&offset=${offset}&lang=en`)
        .then(res => {
          this.loading = false
          if(res.data.data.length == 0) {
            if(this.gifs.length == 0) {
              this.errorMsg = "We found no gifs for that keyword :("
            } else {
              this.end = true
            }
          } else {
            this.end = false
            if(reset) {
              this.gifs = res.data.data
            } else {
              this.gifs.push(...res.data.data)
            }
          }
        })
        .catch(err => {
          this.loading = false
          this.errorMsg = err.data
        })
    },
    gotoLink(gif) {
      window.open(gif.url, "_blank")
    },
    scroll () {
      window.onscroll = () => {
        let bottomOfWindow = document.documentElement.scrollTop + window.innerHeight + 1 > document.documentElement.offsetHeight;

        if (bottomOfWindow && !this.loading) {
          this.getGifs(false)
        }
      };
    },
  },
  mounted() {
    this.scroll()
  }
}
</script>

<style>
* {
  font-family:'Courier New', Courier, monospace;
}
body {
  background-color: rgb(40, 40, 40);
}

.waterfallItem {
  border: 2px solid #1C6EA4;
  margin: 3px;
}

.waterfallItem:hover {
  cursor: pointer;
}

.centered-text {
  text-align: center;
}

#errorMsg {
  color: rgb(253, 130, 130)
}

.searchbar {
  margin: auto !important; 
  width: 30%;
}

@media only screen and (max-width: 1000px) {
  .searchbar {
    width: 70%;
  }
}
</style>
