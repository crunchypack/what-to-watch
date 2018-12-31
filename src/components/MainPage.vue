<template>
  <!-- Main div -->
  <div class="main">
    <h1>Movies</h1>
    <!-- Searchbar -->
    <b-container fluid class="search">
      <b-row>
        <b-col sm="3">
          <label class="big">Search Movies</label>
        </b-col>
        <b-col sm="6">
          <!-- input field two-way bound to search variable-->
          <b-form-input
            type="text"
            v-model="search"
            placeholder="Enter title, actor, director, genre or service (Netflix or Amazon Prime)"
          ></b-form-input>
          <!--END input field-->
        </b-col>
      </b-row>
    </b-container>
    <!--END Searchbar-->
    <!-- b-card-group start -->
    <b-card-group>
      <!-- div, one created for every movie -->
      <div v-for="(movie, index) in searchedMovies" :key="index" class="movie">
        <!-- b-card start -->
        <!-- Calling getPoster(title) with current movie title to get the image url in the img-src attribute -->
        <b-card
          no-body
          style="max-width: 20rem"
          :footer="'Genre: '+movie.genre"
          :img-src="getPoster(movie.title)"
          img-alt="Movie poster"
          img-top
        >
          <!-- card contents -->
          <!-- using the for loop dynamically write out movie title, description and so on... -->
          <h4 slot="header">{{movie.title}}</h4>
          <b-card-body>
            <p class="card-text">{{movie.desc}}</p>
          </b-card-body>
          <b-list-group flush>
            <!-- starring is an array so annother for loop is required -->
            <b-list-group-item v-for="(cast, index) in movie.starring" :key="index">{{cast}}</b-list-group-item>
            <b-list-group-item>
              <span class="dir">Director:</span>
              {{movie.director}}
            </b-list-group-item>
            <b-list-group-item>
              Available:
              <span v-for="(stream, i) in movie.available" :key="i">{{stream+" "}}</span>
            </b-list-group-item>
          </b-list-group>
          <b-card-body>
            IMBb:
            <a :href="movie.url" class="card-link" target="_blank">{{movie.title}}</a>
            <p>Tomato score:
              <!-- Calling getRating(title) with current movie title to get rating  -->
              <span class="rating">{{getRating(movie.title)}}</span>
            </p>
            <!-- END card contents -->
          </b-card-body>
          <!--END b-card -->
        </b-card>
        <!-- END div -->
      </div>
    </b-card-group>
  </div>
</template>

<script>
// Import axios for ajax requests
import axios from "axios";
export default {
  name: "MainPage",
  data() {
    return {
      // data
      movies: [],
      search: "",
      poster: [],
      ratings: []
    };
  },
  // Funcions called after instance has been mounted
  mounted() {
    // Get all the movies and store them in an array
    axios({
      method: "GET",
      url: "https://lobonode.ddns.net/api"
    }).then(res => {
      this.movies = res.data;
      // Run function for each movie in the database
      this.movies.forEach(title => {
        this.fillIMDb(title.title);
      });
    });
  },
  // Computed functions
  computed: {
    // Search function
    searchedMovies: function() {
      let filter = new RegExp(this.search, "i");
      let final;
      // Filter array
      return this.movies.filter(movie => {
        // check title,actor, director, stream service and genre
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
        // return filtered array
        return final;
      });
    }
  },
  // Functions
  methods: {
    // get imdb data
    fillIMDb: function(title) {
      // Ajax get call to API omdbapi
      /** example part of response
         * {"Title":"Star Wars: The Last Jedi",
         * "Year":"2017","Rated":"PG-13","Released":"15 Dec 2017",
         * "Runtime":"152 min","Genre":"Action, Adventure, Fantasy, Sci-Fi",
         * "Poster":"https://m.media-amazon.com/images/M/MV5BMjQ1MzcxNjg4N15BMl5BanBnXkFtZTgwNzgwMjY4MzI@._V1_SX300.jpg",
         * "Ratings":[{"Source":"Internet Movie Database","Value":"7.2/10"},{"Source":"Rotten Tomatoes","Value":"91%"},
         * {"Source":"Metacritic","Value":"85/100"}],

         */
      axios({
        method: "GET",
        url: "http://www.omdbapi.com/?t=" + title + "&apikey=dabd7b02"
      }).then(res => {
        // Add poster url and ratings from rotten tomatoes to two arrays
        this.poster.push(title + " " + res.data.Poster);
        this.ratings.push(title + " " + res.data.Ratings[1].Value);
      });
    },
    // get poster url
    getPoster: function(title) {
      var source;
      // Gag movie
      if (title == "Natalies Crazy Cats") {
        source = "https://placekitten.com/g/300/450";
      } else {
        // check the array
        this.poster.forEach(item => {
          // if it includes title
          if (item.includes(title)) {
            // remove title and keep url
            source = item.split(title + " ").pop();
          }
        });
      }
      // return url
      return source;
    },
    // same as previous function but for ratings
    getRating: function(title) {
      var rating;
      if (title == "Natalies Crazy Cats") {
        rating = "100%";
      } else {
        this.ratings.forEach(item => {
          if (item.includes(title)) {
            rating = item.split(title + " ").pop();
          }
        });
      }

      return rating;
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
#main {
  width: 100%;
  margin: 0 auto;
}
.dir {
  font-weight: bold;
}
body {
  width: 100%;
  margin: 0 auto;
}
h1 {
  text-align: center;
}
.search {
  margin-bottom: 5%;
}
.movie {
  margin: 1%;
}
</style>
