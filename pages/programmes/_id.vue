<template>
  <Transition name="fade">
    <section class="container mx-auto">
      <button
        @click="goBack()"
        class="outline-none border bg-gray-200 text-black px-3 py-2 mt-6 rounded-md text-sm"
      >
        <svg
          xmlns="http://www.w3.org/2000/svg"
          width="23"
          height="23"
          viewBox="0 0 24 24"
          fill="none"
          stroke="#747070"
          stroke-width="2"
          stroke-linecap="square"
          stroke-linejoin="bevel"
        >
          <path d="M19 12H6M12 5l-7 7 7 7" />
        </svg>
      </button>

      <div v-if="$nuxt.isOffline" class="text-gray-500 text-base relative">
        <img src="@/assets/noInternet.jpeg" />
        <div class="absolute -bottom-14 left-10">
          <p class="text-center">You are currently offline</p>
          <p class="text-center">Check your internet connect and try again!</p>
        </div>
      </div>
      <main
        v-if="$nuxt.isOnline"
        class="md:w-9/12 rounded-md shadow-md border space-y-5 my-6"
      >
        <div v-if="loading" class="border-green-500 p-6 my-6">
          <div class="animate-pulse flex space-y-6 flex-col">
            <div class="rounded-md bg-slate-300 h-5 w-full"></div>
            <div class="rounded-md bg-slate-300 h-5 w-full"></div>
            <div class="flex items-center space-x-3">
              <div class="rounded-md bg-slate-300 h-6 w-10"></div>
              <div class="rounded-md bg-slate-300 h-4 w-6/12"></div>
            </div>
            <div class="flex items-center space-x-3">
              <div class="rounded-md bg-slate-300 h-10 w-10"></div>
              <div class="rounded-md bg-slate-300 h-10 w-10"></div>
              <div class="rounded-md bg-slate-300 h-10 w-10"></div>
            </div>
          </div>
        </div>
        <div class="px-6 py-6" v-else>
          <div class="">
            <p
              v-if="programme.title !== 'null'"
              class="text-white rounded-md bg-green-400 p-2"
            >
              Title:
            </p>
            <h6 class="text-lg md:text-xl text-gray-900 lowercase pt-3">
              {{ programme.title }}
            </h6>
          </div>
          <div class="mt-3">
            <p
              v-if="programme.theme !== 'null'"
              class="text-white rounded-md bg-green-400 p-2"
            >
              Theme:
            </p>
            <p class="text-gray-900 text-sm md:text-base pt-3">
              {{ programme.theme === "null" ? "" : programme.theme }}
            </p>
          </div>
          <div
            :class="[programme.status === 'pending' ? 'hidden' : '']"
            class="flex items-center space-x-2 mt-3"
          >
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="20"
              height="20"
              viewBox="0 0 24 24"
              fill="none"
              stroke="#5bc747"
              stroke-width="1.5"
              stroke-linecap="butt"
              stroke-linejoin="bevel"
            >
              <rect x="3" y="4" width="18" height="18" rx="2" ry="2"></rect>
              <line x1="16" y1="2" x2="16" y2="6"></line>
              <line x1="8" y1="2" x2="8" y2="6"></line>
              <line x1="3" y1="10" x2="21" y2="10"></line>
            </svg>
            <p class="text-sm text-gray-900 text-justify">
              {{ timeFrame }}
            </p>
          </div>

          <div class="pt-4" v-if="programme?.zoomMeetingUrl !== 'undefined'">
            <a :href="programme.zoomMeetingUrl" class="text-green-500 text-sm"
              >Click to join programme</a
            >
          </div>

          <div class="pt-4" v-if="programme?.uploadedVideoUrl !== ''">
            <video-player :src="programme?.uploadedVideoUrl" />
          </div>

          <p class="font-medium text-base pt-3">
            {{
              !programme.uploadedDocumentFiles.length ? "" : "Paper download"
            }}
          </p>
          <section
            v-if="programme.uploadedDocumentFiles"
            class="grid grid-cols-6 md:grid-cols-10"
          >
            <a
              :href="eachFile"
              class=""
              v-for="(eachFile, index) in programme.uploadedDocumentFiles"
              :key="index"
              download
            >
              <img src="@/assets/pdfIcon.svg" class="h-16 w-16" alt="" />
            </a>
          </section>
        </div>
      </main>
    </section>
  </Transition>
</template>

<script>
import VideoPlayer from "nuxt-video-player";
require("nuxt-video-player/src/assets/css/main.css");
export default {
  scrollToTop: true,
  data() {
    return {
      loading: true,
      programme: {},
    };
  },
  created() {
    this.fetchProgrammes();
  },
  methods: {
    exportToPDF() {
      html2pdf(document.getElementById("element-to-convert"));
    },
    async fetchProgrammes() {
      this.loading = true;
      try {
        let res = await this.$axios.get(
          `https://panafstrag.onrender.com/api/panAfrica/programmes/${this.$route.params.id}`
        );
        this.programme = res.data;
        this.loading = false;
      } catch (err) {
        this.$toast
          .error("Something went wrong, please try again.")
          .goAway(1500);
      } finally {
        this.loading = false;
      }
    },
    goBack() {
      this.$router.go(-1);
    },
    // downloadFile(fileUrl) {
    //   axios({
    //     url: fileUrl,
    //     method: "GET",
    //     responseType: "blob",
    //   }).then((res) => {
    //     var FILE = window.URL.createObjectURL(new Blob([res.data]));

    //     var docUrl = document.createElement("x");
    //     docUrl.href = FILE;
    //     docUrl.setAttribute("download", "file.pdf");
    //     document.body.appendChild(docUrl);
    //     docUrl.click();
    //   });
    // },
  },
  computed: {
    timeFrame() {
      let startDay = this.$moment(this.programme.startDate)
        .format("YYYY-MM-Do")
        .split("-")[2];
      let endDay = this.$moment(this.programme.endDate)
        .format("YYYY-MM-Do")
        .split("-")[2];
      let month = this.$moment(this.programme.endDate)
        .format("YYYY-MMMM-Do")
        .split("-")[1];
      let year = this.$moment(this.programme.startDate)
        .format("YYYY-MM-DD")
        .split("-")[0];
      return `${startDay} to ${endDay}, ${month}, ${year}`;
    },
  },
};
</script>


<style scoped>
#app {
  margin-top: 60px;
}
.fade-enter-active {
  transition: all 0.3s cubic-bezier(0.52, 0.02, 0.19, 1.02) 0.15s;
}
.fade-leave-active {
  transition: all 0.3s cubic-bezier(0.52, 0.02, 0.19, 1.02);
}
.fade-enter-from {
  opacity: 0;
  transform: scale(0.8);
}
.fade-leave-to {
  transform: scale(0.8);
}
</style>
