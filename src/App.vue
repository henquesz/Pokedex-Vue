<template>
  <v-app>
    <v-container>
      <v-row>
        <v-container>
          <v-img
            :src="require('../src/assets/pokedex.png')"
            class=""
            contain
            height="300"
          />
          <!-- <h1 color="white" class="mx-5">Procure pelos teus pokemons favoritos abaixo.</h1> -->
        </v-container>
      </v-row>
      <v-container>
        <v-text-field
          dark
          v-model="search"
          label="Pesquisar"
          placeholder="Bulbassaur"
          outlined
          color="red"
        ></v-text-field>
        <v-row>
          <v-col
            sm="4"
            md="3"
            lg="2"
            v-for="pokemon in filtered_pokemons"
            :key="pokemon.name"
          >
            <v-card @click="show_pokemon(get_id(pokemon))">
              <v-container>
                <v-row class="mx-0 d-flex justify-center">
                  <img
                    :src="`https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${get_id(
                      pokemon
                    )}.png`"
                    :alt="pokemon.name"
                    width="80%"
                  />
                </v-row>
                <h2 class="text-center">{{ get_name(pokemon) }}</h2>
                <h3 class="text-center">ID: {{ get_id(pokemon) }}</h3>
              </v-container>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
    </v-container>

    <v-dialog v-model="show_dialog" width="500">
      <v-card v-if="selected_pokemon" class="px-4">
        <v-container>
          <v-row class="d-flex align-center">
            <v-col sm="4" md="3" lg="2">
              <img
                :src="`https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${selected_pokemon.id}.png`"
                :alt="selected_pokemon.name"
                width="80%"
              />
            </v-col>
            <v-col cols="8">
              <h1>
                {{ get_name(selected_pokemon) }}
              </h1>
              <v-chip
                label
                v-for="type in selected_pokemon.types"
                :key="type.slot"
                class="mr-2"
                >{{ type.type.name }}
              </v-chip>
              <v-divider class="my-4"></v-divider>
              <v-chip label>
                Altura: {{ selected_pokemon.height * 2.54 }}cm</v-chip
              >
              <v-chip label class="ml-2">
                Peso:
                {{
                  (selected_pokemon.weight * 0.45359237).toFixed(0)
                }}Kg</v-chip
              >
            </v-col>
          </v-row>
          <h2>moves</h2>
          <v-simple-table>
            <template>
              <thead>
                <tr>
                  <th class="text-left">Level</th>
                  <th class="text-left">Name</th>
                </tr>
              </thead>
              <tbody>
                <tr
                  v-for="item in filter_moves(selected_pokemon)"
                  :key="item.move.name"
                >
                  <td>{{ get_move_level(item) }}</td>
                  <td>{{ item.move.name }}</td>
                </tr>
              </tbody>
            </template>
          </v-simple-table>
        </v-container>
      </v-card>
    </v-dialog>
    <v-container>
      <v-row justify="center">
        <v-col cols="2">
          <v-container class="d-flex align-center">
            <v-btn
              elevation="5"
              v-on:click="prevPage"
              class="d-flex align-center"
              >prev</v-btn
            >
            -
            <v-btn
              elevation="2"
              v-on:click="nextPage"
              class="d-flex align-center"
              >next</v-btn
            >
          </v-container>
        </v-col>
      </v-row>
    </v-container>
  </v-app>
</template>


<script>
// import HelloWorld from './components/HelloWorld';
import axios from "axios";
export default {
  name: "App",

  components: {},

  data() {
    return {
      url: "https://poke-14cytrf7r-henquesz.vercel.app/api/poke",
      pokemons: [],
      pages: [],
      search: "",
      show_dialog: false,
      selected_pokemon: null,
    };
  },

  mounted() {
    //Local de url para substituir por rota da ApiRest desenvolvida. (https://github.com/henquesz/ApiRestSX)
    axios.get(this.url).then((response) => {
      this.pokemons = response.data.pokemon.results;
      this.pages = response.data.pokemon;
      console.log(this.pages);
    });
  },
  methods: {
    nextPage() {
      try {
        this.url = this.pages.next;
        console.log(this.url);
        axios.get(this.url).then((response) => {
          this.pokemons = response.data.results;
          this.pages = response.data;
        });
      } catch (error) {
        console.log("erro next page");
      }
    },
    prevPage() {
      try {
        this.url = this.pages.previous;
        console.log(this.pages.previous);

        axios.get(this.url).then((response) => {
          this.pokemons = response.data.results;
          this.pages = response.data;
        });
      } catch (error) {
        console.log("erro prev page");
      }
    },
    get_id(pokemon) {
      return Number(pokemon.url.split("/")[6]);
    },
    get_name(pokemon) {
      return pokemon.name.charAt(0).toUpperCase() + pokemon.name.slice(1);
    },
    show_pokemon(id) {
      axios.get(`https://pokeapi.co/api/v2/pokemon/${id}`).then((response) => {
        this.selected_pokemon = response.data;
        this.show_dialog = !this.show_dialog;
      });
    },
    get_move_level(move) {
      for (let version of move.version_group_details) {
        if (
          version.version_group.name == "sword-shield" &&
          version.move_learn_method.name == "level-up"
        ) {
          return version.level_learned_at;
        }
      }
      return 0;
    },
    filter_moves(pokemon) {
      return pokemon.moves.filter((item) => {
        let include = false;
        for (let version of item.version_group_details) {
          if (
            version.version_group.name == "sword-shield" &&
            version.move_learn_method.name == "level-up"
          ) {
            include = true;
          }
        }
        return include;
      });
    },
  },
  computed: {
    filtered_pokemons() {
      return this.pokemons.filter((item) => {
        return item.name.includes(this.search);
      });
    },
    // filtered_species() {
    //   return this.pokemons.filter((item) => {
    //     return item.type.type.name.includes(this.search);
    //   });
    // },
  },
};
</script>

<style>
#app {
  background: rgb(0, 43, 48);
  background: radial-gradient(
      circle,
      rgba(0, 43, 48, 1) 0%,
      rgba(22, 22, 22, 1) 100%
    )
    no-repeat center center fixed !important;
  -webkit-background-size: cover;
  -moz-background-size: cover;
  -o-background-size: cover;
  background-size: cover !important;
  background-position: center;
  min-height: 100vh;
}
</style>