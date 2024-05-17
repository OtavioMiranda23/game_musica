<script setup lang="ts">
import { RouterLink, RouterView } from 'vue-router'
import HelloWorld from './components/HelloWorld.vue'
import Axios from 'axios'
</script>

<template>
  <header>
    <img alt="Vue logo" class="logo" src="@/assets/logo.svg" width="125" height="125" />

    <div class="wrapper">
      <div id="embed-iframe"></div>

      <button ref="btnPlay">Play</button>
      <HelloWorld msg="You did it!" />

      <nav>
        <RouterLink to="/">Home</RouterLink>
        <RouterLink to="/about">About</RouterLink>
      </nav>
    </div>
  </header>

  <RouterView />
</template>

<script lang="ts">
export default {
  data() {
    return {
      deviceId: '',
      musicUri: '',
      isExpired: false,
      token: ''
    }
  },
  methods: {
    verifyTimeToken(expiresTime: number) {
      if (expiresTime < 2) {
        this.isExpired = true
      }
    },

    async authenticate() {
      try {
        const url = 'https://accounts.spotify.com/api/token'
        const clientId = import.meta.env.VITE_SPOTIFY_CLIENT_ID
        const clientSecret = import.meta.env.VITE_SPOTIFY_CLIENT_SECRET
        const data = new URLSearchParams()
        const config = {
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded'
          }
        }
        data.append('grant_type', 'client_credentials')
        data.append('client_id', clientId)
        data.append('client_secret', clientSecret)

        const res = await Axios.post(url, data, config)
        this.token = res.data.access_token
      } catch (err) {
        console.error('Houve um erro na autenticação', err)
      }
    },
    async getMusic(idMusic: string) {
      try {
        const config = {
          headers: {
            Authorization: `Bearer ${this.token}`
          }
        }
        console.log(config)
        const url = `https://api.spotify.com/v1/tracks/${idMusic}`
        const res = await Axios.get(url, config)
        this.musicUri = res.data.uri
        console.log(res.data)
      } catch (err) {
        console.error('Houve um erro no usuario', err)
      }
    }
  },
  created() {
    this.authenticate()
    // this.getUserDetails('6nfua5e3chr8c0o2s09if3wwy');
    const scriptPlayer = document.createElement('script')
    scriptPlayer.src = 'https://open.spotify.com/embed/iframe-api/v1' // or any other cdn
    scriptPlayer.async = true
    document.body.appendChild(scriptPlayer)
    setTimeout(() => {
      window.onSpotifyIframeApiReady = (IFrameAPI) => {
        const element = document.getElementById('embed-iframe')
        const options = {
          width: 400,
          height: 400,
          uri: this.musicUri
        }
        const callback = (EmbedController) => {
          this.$refs.btnPlay.addEventListener('click', () => {
            EmbedController.play()
          })
        }
        IFrameAPI.createController(element, options, callback)
      }
    }, '3000')

    // const script = document.createElement('script')
    // script.src = 'https://sdk.scdn.co/spotify-player.js'
    // script.async = true
    // document.body.appendChild(script)
    // script.onload = () => {
    //   window.onSpotifyWebPlaybackSDKReady = () => {
    //     const player = new window.Spotify.Player({
    //       name: 'Web Playback SDK',
    //       getOAuthToken: (cb: any) => {
    //         cb(
    //           'BQDASzRmVtVooA2RBxjeDCvGhz7Qy1Y8Lpo7GijyUca79N30MEXUa7aTVYkD3hPcclhdq6PJjxThUiXDImmb96G7IbHg54J2DsnjygzgFQWG8Gg1ZiPyPC1pMoHGwhaVFma0xLthF9ak8JXkJVdV8zoS1DtF3WGI9RIJN-0ocK3aN85eHoIHcgL0tLoonV6s209pJxSDAA'
    //         )
    //       },
    //       volume: 0.5
    //     })
    //     player.connect().then((success: any) => {
    //       if (success) {
    //         console.log('The Web Playback SDK successfully connected to Spotify!')
    //       }
    //     })
    //     player.addListener('initialization_error', ({ message }) => {
    //       console.error(message)
    //     })

    //     player.addListener('authentication_error', ({ message }) => {
    //       console.error(message)
    //     })

    //     player.addListener('account_error', ({ message }) => {
    //       console.error(message)
    //     })
    //     player.on('ready', (data) => {
    //       this.deviceId = data.device_id
    //     })
    //   }
    // }
  },
  mounted() {
    setTimeout(() => {
      this.getMusic('3ZRWcArefd1sfpdw5icCM7')
    }, '1000')
  }
}
</script>

<style scoped>
header {
  line-height: 1.5;
  max-height: 100vh;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

nav {
  width: 100%;
  font-size: 12px;
  text-align: center;
  margin-top: 2rem;
}

nav a.router-link-exact-active {
  color: var(--color-text);
}

nav a.router-link-exact-active:hover {
  background-color: transparent;
}

nav a {
  display: inline-block;
  padding: 0 1rem;
  border-left: 1px solid var(--color-border);
}

nav a:first-of-type {
  border: 0;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }

  nav {
    text-align: left;
    margin-left: -1rem;
    font-size: 1rem;

    padding: 1rem 0;
    margin-top: 1rem;
  }
}
</style>
