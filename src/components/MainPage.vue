<template>
  <div class="main">
    <h1>Movies</h1>
    <b-container fluid>
      <b-row>
        <b-col sm="3">
          <label class="big">Search Movies</label>
        </b-col>
        <b-col sm="6">
          <b-form-input
            type="text"
            v-model="search"
            placeholder="Enter title, actor, genre or service (Netflix or Amazon Prime)"
          ></b-form-input>
        </b-col>
      </b-row>
    </b-container>
    <div v-for="(movie, index) in searchedMovies" :key="index">
      <h2>{{movie.title}}</h2>
      <p>
        Cast:
        <span v-for="(actor, idx) in movie.starring" :key="idx">
          {{actor}}
          <span v-if="movie.starring.length -1 != idx">,</span>
        </span>
      </p>
      <p>{{movie.desc}}</p>
      <p>
        Genre:
        <span class="genre" v-for="(genre, index) in movie.genre" :key="index">
          {{genre}}
          <span v-if="movie.genre.length -1 != index">,</span>
        </span>
      </p>
      <p>Director: {{movie.director}}</p>
      <p v-for="(stream, index) in movie.available" :key="index">Available at: {{stream}}</p>
      <a :href="movie.url" target="_blank">
        <p>IMDb Link</p>
      </a>
    </div>
  </div>
</template>

<script>
import axios from "axios";
export default {
  name: "MainPage",
  data() {
    return {
      movies: [],
      search: ""
    };
  },
  mounted() {
    axios({
      method: "GET",
      url: "https://lobonode.ddns.net/api"
    }).then(res => {
      this.movies = res.data;
    });
  },
  computed: {
    searchedMovies: function() {
      let filter = new RegExp(this.search, "i");
      let final;
      return this.movies.filter(movie => {
        final = movie.title.match(filter);
        final += movie.director.match(filter);

        for (let i = 0; i < movie.available.length; i++) {
          final += movie.available[i].match(filter);
        }
        for (let i = 0; i < movie.starring.length; i++) {
          final += movie.starring[i].match(filter);
        }
        for (let i = 0; i < movie.genre.length; i++) {
          final += movie.genre[i].match(filter);
        }

        return final;
      });
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
