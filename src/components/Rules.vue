<template>
  <div>
    <section v-if="showRules" class="fixed bg-gray-200 rounded-r-xl top-1/2 -translate-y-1/2 max-w-lg z-40 text-left">
      <div class="max-h-[65vh] overflow-auto py-4 px-6 relative">
        <button @click="toggleRules()" class="btn hover:scale-110 absolute right-10 top-10 border-2 border-black">X</button>
        <div class="sm:px-2">
          <h1>Rules</h1>
          <h2>Game Rules</h2>
          <ul>
            <li>The Dealer always has an hidden Card, which will be revealed, when the round is over.</li>
            <li>The Dealer draws before the player.</li>
            <li>The Dealer has higher Priority, means when both player and dealer's card values are over 21. The Dealer
              still wins</li>
          </ul>
          <h2>Card Values</h2>
          <ul>
            <li>Cards 2-10 are worth the value of the number on the face of the card.
            </li>
            <li>
              Face cards are worth 10, except for the Ace, which is worth 1 or 11.
            </li>
          </ul>
          <h2>21 Card Game</h2>
          <ul>
            <li>The highest value one can reach is 21.</li>
            <li>The one which oversteps 21, has lost.</li>
            <li>Wining with 21 is called "Blackjack".</li>
          </ul>
        </div>
      </div>
    </section>
    <div v-if="!showRules" class="absolute top-1/2 -translate-y-1/2 z-40">
      <button @click="toggleRules()" class="btn">
        <p class="py-9 text-3xl">
          >
        </p>
      </button>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue";

export default defineComponent({
  data() {
    let showRules: boolean = false;

    return { showRules }
  },
  setup() { },
  mounted() { },
  methods: {
    /**
     * shows or hides the rules
     */
    async toggleRules(): Promise<void> {
      this.showRules = !this.showRules;
    },
  },
});
</script>


<style scoped>
h1,
h2,
h3,
h4,
h5,
h6 {
  margin-top: .75rem;
  margin-bottom: .75rem;
}
</style>


