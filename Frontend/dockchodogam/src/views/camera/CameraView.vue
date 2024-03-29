<template>
  <LoadingPage v-if="this.isLoading" />
  <NavBar @overflow="overflow" />
  <div class="camera">
    <div class="camera__body">
      <camera ref="camera" @error="(error) => logEvent('error: ' + error)">
      </camera>
      <button type="button" @click="snapshot" class="photo__btn">
        <font-awesome-icon icon="fa-solid fa-camera" class="photo__icon" />
      </button>
    </div>
  </div>
</template>

<script lang="ts">
import { useRouter } from 'vue-router'
import { BASE_URL } from '@/constant/BASE_URL'
import NavBar from '@/components/main/NavBar.vue'
import axios from 'axios'
import { defineComponent, onMounted, Ref, ref } from 'vue'
import Camera from '@/components/camera/Camera.vue'
import swal from 'sweetalert'
import { useStore } from 'vuex'

const useRouterCustom = () => {
  const router = useRouter()

  const goToResult = () => {
    router.push('/camera/result')
  }
  return {
    goToResult
  }
}

const useRouterMain = () => {
  const router = useRouter()

  const goToMain = () => {
    router.push('/')
  }
  return {
    goToMain
  }
}

export default defineComponent({
  name: 'App',
  components: {
    Camera,
    NavBar
  },
  setup() {
    const store = useStore()

    const { goToResult } = useRouterCustom()

    const { goToMain } = useRouterMain()

    const camera = ref<InstanceType<typeof Camera>>()

    const cameras: Ref<MediaDeviceInfo[]> = ref([])

    onMounted(async () => {
      if (!camera.value) return
      try {
        cameras.value = await camera.value.devices(['videoinput'])
      } catch (e) {
        console.error(e)
      }
    })

    const start = () => camera.value?.start()
    const stop = () => camera.value?.stop()
    const pause = () => camera.value?.pause()
    const resume = () => camera.value?.resume()
    const snapshot = async () => {
      var audio = new Audio(process.env.VUE_APP_S3_URL + '/camera.mp3')
      audio.play()
      const blob = await camera.value?.snapshot({
        width: 500,
        height: 500
      })

      const formdata = new FormData()
      formdata.append('file', blob)

      const url = URL.createObjectURL(blob!)
      const result = ref()
      currentSnapshot.value = URL.createObjectURL(blob!)

      axios({
        url: BASE_URL + '/api/v1/dokcho/judge/',
        method: 'POST',
        headers: {
          AUTHORIZATION: 'Bearer ' + localStorage.getItem('accessToken'),
          'Content-Type': 'application/x-www-form-urlencoded'
        },
        data: formdata
      })
        .then((res) => {
          result.value = res.data
          store.dispatch('fetchphotoResult', res.data)
          goToResult()
        })
        .catch((err) => {
          swal({
            title: '사진을 다시 찍어주세요 😢',
            text: '찍어 주신 사진을 인식하지 못했어요 ...',
            icon: 'error',
            timer: 1500
          })
          console.log(err)
        })
    }

    const logEvent = (name: string) => console.log(name)

    const currentSnapshot = ref()

    const changeCamera = (event: Event) => {
      const target = event.target as HTMLSelectElement
      camera.value?.changeCamera(target.value)
    }

    return {
      camera,
      start,
      stop,
      pause,
      resume,
      logEvent,
      snapshot,
      currentSnapshot,
      cameras,
      changeCamera,
      goToMain
    }
  }
})
</script>

<style scoped>
.camera {
  height: 100vh;
  width: 100vw;
  position: absolute;
  top: 0;
  left: 0;
  z-index: -1;
  overflow: hidden;
}
.camera__body {
  width: 100%;
  height: 100%;
}
.photo__btn {
  position: absolute;
  bottom: 5vh;
  left: 50vw;
  display: block;
  z-index: 999;
  transform: translate(-50%, 0%);
  width: 7vw;
  height: 7vw;
  border-radius: 50%;
  border: 1vw double #467302;
  background: #eee;
  transition: 0.3s;
}
.photo__btn:hover {
  border: 1.2vw double #467302;
  background: white;
}
.photo__btn:hover > .photo__icon {
  width: 2.5vw;
  height: 2.5vw;
}
.photo__icon {
  width: 2vw;
  height: 2vw;
  color: #467302;
  transition: 0.3s;
}
@media screen and (max-width: 850px) {
  .photo__btn {
    width: 10vh;
    height: 10vh;
    bottom: 12vh;
    border: 1vh double #467302;
  }
  .photo__btn:hover {
    border: 1vh double #467302;
  }
  .photo__icon {
    width: 3vh;
    height: 3vh;
  }
  .photo__btn:hover > .photo__icon {
    width: 3vh;
    height: 3vh;
  }
}
</style>

<style>
#video {
  object-fit: cover;
}
</style>
