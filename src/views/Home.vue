<template>
  <div class="home">
    <div id="cube-loader" v-if="isLoading">
      <div class="caption">
        <div class="cube-loader">
          <div class="cube loader-1"></div>
          <div class="cube loader-2"></div>
          <div class="cube loader-4"></div>
          <div class="cube loader-3"></div>
        </div>
      </div>
    </div>
    <div class="container">
      <div class="col-8">
        <div class="group"
             v-for="post in filteredList"
             :key="post.id"
        >
          <div class="comments comments-big">
            <img src="https://placekitten.com/100/100" alt="Cat">
            <p class="name">{{ post.userName }}</p>
            <h2>{{ post.title }}</h2>
            <p class="description">{{ post.body }}</p>
            <div class="clear"></div>
            <div class="button-container" v-if="!post.show">
              <button @click="getComments(post)">Открыть комментарии</button>
            </div>
          </div>
          <template v-if="post.show">
            <div class="comments comments-small" v-for="comment in post.comments" :key="comment.id">
              <img src="https://placekitten.com/40/60" alt="Cat">
              <h2>{{ comment.name }}</h2>
              <p class="description">{{ comment.body }}</p>
              <div class="clear"></div>
            </div>
            <div class="button-container hide comments-small">
              <button @click="post.show = !post.show" v-if="post.show">Скрыть комментарии</button>
            </div>
          </template>
        </div>
      </div>
      <div class="col-4">
        <div class="filter">
          <div class="filter__form">
            <p>Фильтры</p>
            <div class="form-group">
              <label for="name">Имя/название</label>
              <input type="text" id="name" v-model="name">
            </div>
            <div class="form-group">
              <label for="content">Контент</label>
              <input type="text" id="content" v-model="content">
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'home',
  data() {
    return {
      posts: [],
      comments: [],
      name: '',
      content: '',
      isLoading: true
    }
  },
  methods: {
    getPosts() {
      this.axios.get('https://jsonplaceholder.typicode.com/posts', {
        headers: {
          common: {
            'X-Requested-With': 'XMLHttpRequest',
            'Access-Control-Allow-Origin': '*'
          }
        }
      }).then((response) => {
        for (let i = 0; i < response.data.length; i++) {
          this.posts.push({
              userId: response.data[i].userId,
              id: response.data[i].id,
              title: response.data[i].title,
              body: response.data[i].body,
              show: false
          })
        }
        this.axios.get('https://jsonplaceholder.typicode.com/users', {
          headers: {
            common: {
              'X-Requested-With': 'XMLHttpRequest',
              'Access-Control-Allow-Origin': '*'
            }
          }
        }).then((response) => {
          for (let i = 0; i < this.posts.length; i++) {
            for (let j = 0; j < response.data.length; j++) {
              if (this.posts[i].userId === response.data[j].id) {
                this.$set(this.posts[i], 'userName', response.data[j].name)
              }
            }
          }
          this.isLoading = !this.isLoading
        })
      })
    },
    getComments(post) {
      if (!this.posts[post.id-1].comments) {
        this.isLoading = !this.isLoading
        this.axios.get('https://jsonplaceholder.typicode.com/comments', {
          headers: {
            common: {
              'X-Requested-With': 'XMLHttpRequest',
              'Access-Control-Allow-Origin': '*'
            }
          },
          params: {
            postId: post.id
          }
        }).then((response) => {
          this.comments = []
          for (let i = 0; i < this.posts.length; i++) {
            for (let j = 0; j < response.data.length; j++) {
              if (this.posts[i].id === response.data[j].postId) {
                this.comments.push(response.data[j])
              }
            }
          }
          this.$set(this.posts[post.id-1], 'comments', this.comments)
          post.show = !post.show
          this.isLoading = !this.isLoading
        })
      } else {
        post.show = !post.show
      }
    }
  },
  computed: {
    filteredList() {
      let name = this.name
      let content = this.content
      return this.posts.filter(function(elem) {
        return (elem.title.indexOf(name) > -1 || elem.userName.indexOf(name) > -1) && elem.body.indexOf(content) > -1
      })
    }
  },
  mounted() {
    this.getPosts()
  }
}
</script>
