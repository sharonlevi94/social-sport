<template>
  <div class="q-pa-md">
    {{titleName}}
    <div v-if="isMyPhotos()">

      <q-file filled bottom-slots v-model="myFile" label="Click to choose file" counter>

        <template v-slot:prepend>
          <q-icon name="cloud_upload" @click.stop/>
        </template>

        <template v-slot:append>
          <q-icon name="close" @click.stop="myFile = null" class="cursor-pointer"/>
        </template>

        <template v-slot:hint>
          {{ hint }}
        </template>
      </q-file>

      <q-btn @click="upload()">
        {{ buttonName }}
      </q-btn>

    </div>

    <q-carousel
        animated
        v-model="slide"
        arrows
        navigation
        infinite
        v-if="display"
    >
      <q-carousel-slide v-for="photo in slider"
                        :key="photo.id"
                        :name="photo.id"
                        :img-src="photo.downloadURL"/>
    </q-carousel>

  </div>
</template>

<script>
import { mapState, mapActions, mapMutations } from 'vuex'

export default {
  name: "UploadPhotos",
  props: ['titleName', 'buttonName', 'hint', 'display','id'],
  data() {
    return {
      myFile: null,
      slider: [],
      slide: 1
    }
  },
  computed:{
    ...mapState('images',[ 'editedImageId','editedImage','images','currProfilePictureURL','profilePictureId'])
  },
  methods: {
    ...mapActions('images', ['uploadImage','getImages','uploadProfilePicture','getProfilePicture']),

    ...mapMutations('images',['setEditedImage','resetEditedImageId','setNewProfilePicture']),

    async upload() {
      if(this.buttonName == 'Upload') {
        this.setEditedImage(this.myFile)
        await this.uploadImage()
        this.resetEditedImageId()
        await this.read()
        await this.getProfilePicture()
        window.user.downloadURL = this.currProfilePictureURL;
      }
      else {
        this.setNewProfilePicture(this.myFile)
        await this.uploadProfilePicture();
      }
    },
    async read() {
      this.slider = [];
      await this.getImages(this.$route.params.id);
      let counter = 1;
      //make the images to objects with Ids:
      for (let url of this.images) {
        let imgObj = {}
        imgObj.downloadURL = url;
        imgObj.id = counter;
        counter++;
        this.slider.push(imgObj);
      }
    },
    isMyPhotos(){
      return this.$route.params.id == window.user.uid
    }
  },
  created() {
    this.read()
    this.getProfilePicture()
  }
}
</script>

<style scoped>
.q-pa-md {
  font-size: 40px;
  font-family: "Berlin Sans FB";
  width: 100%;
}
</style>