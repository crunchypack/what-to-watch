<template>
  <!-- Main div -->
  <div class="main">
    <h1>Movies</h1>
    <!-- Searchbar -->
    <b-container>
      <b-row>
        <b-col sm="3">
          <label class="big">Search Movies</label>
        </b-col>
        <b-col sm="6">
          <!-- input field two-way bound to search variable-->
          <b-form-input type="text" v-model="search" placeholder="Title, Actor, Director or Genre"></b-form-input>
          <!--END input field-->
          Sort by / Ordenar por
          <b-form-select v-model="sort" :options="sortBy" size="small"/>
          <b-form-select v-if="popular" v-model="asc" :options="descend"/>
        </b-col>
      </b-row>
      <br>
      <b-row class="center">
        <b-form-group>
          <b-form-checkbox-group
            id="servicebox"
            name="service"
            v-model="service"
            :options="options"
          ></b-form-checkbox-group>
        </b-form-group>
      </b-row>
    </b-container>
    <!--END Searchbar-->
    <!-- b-card-group start -->
    <b-card-group>
      <!-- div, one created for every movie -->
      <div v-for="(movie, index) in searchedMovies" :key="index" class="movie">
        <!-- b-card start -->
        <span style="visibility:" hidden>{{movie["imdbScore"]= getScore(movie.title,movie.year)}}</span>
        <!-- Calling getPoster(title) with current movie title to get the image url in the img-src attribute -->
        <b-card no-body :img-src="getPoster(movie.title,movie.year)" img-alt="Movie poster" img-top>
          <!-- card contents -->
          <!-- using the for loop dynamically write out movie title, description and so on... -->
          <h4 slot="header" class="view-item" @click="showHide(movie)">{{movie.title}}</h4>

          <b-card-body v-if="movie.visible">
            <p class="card-text">{{movie.desc}}</p>
          </b-card-body>
          <b-list-group flush v-if="movie.visible">
            <!-- starring is an array so annother for loop is required -->
            <b-list-group-item v-for="(cast, c) in movie.starring" :key="c">
              <span @click="searchAll(cast)" class="clickable">{{cast}}</span>
            </b-list-group-item>
            <b-list-group-item>
              <span class="bold">Released:</span>
              {{movie.year}}
            </b-list-group-item>
            <b-list-group-item>
              <span class="bold">Director:</span>
              <span @click="searchAll(movie.director)" class="clickable">{{" "+movie.director}}</span>
            </b-list-group-item>
          </b-list-group>
          <b-list-group>
            <b-list-group-item style="background-color:#46bf68" v-if="movie.available[0] != null">
              <span class="bold">Available:</span>
              <span v-for="(stream, i) in movie.available" :key="i">
                <span v-if="i==1">
                  ,
                  <span class="stream" v-if="stream == 'Netflix'">
                    <a href="https://www.Netflix.com" target="_blank">{{stream}}</a>
                  </span>
                  <span class="stream" v-else>
                    <a
                      href="https://www.amazon.com/Amazon-Video/b/ref=sv_atv_logo?node=2858778011&ie=UTF8"
                      target="_blank"
                    >{{stream}}</a>
                  </span>
                </span>
                <span v-else class="stream">
                  <span class="stream" v-if="stream == 'Netflix'">
                    <a href="https://www.Netflix.com" target="_blank">{{" "+ stream}}</a>
                  </span>
                  <span class="stream" v-else>
                    <a
                      href="https://www.amazon.com/Amazon-Video/b/ref=sv_atv_logo?node=2858778011&ie=UTF8"
                      target="_blank"
                    >{{" "+ stream}}</a>
                  </span>
                </span>
              </span>
            </b-list-group-item>
            <b-list-group-item style="background-color:#bf464c" v-else>
              <span class="bold">Not available</span>
            </b-list-group-item>
          </b-list-group>
          <b-card-body v-if="movie.visible">
            IMBb:
            <a :href="movie.url" class="card-link" target="_blank">{{movie.title}}</a>
            <p>
              Tomato score:
              <!-- Calling getRating(title) with current movie title to get rating  -->
              <span class="rating">{{getRating(movie.title,movie.year)}}</span>
            </p>
            <p>
              IMDb Score:
              {{movie.imdbScore}}
            </p>
          </b-card-body>
          <b-card-footer v-if="movie.visible">
            <div v-for="(genre, g) in movie.genre" :key="g">
              <span v-if="g != movie.genre.length-1">
                <span @click="searchAll(genre)" class="clickable">{{genre}}</span>,
              </span>
              <span v-else class="clickable" @click="searchAll(genre)">{{genre}}</span>
            </div>
          </b-card-footer>
          <!--END b-card -->
        </b-card>
        <!-- END div -->
      </div>
      <!-- b-card-group END -->
    </b-card-group>
    <!-- Main div END -->
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
      service: [],
      sortBy: [
        { value: "pop", text: "High rated /Puntaje Alto" },
        { value: "_id", text: "Date Added / Fecha Agregada" },
        { value: "title", text: "Title / Título" },
        { value: "year", text: "Year / Año" }
      ],
      sort: "_id",
      asc: -1,
      descend: [
        { value: 1, text: "Ascending / Ascendiendo" },
        { value: -1, text: "Descending / Descendiendo" }
      ],
      options: ["Netflix", "Amazon Prime"],
      movies: [],
      search: "",
      poster: [],
      ratings: [],
      imdb: [],
      popular: true
    };
  },
  // Funcions called after instance has been mounted
  mounted() {
    // Get all the movies and store them in an array
    this.init();
  },
  // Computed functions
  computed: {
    // Search function
    searchedMovies: function() {
      let filter = new RegExp(this.search, "i");
      let final;
      // If nothing is written
      if (this.search == "") {
        return this.movies.filter(movie => {
          // Return all movies
          final = movie.title.match(filter);
          return final;
        });
      } else {
        return this.movies.filter(movie => {
          // check title,actor, director, stream service and genre
          final = movie.title.match(filter);
          final += movie.director.match(filter);
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
    }
  },
  //watcher
  watch: {
    service: {
      handler: function() {
        this.updateMovies();
      },
      deep: true
    },
    sort: {
      handler: function() {
        this.updateMovies();
      },
      deep: true
    },
    asc: {
      handler: function() {
        this.updateMovies();
      },
      deep: true
    }
  },
  // Functions
  methods: {
    init: function() {
      axios({
        method: "GET",
        url: "https://lobonode.ddns.net/api/" + this.sort + "/" + this.asc
      }).then(res => {
        this.movies = res.data;
        // Run function for each movie in the database
        this.movies.forEach(title => {
          if (title.title.includes("&")) {
            let temp = title.title.replace("&", "%26");
            this.fillIMDb(temp, title.year);
          } else {
            this.fillIMDb(title.title, title.year);
          }
          title["visible"] = false;
        });
      });
    },
    // get imdb data
    fillIMDb: function(title, year) {
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
        url:
          "https://www.omdbapi.com/?t=" + title + "&apikey=d5c6232e&y=" + year
      }).then(res => {
        // Add poster url and ratings from rotten tomatoes to two arrays
        if (title.includes("%26")) {
          title = title.replace("%26", "&");
        }
        this.poster.push(title + " " + year + " " + res.data.Poster);
        // Sometimes rotten tomatoes score is not available
        try {
          this.ratings.push(
            title + " " + year + " " + res.data.Ratings[1].Value
          );
        } catch {
          this.ratings.push(title + " " + year + " " + "  N/A");
        }
        this.imdb.push(title + " " + year + " " + res.data.imdbRating);
      });
    },
    // get poster url
    getPoster: function(title, year) {
      var source;
      // Gag movie
      if (title == "Natalies Crazy Cats") {
        source = "https://placekitten.com/g/300/450";
      } else {
        // check the array
        this.poster.forEach(item => {
          // if it includes title
          if (item.includes(title + " " + year)) {
            // remove title and keep url
            source = item.split(title + " " + year + " ").pop();
          }
        });
      }
      // return url
      return source;
    },
    updateMovies: function() {
      function compare(a, b) {
        if (a.imdbScore < b.imdbScore) return 1;
        if (a.imdbScore > b.imdbScore) return -1;
        return 0;
      }
      if (this.sort == "pop") {
        this.popular = false;
        this.movies.sort(compare);
      } else {
        this.popular = true;
        if (this.service.length < 1) {
          axios({
            method: "GET",
            url: "https://lobonode.ddns.net/api/" + this.sort + "/" + this.asc
          }).then(res => {
            this.movies = res.data;
          });
        } else if (this.service.length == 1) {
          axios({
            method: "GET",
            url:
              "https://lobonode.ddns.net/api/movies/" +
              this.service[0] +
              "/" +
              this.sort +
              "/" +
              this.asc
          }).then(res => {
            this.movies = res.data;
          });
        } else if (this.service.length == 2) {
          axios({
            method: "GET",
            url:
              "https://lobonode.ddns.net/api/movies/" +
              this.service[0] +
              "/" +
              this.service[1] +
              "/" +
              this.sort +
              "/" +
              this.asc
          }).then(res => {
            this.movies = res.data;
          });
        }
      }

      this.search = this.search + " ";
      this.search = this.search.slice(0, -1);
    },
    // same as poster function but for ratings
    getRating: function(title, year) {
      var rating;
      if (title == "Natalies Crazy Cats") {
        rating = "100%";
      } else {
        this.ratings.forEach(item => {
          if (item.includes(title + " " + year)) {
            rating = item.split(title + " " + year + " ").pop();
          }
        });
      }

      return rating;
    },
    getScore: function(title, year) {
      var score;
      if (title == "Natalies Crazy Cats") {
        score = "10.0";
      } else {
        this.imdb.forEach(item => {
          if (item.includes(title + " " + year)) {
            score = item.split(title + " " + year + " ").pop();
          }
        });
      }
      if (score == "N/A") {
        score = "5.0";
      }

      return score;
    },
    searchAll: function(s) {
      this.movies.forEach(movie => {
        movie.visible = false;
      });

      this.search = s;
      window.scrollTo(0, 0);
    },
    showHide: function(movie) {
      movie.visible = !movie.visible;
      this.search = this.search + " ";
      this.search = this.search.slice(0, -1);
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
.card {
  width: 20rem;
}
.center {
  margin: auto;
  max-width: 250px;
}
.clickable {
  color: rgb(19, 128, 230);
  font-style: italic;
}
.clickable:hover {
  cursor: pointer;
  color: rgb(4, 68, 128);
}
.view-item {
  color: rgb(4, 59, 4);
}
.view-item:hover {
  cursor: pointer;
  color: hotpink;
}
.stream,
a {
  color: rgb(27, 25, 25);
  font-weight: bold;
}
.stream:hover,
a:hover {
  cursor: pointer;
  color: rgb(255, 255, 255);
  text-decoration: none;
}
.bold {
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
@media screen and (max-width: 800px) {
  .card {
    width: 70%;
    margin: 0 auto;
  }
}
</style>
