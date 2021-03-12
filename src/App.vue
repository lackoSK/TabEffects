<template>
  <div class="modal" v-show="settings.nameModal">
    <div class="modal-content no-select">
      <label for="name_modal" class="modal-label">How should we call you?</label>
      <input v-model="name" type="text" name="first_name" autocomplete="given-name" id="name_modal"
             class="modal-input no-outline"
             placeholder="Your name">
      <button class="name-confirm no-outline" @click="settings.nameModal = false">
        Close
       </button>
    </div>
  </div>
  <div class="modal" v-show="settings.imageModal">
    <div class="modal-content no-select">
      <label for="key-modal" class="modal-label">Your unsplash access key</label>
      <input v-model="accessKey" type="text" id="key-modal"
             class="modal-input no-outline"
             placeholder="Access key">
      <label for="image-modal" class="modal-label">Your preference for images?</label>
      <input v-model="imageQuery" type="text" name="first_name" autocomplete="given-name" id="image-modal"
             class="modal-input no-outline"
             placeholder="Whatever you like">
      <button class="name-confirm no-outline" @click="settings.imageModal = false">
        Close
       </button>
    </div>
  </div>
  <div>
    <div class="wrapper no-select" :style="{ backgroundImage: `url('${fetchImageUrl}')` }">
      <main>
        <span class="time">{{ clock.time }}</span>
        <span class="slogan">{{ slogan }}, {{ name }}.</span>
      </main>
      <aside>
        <button class="focus:outline-none transition transform hover:-rotate-12"
                @click="settings.shown = !settings.shown">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor">
            <path fill-rule="evenodd"
                  d="M11.49 3.17c-.38-1.56-2.6-1.56-2.98 0a1.532 1.532 0 01-2.286.948c-1.372-.836-2.942.734-2.106 2.106.54.886.061 2.042-.947 2.287-1.561.379-1.561 2.6 0 2.978a1.532 1.532 0 01.947 2.287c-.836 1.372.734 2.942 2.106 2.106a1.532 1.532 0 012.287.947c.379 1.561 2.6 1.561 2.978 0a1.533 1.533 0 012.287-.947c1.372.836 2.942-.734 2.106-2.106a1.533 1.533 0 01.947-2.287c1.561-.379 1.561-2.6 0-2.978a1.532 1.532 0 01-.947-2.287c.836-1.372-.734-2.942-2.106-2.106a1.532 1.532 0 01-2.287-.947zM10 13a3 3 0 100-6 3 3 0 000 6z"
                  clip-rule="evenodd"/>
          </svg>
        </button>
        <span class="credits" v-html="credits"></span>
      </aside>
        <transition name="fade">
           <div class="settings" v-show="settings.shown">
              <settingsButton header="Name" description="Click to change your name which is displayed."
                              @click="settings.nameModal = true"/>
              <settingsButton header="Image Settings" description="Click to enter image settings."
                              @click="settings.imageModal = true"/>
          </div>
        </transition>
    </div>
  </div>
</template>

<script>
import SettingsButton from "@/components/settings-button";
import axios from "axios";

const apiEndpoint = "https://api.unsplash.com/";
const randomPhotoEndpoint = `${apiEndpoint}photos/random`;
let apiKey = ""

export default {
  setting: 'App',
  components: {
    SettingsButton,
  },
  data() {
    return {
      name: "unset",
      imageQuery: "nature",
      accessKey: "",
      clock: {
        interval: null,
        time: null,
      },
      unsplash: {
        imageData: null,
      },
      settings: {
        shown: false,
        nameModal: false,
        imageApiKey: "nature",
        imageModal: false,
      },
      slogan: "",
    }
  },
  methods: {
    getRandomPhoto: async param => {
      console.log("here")
      try {
        const res = await axios.get(randomPhotoEndpoint, {
          params: {
            client_id: apiKey,
            count: 1,
            ...param
          }
        });

        if (res.status === 200) return res.data[0];
        else return null;
      } catch (exc) {
        console.error(exc);
        return null;
      }
    },
    searchPhoto() {
      const param = {
        query: this.imageQuery
      };

      this.getRandomPhoto(param).then(response => {
        this.unsplash.imageData = response;
      });
    },
    fetchTime() {
      this.clock.time = Intl.DateTimeFormat(navigator.language, {
        hour: 'numeric',
        minute: 'numeric',
      }).format()
    },
    updateSlogan() {
      Array.prototype.sample = function () {
        return this[Math.floor(Math.random() * this.length)];
      }

      const morning = ["Good morning", "Morning", "Howdy", "Enjoy the day"]
      const lunch = ["Enjoy", "Have a nice day", "Enjoy the day", "Be productive", "Hold tight"]
      const evening = ["Good evening", "Evening", "Relax", "Take a break", "Don't worry"]

      const hours = Number.parseInt(this.clock.time)

      if (hours < 12) {
        return morning.sample();
      } else if (hours < 17) {
        return lunch.sample();
      } else
        return evening.sample();
    },
  },
  computed: {
    fetchImageUrl() {
      const data = this.unsplash.imageData;
      if (data) return data.urls.raw + "?q=100&fm=jpg&dpr=2&crop=faces&fit=crop" + "&h=" + window.outerHeight + "&w=" + window.outerWidth;
      return null;
    },
    credits() {
      const data = this.unsplash.imageData;
      let html = null;

      if (data)
        html = "<a href='" + data.links.html + "'>" + "Photo" + "<a/>  by <a href='" + data.user.links.html + "' target='_blank'>" + data.user.name + "</a> on <a href='https://unsplash.com' target='_blank'>" + "Unsplash" + "</a>"

      return html;
    }
  },
  mounted() {
    if (localStorage.name) this.name = localStorage.name;
    if (localStorage.imageQuery) this.imageQuery = localStorage.imageQuery;
    if (localStorage.accessKey) this.accessKey = localStorage.accessKey;
    apiKey = this.accessKey

    this.searchPhoto();
    this.fetchTime();
    this.slogan = this.updateSlogan();

    // start runnable to fetch time every 30 seconds
    this.clock.interval = setInterval(() => {
      this.fetchTime();
    }, 30000)

  },
  beforeUnmount() {
    clearInterval(this.clock.interval); // prevent memory leak
  },
  watch: {
    name(newName) {
      localStorage.name = newName;
    },
    imageQuery(newQuery) {
      localStorage.imageQuery = newQuery;
    },
    accessKey(newKey) {
      localStorage.accessKey = newKey;
    }
  },
}
</script>

<style scoped>
.wrapper {
  @apply h-screen bg-gray-900 overflow-hidden bg-cover;
  @apply flex items-center justify-center flex-col;
}

main {
  @apply text-white text-7xl font-bold;
}

.time, .slogan {
  @apply block text-center;
}

.time {
  @apply text-9xl;
}

.slogan {
  @apply text-6xl font-medium;
}

aside {
  @apply absolute bottom-2 left-3 flex justify-center items-center;
}

aside button {
  @apply inline-block rounded-md;
}

aside svg {
  @apply h-6 w-6 text-gray-100;
}

aside .credits {
  @apply inline-block text-white text-sm text-gray-100 ml-2;
}

.settings {
  @apply absolute bottom-12 left-3;
  @apply bg-white text-gray-900 rounded-md p-4 space-y-2;
}

.modal {
  @apply h-screen w-screen absolute top-0 left-0 bg-gray-900 bg-opacity-50 flex justify-center items-center;
}

.modal-content {
  @apply w-64 rounded-md p-4 max-w-md bg-white;
}

.modal-label {
  @apply text-sm font-medium text-gray-900 pb-1;
}

.modal-input {
  @apply border-transparent focus:border-transparent shadow border-gray-500 w-full rounded-md block;
}

.name-confirm {
  @apply bg-gray-700 hover:bg-gray-800 shadow text-white py-2 px-3 rounded-md mt-4 transition;
}

.fade-enter-active, .fade-leave-active {
  @apply transition-all ease-in-out duration-200;
}

.fade-enter, .fade-leave-to {
  @apply transition-all ease-in-out duration-200;
}
</style>
