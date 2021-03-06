<template>
  <div v-if="asyncDataStatus_ready" class="col-large push-top">
    <h1 class="text-white">
      {{thread.title}}
      <router-link
        class="text-white"
        v-if="authUser._id === thread.author._id"
        :to="{name: 'ThreadEdit', slug: this.slug}"
      >
        <i class="fa fa-pencil"></i>
      </router-link>
    </h1>

    <p class="text-white">{{thread.text}}</p>
    <p class="text-white">
      By
      <a href="#" class="link-unstyled">{{thread.author.username}}</a>,
      <AppDate :timestamp="thread.publishedAt" />.
    </p>
    <PostList v-if="posts" :posts="posts" :threadId="thread._id" />
    <PostEditor v-if="authUser" :threadId="thread._id" :threadSlug="thread.slug" />
    <div v-else class="text-center" style="margin-bottom: 50px;">
      <router-link
        :to="{name: 'SignIn', query: {
        redirectTo: $route.path
        }
        }"
      >Sign in</router-link>or
      <router-link
        :to="{name: 'Register', query: {
        redirectTo: $route.path
        }
        }"
      >Register</router-link>to post a reply.
    </div>
  </div>
</template>

<script>
import { mapActions, mapState } from 'vuex';
import PostList from '@/components/PostList';
import PostEditor from '@/components/PostEditor';
import asyncDataStatus from '@/mixins/asyncDataStatus';

export default {
  components: {
    PostList,
    PostEditor
  },

  mixins: [asyncDataStatus],

  props: {
    slug: {
      required: true,
      type: String
    }
  },

  computed: {
    ...mapState('threads', {
      thread: state => state.thread
    }),

    ...mapState('auth', {
      authUser: state => state.currentUser.user
    }),

    ...mapState('posts', {
      posts: state => state.posts
    }),

    repliesCount() {
      return this.thread.posts.length;
    },

    contributorsCount() {
      return this.thread.contributors.length;
    }
  },

  methods: {
    ...mapActions('threads', ['fetchThreadBySlug']),
    ...mapActions('posts', ['fetchPosts'])
  },

  created() {
    // fetch thread
    this.fetchThreadBySlug(this.slug).then(thread => {
      this.fetchPosts(thread._id);
      this.asyncDataStatus_fetched();
    });
  }
};
</script>
