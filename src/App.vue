<template lang="pug">
  #app
    label
      span Search for a dog breed
      autocomplete(:search="search" :debounceTime='100')
    .columns
      div.dog-list-container
        div(v-if="currentInputValue")
          h2 Seach Results
          ul.dog-list.results
            li(v-for="(dog, i) in filteredDogs" :key="i")
              button(@click='handleClicked(dog)' :data-url="dog.imageUrl" :data-name="dog.name") Add as favorite
              div
                img.thumb.lazy(:src="dog.imageUrl")
                p {{dog.name}}
        div(v-else)
          h2 A Random Dog Picture
          ul.dog-list.results
            li(v-for="(dog, i) in randomDog" :key="i")
              button(@click='handleClicked(dog)' :data-url="dog.imageUrl" :data-name="dog.name") Add as favorite
              div
                img.thumb.lazy(:src="dog.imageUrl")
                p {{dog.name}}

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
import axios from 'axios'

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

  data: function(){
    return{
      dogs: [],
      tempDogs: [],
      checked: dogStorage.fetch(),
      currentInputValue: '',
      randomDog: [],
      loading: true
    }
  },
  mounted() {
    axios.get('https://dog.ceo/api/breeds/list/all')
      .then((dogs) => {
        for (var breed in dogs.data.message) {
          this.dogs.push({'name': breed, 'imageUrl': '' })
        }
        for (var dog in dogs.data.message) {
          let my_dog = dog
          axios.get(`https://dog.ceo/api/breed/${dog}/images/random`)
            .then(response=>{
              this.tempDogs.push({'name': my_dog, 'imageUrl': response.data.message, 'checked': false})
            })
            .finally(
              this.getRandomDog,
              this.dogs = this.tempDogs,
            )
        }
      })
  },

  computed: {
    filteredDogs(){
      return this.returnFullSearch(this.currentInputValue)
    } 
  },
  methods: {
    getRandomDog(){
      if (this.randomDog.length  < 1) {
        var item = this.dogs[Math.floor(Math.random() * this.dogs.length)];
        this.randomDog.push({name: item.name, imageUrl: item.imageUrl})
      }

    },
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
    }

  },
  watch: {
    checked: {
      deep: true,
      handler: dogStorage.save
    },
    loading: ()=>{
      this.getRandomDog()
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
    grid-template-columns: 33.33% 33.33% 33.33%;
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
</style>
