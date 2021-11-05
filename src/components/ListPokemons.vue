<template lang="pug">
    b-container(fluid style="align:center")
      h1.my-3 Pokemons
      div(style="float:right;margin-top:-3px;margin-right:20px") 
        b-button(variant="outline-primary" @click="switchView();") Switch View
      <p v-if="isLoading">Loading...</p>
      b-row
       b-col
        <div v-if="view==1" class="overflow-auto" >
            b-pagination(
                v-model="currentPage"
                :total-rows="rows"
                :per-page="perPage"
                aria-controls="my-table"
                size="md"
            )
            b-table(
            id="my-table"
            :fields="pokemonColumns"
            :items="pokemons"
            :per-page="perPage"
            :current-page="currentPage"
            bordered
            hover=true
            responsive="md"
            )
              template(v-slot:cell(name)="data")
                span <b>{{ data.value}}</b>
              template(v-slot:cell(url)="data")
                img(:src="data.value" :alt="data.value")         
              template(v-slot:cell(height)="data")
                span {{ data.value}}m
              template(v-slot:cell(weight)="data")
                span {{ data.value }}kg
        </div>
        <div v-if="view==2" class="viewtwo">
          div(class="list")
            article(v-for="pokemon in paginatedPokemons")
              <img :src="pokemon.url" width="96" height="96" alt="">
              <h5>{{ pokemon.name }}</h5>
              div Height: {{ pokemon.height }}m <br> Weight: {{ pokemon.weight }}kg
            <button @click="loadMore" v-if="viewtwocurrentPage * viewtwomaxPerPage < pokemons.length">Load More</button>
          </div>
        </div>
</template>

<script>
export default {
  components: {
  },
  data() {
    return {
      imageUrl: 'https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/',
      nextUrl:'',
      prevUrl:'',
      view:1,
      pokemons: [
          {
              name:'',
              url:'',
              height:'',
              weight:''
          }
      ],
      isLoading: false,
      error: null,
      perPage: 30,
      currentPage: 1,
      viewtwocurrentPage: 1,
      viewtwomaxPerPage: 15,
      viewtwoshowReadMore: true
    };
  },
  computed: {
    pokemonColumns() {
      return [
        { label: "Name", key: "name", thStyle: 'width: 10%' , tdClass: 'small', sortable: false },
        { label: "Image", key: "url", thStyle: 'width: 15%', tdClass: 'small', sortable: false, image: true },
        { label: "Height", key: "height", thStyle: 'width: 15%', tdClass: 'small', sortable: false },
        { label: "Weight", key: "weight", thStyle: 'width: 20%', tdClass: 'small', sortable: false },
      ];
    },
    rows() {
      return this.pokemons.length
    },
    pageCount() {
      return Math.ceil(this.pokemons.length / this.viewtwomaxPerPage);
    },
    pageOffest() {
      return this.viewtwomaxPerPage * this.viewtwocurrentPage;
    },
    paginatedPokemons() {
      return this.pokemons.slice(0, this.viewtwocurrentPage * this.viewtwomaxPerPage);
    }
    },
  methods: {
    loadMore() {
      this.viewtwocurrentPage += 1;
    },
    switchView(){
      if(this.view==1)
        this.view=2;
      else
        this.view=1;
    },
    loadPokemons() {
      this.isLoading = true;
      this.error = null;
      fetch('https://pokeapi.co/api/v2/pokemon?limit=150&offset=0')
        .then((resp) => {
            if(resp.status === 200)
              return resp.json();
          })
          .then((data) => {
            this.isLoading = false;
            this.nextUrl = data.next;
            let pheight = null;
            let pweight = null;
            let iurlhtml = '';
            data.results.forEach(pokemon => {
              pokemon.id = pokemon.url.split('/')
                .filter(function(part) { return !!part }).pop();
             //this.pokemons.push(pokemon);
              fetch(pokemon.url)
                .then((resp) => {
                    if(resp.status === 200)
                    return resp.json();
                })
                .then((data) => {
                    pheight = data.height/10;
                    pweight = data.weight/10;
                    //console.log(iurlhtml);
                    iurlhtml = this.imageUrl + pokemon.id +".png";
                    this.pokemons.push({name:pokemon.name,url:iurlhtml,height:pheight,weight:pweight});
                })
                .catch((error) => {
                console.log(error);
                this.isLoading = false;
                this.error = 'Failed to fetch details - please try again later.';
                });
                this.pokemons.shift();
            });
          })
        .catch((error) => {
          console.log(error);
          this.isLoading = false;
          this.error = 'Failed to fetch data - please try again later.';
        });
    },
  },
  mounted() {
    this.loadPokemons();
  },
};
</script>

<style scoped>
.viewtwo {
    display:flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
    padding:10px;
    width: calc(100% - 20px);
    min-height: calc(100vh - 20px);
    font-family: 'Acme', arial;
    font-size: 0.9rem;
    font-weight: normal;
}
.list {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
    grid-gap: 10px;
    width: 70%;
    max-width: 790px;
}
article {
    height: 200px;
    background-color: #efefef;
    text-align: center;
    text-transform: capitalize;
    border-radius: 5px;
    cursor: pointer;
    box-shadow: 0 15px 30px rgba(0,0,0,.2),
                0 10px 10px rgba(0,0,0,.2);
    
}
#scroll-trigger {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%;
    height: 150px;
    font-size: 2rem;
    color: #efefef;
}
</style>