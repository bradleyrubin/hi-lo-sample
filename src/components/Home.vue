<template>
  <div style="display:flex">
    <app-scoreboard
      v-if="players"
      style="flex:1"
      :players="players"
      :currentPlayerIndex="currentPlayerIndex"/>
    <div style="flex:4">
      <div>
        <h4 v-if="isGameOver">
          Game Over, Winner is Player {{ winner.name }}
        </h4>
        <h5>Cards Remaining In Deck: {{ cardsRemaining }} </h5>
        <h5>Cards In Face-Up Pile: {{ cardsInFaceUpPile }} </h5>
      </div>
      <div v-if="activeCardURL">
        <img
          :src="activeCardURL"
          style="height:100px"/>
      </div>
      <div v-if="!isGameOver">
        <button
          :disabled="!passButtonActive"
          @click="pass()">
          Pass
        </button>
        <h5>Guess High or Low</h5>
        <button
          @click="highClicked()">
          Hi
        </button>
        <button
          @click="lowClicked()">
          Low
        </button>
      </div>
      <div v-if="!isGameOver">
        <h5>Player {{ currentPlayerName }}'s turn</h5>
        <h5>Current correct streak: {{ correctGuessesInARow }} </h5>
      </div>
      <div v-if="guess">
        You guessed {{ guess }}
      </div>
      <div v-if="resultDescription">
        <p> {{ resultDescription }} </p>
      </div>
    </div>
    <div style="flex:1">
      <button
        @click="resetGame()">
        Reset
      </button>
    </div>
  </div>
</template>

<script src="https://cdn.jsdelivr.net/npm/vue-resource@1.5.1"></script>
<script>

import _ from 'lodash';
import Scoreboard from './Scoreboard';

export default {
  components: {
    'app-scoreboard': Scoreboard,
  },
  name: 'Home',
  data() {
    return {
      deck: null,
      cardsRemaining: 0,
      cardDrawn: null,
      cardsInFaceUpPile: 0,
      previousCard: null,
      guess: null,
      resultDescription: null,
      players: [{
        name: 'A',
        points: 0,
      }, {
        name: 'B',
        points: 0,
      }],
      currentPlayerIndex: 0,
      correctGuessesInARow: 0,
      passButtonActive: false,
      isLoading: false,
    };
  },
  computed: {
    winner() {
      let winningPlayer = null;
      let isTie = false;
      this.players.forEach(player => {
        if (!winningPlayer) {
          winningPlayer = player;
        } else if (player.points < winningPlayer.points) {
          winningPlayer = player;
        } else if (player.points === winningPlayer.points) {
          isTie = true;
        }
      });
      if (isTie) {
        return {
          name: 'Tie',
          points: winningPlayer.points,
        };
      }
      return winningPlayer;
    },
    isGameOver() {
      return this.deck != null && this.cardsRemaining === 0;
    },
    currentPlayer() {
      return this.players[this.currentPlayerIndex];
    },
    currentPlayerName() {
      return this.currentPlayer.name;
    },
    activeCardURL() {
      if (!this.cardDrawn) return null;
      return this.cardDrawn.image;
    },
    deckID() {
      if (!this.deck) return null;
      return this.deck.deck_id;
    },
  },
  created() {
    this.resetPlayerPoints();
    this.fetchDeck();
  },
  methods: {
    pass() {
      if (!this.passButtonActive || this.isLoading) return;
      this.passButtonActive = false;
      this.currentPlayerIndex = (this.currentPlayerIndex + 1) % this.players.length;
    },
    resetGame() {
      this.resetPlayerPoints();
      this.fetchDeck();
      this.correctGuessesInARow = 0;
      this.currentPlayerIndex = 0;
      this.resultDescription = null;
      this.guess = null;
    },
    resetPlayerPoints() {
      this.players.forEach(player => player.points = 0);
    },
    readable(card) {
      return `${card.value} of ${card.suit}`;
    },
    fetchDeck() {
      this.isLoading = true;
      Vue.http.get('https://deckofcardsapi.com/api/deck/new/shuffle/?deck_count=1').then(response => {
        this.isLoading = false;
        this.cardDrawn = null;
        this.deck = response.body;
        this.cardsRemaining = this.deck.remaining;
        this.cardsInFaceUpPile = 0;
        this.drawCard();
      });
    },
    drawCard(processGuess = false) {
      this.passButtonActive = false;
      this.isLoading = true;
      Vue.http.get(`https://deckofcardsapi.com/api/deck/${this.deckID}/draw/?count=1`).then(response => {
        this.isLoading = false;
        var body = response.body;
        if (body.success === true) {
          this.cardDrawn = body.cards[0];
          this.cardsRemaining = body.remaining;
          this.cardsInFaceUpPile += 1;
          if (processGuess) {
            this.processGuess();
          }
        } else {
          console.log("Error drawing card!!!\n");
        }
      });
    },
    setResultText(oldCard, newCard, guess, comparison, isCorrect) {
      this.resultDescription = `${isCorrect ? 'Correct' : 'Incorrect'}, ${this.readable(newCard)}`;
      this.resultDescription = `${this.resultDescription} is ${isCorrect ? '' : ' not '}`;
      this.resultDescription = `${this.resultDescription} ${comparison} compared to`;
      this.resultDescription = `${this.resultDescription} ${this.readable(oldCard)}`;
    },
    processGuess() {
      const comparison = this.getAComparedToBeCondition(this.cardDrawn, this.previousCard);
      const isCorrect = (comparison === this.guess);
      this.setResultText(this.previousCard, this.cardDrawn, comparison, this.guess, isCorrect);
      if (isCorrect) {
        this.correctGuessesInARow += 1;
        if (this.correctGuessesInARow === 3) {
          this.correctGuessesInARow = 0;
          this.passButtonActive = true;
        }
      } else {
        this.currentPlayer.points += this.cardsInFaceUpPile;
        this.cardsInFaceUpPile = 0;
        this.drawCard();
        this.correctGuessesInARow = 0;
      }
    },
    getAComparedToBeCondition(a, b) {
      if (!a || !b) {
        console.log("Error, one card is empty\n");
        return false;
      }
      const aValue = this.getNumericValueFromCardValue(a.value)
      const bValue = this.getNumericValueFromCardValue(b.value);

      if (aValue > bValue) return 'high';
      if (aValue < bValue) return 'low';
      return 'same';
    },
    getNumericValueFromCardValue(cardValue) {
      switch(cardValue) {
        case "ACE": return 14;
        case "KING": return 13;
        case "QUEEN": return 12;
        case "JACK": return 11;
        default: return parseInt(cardValue, 10);
      }
    },
    highClicked() {
      if (this.isLoading) return;
      this.guess = "high";
      this.previousCard = this.cardDrawn;
      this.drawCard(true);
    },
    lowClicked() {
      if (this.isLoading) return;
      this.guess = "low";
      this.previousCard = this.cardDrawn;
      this.drawCard(true);
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
</style>
