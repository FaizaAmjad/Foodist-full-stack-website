<template>
  <b-container>
    <b-row >
      <b-col class="mt-2 mb-2 mx-1">
        <b-form-spinbutton class="shadow-lg" v-model="scaleFactor" inline align-h="start" min="1"/>
      </b-col>
      <b-col align-h="end">
        <b-button pill class="mt-2 mb-2 mx-1 buttonStyle" @click="$router.push({ name: 'create-recipe' })" variant="primary">
          <span class="plus-sign">+</span> Add Recipe
        </b-button>
      </b-col>
    </b-row>
    <b-row class="border-0">
        <b-col v-for="(recipe, index) in recipes" :key="index" cols="12" md="4">
          <div class="rounded">
            <bcard :scaleFactor="scaleFactor" class="mb-2 highlightCard" @modalEvent="cardModal(index)" @showDeleteModal="showDeleteModal" @editEvent="editRecipe" :displayData="recipe" />
          </div>
        </b-col>
    </b-row>

    <b-modal  hide-header hide-footer v-model="showCardModal" tall size="md" body-class="m-0 p-0" content-class="custom-rounded-card">
      <bcardrec :scaleFactor="scaleFactor" @closeCardModal="closeCardModal" :displayData="recipes[cardDisplay]" />
    </b-modal>

    <b-modal v-model="showModal" title="Confirm Delete" hide-footer>
      <div>
        <p>Are you sure you want to delete this recipe?</p>
      </div>
      <b-row align-h="end" class="justify-content-around">
        <b-button variant="danger" @click="confirmDelete">Delete</b-button>
        <b-button variant="secondary" @click="cancelDelete">Cancel</b-button>
      </b-row>
    </b-modal>
    <spinner v-if="isLoading"></spinner>
  </b-container>
</template>

<script>
import { Api } from '@/Api'
import BCard from '@/components/BCard.vue'
import BCardrec from '@/components/BCardRec.vue'
import Spinner from '@/components/Spinner.vue'

export default {
  name: 'recipes',
  props: {
    currentUser: {
      type: String
    }
  },
  components: {
    bcard: BCard,
    bcardrec: BCardrec,
    spinner: Spinner
  },
  data() {
    return {
      recipes: [],
      selectedRecipe: null,
      showModal: false, // To control the visibility of the modal
      selectedItem: null,
      errorMessage: '',
      isLoading: true,
      showCardModal: false,
      cardDisplay: -1,
      scaleFactor: 1
    }
  },
  methods: {
    getRecipes() {
      const currentUser = this.currentUser
      Api.get('/v1/users/' + currentUser + '/recipes')
        .then((response) => {
          if (response.status === 200) {
            this.recipes = response.data
            this.isLoading = false
          }
        }).catch((error) => {
          if (error.response) {
            if ((error.response.status === 404)) {
              this.errorMessage = 'Not found'
            }
          } else if (error.request) {
            this.$router.push('/error')
          } else {
            this.errorMessage = 'Server error'
          }
        })
    },
    editRecipe(event) {
      const recipe = this.recipes.find((recipe) => recipe._id === event)
      this.$emit('editEvent', recipe)
      this.$router.push({ name: 'edit-recipe' })
    },
    removeList(event) {
      const recipe = this.recipes.find((recipe) => recipe._id === event)
      Api.delete('v1/users/' + this.currentUser + '/recipes/' + recipe.name)
        .then((res) => {
          this.recipes = res.data.recipes
        })
    },
    showDeleteModal(item) {
      // Set the selected item and show the modal
      this.selectedItem = item
      this.showModal = true
    },
    cardModal(index) {
      this.showCardModal = true
      this.cardDisplay = index
    },
    confirmDelete() {
      this.removeList(this.selectedItem)
      this.hideModal()
    },
    cancelDelete() {
      // Handle cancel deletion logic here
      // Hide the modal without performing any deletion
      this.hideModal()
    },
    hideModal() {
      // Hide the modal and clear the selected item
      this.showModal = false
      this.selectedItem = null
    },
    closeCardModal() {
      this.showCardModal = false
    }
  },
  created() {
    this.getRecipes()
  }
}
</script>
