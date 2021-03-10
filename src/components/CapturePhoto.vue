<template>
  <div>
    <div class="photo-frame"
        :style="`width: ${ width }; height: ${ height };`"
        @mouseover="hideButton = false"
        @mouseout="hideButton = true"
      >
      <q-img
            :src="imageSrc"
            style="width: 100%; height: 100%; object-fit: contain;"
            @dblclick.native="photoDialogView = imageSrc ? true : false"
          >
          <q-btn
            v-show="!hideButton && !readOnly && imageSrc"
            color="primary"
            icon="delete"
            class="absolute"
            style="top: 10; left: 5px;width:2rem;"
            @mouseover="hideButton = false"
            @mouseout="hideButton = true"
            @click="buttonDelete"
          />
          <q-btn
            v-show="!hideButton && !readOnly"
            @click="photoDialog = true"
            color="primary"
            icon="edit"
            class="absolute"
            style="top: 10; right: 5px;width:2rem;"
            @mouseover="hideButton = false"
            @mouseout="hideButton = true"
          />
        </q-img>
    </div>

    <q-dialog v-model="photoDialogView" >
      <q-card style="width: 100%; max-width: 500px" >
        <q-img :src="imageSrc" style="width: 100%; height: 100%; object-fit: contain;" />
        <q-card-actions>
          <q-btn class="full-width" color="primary" @click="photoDialogView = false" >Aceptar</q-btn>
        </q-card-actions>
      </q-card>
    </q-dialog>

    <q-dialog v-model="photoDialog" >
      <div class="text-center q-pa-md bg-white" >
        <div class="camera-frame q-pa-md">
          <video
            v-show="!imageCaptured"
            ref="video"
            class="full-width"
            autoplay
          />
          <!-- style="background-color: white; max-height: 240px;" -->
          <canvas
            v-show="imageCaptured"
            ref="canvas"
            class="full-width"
            height="240"
          />
        </div>
        <div class="text-center q-pa-md">
          <q-btn
            v-if="hasCameraSupport && isChnagePhoto"
            @click="captureImage"
            :disable="imageCaptured"
            color="grey-10"
            icon="eva-camera"
            size="lg"
            round
          />
          <q-file
            v-else
            v-model="imageUpload"
            @input="captureImageFallback"
            label="Seleccione una imagen"
            accept="image/*"
            outlined
          >
            <template v-slot:prepend>
              <q-icon name="eva-attach-outline" />
            </template>
          </q-file>

          <div class="row justify-center q-mt-lg">
            <q-btn
              @click="buttonChange"
              v-show="canChange"
              color="primary"
              :label="isChnagePhoto ? 'Cambiar a Búsqueda' : 'Cambiar Tirar Foto'"
              rounded
              unelevated
            />
          </div>

          <div class="row justify-center q-mt-lg">
            <q-btn
              @click="photoDialog = false"
              color="red"
              label="Cancelar"
              rounded
              unelevated
            />
            <q-space/>
            <q-btn
              @click="buttonOk"
              :disable="!photo"
              color="primary"
              label="Aceptar"
              rounded
              unelevated
            />
          </div>
        </div>
      </div>
    </q-dialog>
  </div>
</template>

<script>
import { uid } from 'quasar'
require('md-gum-polyfill')
// import imagenPlaceholder from 'src/assets/Agua.jpg'

export default {
  name: 'CapturePhoto',
  props: {
    width: {
      type: String,
      default: '200px'
    },
    height: {
      type: String,
      default: '160px'
    },
    readOnly: {
      type: Boolean,
      default: false
    }
  },
  data () {
    return {
      imageSrc: '',
      hideButton: true,
      photoDialog: false,
      photoDialogView: false,
      photo: null,
      namePhoto: '',
      extPhoto: '',
      imageCaptured: false,
      imageUpload: [],
      hasCameraSupport: true,
      isLoad: true,
      canChange: true,
      isChnagePhoto: true
    }
  },
  watch: {
    photoDialog (val) {
      if (val) {
        this.imageCaptured = false
        this.photo = null
        this.imageUpload = []
        this.hasCameraSupport = true
        this.isLoad = true
        this.canChange = true
        this.isChnagePhoto = true
        this.initCamera()
      } else {
        this.disableCamera()
      }
    }
  },
  methods: {
    initCamera () {
      this.$q.loading.show()
      navigator.mediaDevices.getUserMedia({
        video: true
      }).then(stream => {
        this.$refs.video.srcObject = stream
      }).catch(error => {
        console.error(error)
        this.hasCameraSupport = false
      })
      this.canChange = this.hasCameraSupport
      this.$q.loading.hide()
    },
    captureImage () {
      this.$q.loading.show()
      const video = this.$refs.video
      const canvas = this.$refs.canvas
      canvas.width = video.getBoundingClientRect().width
      canvas.height = video.getBoundingClientRect().height
      const context = canvas.getContext('2d')
      context.drawImage(video, 0, 0, canvas.width, canvas.height)
      this.imageCaptured = true
      this.photo = this.dataURItoBlob(canvas.toDataURL())
      this.namePhoto = uid() + '.png'
      this.extPhoto = '.png'
      this.isLoad = false
      this.disableCamera()
      this.$q.loading.hide()
    },
    captureImageFallback (file) {
      this.$q.loading.show()
      this.photo = file
      this.isLoad = true
      this.namePhoto = this.photo.name
      const array = this.photo.name.split('.')
      this.extPhoto = '.' + array[array.length - 1]

      const canvas = this.$refs.canvas
      const context = canvas.getContext('2d')

      var reader = new FileReader()
      reader.onload = event => {
        var img = new Image()
        img.onload = () => {
          canvas.width = img.width
          canvas.height = img.height
          context.drawImage(img, 0, 0)
          this.imageCaptured = true
        }
        img.src = event.target.result
      }
      reader.readAsDataURL(file)
      this.$q.loading.hide()
    },
    disableCamera () {
      try {
        this.$refs.video.srcObject.getVideoTracks().forEach(track => {
          track.stop()
        })
      } catch (error) {
        console.log(error)
      }
    },
    dataURItoBlob (dataURI) {
      // convert base64 to raw binary data held in a string
      // doesn't handle URLEncoded DataURIs - see SO answer #6850276 for code that does this
      var byteString = atob(dataURI.split(',')[1])
      // separate out the mime component
      var mimeString = dataURI.split(',')[0].split(':')[1].split(';')[0]
      // write the bytes of the string to an ArrayBuffer
      var ab = new ArrayBuffer(byteString.length)
      // create a view into the buffer
      var ia = new Uint8Array(ab)
      // set the bytes of the buffer to the correct values
      for (var i = 0; i < byteString.length; i++) {
        ia[i] = byteString.charCodeAt(i)
      }
      // write the ArrayBuffer to a blob, and you're done
      var blob = new Blob([ab], { type: mimeString })
      return blob
    },
    buttonChange () {
      this.$q.loading.show()
      console.log('buttonChange')
      console.log('IsLoad', this.isLoad)
      if (this.isLoad) {
        if (this.hasCameraSupport) {
          console.log('tiene soporte')
          this.disableCamera()
        }
        // console.log('cambiar')
        this.imageCaptured = true
        this.isChnagePhoto = false
      } else {
        if (this.hasCameraSupport) {
          this.initCamera()
        }
        this.imageCaptured = false
        this.isChnagePhoto = true
      }
      this.$q.loading.hide()
    },
    buttonOk () {
      this.imageSrc = URL.createObjectURL(this.photo)
      this.photoDialog = false
      // Return image to parent
      this.$emit('getImage', { photo: this.imageSrc, namePhoto: this.namePhoto, extPhoto: this.extPhoto })
    },
    buttonDelete () {
      this.imageSrc = null
      this.$emit('getImage', null)
    }
    // addPost () {
    //   this.$q.loading.show()

    //   const formData = new FormData()
    //   // formData.append('id', this.post.id)
    //   // formData.append('caption', this.post.caption)
    //   // formData.append('location', this.post.location)
    //   // formData.append('date', this.post.date)
    //   const id = 'kdjflkjdj'
    //   formData.append('file', this.photo, id + '.png')

    //   this.$axios.post(`${process.env.API}/createPost`, formData).then(response => {
    //     console.log('response: ', response)
    //     this.$router.push('/')
    //     this.$q.notify({
    //       message: 'Post created!',
    //       actions: [
    //         { label: 'Dismiss', color: 'white' }
    //       ]
    //     })
    //     this.$q.loading.hide()
    //   }).catch(err => {
    //     console.log('err: ', err)
    //     this.$q.dialog({
    //       title: 'Error',
    //       message: 'Sorry, could not create post!'
    //     })
    //     this.$q.loading.hide()
    //   })
    // }
  },
  // mounted () {
  //   this.initCamera()
  // },
  beforeDestroy () {
    if (this.hasCameraSupport) {
      this.disableCamera()
    }
  }
}
</script>

<style lang="sass">
  .camera-frame
    border: 2px solid $grey-10
    border-radius: 10px
  .photo-frame
    border: 1px solid $grey-10
    border-radius: 10px
  .cover
    background-size: cover !important
</style>
