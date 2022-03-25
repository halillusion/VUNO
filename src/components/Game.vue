<script>
import Progress from "./Progress.vue"
import Start from "./Start.vue"
import Countdown from "./Countdown.vue"
export default {
  data() {
    return {
      progress: 0,
      screen: '',
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
      finishModal: null,
      selectedColor: null,
      selectedCard: null,
      drawnCard: null,
      rightToPass: null,
      tempFirstCard: null,
      unoClicked: false,
      history: [],
      gameAfter: {
        title: null,
        table: [],
        keep: true,
      },
      playerActions: {
        top: {
          pass: false,
          uno: false
        },
        right: {
          pass: false,
          uno: false
        },
        bottom: {
          pass: false,
          uno: false
        },
        left: {
          pass: false,
          uno: false
        }
      }
    }
  },
  components: {
    Progress, Start, Countdown
  },
  methods: {
    /* Init game */
    async startGame(keepGoing = false) {

      this.numberOfPlayer = parseInt(this.numberOfPlayer)
      if (this.playerName == '') {
        this.alertMessage = 'Player name cannot be empty!'
      } else if (this.numberOfPlayer < 2 || this.numberOfPlayer > 4) {
        this.alertMessage = 'The number of players must be chosen correctly!'
      } else {
        this.alertMessage = ''
        this.preparingSecond = 3
        this.screen = 'countdown'

        console.log("Game starting...")
        if (keepGoing) {
          this.finishModal.hide()
          this.history.push(this.gameAfter)
          this.gameAfter = {
            title: null,
            table: [],
            keep: true,
          }
          this.cardsPlayed = []
          this.cards = []
          for (let number = 0; number < this.numberOfPlayer; number++) {
            this.playerData[number].cards = []
          }
        } else {
          for (let number = 0; number < this.numberOfPlayer; number++) {
            let playerName = this.fakePlayerNames[Math.floor(Math.random()*this.fakePlayerNames.length)]
            if (! number) {
              playerName = this.playerName
            }
            this.playerData[number] = {
              name: playerName,
              score: 0,
              cards: []
            }
          }
          await this.prepareOrderRelation()
        }
        
        await this.prepareCards()
        await this.shuffleCards()
        await new Promise(async (resolve) => {
          const COUNT_DOWN = setInterval(() => {
            console.log("Countdown: " + this.preparingSecond)
            this.preparingSecond--
            if (this.preparingSecond <= 0) {
              clearInterval(COUNT_DOWN)
              resolve(console.log("Start"))
            }
          }, 1000)
        })
        await new Promise(async (resolve) => {
          this.screen = 'game'
          await this.dealCards()
          resolve(console.log("Visible game display..."))
        })
        await new Promise(async (resolve) => {
          await this.setOrder()
          resolve(console.log("Like everything is ready..."))
        })
        await this.firstMove()
      }

      this.play()
    },
    /* Prepare first move after start */
    async firstMove() {
      console.log("first move")
      return new Promise(async (resolve) => {
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
                resolve(console.log("Machine chose color: " + this.color))
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
                  resolve(console.log("Human chose color: " + this.color))
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
                resolve(console.log("Joker +4 draw to " + nextUser))
              })
              resolve(console.log("Joker +4"))
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
                  resolve(console.log(this.color + " +2 draw to " + nextUser))
                })
                resolve(console.log(this.color + " +2"))
              })
              nextUser = this.nextUser(nextUser) // skip 1 more user
            } else if (currentCard[1] === 'block') { // block next user and skip
              await new Promise(async (resolve) => { // we will add block animation
                this.order = nextUser
                await this.pleaseWait(500);
                resolve(console.log(this.order + " blocked."))
              })
              nextUser = this.nextUser(nextUser) // skip 1 more user
            } else { // switch direction and skip
              await this.switchDirection()
              nextUser = this.nextUser() // assign again
            }
            this.order = nextUser
          }
        }
        resolve()
      })
    },
    /* Playing circle */
    async play() {
      if (this.gameAfter.title === null) {
        return new Promise (async (resolve) => {

          const ORDER = this.order
          console.log("playing " + ORDER)
          console.log(
            this.playerData[this.orderRelation[ORDER]].name,
            this.playerData[this.orderRelation[ORDER]].cards.length
          )
          this.drawnCard = null
          this.rightToPass = null
          this.unoClicked = false
          let currentCard = this.cardsPlayed[this.cardsPlayed.length - 1].split(':')
          if (ORDER === 'bottom') { // Human
            console.log("Human playing...")
            await new Promise(async (resolve) => {
              const CARD_PICK_INTERVAL = setInterval(async () => {
                if (this.selectedCard !== null) {
                  const SELECTED_CARD = this.selectedCard[0]
                  const SELECTED_CARD_KEY = this.selectedCard[1]
                  const SELECTED_CARD_SPLITTED = SELECTED_CARD.split(':')
                  console.log("Human's selected card: " + SELECTED_CARD)
                  this.selectedCard = null
                  if (SELECTED_CARD_SPLITTED[0] === 'joker' || SELECTED_CARD_SPLITTED[0] === this.color || SELECTED_CARD_SPLITTED[1] === currentCard[1]) {
                    console.log("Human's selected card is available: " + SELECTED_CARD)
                    clearInterval(CARD_PICK_INTERVAL)
                    this.playerData[this.orderRelation[ORDER]].cards.splice(SELECTED_CARD_KEY, 1)
                    this.cardsPlayed.push(SELECTED_CARD)
                    await this.calculateTheMove()
                    resolve(console.log("Move calculated"))
                  }
                } else if (this.rightToPass !== null) {
                  this.order = this.nextUser()
                  clearInterval(CARD_PICK_INTERVAL)
                  resolve(console.log("Human's used right to pass"))
                }
              }, 50)
            })

          } else { // Machine
            console.log("Machine playing...")
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
          
                const RAND_KEY = availableCards[Math.floor(Math.random()*(availableCards.length-1))]
                const CARD_NAME = this.playerData[this.orderRelation[ORDER]].cards[RAND_KEY]
                console.log("Machine's selected card: " + CARD_NAME)
                await this.pleaseWait(2000)
                console.log("Machine's selected card is available: " + CARD_NAME)
                this.playerData[this.orderRelation[ORDER]].cards.splice(RAND_KEY, 1)
                this.cardsPlayed.push(CARD_NAME)
                await this.calculateTheMove()

              } else { // hasn't a playable card
                
                await this.pleaseWait(2500)
                const DRAW_CARD = this.drawCard()
                this.playerData[this.orderRelation[this.order]].cards.push(DRAW_CARD)
                let splittedDrawCard = DRAW_CARD.split(':')
                
                console.log("Machine's used right to pass " + DRAW_CARD)
                await this.pleaseWait(1500)

                if (splittedDrawCard[0] === 'joker' || splittedDrawCard[0] === this.color || splittedDrawCard[1] === currentCard[1]) {

                  this.cardsPlayed.push(DRAW_CARD)
                  let machineCard = this.playerData[this.orderRelation[ORDER]].cards
                  this.playerData[this.orderRelation[ORDER]].cards.splice(machineCard.indexOf(DRAW_CARD), 1)
                  console.log("Machine's pass card is available: " + DRAW_CARD)
                  await this.calculateTheMove()
                } else {
                  await this.pleaseWait(1500)
                  console.log("Machine's pass to right")
                  this.pass()
                }
              }
              resolve()
            })
          }
          console.log(
            this.playerData[this.orderRelation[ORDER]].name,
            this.playerData[this.orderRelation[ORDER]].cards.length
          )

          if (this.playerData[this.orderRelation[ORDER]].cards.length === 0) {
            await this.finish()
          } else if (this.playerData[this.orderRelation[ORDER]].cards.length === 1) {
            await this.uno(ORDER);
          } else if (this.playerActions[ORDER].uno) {
            this.playerActions[ORDER].uno = false
          }
          this.play()
          resolve()
        })
      } else {
        return true
      }
    },
    finish () {
      return new Promise((resolve) => {
        let keepGoing = true
        let currentUser = null
        Object.entries(this.playerData).forEach((playerData, key) => {
          const KEY = key
          const PLYR = {...playerData[1]}
          PLYR.cards = {...PLYR.cards}

          if (this.gameAfter.table[KEY] === undefined) {
            this.gameAfter.table[KEY] = {}
          }

          this.gameAfter.table[KEY].name = PLYR.name
          this.gameAfter.table[KEY].current = this.orderRelation.bottom === KEY ? true : false
          if (this.gameAfter.table[KEY].current) {
            currentUser = KEY
          }
          this.gameAfter.table[KEY].score = 500
          if (Object.values(PLYR.cards).length) {
            const CARDS = Object.values(PLYR.cards)
            for (let index = 0; index < CARDS.length; index++) {
              const CARD = CARDS[index];
              this.gameAfter.table[KEY].score -= this.getCardPoint(CARD)
            }
          }

          this.playerData[KEY].score += (500 - this.gameAfter.table[KEY].score)
          if (this.playerData[KEY].score >= 500) {
            keepGoing = false
          }
        })

        this.gameAfter.table.sort((a,b) => (a.score > b.score) ? 1 : ((b.score > a.score) ? -1 : 0)).reverse()
        this.gameAfter.title = this.gameAfter.table[0].name + ' WON!'
        this.gameAfter.keep = keepGoing
        this.finishModal.show()
        resolve()
      }) 
    },
    uno (playerOrder) {
      return new Promise(async (resolve) => {
        if (playerOrder === 'bottom') {
          const PASS_LIMIT = 1000 // ~ 1 second
          let currentTime = 0
          const PASS_INTERVAL = setInterval(() => {
            if (currentTime > PASS_LIMIT) {
              clearInterval(PASS_INTERVAL)
              if (this.unoClicked) {
                this.playerActions[playerOrder].uno = true
                resolve()
              } else { // punishment
                this.playerActions[playerOrder].uno = null
                this.playerData[this.orderRelation[playerOrder]].cards.push(this.drawCard())
                this.playerData[this.orderRelation[playerOrder]].cards.push(this.drawCard())
                setTimeout(() => {
                  this.playerActions[playerOrder].uno = false
                }, 1000)
                resolve()
              }
            }
            currentTime+=500
          }, 500)
        } else {
          const NUMBERS = [0,1,2,3,4,5,6,7,8,9,10]
          const RAND = NUMBERS[Math.floor(Math.random()*NUMBERS.length)]
          if ((RAND%2) === 0) { // random pass punishment
            this.playerActions[playerOrder].uno = true
            resolve()
          } else {
            await this.pleaseWait(1000)
            this.playerActions[playerOrder].uno = null
            this.playerData[this.orderRelation[playerOrder]].cards.push(this.drawCard())
            this.playerData[this.orderRelation[playerOrder]].cards.push(this.drawCard())
            setTimeout(() => {
              this.playerActions[playerOrder].uno = false
              resolve()
            }, 1000)
          }
        }
      })
    },
    async calculateTheMove() {
      return new Promise(async (resolve) => {

        console.log(this.playerData, this.orderRelation)

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
              nextUser = Object.entries(this.playerData).length === 2 ? this.order : this.nextUser(nextUser) // assign again
              await this.switchDirection()
            }
            this.order = nextUser
          } else {
            this.order = nextUser
          }
        }
        await this.pleaseWait(500)
        resolve()
      })
    },
    pass() {
      const ORDER = this.order
      if (ORDER === 'bottom') {
        this.rightToPass = true
      } else {
        this.order = this.nextUser()
      }
      this.playerActions[ORDER].pass = true
      setTimeout(() => {
        this.playerActions[ORDER].pass = false
      }, 1500)
      
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
    switchDirection() {
      return new Promise((resolve) => {
        resolve(this.direction = this.direction === 'left' ? 'right' : 'left')
      })
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
        
        console.log("Deal cards...")
        const PLAYER_DATA = Object.entries(this.playerData)
        // Draw player cards
        for (let card = 0; card < 7; card++) {
          for (let playerIndex = 0; playerIndex < PLAYER_DATA.length; playerIndex++) {
            let card = this.drawCard()
            PLAYER_DATA[playerIndex][1].cards.push(card)
            console.log("DEAL CARD: " + PLAYER_DATA[playerIndex][1].name + " - " + card)
            await this.pleaseWait(250)
          }
          await this.pleaseWait(250)
        }
          
        // Draw table card
        const TABLE_CARD = (this.tempFirstCard !== null ? this.tempFirstCard : this.drawCard())
        console.log("DEAL CARD: TABLE - " + TABLE_CARD)
        this.cardsPlayed.push(TABLE_CARD)
        await this.pleaseWait(500)
        resolve(console.log("deadCards() OK"))
      })
    },
    /**
     * Prepare Order Relation
     */
    prepareOrderRelation() {
      /* TEMP */
      return new Promise((resolve) => {
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
        resolve(console.log("Order relations prepared..."))
      })
    },
    /**
     * Prepare Cards
     */
    prepareCards() {
      return new Promise((resolve) => {
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
        resolve(console.log("Cards generated..."))
      })
    },
    /**
     * Shuffle Generated Cards
     */
    shuffleCards() {
      return new Promise ((resolve) => {
        this.cards = this.cards.sort(() => Math.random() - 0.5)
        resolve(console.log("Card shuffle..."))
      })
    },
    /**
     * Draw a Card
     */
    drawCard() {
      if (this.cards.length) {
        return this.cards.pop()
      } else {
        return this.cardsPlayed.shift()
      }
    },
    drawCardToSlot() {
      if (this.drawnCard === null && this.order === 'bottom') {
        this.drawnCard = this.drawCard()
        this.playerData[this.orderRelation[this.order]].cards.push(this.drawnCard)
      }
    },
    setOrder() {
      return new Promise(async (resolve) =>  {
        const orders = this.orders.filter((el) => {
          return this.orderRelation[el] !== null;
        });
        this.order = orders[Math.floor(Math.random()*orders.length)]
        await this.pleaseWait(500)
        resolve(console.log("Calculated order..."))
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
    reloadPage () {
      window.location.reload()
    },
    test (data) {
      console.log(data)
    }
  },
  watch: {
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
  created() {
    this.screen = 'progress'
    const WIDTH = setInterval(async () => {
      if ((this.progress + 1) <= 100)
        this.progress+=1
        
      if (this.progress>99) {
        clearInterval(WIDTH)
        await this.pleaseWait(700)
        this.screen = 'start'
      }
    }, 11)
  }
}
</script>
<template>
  <div class="container-fluid">
    <div class="row align-items-center justify-content-center">
      <!-- Progress -->
      <Progress v-if="screen === 'progress'" :progress="progress" />
      <!-- /Progress -->

      <!-- Start -->
      <Start v-if="screen === 'start'" 
        @changed="(e) => { playerName = e._name, numberOfPlayer = e._number }"
        @starting="startGame()"
        :content="{_name: playerName, _number: numberOfPlayer}"  />
      <!-- /Start -->

      <!-- Countdown -->
      <Countdown v-if="screen === 'countdown'" :count="preparingSecond" />
      <!-- /Countdown -->
      
      <div v-if="screen === 'game'" class="col-12 game">
        <div class="row">
          <div class="col" v-if="orderRelation.top !== null">
            <div class="cards">
              <div v-for="(card, key) in getTopCards" :key="key" class="game-card rival" :data-card-type="card">
              </div>
            </div>
          </div>
        </div>
        <div class="row">
           <div class="col" v-if="orderRelation.left !== null">
              <div class="cards">
                <div v-for="(card, key) in getLeftCards" :key="key" class="game-card rival" :data-card-type="card">
                </div>
              </div>
            </div>
            <div class="col">
              center
            </div>
            <div class="col" v-if="orderRelation.right !== null">
              <div class="cards">
                <div v-for="(card, key) in getRightCards" :key="key" class="game-card rival" :data-card-type="card">
                </div>
              </div>
            </div>
        </div>
        <div class="row">
          <div class="col" v-if="orderRelation.bottom !== null">
            <div class="cards">
                <div v-for="(card, key) in getBottomCards" :key="key" class="game-card" :data-card-type="card">
                </div>
              </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>