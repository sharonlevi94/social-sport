<template>
  <div class="text-center items-center">
    <div v-for="post of posts">

      <!--Post Content-->
      <q-card
          bordered
          class="my-card text-black text-left glossy"
      >

        <q-card-section>
          <div class="sectionTitle" v-if="isPost()"> {{ post.author }} says:</div>
          <div>{{ post.content }}</div>
        </q-card-section>

        <q-card-section>
          <div>
            <q-btn push class="post-buttons"
                   icon="img:https://firebasestorage.googleapis.com/v0/b/fitnesspot-10d17.appspot.com/o/assets%2Fdelete.png?alt=media&token=272b9639-e56b-4f61-b259-6e2906537429"
                   @click="deleteObj(post.id)"
                   v-if="isBelong(post.id)"/>
            <q-btn push class="post-buttons"
                   icon="img:https://firebasestorage.googleapis.com/v0/b/fitnesspot-10d17.appspot.com/o/assets%2Flike.png?alt=media&token=bbf24394-9b02-4cf5-a064-69f69703b6c8"
                   @click="like(post.id)"
            />
            <q-btn push class="post-buttons"
                   icon="img:https://firebasestorage.googleapis.com/v0/b/fitnesspot-10d17.appspot.com/o/assets%2Fcomment.png?alt=media&token=5cad8e5f-d33a-429e-8aae-bed65f666e2a"
                   @click="showPostComments(post.id)"
            />
            {{ post.likes }} likes
          </div>
        </q-card-section>

      </q-card>

      <!--Write A Comment-->
      <div class="comment-parent">
        <q-input outlined bottom-slots v-model="post.currComment" label="left your comment here" :dense="dense">
          <template v-slot:before>
            <q-avatar>
              <img :src="this.currProfilePictureURL">
            </q-avatar>
          </template>

          <template v-slot:append>
            <q-icon v-if="post.currComment !== ''" name="close" @click="post.currComment = ''" class="cursor-pointer"/>
          </template>

          <template v-slot:after>
            <q-btn round dense flat icon="send" @click="leftComment(post.id, post.currComment)"/>
          </template>
        </q-input>
      </div>

      <!--View Comments-->
      <q-dialog v-model="commentDialog"
                transition-show="rotate"
                transition-hide="rotate">
        <Comments/>
      </q-dialog>

    </div>
  </div>
</template>

<script>
import firebaseDataBase from '../middleware/firebase/database';
import {mapMutations, mapActions, mapState} from 'vuex';
import Comments from "./Comments";

export default {
  name: "CardViewer",
  components: {
    Comments
  },
  props: ['cardObj', 'cardName', 'settingsName', 'id'],
  data() {
    return {
      settings: [],
      counter: 0,
      userName: '',
      dense: false,
      commentDialog: false,
      authorImage: '',
      lock: false,
    }
  },
  computed: {
    ...mapState('posts', ['editedObj', 'editedPostId', 'posts', 'currComment']),
    ...mapState('images', ['currProfilePictureURL'])
  },
  methods: {
    ...mapActions('posts', ['getPosts', 'deletePost', 'updatePost',
      'setEditPostById', 'getPost']),

    ...mapActions('images', ['getProfilePicture', 'getProfilePictureById']),

    ...mapMutations('posts', ['setEditedPostId', 'setEditedPost',
      'resetEditedPostId', 'setComments']),

    async readSettings() {
      this.settings = [];
      let cols = await firebaseDataBase.readSettings({entity: this.settingsName});
      for (let i in cols) {
        this.settings.push(cols[i]['label']);
      }
    },

    deleteObj(id) {
      this.setEditedPostId(id);
      this.deletePost();
      this.resetEditedPostId();
    },

    isPost() {
      if (this.cardName === 'posts') {
        this.userName = window.user.displayName;
        return true;
      }
      return false;
    },

    async like(id) {
      await this.setEditedPostId(id);
      await this.setEditPostById();
      let localObj = {}
      Object.assign(localObj, this.editedObj)
      localObj.likes++;
      await this.setEditedPost(localObj);
      await this.updatePost();
    },

    async leftComment(id, currComment) {
      await this.setEditedPostId(id);
      await this.setEditPostById();
      let localObj = {}

      Object.assign(localObj, this.editedObj)
      let comment = {
        author: window.user.displayName,
        image: this.currProfilePictureURL,
        content: currComment
      }

      localObj.comments.push(comment)
      //reset current comment
      await this.setEditedPost(localObj);
      await this.updatePost();
    },

    async isBelong(id) {
      let post = await this.getPost(id)
      return post.authorId == window.user.uid
    },

    getAuthorImage(id) {
      return this.getProfilePictureById(id).then((image) => {
        return image;
      })
    },

    showPostComments(id) {
      this.setEditedPostId(id)
      console.log(id)
      // this.getPost(id)
      //console.log(this.editedObj)
      // this.setComments(this.editedObj.comments)
      this.commentDialog = true
    }
  },
  created() {
    this.getProfilePicture()
  }
  //---------------------------------------------------------------------------------------

}
</script>

<style scoped>
.my-card {
  width: 100%;
  margin: 10px;
  border-radius: 15px;
  font-size: 20px;
  font-family: "Berlin Sans FB";
}

.sectionTitle {
  color: #000000;
}

</style>