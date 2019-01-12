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
          <b-form-input
            type="text"
            v-model="search"
            placeholder="Enter title, actor, director, genre or service (Netflix or Amazon Prime)"
          ></b-form-input>
          <!--END input field-->
        </b-col>
      </b-row>
      <br>
      <b-row class="text-center">
        <b-col col>
          <b-button align="center" :variant="'success'" @click="searchAll('Netflix')">Netflix</b-button>
        </b-col>
        <b-col col>
          <b-button align="center" :variant="'success'" @click="searchAll('Amazon')">Amazon</b-button>
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
          :footer="'Genre: '+movie.genre"
          :img-src="getPoster(movie.title,movie.year)"
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
            <b-list-group-item v-for="(cast, index) in movie.starring" :key="index">
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
            <b-list-group-item style="background-color:#46bf68" v-if="movie.available[0] != null">
              <span class="bold">Available:</span>
              <span v-for="(stream, i) in movie.available" :key="i">
                <span @click="searchAll(stream)" v-if="i==1">,
                  <span class="stream">{{stream}}</span>
                </span>
                <span @click="searchAll(stream)" v-else class="stream">{{" "+ stream}}</span>
              </span>
            </b-list-group-item>
            <b-list-group-item style="background-color:#bf464c" v-else>
              <span class="bold">Not available</span>
            </b-list-group-item>
          </b-list-group>
          <b-card-body>
            IMBb:
            <a :href="movie.url" class="card-link" target="_blank">{{movie.title}}</a>
            <p>Tomato score:
              <!-- Calling getRating(title) with current movie title to get rating  -->
              <span class="rating">{{getRating(movie.title,movie.year)}}</span>
            </p>
            <!-- END card contents -->
          </b-card-body>
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
        if (title.title.includes("&")) {
          let temp = title.title.replace("&", "%26");
          this.fillIMDb(temp, title.year);
        } else {
          this.fillIMDb(title.title, title.year);
        }
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
          "https://www.omdbapi.com/?t=" + title + "&apikey=dabd7b02&y=" + year
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
    // same as previous function but for ratings
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
    searchAll: function(s) {
      this.search = s;
      window.scrollTo(0, 0);
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
.clickable {
  color: rgb(19, 128, 230);
  font-style: italic;
}
.clickable:hover {
  cursor: pointer;
  color: rgb(4, 68, 128);
}
.stream {
  color: rgb(27, 25, 25);
  font-weight: bold;
}
.stream:hover {
  cursor: pointer;
  color: rgb(255, 255, 255);
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
