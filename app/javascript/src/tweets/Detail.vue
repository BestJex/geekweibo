  <template>
  <div>
    <div class="flex flex-row mb-4 bg-white shadow sm:rounded-lg">
      <img
        :src="tweet.user.avatar"
        class="inline-block object-cover w-12 h-12 mt-4 ml-4 rounded-full"
      />
      <div class="w-full p-4">
        <h3 class="text-lg font-medium text-gray-900 leading-6">
          {{ tweet.user.nickname }}
        </h3>
        <div
          class="mt-1 text-sm text-gray-500 leading-5 markdown"
          v-html="markdown(tweet.body)"
        ></div>
        <div class="flex items-center justify-end mt-4">
          <svg
            fill="none"
            class="w-4 h-4"
            stroke-width="2"
            viewBox="0 0 24 24"
            stroke="currentColor"
            @click.stop="showTweetComment(tweet)"
          >
            <path
              d="M8 12h.01M12 12h.01M16 12h.01M21 12c0 4.418-4.03 8-9 8a9.863 9.863 0 01-4.255-.949L3 20l1.395-3.72C3.512 15.042 3 13.574 3 12c0-4.418 4.03-8 9-8s9 3.582 9 8z"
            ></path>
          </svg>
          <span
            class="inline-block ml-1 text-xs"
            v-if="tweet.comments.length"
            >{{ tweet.comments.length }}</span
          >
          <svg
            viewBox="0 0 24 24"
            class="w-4 h-4 ml-4 fill-current"
            :class="tweet.liked ? 'text-red-500' : 'text-black'"
            @click.stop="toggleLike(tweet)"
          >
            <path
              class="heroicon-ui"
              d="M12.76 3.76a6 6 0 018.48 8.48l-8.53 8.54a1 1 0 01-1.42 0l-8.53-8.54a6 6 0 018.48-8.48l.76.75.76-.75zm7.07 7.07a4 4 0 10-5.66-5.66l-1.46 1.47a1 1 0 01-1.42 0L9.83 5.17a4 4 0 10-5.66 5.66L12 18.66l7.83-7.83z"
            />
          </svg>
          <span v-if="tweet.like_count" class="inline-block ml-1 text-xs">{{
            tweet.like_count
          }}</span>
        </div>
        <div class="mt-2 rounded">
          <div class="flex flex-col p-2 bg-gray-100 rounded">
            <div class="flex flex-row">
              <img
                v-if="currentUser"
                :src="currentUser.avatar"
                class="object-cover w-8 h-8 rounded-full"
              />
              <div
                contenteditable="true"
                ref="tweet_comment_inbox"
                @input="updateNewComment"
                @focus="confirmLogin"
                class="w-full p-2 ml-2 bg-white border-indigo-100 border-solid rounded outline-none"
              ></div>
            </div>
            <button
              @click="postComment"
              class="self-end px-3 py-2 mt-2 text-xs text-white bg-indigo-500 rounded outline-none focus:outline-none active:bg-indigo-600"
            >
              评论
            </button>
          </div>
          <div
            v-for="comment in tweet.comments"
            class="flex flex-row mt-3"
            v-bind:key="comment.id"
          >
            <img
              :src="comment.user.avatar"
              class="object-cover w-8 h-8 rounded-full"
            />
            <div class="w-full ml-2">
              <div class="text-xs font-medium text-gray-900">
                {{ comment.user.nickname }}
              </div>
              <div
                class="text-sm text-gray-500 markdown"
                v-html="markdown(comment.body)"
              ></div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
const marked = require("marked");
marked.setOptions({
  renderer: new marked.Renderer(),
  gfm: true,
  breaks: true,
  pedantic: false,
  smartLists: true,
  smartypants: false
});

import LoginDialog from "../common/LoginDialog.vue";
export default {
  props: ["origin_tweet"],
  data() {
    return {
      tweet: this.origin_tweet,
      show_comment: true,
      currentUser: window.currentUser()
    };
  },
  methods: {
    toggleLike() {
      if (!this.currentUser) {
        this.showLoginDialog();
        return;
      }
      const action = this.tweet.liked ? window.delete : window.post;
      action("/likes", {
        likable_id: this.tweet.id,
        likable_type: "Tweet"
      }).then(res => console.log("like/unlike succeed"));

      this.tweet.liked = !this.tweet.liked;
      this.tweet.like_count += this.tweet.liked ? 1 : -1;
    },
    confirmLogin() {
      if (!this.currentUser) this.showLoginDialog();
    },
    showLoginDialog() {
      this.$modal.show(LoginDialog, {}, { height: "auto", width: 400 });
    },
    updateNewComment(e) {
      this.new_comment = e.target.innerText;
    },
    postComment() {
      if (!this.new_comment) return;

      post("/comments", {
        tweet_id: this.tweet.id,
        body: this.new_comment
      }).then(data => {
        this.tweet.comments.unshift({
          user: this.currentUser,
          body: data.body
        });
        this.new_comment = "";
        this.$refs.tweet_comment_inbox.innerText = "";
      });
    },
    markdown(text) {
      return marked(text);
    }
  }
};
</script>
