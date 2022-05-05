<template lang="pug">
h1 Dog Picture Favoriter™
label
  label Search for a dog breed
    input(list="dog_list" name="selected_dog" id="selected_dog" v-model="currentInputValue" v-on:input="getDogImage")
  datalist(id="dog_list")
    option(v-for="(dog, i) in dogs" :value="dog.name") {{dog.name}}
.columns
  div.dog-list-container
    div(v-if="currentInputValue")
      h2 Seach Results
      ul.dog-list.results
        li(v-for="(dog, i) in searchResults" :key="i")
          button(@click='handleClicked(dog)' :data-url="dog.imageUrl" :data-name="dog.name") Add as favorite
          div
            img.thumb.lazy(:src="dog.imageUrl")
            p {{dog.name}}
    div(v-else)
      h2 A Random Dog Picture
      ul.dog-list.results(v-if="randomDog")
        li
          button(@click='handleClicked(randomDog)' :data-url="randomDog.imageUrl" :data-name="randomDog.name") Add as favorite
          div
            img.thumb.lazy(:src="randomDog.imageUrl")
            p {{randomDog.name}}

  div.dog-list-container
    h2 Favorites
    ul.dog-list.favorites
      li(v-for="(dog, i) in checked" :key="i") 
        button(@click='removeDog(i)') Remove from favorites
        div
          img.thumb.lazy(:src="dog.imageUrl")
          p {{dog.name}}


</template>

<script>
let DOGSTORAGE_KEY = 'kpmcguire-dog-storage'

var dogStorage = {
  fetch: function(){
    return JSON.parse(localStorage.getItem(DOGSTORAGE_KEY) || '[]')
  },
  save: function (checked){
    localStorage.setItem(DOGSTORAGE_KEY, JSON.stringify(checked))
  }
}



export default {
  name: 'App',

  data(){
    return{
      dogs: [],
      tempDogs: [],
      checked: dogStorage.fetch(),
      currentInputValue: '',
      randomDog: [],
      loading: true,
      searchResults: []
    }
  },
  async created() {
    let response = await fetch('https://dog.ceo/api/breeds/list/all')
    
    let dogs = await response.json()
    
    for (var breed in dogs.message) {
      this.dogs.push({'name': breed, 'imageUrl': '' })
    }

    this.randomDog = this.dogs[Math.floor(Math.random()*this.dogs.length)];
  },
  
  computed: {
    filteredDogs(){
      return this.returnFullSearch(this.currentInputValue)
    },

  },
  methods: {
    search(input) {
      this.currentInputValue = input
      if (input.length < 1) { return [] }
      return this.dogs.filter(dog => {
        return dog.name.toLowerCase()
          .startsWith(input.toLowerCase())
      })
      .map(function(obj) {
        return obj.name;
      });
    },
    returnFullSearch(input) {
      if (input.length < 1) { return [] }
      return this.dogs.filter(dog => {
        return dog.name.toLowerCase()
          .startsWith(input.toLowerCase())
      })
    },
    handleSubmit(result){
      this.checked.push(result)
    },
    handleClicked(target){
      this.checked.push({'name': target.name, 'imageUrl': target.imageUrl})
    },
    getResultValue(result) {
      return result.name
    },
    removeDog(dog) {
      this.checked.splice(dog, 1);
    },
    debounce(fn, time) {
      let timeoutId;
        return (...args) => {
          clearTimeout(timeoutId);
          timeoutId = setTimeout(fn, time, ...args);
        };
    },
    getDogImage() {
      
      let fn = this.debounce(()=>{
        this.searchResults = []
        
        
        this.filteredDogs.forEach((dog)=>{
          let breed = JSON.parse(JSON.stringify(dog.name))
          fetch(`https://dog.ceo/api/breed/${breed}/images/random`)
            .then(response=>response.json())
            .then((url)=>
            {
              
              this.searchResults.push({name: breed, imageUrl: url.message})
              
              let uniqueSearchResults = [...new Set(this.searchResults)];
              
              this.searchResults = uniqueSearchResults;
              
            }
            )        
        })
      }, 250);
      
      fn()
    }
    

  },
  watch: {
    checked: {
      deep: true,
      handler: dogStorage.save
    },
    randomDog() {
      let dog_name = this.randomDog.name
      fetch(`https://dog.ceo/api/breed/${dog_name}/images/random`)
        .then(response=>response.json())
        .then((url)=>this.randomDog.imageUrl = url.message)
    }
  }

}
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.thumb {
  max-width: 100%;
  height: auto;
}

.columns {
  margin-top: 1em;
  display: flex;
  background-color: silver;
  align-items: stretch;

  > div {
    flex: 1;
  }

  .autocomplete {
    text-align: left;
  }

  .dog-list-container {
    background-color: gainsboro;
    margin: 2em;
  }

  .dog-list {
    display: grid;
    grid-template-columns: 33.333% 33.333% 33.333%;
    list-style-type: none;

    li {
      padding: 1em;
      margin: 1em;
      border-radius: 1em;
      display: flex;
      flex-direction: column;
      align-items: center;
      background-color: whitesmoke;

      button {
        border: 0;
        background: rgb(76, 89, 109);
        margin-bottom: 0.5em;
        color: white;
        padding: 1em 2em;
        border-radius: 0.5em;
        cursor: pointer;

        &:hover, &:focus {
          background-color: rgb(42, 54, 71)
        }
      }
    }
  }
}

#selected_dog {
  display: block;
  width: 100%;
  max-width: 600px;
  margin: 0 auto;
  line-height: 2;
  border-radius: 0.125rem;
  border: 1px solid #767676;
}

</style>
