<!-- This card is BCard but without buttons, will be used as a component in other pages -->
<script>

export default {
  name: 'Card',
  components: { },
  props: {
    displayData: []
  },
  data() {
    return {
      isOverflowing: Boolean
    }
  },
  mounted() {
    const card = this.$refs.card
    console.log(card.scrollHeight)
    console.log(card.clientHeight)
    this.isOverflowing = card.scrollHeight > card.clientHeight
  },
  computed: {
    numberOfMatches() {
      if (this.displayData.ingredientMatches !== undefined) {
        return `Food items in your kitchen: ${this.displayData.ingredientMatches}`
      }
      return ''
    }
  },
  methods: {
    capitalizeFirst(toBeCapitalized) {
      return toBeCapitalized.charAt(0).toUpperCase() + toBeCapitalized.slice(1)
    },
    formatField(name, value) {
      if (name === 'expiryDate' && value) {
        const date = new Date(value)
        const options = { day: 'numeric', month: 'short', year: 'numeric' }
        return 'Expiry date: ' + date.toLocaleDateString(undefined, options)
      } else {
        return value
      }
    }
  }
}
</script>

<template>
  <b-card class="custom-rounded-card shadow-lg fixedHeight" max-width="374">
    <!-- Header slot -->
    <template #header>
      <b-row align-h="between">
        <div class="mt-2 ml-2">{{ displayData.name }}</div>
      </b-row>
    </template>
    <template #default>
      <p class="expandText" v-if="isOverflowing">Click to show more</p>
      <p><i>{{numberOfMatches}} </i></p>
      <b-row v-for="(field, name) in displayData" :key="name" align-h="center">
        <!-- Array slot -->
        <!-- Non array slot -->
        <b-col v-if="!Array.isArray(field)">
          <b-card-text v-if="name !== 'name' && name !=='_id'&& name !=='reminder' && name !=='expired' && name !=='ingredientMatches'">{{formatField(name, field)}}</b-card-text>
        </b-col>
      </b-row>
    </template>
  </b-card>
</template>
