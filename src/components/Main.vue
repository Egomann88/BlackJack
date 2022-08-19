<template>
  <!-- Result - popup -->
  <div v-if="showPopup">
    <section class="h-screen w-screen bg-gray-700 fixed top-0 opacity-75 z-40">
      <div @click="reset()" class="absolute inset-0 z-40"></div>
    </section>
    <div class="w-full md:w-1/2 fixed top-1/2 left-1/2 -translate-y-1/2 -translate-x-1/2 z-40">
      <div class="flex h-full">
        <div :class="doubleBlackJck" class="flex flex-col m-auto bg-gray-300 p-8 rounded-2xl shadow-2xl">
          <h3>{{ result }}</h3>
          <button @click="reset()" class="btn mt-6 border-2 border-black tracking-widest">O.K</button>
        </div>
      </div>
    </div>
  </div>

  <!-- sidebar -->
  <Rules />

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
import Rules from "./Rules.vue";

export default defineComponent({
  components: {
    Rules,
  },
  data() {
    let dealerPts: number = 0,
      playerPts: number = 0,
      dealerAceCount: number = 0,
      playerAceCount: number = 0,
      dealerHiddenValue: number = 0,
      playerCardsOnField: number = 0;

    let cardImgTopSrc: string = "/src/assets/cards/",
      dealerHiddenCardImgSrc: string = "/src/assets/cards/BACK.png",
      result: string = "",
      doubleBlackJck: string = "";

    let dealerCardImgSrc: string[] = [],
      playerCardImgSrc: string[] = [],
      deck: string[] = [];

    let hidden: any = ""; // string | undefinded

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
      doubleBlackJck,
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
     * filles the deck with all cards
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

    /**
     * declaires all cards of the dealer (pulls cards until he reaches a value over equal 17 [with hidden cards])
     * @func buildDeck is called to rebuild deck, if it wasn't created properly
     * @func shuffleDeck shuffels the deck new
     * @func startGame restarts the function, if something was misdone
     * @func getValue adds the Value of the card to the max points
     * @func checkAce adds if an ace was drawn
     */
    startGame() {
      let i: number = 0;
      let card: string | undefined = "";

      this.hidden = this.deck.pop();

      if (this.hidden == undefined) { // prevents site crash, when game resets
        console.error("fatal error: hidden is undefined", this.hidden);
        this.reset(); // restart
      }

      // reckon with the hidden value right from the start
      this.dealerHiddenValue += this.getValue(this.hidden);
      this.dealerPts += this.getValue(this.hidden);
      this.dealerAceCount += this.checkAce(this.hidden);

      while (this.dealerPts < 17) {

        do {
          card = this.deck.pop(); // cut last value
        } while (card == undefined);  // prevents loading crash

        this.dealerCardImgSrc[i] = this.cardImgTopSrc + card + ".png";  // add cardSrc
        this.dealerPts += this.getValue(card);  // adds value of card to dealers points
        this.dealerAceCount += this.checkAce(card);  // check if aces exists
        i++;
      }

      for (i = this.playerCardsOnField; i < 2; i++) { // player begins with two cards

        do {
          card = this.deck.pop(); // cut last value
        } while (card == undefined);  // prevents loading crash

        this.playerCardImgSrc[i] = this.cardImgTopSrc + card + ".png";  // add cardSrc
        this.playerPts += this.getValue(card);  // adds value of card to dealers points
        this.playerAceCount += this.checkAce(card);  // check if aces exists
        this.playerCardsOnField = i + 1;
      }
    },

    /**
     * checks the value of the card and returns it
     * @param {string} card full name of the card
     * @returns number -> value of card
     */
    getValue(card: string): number {
      switch (card[0]) {  // is first diggit A, J, Q or K
        case "A":
          return 11;
        case "J":
        case "Q":
        case "K":
          return 10;
      }

      return parseInt(card[0]);
    },

    /**
     * Checks if the card is an ace, if yes returns an 1
     * @param card full name of card
     * @returns 1 if is an a ace / 0 when not
     */
    checkAce(card: string): number {

      if (card[0] == "A") { // A-D, A-S, K-D, 10-S,...
        return 1;
      }
      return 0;
    },

    /**
     * checks, if the playersPts are more than 21 and he has one or more aces
     * if so, the player loses 10 Pts (11 - 10) for the alternate value of ace
     * 
     * @param playerPts all points of the player 
     * @param playerAceCount all the aces the player has
     * @returns number -> playerPts 
     */
    reduceAce(playerPts: number, playerAceCount: number): number {
      while (playerPts > 21 && playerAceCount > 0) {
        playerPts -= 10;
        playerAceCount -= 1;
      }

      return playerPts;
    },

    /**
     * As first, it checks if the player is still able to draw cards.
     * Then it draws a new Card
     * Deaktivates the Hit Button is nessesary 
     * 
     * @func play restart the function 
     * @func adds the Value of the card to the max points
     * @func checkAce adds if an ace was drawn
     * @func reduceAce 
     * @param { number } mult
     * @returns number
     */
    play(mult: number): number {
      if (!this.canHit) {
        alert('You cannot play anymore.');
        return 0;
      }

      let card: string | undefined = this.deck.pop(); // cut last value

      if (card == undefined) {
        console.error("card is undefined", card);
        this.play(mult);  // restart func
        return 0;
      }

      this.playerCardImgSrc[this.playerCardsOnField++] = this.cardImgTopSrc + card + ".png";  // add cardSrc
      this.playerPts += this.getValue(card) * mult;  // adds value of card to dealers points
      this.playerAceCount = this.checkAce(card) * mult;  // check if aces exists

      if (this.playerPts > 21) {  // if playerPts is over 21, reduce the ace value (if possible)
        this.playerPts = this.reduceAce(this.playerPts, this.playerAceCount);
        this.playerAceCount = 0;
      }

      if (this.playerPts > 21 || mult == 2) {  // if playerpts over 21 or doubled 
        this.canHit = false;
      }
      return 1;
    },

    /**
     * Reduces Ace values if needed, reveals hidden card value and declares if player won or not
     * 
     * @func reduceAce reducesAce value if needed
     * @func togglePopup shows won / lose Popup
     */
    stay(): void {
      this.dealerPts > 21 ? this.reduceAce(this.dealerPts, this.dealerAceCount) : this.dealerPts; // reduces Points, if nessesary
      this.playerPts > 21 ? this.reduceAce(this.playerPts, this.playerAceCount) : this.playerPts; // reduces Points, if nessesary

      this.canHit = false;  // show true value of dealer
      this.dealerHiddenCardImgSrc = this.cardImgTopSrc + this.hidden + ".png"; // hidden img

      let msg: string = "";
      if ((this.playerPts == 21 || this.dealerPts == 21) && this.playerPts != this.dealerPts) {
        msg = "Blackjack! ";
        this.playerPts != 21 ? msg += "You Lose!" : msg += "You Win!";
      } else if (this.playerPts > 21) {  // even if dealer is also over 21, you still lose
        msg = "You Lose!";
      } else if (this.dealerPts > 21) {
        msg = "You Win!";
      } else if (this.playerPts == this.dealerPts) {
        if (this.playerPts == 21) {
          msg = "double Blackjack!";
          this.doubleBlackJck = "bg-gradient-to-r from-red-400 to-black text-white animation-scaleUp";
        } else {
          msg = "Tie!";
        }
      } else if (this.playerPts < this.dealerPts) {
        msg = "You Lose!";
      } else if (this.playerPts > this.dealerPts) {
        msg = "You Win!";
      } else {
        console.error("Unknown Error");
      }

      this.result = msg;

      this.togglePopup();
    },

    /**
     * Toggles won / lose Popup
     */
    async togglePopup() {
      this.showPopup = !this.showPopup;
    },

    /**
     * sets all variabels to default and restarts all functions
     * @func buildDeck refills the deck
     * @func shuffleDeck shuffle the deck
     * @func startGame gives dealer and player starter cards
     * @func togglePopup disables / hides  Popup
     */
    reset(): void {
      // sets variables to default
      this.dealerPts = 0;
      this.playerPts = 0;
      this.dealerAceCount = 0;
      this.playerAceCount = 0;
      this.dealerHiddenValue = 0;
      this.playerCardsOnField = 0;
      this.dealerHiddenCardImgSrc = "/src/assets/cards/BACK.png";
      this.doubleBlackJck = "";
      this.deck = []; // clear deck
      this.dealerCardImgSrc = [];
      this.playerCardImgSrc = [];
      this.hidden = "";
      this.canHit = true;
      this.restructure = true;  // show loading / restructure screen 

      this.buildDeck(); // refills the deck
      this.shuffleDeck(); // shuffle new
      this.startGame();

      this.togglePopup(); // disables / hides  Popup

      setTimeout(() => {
        this.restructure = false;  // show loading / restructure screen 
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

.animation-scaleUp {
  -webkit-animation: scaleUp 2.5s ease 0s infinite;
  -moz-animation: scaleUp 2.5s ease 0s infinite;
  -ms-animation: scaleUp 2.5s ease 0s infinite;
  animation: scaleUp 2.5s ease 0s infinite;
}

@-webkit-keyframes scaleUp {
  0% {
    transform: rotate(1.45);
    transform: scale(1.25);
  }

  50% {
    transform: rotate(1.45);
    transform: scale(2);
  }

  100% {
    transform: rotate(1.45);
    transform: scale(1.25);
  }
}

@keyframes scaleUp {
  0% {
    transform: rotate(1.45);
    transform: scale(1.25);
  }

  50% {
    transform: rotate(1.45);
    transform: scale(2);
  }

  100% {
    transform: rotate(1.45);
    transform: scale(1.25);
  }
}
</style>
