<template>
  <div :class="[classes, 'container']">
    <div
      class="close"
      @click.prevent="$store.commit('download/toggleShowing')"
    >
      <font-awesome-icon
        :icon="faTimes"
        size="1x"
      />
    </div>

    <template v-if="$fetchState.pending">
      <font-awesome-icon
        :icon="faSpinner"
        size="3x"
        spin
      />
    </template>

    <template v-else-if="$fetchState.error">
      <sys-header-2>
        An Error Occured
      </sys-header-2>

      <sys-paragraph-1>
        Please try again
      </sys-paragraph-1>
    </template>

    <template v-else>
      <sys-header-2>
        <template v-if="isLts">
          Download {{ version }} LTS
        </template>
        <template v-else>
          Download Latest {{ version }}
        </template>
      </sys-header-2>

      <div class="buttons">
        <sys-form-button
          rel="noopener"
          target="_blank"
          color="secondary"
          title="Download Pop!_OS"
          :ghost="!preferIntel"
          :href="intelUrl"
          @click="$ga.event('download', 'download', 'intel', intelUrl)"
        >
          Download
        </sys-form-button>

        <sys-form-button
          rel="noopener"
          color="secondary"
          target="_blank"
          title="Download Pop!_OS for systems with NVIDIA GPUs"
          :ghost="!preferNvidia"
          :href="nvidiaUrl"
          @click="$ga.event('download', 'download', 'nvidia', nvidiaUrl)"
        >
          Download (nVidia)
        </sys-form-button>
      </div>

      <sys-paragraph-1 class="disclaimer">
        If you have NVIDIA graphics, download the ISO with the proprietary
        NVIDIA driver preinstalled.
      </sys-paragraph-1>
      <sys-paragraph-1 class="disclaimer">
        Disable Secure Boot in your BIOS to install Pop!_OS.
      </sys-paragraph-1>
      <sys-paragraph-1 tag="dl">
        <dt>Requirements:</dt>
        <dd>2 GB RAM, 16 GB storage, 64-bit processor</dd>

        <dt>Recommended:</dt>
        <dd>4 GB RAM, 16 GB storage, 64-bit processor</dd>

        <dt>Filesize:</dt>
        <dd>{{ intelSize }} GB, {{ nvidiaSize }} GB (NVIDIA)</dd>

        <dt>SHA256 Sum:</dt>
        <dd class="sha">
          <code><span>{{ intelSha }}</span></code>
          <div>Pop!_OS ISO Image</div>

          <code><span>{{ nvidiaSha }}</span></code>
          <div>Pop!_OS ISO Image (NVIDIA)</div>
        </dd>
      </sys-paragraph-1>

      <p>
        <a
          class="help"
          target="_blank"
          href="https://support.system76.com/articles/install-pop/"
        >
          Learn how to create installation media.
        </a>
      </p>

      <div class="foot">
        <sys-form-button
          v-if="canSwitchRelease"
          color="primary"
          ghost
          rel="noopener"
          target="_blank"
          @click.prevent="toggle"
        >
          <template v-if="isLts">
            Download latest {{ alternativeVersion }}
          </template>
          <template v-else>
            Download {{ alternativeVersion }} LTS
          </template>
        </sys-form-button>
      </div>
    </template>
  </div>
</template>

<style scoped>
  .container {
    border-radius: 3px;
    border: 1px solid transparent;
    padding: 1rem;
    position: relative;
  }

  .light-mode {
    background-color: #fff;
    border-color: #C8C8C8;
    color: #272727;
  }

  .dark-mode {
    background-color: #272727;
    border-color: #171717;
    color: #ccc;
  }

  .close {
    align-content: center;
    align-items: center;
    border-radius: 50%;
    cursor: pointer;
    display: flex;
    height: 48px;
    justify-content: center;
    position: absolute;
    right: 6px;
    top: 6px;
    width: 48px;
  }

  .close:hover {
    background-color: rgba(0, 0, 0, 0.2);
  }

  h2 {
    text-align: center;
  }

  .buttons {
    align-content: center;
    align-items: center;
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    margin: 0 -0.5rem;
  }

  .buttons > * {
    flex: 1 1 40%;
    margin: 0.5rem;
    max-width: 40ch;
    min-width: 28ch;
  }

  .disclaimer {
    margin: 2rem auto 2rem;
    max-width: 60ch;
    text-align: center;
  }

  .foot {
    display: flex;
    flex-direction: column;
    margin: -0.5rem 0;
  }

  dt {
    font-weight: bold;
    margin: 0;
    padding: 0;
  }

  dd {
    margin: 0 0 2rem;
    padding: 0;
  }

  .sha {
    text-align: center;
  }

  .sha > * {
    display: block;
  }

  .sha code {
    background-color: var(--color-black);
    border-radius: 3px;
    color: var(--color-white);
    margin: 0.2rem 0;
    padding: 0.4rem;
    width: 100%;
  }

  .sha span {
    display: block;
    max-width: 100%;
    overflow: auto;
    width: 100%;
  }

  .sha div {
    margin: 0.2rem auto 1rem;
  }

  .foot {
    margin: calc(-0.5rem - 0.4em) -0.6em;
  }

  .foot > * {
    margin: 0.5rem auto 0.5rem 0;
  }

  a.help {
    color: inherit;
    display: block;
    padding: 1rem 0;
    text-decoration: underline;
  }
</style>

<script>
  import { faSpinner, faTimes } from '@fortawesome/free-solid-svg-icons'
  import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome'
  import { mapGetters, mapMutations, mapActions } from 'vuex'

  import color from '~/mixins/color'

  export default {
    components: {
      FontAwesomeIcon
    },

    mixins: [
      color
    ],

    computed: {
      ...mapGetters('download', [
        'loaded',
        'canSwitchRelease',
        'release',
        'isLts',
        'version',
        'alternativeVersion',
        'channel',

        'intelSha',
        'nvidiaSha',
        'intelSize',
        'nvidiaSize',
        'intelUrl',
        'nvidiaUrl',

        'hasPreference',
        'preferIntel',
        'preferNvidia'
      ]),

      faSpinner: () => faSpinner,
      faTimes: () => faTimes
    },

    async fetch () {
      if (!this.loaded) {
        await this.fetch(this.release)
      }
    },

    methods: {
      ...mapMutations('download', [
        'switchRelease'
      ]),

      ...mapActions('download', [
        'fetch'
      ]),

      toggle () {
        this.switchRelease()
        this.$fetch()
      }
    }
  }

</script>
