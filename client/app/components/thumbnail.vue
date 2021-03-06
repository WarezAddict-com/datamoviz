<template>
  <div class="movie-container" :class="{ highlighted: highlighted, hidden: hidden }" @keyup.escape="close()">
    <a href="#" @click.prevent="select()">
      <img :src="posterPath" height="300" />
      <span class="title">
        <span class="badge">{{ movie.release_date|date("%Y") }}</span>
        {{ movie.title }}
      </span>
    </a><!--
  --><div class="details">
      <h3>
        <small v-for="country in movie.production_countries" :key="country.iso_3166_1"><img
                :src="`https://ssl-proxy.my-addr.org/myaddrproxy.php/http/www.geonames.org/flags/s/${country.iso_3166_1.toLowerCase()}.png`"
                :alt="country.name"
                :title="country.name" />&nbsp;</small>{{ movie.original_title }}
        <small><i class="fa fa-language"></i> {{ movie.spoken_languages.map(a => a.name).join(', ') }}</small>
      </h3>
      <p>
        <span class="badge">{{ movie.release_date|date("%m\/%d\/%Y") }}</span>
        <span class="badge" v-if="movie.runtime"><i class="fa fa-clock-o"></i> {{ movie.runtime }} min</span>
        <small>IMDb {{ movie.imdb_id }}</small>
      </p>
      <p class="overview"><strong>{{ movie.tagline }}</strong> {{ movie.overview }}</p>
      <gauge-chart
              name="Vote average"
              :value="movie.imdb_rating"
              :max="10" :size="100"
              :title="`${movie.imdb_nb_reviews} votes`"
              :thresholds="[4, 7]"
              :imdb="movie.imdb_id"
              v-if="movie.imdb_rating !== undefined"></gauge-chart>
      <gauge-chart
              name="Movie popularity"
              :value="movie.imdb_nb_reviews"
              :max="popularity" :size="100"
              :title="'Popularity'"
              :thresholds="[popularity/8, popularity/2]"
              :imdb="movie.imdb_id"></gauge-chart>
      <profitability-chart
              v-if="movie.budget !== 0 && movie.revenue !== undefined"
              :budget="movie.budget"
              :revenue="movie.revenue"
              :imdb="movie.imdb_id"></profitability-chart>
      <p><span v-for="genre, key in movie.genres" :key="key"><span class="badge">{{ genre }}</span>&nbsp;</span>
        <small v-if="movie.homepage"><a :href="movie.homepage" title="Go to movie official website">{{ movie.homepage }}</a></small></p>
      <small><span class="badge badge-info">Pro tip</span> Press <span class="badge">esc</span> to deselect the movie.</small>
  </div>
  </div>
</template>

<script>
  import { FILTERS_UPDATE, MOVIE_SELECTED } from '../event-bus';
  import GaugeChart from './chart-gauge.vue';
  import ProfitabilityChart from './chart-profitability.vue';

  export default {
    name: 'thumbnail',
    props: ['movie', 'total', 'popularity'],
    components: {
      GaugeChart,
      ProfitabilityChart
    },
    computed: {
      posterPath () {
        if (this.movie.poster_path === null) {
          return 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAL4AAAEDCAMAAAC8iU6wAAAAGFBMVEWZmZmwsLCcnJyqqqqkpKSnp6efn5+tra2TtCY9AAACRklEQVR4nO3Z626CMABAYWhLef83HhdRaEzUWTw0OSf7sS4ZfCiWrutyaLium75aTj6ZfDL5ZPLJ5JPJJ5NPJp9MPpl8Mvlk8snkk8knk08mn0w+mXwy+WTyyeSTySeTTyafTD6ZfDL5ZPLJ5JPJJ5NPJp9MPpl8Mvlk8snkk8knk08mn0w+mXwy+WTyyeSTySeTTyafTD6ZfDL55RHTVDk+61Wqfdwc+6VhO/6wjuM5F1D5qKnfGpcjh/H+g/Tqd/9TXX7u+4N/p+/7XPVUa3X5e+18/8T9eKx6qrWq/NQfCuE4PuH2qcqPBbe4nOH1ET7tl/xY81xr8h81zi8/uvk4vvpH9zjTxGIivfzEeXz5w/Exdspjt/KiYSi0u+s5Ydqsv2RLt/tlvC0R8jY+ZclzwoI5DTEOu/VNnsclPox1pqGa/Mn5ou0qpvfkSvw0w4pZ/2mrP1V7CFThz5zcvaFf0cP9u6+rxk9v8+PjQr7u9/y4ex++7sf84jb6Ovny/5385vlh2dVplt+FeROzXf6S/I+TvyX/437Mf2yetMq/7fm3yr+9/u3yl78WG+bPi6TL8cfX+vtWW7rePk/Y7+g8v5bqW+T+V51MPpl8Mvlk8snkk8knk08mn0w+mXwy+WTyyeSTySeTTyafTD6ZfDL5ZPLJ5JPJJ5NPJp9MPpl8Mvlk8snkk8knk08mn0w+mXwy+WTyyeSTySeTTyafTD6ZfDL5ZPLJ5JPJJ5NPJp9MPpl8Mvlk8snkk8knk08mnyyHhuv+AMviDaCX0YnaAAAAAElFTkSuQmCC';
        }

        return `https://image.tmdb.org/t/p/w342${this.movie.poster_path}`;
      }
    },
    data() {
      return {
        highlighted: false,
        hidden: false
      };
    },
    methods: {
      select() {
        this.highlighted = true;
        this.$bus.$emit(MOVIE_SELECTED, this.movie);
      },
      close() {
        const { filters } = this.$bus;
        this.$bus.$emit(FILTERS_UPDATE, filters);
      }
    },
    mounted() {
      this.$bus.$on(MOVIE_SELECTED, (movie) => {
        this.highlighted = movie.imdb_id === this.movie.imdb_id;
        this.hidden = !this.highlighted;
      });
      this.$bus.$on(FILTERS_UPDATE, () => {
        this.hidden = false;
        this.highlighted = false;
      });
    }
  };
</script>

<style scoped lang="scss" ref="stylesheet/scss">
  @import '../scss/vars';

  .movie-container {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;

    @media screen and (min-width: 576px) {
      flex-wrap: nowrap;
    }

    a {
      opacity: 0.5;
      transition: opacity .4s;
      color: white;
      width: 200px;
      min-width: 200px;
      max-width: 200px;
      overflow: hidden;
      margin-bottom: 20px;
      position: relative;

      img {
        display: block;
        width: 100%;
      }

      .badge-year {
        position: absolute;
        top: 5px;
        right: 5px;
        display: none;
      }

      &:hover {
        opacity: 1;

        .badge-year {
          display: block;
        }
      }

      &:focus {
        opacity: 1;
        outline: none;
      }

      .title {
        display: inline-block;
        text-align: center;
        width: 200px;
        font-size: 0.9em;
        padding: 10px 5px;
      }
    }

    .details {
      width: 0;
      height: 0;
      overflow: hidden;
      padding: 0;
      opacity: 0;

      h3 small img {
        vertical-align: initial;
      }

      p.overview {
        text-align: justify;
        margin-bottom: 10px;
      }
    }

    &.highlighted {
      a {
        opacity: 1;

        img {
          border-top-left-radius: 5px;
          border-top-right-radius: 5px;
        }

        .title {
          background: $global-color-primary;
          border-bottom-left-radius: 3px;
          border-bottom-right-radius: 3px;
        }
      }

      .details {
        opacity: 1;
        transition: opacity .4s linear .75s;
        padding-right: 15px;
        width: auto;
        height: auto;

        @media screen and (min-width: 576px) {
          margin-left: 20px;
        }
      }
    }

    &.hidden {
      a {
        width: 0;
        min-width: 0;
        opacity: 0;
        pointer-events: none;
        transition: opacity .2s, width .6s linear .2s;
      }
    }
  }
</style>
