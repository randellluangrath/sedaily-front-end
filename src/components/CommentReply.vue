<template>
  <div v-if="me" class="col-md-8">
    <div v-if="me.name">
      <div class='reply-container'>
        <div v-if="justSubmitted">
          Thanks for submitting!
          <spinner :show="true"></spinner>
        </div>
        <div v-else>
          <textarea placeholder='Your message here...'
          class='reply-box'
          type='text'
          v-model='commentContent' />
          <button class='button-submit-small' @click='submitComment'>
            Reply
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import UpdateProfile from 'components/UpdateProfile.vue'
import Spinner from 'components/Spinner'
import { mapState, mapActions } from 'vuex'

export default {
  name: 'comment-reply',
  props: {
    parentComment: {
      type: Object,
      required: true
    },
    rootEntityType: {
      type: String,
      required: false
    }
  },
  components: {
    UpdateProfile,
    Spinner
  },
  beforeMount () {
    console.log('rootEntityType--reply', this.rootEntityType)
  },
  data () {
    return {
      commentContent: '',
      justSubmitted: false,
      username: null,
      loading: true
    }
  },

  computed: {
    // local computed methods +
    ...mapState({
      me (state) {
        return state.me
      },
      entityId (state) {
        return state.route.params.id // TODO: pass into component
      }
    })
  },

  methods: {
    ...mapActions(['commentsCreate', 'commentsFetch']),
    submitComment () {
      this.justSubmitted = true
      this.commentsCreate({
        entityId: this.entityId,
        rootEntityType: this.rootEntityType,
        parentCommentId: this.parentComment._id,
        content: this.commentContent
      })
        .then((response) => {
          this.commentContent = ''
          // NOTE: this won't work too well once we are paginating comments:
          this.justSubmitted = false
          this.commentsFetch({
            entityId: this.entityId
          })
        })
        .catch((error) => {
          this.$toasted.error(error.response.data.message)
        })
    } }
}
</script>

<style scoped lang="stylus">
.reply-container
  margin-left 40px
  margin-top 15px

.collapsed-area
  margin-left 20px
  margin-top 10px

.reply-box
  width 100%
  padding 5px 5px
  margin-bottom 12px
  border-radius 2px
  border-color #c5c5c5
</style>
