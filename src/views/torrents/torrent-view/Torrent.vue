<template>
  <div class="min-h-fit flex flex-col grow">
    <div class="max-w-full flex flex-col items-center">

      <!-- MOBILE -->
      <div id="torrent-view" class="mb-8 max-w-full flex lg:hidden flex-col items-center">
        <TorrentActionCard class="mb-8 w-full" :torrent="torrent" @updated="reload"/>
        <div id="torrent-view-details" class="w-full flex flex-col flex-auto items-center">
          <div id="torrent-view-details-body" class="w-full flex flex-col grow">
            <div class="flex flex-col items-center">
              <div id="details-switcher" class="mb-4 max-w-md h-12">
                <ul class="flex flex-row border-b-2 border-slate-600 dark:border-dark-600">
                  <li v-for="(option) in tabs">
                    <button
                        class="inline-flex py-2 px-6 font-medium text-center text-slate-400 dark:text-dark-200 hover:text-slate-200 border-b-2 border-transparent duration-200"
                        :class="{ 'active': option.tab === tab }"
                        style="margin-bottom: -2px;"
                        @click="tab = option.tab"
                    >
                      {{ option.name }}
                    </button>
                  </li>
                </ul>
              </div>
              <div class="mt-4 w-full flex flex-col flex-auto">
                <TorrentOverviewTab v-if="tab === Tabs.overview" :torrent="torrent" @updated="reload"/>
                <TorrentFilesTab v-else-if="tab === Tabs.files" :torrent="torrent" @updated="reload"/>
                <TorrentTrackersTab v-else-if="tab === Tabs.trackers" :torrent="torrent" @updated="reload"/>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- DESKTOP -->
      <div id="torrent-view" class="hidden mb-8 max-w-8xl lg:flex flex-row flex-nowrap items-start grow w-full">
        <div id="torrent-view-details" class="mr-8 flex flex-col flex-auto items-center grow w-full">
          <div id="torrent-view-details-body" class="w-full max-w-3xl flex flex-col grow">
            <div class="flex flex-col items-center">
              <div id="details-switcher" class="mb-4 max-w-md h-12">
                <ul class="flex flex-row border-b-2 border-slate-600 dark:border-white/5">
                  <li v-for="(option) in tabs">
                    <button
                        class="inline-flex py-2 px-6 font-medium text-center text-slate-400 dark:text-dark-200 hover:text-slate-200 dark:hover:text-white border-b-2 border-transparent dark:hover:border-white/10 duration-200"
                        :class="{ 'active': option.tab === tab }"
                        style="margin-bottom: -2px;"
                        @click="tab = option.tab"
                    >
                      {{ option.name }}
                    </button>
                  </li>
                </ul>
              </div>
              <div class="mt-4 w-full flex flex-col flex-auto">
                <TorrentOverviewTab v-if="tab === Tabs.overview" :torrent="torrent" @updated="reload"/>
                <TorrentFilesTab v-else-if="tab === Tabs.files" :torrent="torrent" @updated="reload"/>
                <TorrentTrackersTab v-else-if="tab === Tabs.trackers" :torrent="torrent" @updated="reload"/>
              </div>
            </div>
          </div>
        </div>
        <TorrentActionCard class="top-24 sticky max-w-md" :torrent="torrent" @updated="reload"/>
      </div>

    </div>
  </div>
</template>

<script>
import HttpService from "../../../common/http-service";
import TorrentActionCard from "./components/TorrentActionCard.vue";
import TorrentOverviewTab from "./components/TorrentOverviewTab.vue";
import TorrentFilesTab from "./components/TorrentFilesTab.vue";
import TorrentTrackersTab from "./components/TorrentTrackersTab.vue";

export default {
  name: "Torrent",
  components: {
    TorrentTrackersTab,
    TorrentFilesTab,
    TorrentOverviewTab,
    TorrentActionCard
  },
  data: () => ({
    Tabs: {
      overview: 0,
      files: 1,
      trackers: 2
    },
    tabs: [
      { name: 'Overview', tab: 0 },
      { name: 'Files', tab: 1 },
      { name: 'Trackers', tab: 2 },
    ],
    tab: 0,
    torrent: {
      description: ''
    }
  }),
  methods: {
    getTorrent(torrentId) {
      this.loading = true;
      HttpService.get(`/torrent/${torrentId}`, (res) => {
        this.torrent = res.data.data;
        this.loading = false;
        this.setTorrentTitleInUrl();
      }).catch(() => {
        this.loading = false;
      })
    },
    reload() {
      this.getTorrent(this.torrent.torrent_id);
    },
    setTorrentTitleInUrl() {
      if (this.$route.params.title !== this.torrent.title) {
        // Retrieve current params
        const currentParams = this.$router.currentRoute.params;
        // Create new params object
        const mergedParams = { ...currentParams, title: this.torrent.title };
        // When router is not supplied path or name,
        // it simply tries to update current route with new params or query
        // Almost everything is optional.
        this.$router.push({ params: mergedParams });
      }
    },
  },
  created() {
    this.getTorrent(this.$route.params.torrentId);
  }
}
</script>

<style scoped>
.active {
  @apply text-slate-200 dark:text-white border-slate-200 dark:border-dark-accent-600;
}
</style>

<style>
.markdown-body {
  @apply text-slate-400 dark:text-dark-400;
}

.markdown-body a {
  @apply text-sky-500;
}

.markdown-body blockquote {
  @apply text-slate-400 dark:text-dark-400 border-slate-600 dark:border-dark-600;
}

.markdown-body hr {
  @apply bg-slate-200/50 dark:bg-white/5;
}

.markdown-body h1, .markdown-body h2, .markdown-body h3, .markdown-body h4, .markdown-body h5, .markdown-body h6 {
  @apply border-slate-200/50 dark:border-white/5;
}

.markdown-body .highlight pre, .markdown-body pre {
  @apply bg-slate-800 dark:bg-dark-800 text-slate-400 dark:text-dark-400 rounded-md;
}

.markdown-body table tr, .markdown-body table td, .markdown-body table th {
  @apply bg-slate-800 dark:bg-dark-800 border-slate-700 dark:border-white/5;
}
</style>
