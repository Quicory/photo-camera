<template>
  <q-page class="flex flex-center">
      <capture-photo
        :width="'200px'"
        :height="'160px'"
        :readOnly="false"
        @getImage="setImage"
      />
  </q-page>
</template>

<script>
import capturePhoto from 'src/components/CapturePhoto.vue'
export default {
  name: 'PageIndex',
  components: { capturePhoto },
  data () {
    return {
      image: null,
      nameImage: '',
      extImage: ''
    }
  },
  methods: {
    setImage (image) {
      if (image) {
        this.image = image.photo
        this.nameImage = image.namePhoto
        this.extImage = image.extPhoto
      } else {
        this.image = null
        this.nameImage = ''
        this.extImage = ''
      }
      console.log('obj', image)
    }
  }
}
</script>

<style lang="sass">
  .photo-frame
    border: 2px solid $grey-10
    border-radius: 10px
</style>
