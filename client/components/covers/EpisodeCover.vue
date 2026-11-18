<template>
  <div class="relative rounded-xs overflow-hidden bg-primary" :style="wrapperStyle">
    <img :src="coverUrl" loading="lazy" draggable="false" class="w-full h-full object-cover" :style="{ opacity: imageReady ? '1' : '0' }" @load="imageReady = true" @error="onError" />
  </div>
</template>

<script>
/**
 * Episode artwork for podcasts.
 *
 * The server answers /api/podcasts/:id/episode/:episodeId/cover with the
 * episode's own image when it has one, and 302-redirects to the show cover
 * when it does not — so this URL is always usable and needs no null handling.
 * The item-cover fallback below only covers a failed load.
 */
export default {
  props: {
    libraryItemId: String,
    episode: {
      type: Object,
      default: () => null
    },
    width: {
      type: Number,
      default: 48
    },
    height: {
      type: Number,
      default: null
    }
  },
  data() {
    return {
      imageReady: false,
      episodeCoverFailed: false
    }
  },
  watch: {
    'episode.id'() {
      this.imageReady = false
      this.episodeCoverFailed = false
    },
    libraryItemId() {
      this.imageReady = false
      this.episodeCoverFailed = false
    }
  },
  computed: {
    store() {
      // Rows are mounted manually with Vue.extend outside the app context, so
      // $store is not injected — mirror LazyEpisodeRow's fallback.
      return this.$store || this.$nuxt.$store
    },
    wrapperStyle() {
      const height = this.height || this.width
      return {
        width: `${this.width}px`,
        height: `${height}px`,
        minWidth: `${this.width}px`
      }
    },
    coverUrl() {
      if (this.episodeCoverFailed || !this.episode?.id) return this.itemCoverUrl
      return this.store.getters['globals/getPodcastEpisodeCoverSrc'](this.libraryItemId, this.episode.id, this.episode.updatedAt)
    },
    itemCoverUrl() {
      return this.store.getters['globals/getLibraryItemCoverSrcById'](this.libraryItemId)
    }
  },
  methods: {
    onError() {
      if (this.episode?.id && !this.episodeCoverFailed) this.episodeCoverFailed = true
    }
  }
}
</script>
