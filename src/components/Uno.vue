<script>
import Header from './Header.vue'
export default {
  data() {
    return {
      screen: 'start',
      numberOfPlayer: 2,
      playerName: 'Gamer',
      alertMessage: '',
      preparingSecond: 3,
      cards: [],
      cardsPlayed: [],
      playerData: {},
      fakePlayerNames: ["John", "Dave", "Anna", "Jessica", "Scarlett", "Kelly", "Sophia", "Leila"],
      direction: 'left',
      orders: ['top', 'right', 'bottom', 'left'],
      order: '',
      orderRelation: {},
      colors: ['blue', 'green', 'red', 'yellow'],
      color: '',
      colorModal: null,
      pickColorNext: false,
    }
  },
  components: {
    Header
  },
  methods: {
    startGame() {

      this.numberOfPlayer = parseInt(this.numberOfPlayer)
      if (this.playerName == '') {
        this.alertMessage = 'Player name cannot be empty!'
      } else if (this.numberOfPlayer < 2 || this.numberOfPlayer > 4) {
        this.alertMessage = 'The number of players must be chosen correctly!'
      } else {
        this.alertMessage = ''
        this.preparingSecond = 3
        this.screen = 'countdown'

        for (let number = 0; number < this.numberOfPlayer; number++) {
          let playerName = this.fakePlayerNames[Math.floor(Math.random()*this.fakePlayerNames.length)]
          if (! number) {
            playerName = this.playerName
          }
          this.playerData[number] = {
            name: playerName,
            score: 500,
            cards: []
          }
        }

        new Promise((resolve, reject) => {
          resolve(this.prepareOrderRelation())
        }).then(() => {
          return new Promise((resolve, reject) => {
            resolve(this.getCards())
          })
        }).then(() => {
          return new Promise((resolve, reject) => {
            resolve(this.shuffleCards())
          })
        }).then(() => {
          return new Promise(async (resolve, reject) => {
            await new Promise(resolve => setTimeout(() => resolve(this.preparingSecond--), 1000))
            await new Promise(resolve => setTimeout(() => resolve(this.preparingSecond--), 1000))
            await new Promise(resolve => setTimeout(() => resolve(this.preparingSecond--), 1000))
            resolve()
          })
        }).then(() => {
          return new Promise(async (resolve, reject) => {
            this.screen = 'game'
            await this.dealCards()
            resolve()
          })
        }).then(() => {
          return new Promise(async (resolve, reject) => {
            await this.setOrder()
            resolve()
          })
        }).then(() => {
          if (this.order !== 'bottom') {
            this.playingMachine()
          } else {
            console.log("Me!")
          }
        })
      }
    },
    playingMachine() {
      setTimeout(() => {
        let currentCard = this.cardsPlayed[this.cardsPlayed.length - 1].split(':')
        let availableCards = []
        let myCards = this.getMyCards()
        myCards.forEach((element, key) => {
          let elSplitted = element.split(':')
          if (elSplitted[0] === 'joker' || elSplitted[0] === this.color) {
            availableCards.push(key) // this.getCardPoint(element) -> maybe we 'ill use in future for hard mode
          }
        });
        if (availableCards.length) {
          const randKey = availableCards[Math.floor(Math.random()*availableCards.length)]
          const cardName = this.playerData[this.orderRelation[this.order]].cards[randKey]
          this.playerData[this.orderRelation[this.order]].cards.splice(randKey, 1)
          this.cardsPlayed.push(cardName)
        } else {
          
        }
        
        // this.calculateMove(cardName)
      }, 1000)
    },
    getMyCards() {
      return Array.from(this.playerData[this.orderRelation[this.order]].cards)
    },
    /**
     * Deal Shuffle Cards
     */
    dealCards() {
      return new Promise((resolve, reject) => {
        Object.entries(this.playerData).forEach(async el => {
          for (let card = 0; card < 7; card++) {
            await this.pleaseWait(500);
            el[1].cards.push(this.drawCard())
          }
        })
        this.cardsPlayed.push(this.drawCard())
        setTimeout(() => {
          resolve()
        }, (500*7))
      })
    },
    /**
     * Prepare Order Relation
     */
    prepareOrderRelation() {
      switch (parseInt(this.numberOfPlayer)) {
        case 4:
          this.orderRelation = {
            bottom: 0,
            left: 1,
            top: 2,
            right: 3
          }
          break
        case 3:
          this.orderRelation = {
            bottom: 0,
            left: 1,
            top: null,
            right: 2
          }
          break
        default:
          this.orderRelation = {
            bottom: 0,
            left: null,
            top: 1,
            right: null
          }
      }
    },
    /**
     * Generate Cards
     */
    getCards() {

      ["red", "yellow", "green", "blue", "joker"].forEach(color => {
        switch (color) {
          case "joker":
            [...Array(4).keys()].forEach(number => {
              this.cards.push(color + ":+4")
              this.cards.push(color + ":color")
            });
            break;
        
          default:
            [...Array(10).keys()].forEach(number => {
              if (number) this.cards.push(color + ":" + number)
              this.cards.push(color + ":" + number)
            });
            for (let x = 0; x < 2; x++) {
              this.cards.push(color + ":block")
              this.cards.push(color + ":direction")
              this.cards.push(color + ":+2")
            }
            break;
        }
      })

    },
    /**
     * Shuffle Generated Cards
     */
    shuffleCards() {
      this.cards = this.cards.sort(() => Math.random() - 0.5)
    },
    
    /**
     * Draw a Card
     */
    drawCard() {
      return this.cards.shift()
    },
    setOrder() {
      new Promise(async (resolve, reject) =>  {
        const orders = this.orders.filter((el) => {
          return this.orderRelation[el] !== null;
        });
        this.order = orders[Math.floor(Math.random()*orders.length)]
        await this.pleaseWait(1500)
        resolve()
      })
    },
    async play(cardName, first = false) {

      let index = null
      if (cardName.indexOf("-") !== -1) {
        cardName = cardName.split("-")
        index = cardName[0]
        cardName = cardName[1]
      }

      let cardFeatures = cardName.split(':')
      if (first) {
        if (cardFeatures[0] !== 'joker') {
          this.color = cardFeatures[0]
        } else {
          if (this.order == 'bottom') {
            this.pickColorNext = true
            this.calculateMove(cardName)
          } else {
            this.color = this.selectMyColor()
            this.calculateMove(cardName)
          }
        }
        
      } else {
        if (this.order == 'bottom') {
          this.playerData[this.orderRelation.bottom].cards.splice(index, 1)
          this.cardsPlayed.push(cardName)
          this.calculateMove(cardName)
        } else {
          this.computerPlaying()
        }
      }

     
    },
    pickColor(color = null) {
      if (color) {
        this.colorModal.hide()
        this.color = color
      } else {
        this.colorModal.show()
      }
    },
    /**
     *  Get Current Player's Cards
     */

    
    selectMyColor() {

      const myCards = this.getMyCards()
      const colors = {
        red: 0,
        blue: 0,
        green: 0,
        yellow: 0
      }
      myCards.forEach((element) => {
        if (element.includes("red")) {
          colors.red+= this.getCardPoint(element)
        } else if (element.includes("blue")) {
          colors.blue+= this.getCardPoint(element)
        } else if (element.includes("green")) {
          colors.green+= this.getCardPoint(element)
        } else if (element.includes("yellow"))  {
          colors.yellow+= this.getCardPoint(element)
        }
      })

      return Object.keys(colors).reduce((a, b) => colors[a] > colors[b] ? a : b)

    },
    getCardPoint(cardName) {
      let point = 0
      cardName = cardName.split(':')
      if (cardName[1] === 'block' || cardName[1] === 'direction' || cardName[1] === '+2') {
        point = 20
      } else if (cardName[1] === 'color' || cardName[1] === '+4') {
        point = 50 
      } else {
        point = parseInt(cardName[1])
      }
      return point
    },
    calculateMove(cardName) {

      console.log("calculate" + cardName)
    },
    drawACard() {
      if (this.order == 'bottom') {
        this.playerData[0].cards.push(this.drawCard())
      }
    },
    pleaseWait (ms = 1000) {
      return new Promise(resolve => setTimeout(resolve, ms));
    },
    async renderGame() {
      await this.wait(2000)
      this.screen = 'game'
      this.triggergameArea()
      // for table
      setTimeout(() => {
        const startCard = 'joker:color' //this.drawCard()
        this.play(startCard, true)
        this.cardsPlayed.push(startCard)
        setTimeout(() => {
          if (this.pickColorNext) {
            setTimeout(() => {
              this.pickColor()
              this.pickColorNext = false
            })
          }
        }, 100)
      }, 500)
      
    },
    triggergameArea() {
      setTimeout(() => {
        let tooltips = [].slice.call(document.querySelectorAll('[data-bs-toggle="tooltip"]')).map(function (tooltipTriggerEl) {
          return new bootstrap.Tooltip(tooltipTriggerEl)
        })
        if (! this.colorModal) {
          this.colorModal = new bootstrap.Modal(document.getElementById('colorModal'))
        }
      })
    }
  },
  computed: {
    getTopCards() {
      return this.orderRelation.top !== null && this.playerData[this.orderRelation.top].cards
    },
    getBottomCards() {
      return this.orderRelation.bottom !== null && this.playerData[this.orderRelation.bottom].cards
    },
    getLeftCards() {
      return this.orderRelation.left !== null && this.playerData[this.orderRelation.left].cards
    },
    getRightCards() {
      return this.orderRelation.right !== null && this.playerData[this.orderRelation.right].cards
    },
    tableCards() {
      return this.cardsPlayed
    }
  },
  mounted() {
    this.playerName = this.fakePlayerNames[Math.floor(Math.random()*this.fakePlayerNames.length)]
  }
}
</script>

<template>
  <Header :screen="screen" />
  <div class="wrap">
    <div v-if="screen === 'start'" class="start-screen">
      <div class="container">
        <div class="row justify-content-center align-items-center vh-100">
          <div class="col-12 col-md-6 col-lg-4">
            <img class="logo mx-auto d-flex" src="@/assets/logo.svg" alt="VUNO Logo" />
            <div class="mt-5">
              <div v-if="alertMessage" class="alert alert-warning mt-5" role="alert">
                {{alertMessage}}
              </div>
              <div class="form-floating mb-3">
                <input type="text" class="form-control" v-model="playerName" maxlenght="40" placeholder="Player Name">
                <label for="player_name">Player Name</label>
              </div>
              <div class="form-floating mb-3">
                <select class="form-select" id="player_number" v-model="numberOfPlayer" aria-label="Number of Players">
                  <option value="2" selected>2</option>
                  <option value="3">3</option>
                  <option value="4">4</option>
                </select>
                <label for="player_number">Number of Players</label>
              </div>
              <div class="d-grid">
                <button @click="startGame()" class="btn btn-success d-block btn-lg">Play!</button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div v-else class="game-screen" :class="screen == 'game' ? 'active' : (screen == 'countdown' && 'count-active')">
      <div class="counter">
        {{preparingSecond}}
      </div>
      <div class="game-area" :class="order">
        <div v-if="orderRelation.top !== null && orderRelation.top !== undefined" class="position-absolute top-0 start-50 translate-middle-x top" :class="order == 'top' ? 'current' : ''">
          <div class="player-side">
            <div class="player-info">
              <span class="score-badge" data-bs-toggle="tooltip" title="Score">
                {{ playerData[orderRelation.top].score - 500 }}
              </span>
              <span class="name">{{ playerData[orderRelation.top].name }}</span>
              <span class="action-buttons">
                <button>PAS</button>
                <button>UNO</button>
              </span>
            </div>
            <div class="cards">
              <div v-for="(card, key) in getTopCards" :key="key" class="game-card" :data-card-type="card">
              </div>
            </div>
          </div>
        </div>
        <div v-if="orderRelation.left !== null && orderRelation.left !== undefined" class="position-absolute top-50 start-0 translate-middle-y left" :class="order == 'left' ? 'current' : ''">
          <div class="player-side">
            <div class="player-info">
              <span class="score-badge" data-bs-toggle="tooltip" title="Score">
                {{ playerData[orderRelation.left].score - 500 }}
              </span>
              <span class="name">{{ playerData[orderRelation.left].name }}</span>
              <span class="action-buttons">
                <button>PAS</button>
                <button>UNO</button>
              </span>
            </div>
            <div class="cards">
              <div v-for="(card, key) in getLeftCards" :key="key" class="game-card" :data-card-type="card">
              </div>
            </div>
          </div>
        </div>
        <div class="position-absolute top-50 start-50 translate-middle center">
          <div class="game-table" :class="'direction-' + direction + ' ' + color + ' ' + order">
            <div class="other-cards" data-bs-toggle="tooltip" data-bs-placement="top" title="Draw a card" @click="drawACard()">
              <img src="@/assets/logo.svg" alt="Logo" />
            </div>
            <div class="current-card">
              <div class="cards">
              <div v-for="(card, key) in tableCards" :key="key" class="game-card" :data-card-type="card" @click="playerData[0].score+=100">
                </div>
              </div>
            </div>
          </div>
        </div>
        <div v-if="orderRelation.right !== null && orderRelation.right !== undefined" class="position-absolute top-50 end-0 translate-middle-y right" :class="order == 'right' ? 'current' : ''">
          <div class="player-side">
            <div class="player-info">
              <span class="score-badge" data-bs-toggle="tooltip" title="Score">
                {{ playerData[orderRelation.right].score - 500 }}
              </span>
              <span class="name">{{ playerData[orderRelation.right].name }}</span>
              <span class="action-buttons">
                <button>PAS</button>
                <button>UNO</button>
              </span>
            </div>
            <div class="cards">
              <div v-for="(card, key) in getRightCards" :key="key" class="game-card" :data-card-type="card">
              </div>
            </div>
          </div>
        </div>
        <div v-if="orderRelation.bottom !== null && orderRelation.bottom !== undefined" class="position-absolute bottom-0 start-50 translate-middle-x bottom"  :class="order == 'bottom' ? 'current' : ''">
          <div class="player-side">
            <div class="player-info">
              <span class="score-badge" data-bs-toggle="tooltip" title="Score">
                {{ playerData[orderRelation.bottom].score - 500 }}
              </span>
              <span class="name" @click="playerData[orderRelation.bottom].cards.shift()">{{ playerData[orderRelation.bottom].name }}</span>
              <span class="action-buttons">
                <button class="active" @click="playerData[orderRelation.bottom].cards.push('yellow:1')">PAS</button>
                <button class="active" @click="playerData[orderRelation.top].cards.push('yellow:1')">UNO</button>
              </span>
            </div>
            <div class="cards">
              <div v-for="(card, key) in getBottomCards" :key="key" class="game-card" :data-card-type="card" @click="play(key + '-' + card)">
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
  <!-- History -->
  <div class="modal fade" id="historyModal" data-bs-backdrop="static" data-bs-keyboard="false" tabindex="-1" aria-labelledby="historyModalLabel" aria-hidden="true">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title" id="historyModalLabel">History</h5>
          <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
        </div>
        <div class="modal-body">
          soon...
        </div>
      </div>
    </div>
  </div>
  <!-- Color Selection Modal -->
  <div class="modal fade" id="colorModal" data-bs-backdrop="static" data-bs-keyboard="false" tabindex="-1" aria-labelledby="colorModalLabel" aria-hidden="true">
    <div class="modal-dialog modal-dialog-centered">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title" id="colorModalLabel">Pick a color</h5>
        </div>
        <div class="modal-body">
          <div class="btn-group d-flex mx-auto" role="group" aria-label="Pick a color">
            <button class="btn btn-danger" @click="pickColor('red')">Red</button>
            <button class="btn btn-success" @click="pickColor('green')">Green</button>
            <button class="btn btn-primary" @click="pickColor('blue')">Blue</button>
            <button class="btn btn-warning" @click="pickColor('yellow')">Yellow</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>