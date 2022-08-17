<template>
  <!-- popup -->
  <div v-if="showPopup">
    <section class="h-screen w-screen bg-gray-700 fixed top-0 opacity-75 z-30">
      <div @click="reset()" class="absolute top-0 bottom-0 left-0 right-0 z-40"></div>
    </section>
    <div class="w-full md:w-1/2 fixed top-1/2 left-1/2 -translate-y-1/2 -translate-x-1/2 z-40">
      <div class="flex h-full">
        <div class="flex flex-col m-auto bg-gray-300 p-8 rounded-2xl shadow-2xl">
          <h3>{{ result }}</h3>
          <button @click="reset()" class="btn mt-6 border-2 border-black tracking-widest">O.K</button>
        </div>
      </div>
    </div>
  </div>

  <!-- section -->
  <section class="w-full xl:w-3/4 mx-auto px-2 py-12">
    <div class="relative border-2 border-black">
      <section v-if="restructure" class="restructure absolute w-full h-full z-30">
        <p class="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 text-3xl text-white">
          please wait a moment
        </p>
      </section>
      <div class="lg:flex items-center justify-around">
        <!-- dealer -->
        <div class="mt-4 sm:m-4">
          <h2>Dealer: <span v-if="!canHit">{{ dealerPts }}</span><span v-else>{{ dealerPts - dealerHiddenValue }}</span>
          </h2>
          <div class="sm:flex">
            <img :src="dealerHiddenCardImgSrc" alt="" height="175" width="125" class="p-1 mx-auto inline-block" />
            <img v-for="i in dealerCardImgSrc.length" :key="i" :src="dealerCardImgSrc[i - 1]" alt="" height="175"
              width="125" class="p-1 mx-auto inline-block" />
          </div>
        </div>

        <!-- Player -->
        <div class="mt-4 sm:m-4">
          <h2>Player: {{ playerPts }}</h2>
          <div class="sm:flex">
            <img v-for="i in playerCardImgSrc.length" :key="i" :src="playerCardImgSrc[i - 1]" alt="" height="175"
              width="125" class="p-1 mx-auto inline-block">
          </div>
        </div>
      </div>
      <!-- result -->
      <div class="mt-4">
        <button @click="play(1);" :disabled="!canHit" :class="!canHit ? 'disabled' : ''"
          class="btn w-full sm:w-1/4 mt-2 sm:m-2">Hit</button>
        <button @click="play(2);" :disabled="!canHit" :class="!canHit ? 'disabled' : ''"
          class="btn w-full sm:w-1/4 mt-2 sm:m-2">Double</button>
        <button @click="stay()" class="btn w-full sm:w-1/4 mt-2 sm:m-2">Stay</button>
      </div>
    </div>
  </section>
</template>

<script lang="ts">
/* https://www.youtube.com/watch?v=bMYCWccL-3U */
import { defineComponent } from "vue";

export default defineComponent({
  data() {
    let dealerPts: number = 0,
      playerPts: number = 0,
      dealerAceCount: number = 0,
      playerAceCount: number = 0,
      dealerHiddenValue: number = 0,
      playerCardsOnField: number = 0;
    let cardImgTopSrc: string = "/src/assets/cards/",
      dealerHiddenCardImgSrc: string = "/src/assets/cards/BACK.png",
      result: string = "";
    let dealerCardImgSrc: string[] = [],
      playerCardImgSrc: string[] = [],
      deck: string[] = [];
    let hidden: any = "";
    let canHit: boolean = true,
      showPopup: boolean = false,
      restructure: boolean = false;

    return {
      dealerPts,
      playerPts,
      dealerAceCount,
      playerAceCount,
      dealerHiddenValue,
      playerCardsOnField,
      cardImgTopSrc,
      dealerHiddenCardImgSrc,
      result,
      dealerCardImgSrc,
      playerCardImgSrc,
      deck,
      hidden,
      canHit,
      showPopup,
      restructure,
    }
  },
  setup() { },
  mounted() {
    this.buildDeck();
    this.shuffleDeck();
    this.startGame();
  },
  methods: {
    /**
     * filles the player's deck with all cards
     */
    buildDeck() {
      const cardValues = ["A", "2", "3", "4", "5", "6", "7", "8", "9", "10", "J", "Q", "K"];
      const categories = ["C", "D", "H", "S"];

      for (let i = 0; i < categories.length; i++) {
        for (let j = 0; j < cardValues.length; j++) {
          this.deck.push(cardValues[j] + "-" + categories[i]); // A-C, A-D,...
        }
      }
    },

    /**
     * Shuffles all card in the players deck randomly
     */
    shuffleDeck() {
      for (let i = 0; i < this.deck.length; i++) {
        let j = Math.round(Math.random() * this.deck.length);
        let temp = this.deck[i];
        this.deck[i] = this.deck[j];
        this.deck[j] = temp;
      }
    },

    startGame() {
      let i: number = 0;
      let card: string | undefined = "";

      this.hidden = this.deck.pop();

      if (this.hidden == undefined) { // prevents site crash, when game resets
        console.error("hidden is undefined", this.hidden);
        // deck MUST be cleard, or infinite loop will trigge
        this.deck = []; // clear deck
        this.buildDeck(); // create new
        this.shuffleDeck();
        this.startGame(); // restart
      }

      // reckon with the hidden value right from the start
      this.dealerHiddenValue = this.getValue(this.hidden);
      this.dealerPts = this.getValue(this.hidden);
      this.dealerAceCount = this.checkAce(this.hidden);

      while (this.dealerPts < 17) {

        do {
          card = this.deck.pop(); // cut first value
        } while (card == undefined);  // prevents loading crash

        this.dealerCardImgSrc[i] = this.cardImgTopSrc + card + ".png";  // add cardSrc
        this.dealerPts += this.getValue(card);  // adds value of card to dealers points
        this.dealerAceCount = this.checkAce(card);  // check if aces exists
        i++;
      }

      for (i = this.playerCardsOnField; i < 2; i++) { // player begins with two cards

        do {
          card = this.deck.pop(); // cut first value
        } while (card == undefined);  // prevents loading crash

        this.playerCardImgSrc[i] = this.cardImgTopSrc + card + ".png";  // add cardSrc
        this.playerPts += this.getValue(card);  // adds value of card to dealers points
        this.playerAceCount = this.checkAce(card);  // check if aces exists
        this.playerCardsOnField = i + 1;
      }
    },

    getValue(card: string) {
      let data = card.split("-");
      let value = data[0];

      switch (value) {
        case "A":
          return 11;
        case "J":
        case "Q":
        case "K":
          return 10;
      }

      return parseInt(value);
    },

    checkAce(card: string) {
      if (card[0] == "A") {
        return 1;
      }
      return 0;
    },

    reduceAce(playerPts: number, playerAceCount: number) {
      while (playerPts > 21 && playerAceCount > 0) {
        playerPts -= 10;
        playerAceCount -= 1;
      }

      return playerPts;
    },

    play(mult: number) {
      if (!this.canHit) {
        alert('You cannot play anymore.');
        return 0;
      }

      let card: string | undefined = this.deck.pop(); // cut first value

      if (card == undefined) {
        console.error("card is undefined", card);
        return 0;
      }

      this.playerCardImgSrc[this.playerCardsOnField++] = this.cardImgTopSrc + card + ".png";  // add cardSrc
      this.playerPts += this.getValue(card) * mult;  // adds value of card to dealers points
      this.playerAceCount = this.checkAce(card) * mult;  // check if aces exists

      if (this.reduceAce(this.playerPts, this.playerAceCount) > 21 || mult == 2) {
        this.canHit = false;
      }
    },

    stay() {
      this.dealerPts = this.reduceAce(this.dealerPts, this.dealerAceCount);
      this.playerPts = this.reduceAce(this.playerPts, this.playerAceCount);

      this.dealerHiddenCardImgSrc = this.cardImgTopSrc + this.hidden + ".png"; // hidden img

      let msg: string = "";
      if (this.playerPts > 21 || this.playerPts < this.dealerPts) {  // even if dealer is also over 21, you still lose
        msg = "You Lose!";
      } else if (this.dealerPts > 21 || this.playerPts > this.dealerPts) {
        msg = "You Win!";
      } else if (this.playerPts == this.dealerPts) {
        msg = "Tie!";
      } else {
        console.error("Unknown Error");
      }

      this.result = msg;

      this.togglePopup();
    },

    async togglePopup() {
      this.showPopup = !this.showPopup;
    },

    reset() {
      this.dealerPts = 0;
      this.playerPts = 0;
      this.dealerAceCount = 0;
      this.playerAceCount = 0;
      this.dealerHiddenValue = 0;
      this.playerCardsOnField = 0;
      this.dealerHiddenCardImgSrc = "/src/assets/cards/BACK.png";
      this.dealerCardImgSrc = [];
      this.playerCardImgSrc = [];
      this.hidden = "";
      this.canHit = true;
      this.restructure = true;

      this.shuffleDeck();
      this.startGame();

      this.togglePopup();

      setTimeout(() => {
        this.restructure = false;
      }, 300);  // wait, for new cards
    },
  },
});
</script>

<style scoped>
.restructure {
  background-color: #818181;
  opacity: .8;
}
</style>
