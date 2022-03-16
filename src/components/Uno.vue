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
      selectedColor: null,
      selectedCard: null,
      drawnCard: null,
      rightToPass: null,
      tempFirstCard: null,
    }
  },
  components: {
    Header
  },
  methods: {
    /* Init game */
    async startGame() {

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

        console.log("Game starting...")

        await this.prepareOrderRelation()
        await this.prepareCards()
        await this.shuffleCards()
        await new Promise(async (resolve) => {
          const COUNT_DOWN = setInterval(() => {
            this.preparingSecond--
            if (this.preparingSecond <= 0) {
              clearInterval(COUNT_DOWN)
              resolve()
            }
          }, 1000)
        })
        await new Promise(async (resolve) => {
          this.screen = 'game'
          this.triggergameArea()
          await this.dealCards()
          resolve()
        })
        await new Promise(async (resolve) => {
          await this.setOrder()
          resolve()
        })
        this.firstMove()
      }
    },
    /* Prepare first move after start */
    async firstMove() {
      await this.pleaseWait(500);
      let currentCard = this.cardsPlayed[this.cardsPlayed.length - 1]
      let nextUser = this.nextUser()
      // If first card is a joker
      if (currentCard.search("joker") !== -1) {

        // pick start color
        if (this.order !== 'bottom') {
          await new Promise((resolve) => {
            setTimeout(() => {
              this.color = this.selectMyColor()
              resolve()
            }, 1000)
          })
        } else {
          await new Promise((resolve) => {
            this.colorModal.show()
            const COLOR_PICK_INTERVAL = setInterval(() => {
              if (this.selectedColor !== null) {
                this.color = this.selectedColor
                this.selectedColor = null
                this.colorModal.hide()
                clearInterval(COLOR_PICK_INTERVAL)
                resolve()
              }
            }, 200)
          })
        }

        if (currentCard === 'joker:+4') {
          await new Promise(async (resolve) => {
            await new Promise(async (resolve) => {
              for (let index = 0; index < 4; index++) {
                this.playerData[this.orderRelation[nextUser]].cards.push(this.drawCard())
                await this.pleaseWait(200)
              }
              resolve()
            })
            resolve()
          })
          nextUser = this.nextUser(nextUser)
        }
        this.order = nextUser

      } else {
        currentCard = currentCard.split(":")
        // pick start color
        this.color = currentCard[0]
        if (['+2', 'block', 'direction'].indexOf(currentCard[1]) !== -1) {    
          if (currentCard[1] === '+2') { // add 2 card to next user and skip
            await new Promise(async (resolve) => {
              await new Promise(async (resolve) => {
                for (let index = 0; index < 2; index++) {
                  this.playerData[this.orderRelation[nextUser]].cards.push(this.drawCard())
                  await this.pleaseWait(200)
                }
                resolve()
              })
              resolve()
            })
            nextUser = this.nextUser(nextUser) // skip 1 more user
          } else if (currentCard[1] === 'block') { // block next user and skip
            await new Promise(async (resolve) => { // we will add block animation
              this.order = nextUser
              await this.pleaseWait(500);
              resolve()
            })
            nextUser = this.nextUser(nextUser) // skip 1 more user
          } else { // switch direction and skip
            this.switchDirection()
            nextUser = this.nextUser() // assign again
          }
          this.order = nextUser
        }
      }
      
      this.play()
    },
    /* Playing circle */
    async play() {
      this.drawnCard = null
      this.rightToPass = null
      let currentCard = this.cardsPlayed[this.cardsPlayed.length - 1].split(':')
      if (this.order === 'bottom') { // Human

        await new Promise(async (resolve) => {
          const CARD_PICK_INTERVAL = setInterval(async () => {
            if (this.selectedCard !== null) {
              const SELECTED_CARD = this.selectedCard[0]
              const SELECTED_CARD_KEY = this.selectedCard[1]
              const SELECTED_CARD_SPLITTED = SELECTED_CARD.split(':')
              this.selectedCard = null
              
              if (SELECTED_CARD_SPLITTED[0] === 'joker' || SELECTED_CARD_SPLITTED[0] === this.color || SELECTED_CARD_SPLITTED[1] === currentCard[1]) {
                clearInterval(CARD_PICK_INTERVAL)
                this.playerData[this.orderRelation[this.order]].cards.splice(SELECTED_CARD_KEY, 1)
                this.cardsPlayed.push(SELECTED_CARD)
                await this.calculateTheMove()
                resolve()
              }
            } else if (this.rightToPass !== null) {
              this.order = this.nextUser()
              resolve()
            }
          }, 50)
        })

        this.play()
        console.log("We are waiting...")

      } else { // Machine

        await new Promise(async (resolve) => {
          let availableCards = []
          let myCards = this.getMyCards()
          myCards.forEach((element, key) => {
            let elSplitted = element.split(':')
            if (elSplitted[0] === 'joker' || elSplitted[0] === this.color || elSplitted[1] === currentCard[1]) {
              availableCards.push(key) // this.getCardPoint(element) -> maybe we 'ill use in future for hard mode
            }
          });

          if (availableCards.length) { // has a playable card

            await this.pleaseWait(1000)
            const RAND_KEY = availableCards[Math.floor(Math.random()*availableCards.length)]
            const CARD_NAME = this.playerData[this.orderRelation[this.order]].cards[RAND_KEY]
            this.playerData[this.orderRelation[this.order]].cards.splice(RAND_KEY, 1)
            this.cardsPlayed.push(CARD_NAME)
            await this.calculateTheMove()

          } else { // has'nt a playable card
            
            await this.pleaseWait(1000)
            const DRAW_CARD = this.drawCard()
            this.playerData[this.orderRelation[this.order]].cards.push(DRAW_CARD)
            let splittedDrawCard = DRAW_CARD.split(':')
            if (splittedDrawCard[0] === 'joker' || splittedDrawCard[0] === this.color || splittedDrawCard[1] === currentCard[1]) {
              this.cardsPlayed.push(DRAW_CARD)
              await this.calculateTheMove()
            } else {
              await this.pleaseWait(1000)
              this.pass()
            }
          }
          resolve()
        })

        this.play()
      }
    },
    async calculateTheMove() {
      return new Promise(async (resolve) => {
        let currentCard = this.cardsPlayed[this.cardsPlayed.length - 1]
        let nextUser = this.nextUser()
        // If first card is a joker
        if (currentCard.search("joker") !== -1) {

          // pick start color
          if (this.order !== 'bottom') {
            await new Promise((resolve) => {
              setTimeout(() => {
                this.color = this.selectMyColor()
                resolve()
              }, 1000)
            })
          } else {
            await new Promise((resolve) => {
              this.colorModal.show()
              const COLOR_PICK_INTERVAL = setInterval(() => {
                if (this.selectedColor !== null) {
                  this.color = this.selectedColor
                  this.selectedColor = null
                  this.colorModal.hide()
                  clearInterval(COLOR_PICK_INTERVAL)
                  resolve()
                }
              }, 200)
            })
          }

          if (currentCard === 'joker:+4') {
            await new Promise(async (resolve) => {
              await new Promise(async (resolve) => {
                for (let index = 0; index < 4; index++) {
                  this.playerData[this.orderRelation[nextUser]].cards.push(this.drawCard())
                  await this.pleaseWait(200)
                }
                resolve()
              })
              resolve()
            })
            nextUser = this.nextUser(nextUser)
          }
          this.order = nextUser

        } else {
          currentCard = currentCard.split(":")
          // pick start color
          this.color = currentCard[0]
          if (['+2', 'block', 'direction'].indexOf(currentCard[1]) !== -1) {    
            if (currentCard[1] === '+2') { // add 2 card to next user and skip
              await new Promise(async (resolve) => {
                await new Promise(async (resolve) => {
                  for (let index = 0; index < 2; index++) {
                    this.playerData[this.orderRelation[nextUser]].cards.push(this.drawCard())
                    await this.pleaseWait(200)
                  }
                  resolve()
                })
                resolve()
              })
              nextUser = this.nextUser(nextUser) // skip 1 more user
            } else if (currentCard[1] === 'block') { // block next user and skip
              await new Promise(async (resolve) => { // we will add block animation
                this.order = nextUser
                await this.pleaseWait(500);
                resolve()
              })
              nextUser = this.nextUser(nextUser) // skip 1 more user
            } else { // switch direction and skip
              this.switchDirection()
              nextUser = this.nextUser() // assign again
            }
            this.order = nextUser
          } else {
            this.order = nextUser
          }
        }
        resolve()
      })
    },
    pass() {
      this.order = this.nextUser()
    },
    /* Get next user */
    nextUser(from = null) {
      if (from === null) {
        from = this.order
      }
      const DIRECTIONS = {
        left:  ['bottom', 'left', 'top', 'right'],
        right: ['bottom', 'right', 'top', 'left']
      }
      Object.entries(this.orderRelation).forEach(el => {
        if (el[1] === null) {
          DIRECTIONS.left.splice(DIRECTIONS.left.indexOf(el[0]), 1)
          DIRECTIONS.right.splice(DIRECTIONS.right.indexOf(el[0]), 1)
        }
      })

      const CURRENT_INDEX = DIRECTIONS[this.direction].indexOf(from)
      let nextIndex;
      if ((CURRENT_INDEX+1) > (DIRECTIONS[this.direction].length-1)) {
        nextIndex = 0
      } else {
        nextIndex = (CURRENT_INDEX+1)
      }
      return DIRECTIONS[this.direction][nextIndex]
    },
    /*
    async afterMove(firstMove = false) {

      let currentCard = this.cardsPlayed[this.cardsPlayed.length - 1]
      let nextUser = this.nextUser()
      if (firstMove) {
        
      } else {
        if (this.order !== 'bottom') {
          this.playingMachine()
        }
      }
    },
    */
    switchDirection() {
      this.direction = this.direction === 'left' ? 'right' : 'left'
    },
    pickColor(color = null) {
      this.selectedColor = color
    },
    pickCard(card = null) {
      this.selectedCard = card
    },
    getMyCards() {
      return Array.from(this.playerData[this.orderRelation[this.order]].cards)
    },
    /**
     * Deal Shuffle Cards
     */
    dealCards() {
      return new Promise(async (resolve) => {
        const PLAYER_DATA = Object.entries(this.playerData)
        // Draw player cards
        for (let card = 0; card < 7; card++) {
          for (let playerIndex = 0; playerIndex < PLAYER_DATA.length; playerIndex++) {
            PLAYER_DATA[playerIndex][1].cards.push(this.drawCard())
            await this.pleaseWait(250)
          }
          await this.pleaseWait(250)
        }
          
        // Draw table card
        this.cardsPlayed.push((this.tempFirstCard !== null ? this.tempFirstCard : this.drawCard()))
        await this.pleaseWait(500)
        resolve()
      })
    },
    /**
     * Prepare Order Relation
     */
    prepareOrderRelation() {
      /* TEMP */
      return new Promise()
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
     * Prepare Cards
     */
    prepareCards() {

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
    drawCardToSlot() {
      if (this.drawnCard === null && this.order === 'bottom') {
        this.drawnCard = this.drawCard()
        this.playerData[this.orderRelation[this.order]].cards.push(this.drawnCard)
      }
    },
    setOrder() {
      new Promise(async (resolve) =>  {
        const orders = this.orders.filter((el) => {
          return this.orderRelation[el] !== null;
        });
        this.order = orders[Math.floor(Math.random()*orders.length)]
        await this.pleaseWait(500)
        resolve()
      })
    },
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
    /**
     *  Get Current Player's Cards
     */
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
    pleaseWait (ms = 1000) {
      return new Promise(resolve => setTimeout(resolve, ms));
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
            <div class="other-cards" data-bs-toggle="tooltip" data-bs-placement="top" title="Draw a card" @click="drawCardToSlot()">
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
                <button class="active" @click="drawnCard !== null ? rightToPass = true : false">PAS</button>
                <button class="active" @click="nextUser()">UNO</button>
              </span>
            </div>
            <div class="cards">
              <div v-for="(card, key) in getBottomCards" :key="key" class="game-card" :data-card-type="card" @click="selectedCard = [card, key]">
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