<template>
  <div class="container">
    <Header :crumbs="[{ link: `/topic/${topic}`, text: 'topic' }]" />
    <div class="margined">
      <Loading v-if="$fetchState.pending" />
      <Error
        v-if="error"
        :error="error.title"
        :details="error.details"
        :fullwidth="true"
      />
      <div v-if="!$fetchState.pending">
        <div v-if="data && data[0]">
          <h1 style="display: inline-block">{{ data[0].topic_name }}</h1>
          <p>
            id: <b>{{ data[0].topic_id }}</b>
          </p>
          <p>
            category:
            <b
              ><nuxt-link :to="`/browse/${data[0].category_id}`">{{
                data[0].category_name
              }}</nuxt-link></b
            >
          </p>
        </div>
        <PostListSP
          :posts="data"
          :loading="$fetchState.pending"
          @loadMore="loadMore()"
          :showLoadMore="showLoadMore"
          :op="op"
        />
      </div>
      <Footer />
    </div>
  </div>
</template>

<script>
import map from "@/assets/category-map.js";

function getKey(map, value) {
  // thanks https://stackoverflow.com/a/53313763/9918633
  return [...map].find(([key, val]) => val == value)[0];
}

export default {
  head() {
    return {
      title:
        this.data?.hits && this.data.hits[0]
          ? this.data.hits[0].topic.title
          : `topic id ${this.topic}`,
    };
  },
  data() {
    return {
      topic: this.$route.params.topic,
      page: 0,
      data: {},
      category: "",
      showLoadMore: true,
      error: null,
      op: "",
    };
  },
  methods: {
    async loadMore() {
      this.showLoadMore = false;
      this.page++;

      var postsRes = await fetch(
        `https://api.scratchpost.quuq.dev/search/posts?mode=relevance&sort=oldest&category=all&limit=100&offset=${100 * this.page}&topic=${this.topic}&detail=3&displayAuthor=true`,
      ).catch((err) => {
        this.error = {
          title: "Network Error",
          details: "Failed to connect to ScratchDB.",
        };
      });
      var postData = await postsRes.json();
      this.data.push(...postData);

      this.showLoadMore = true;
    },
  },
  async fetch() {
    var postsRes = await fetch(
      `https://api.scratchpost.quuq.dev/search/posts?mode=relevance&sort=oldest&category=all&limit=100&offset=0&topic=${this.topic}&detail=3&displayAuthor=true`,
    ).catch((err) => {
      this.error = {
        title: "Network Error",
        details: "Failed to connect to ScratchDB.",
      };
    });
    var postData = await postsRes.json();

    this.data = postData;
    this.op = postData[0]?.author;
    this.category = postData[0]?.category_id 
    this.showLoadMore = true;
  },
  fetchOnServer: false,
};
</script>

