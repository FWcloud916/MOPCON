<template>
  <Dialog
    v-if="speaker"
    :visible="visible"
    @close="$emit('update:visible', false)"
    @click.native="btnListShow && (btnListShow = false)"
  >
    <div slot="body" class="dialog-body">
      <speaker-info :speakers="speaker" notitle />

      <section class="about">
        <h2>介紹</h2>
        <!-- eslint-disable-next-line vue/no-v-html -->
        <p v-html="parseSummary(speaker.bio)" />
      </section>

      <speaker-topic
        :speaker="speaker"
        class="mt-8"
        @handle-tag-click="toggleTag($event); $emit('update:visible', false);"
      />

      <section v-if="speaker.sponsor_id !== 0" class="sponser">
        <h2>贊助廠商</h2>
        <img :src="getSponsorInfo(speaker.sponsor_id)" class="logo logo-kkbox">
      </section>
    </div>
    <div slot="footer">
      <share-btn :speaker="speaker" :btn-list-show.sync="btnListShow" :share-url="shareUrl" />
    </div>
  </Dialog>
</template>

<script>
import Dialog from '@/components/Dialog'
import SpeakerInfo from './SpeakerInfo'
import SpeakerTopic from './SpeakerTopic'
import ShareBtn from './ShareBtn'

export default {
  name: 'SpeakerDialog',
  components: {
    Dialog,
    SpeakerInfo,
    SpeakerTopic,
    ShareBtn
  },
  props: {
    visible: {
      type: Boolean,
      default: false
    },
    speaker: {
      type: [Object, undefined],
      default: () => {}
    },
    toggleTag: {
      type: Function,
      default: () => {}
    }
  },
  data () {
    return {
      imgUrl: '',
      btnListShow: false
    }
  },
  computed: {
    shareUrl () {
      return process.client ? `${window.location.origin}/2020/schedule/${this.speaker.session_id}` : ''
    }
  },
  methods: {
    parseSummary (summary) {
      return summary.replace(/\n/gi, '<br>')
    },
    getSponsorInfo (id) {
      this.$axios.$get(`/api/2020/sponsor?sponsor_id=${id}`)
        .then(({ success, data, message }) => {
          if (success) {
            this.imgUrl = data[0].logo_path
          }
        })
      return this.imgUrl
    }
  }
}
</script>

<style lang="scss" scoped>
h2 {
  @apply text-lg pl-4 relative mt-3;
  &::before {
    content: '\\\\';
    left: -5px;
    @apply absolute text-yellow-500 text-lg font-medium;
  }
}
p {
  @apply mt-3 text-gray-400;
}
.logo-kkbox {
  width: 72px;
  height: 72px;
  @apply bg-purple-300 mt-4 inline-block;
}

::v-deep .dialog {
  width: 90%;
  @screen lg {
    width: 60%;
  }
  @screen xl {
    width: 50%;
  }
}
</style>
