<template>
  <div class="col-9 col-md-6 col-lg-4">
    <div class="form">
      <div v-if="alertMsg" class="alert">{{ alertMsg }}</div>
      <div class="input">
        <label for="player_name">Player Name</label>
        <input class="element" id="player_name" type="text" v-model="_name">
      </div>
      <div class="input radio">
        <label>Number of Player</label>
        <label for="twoPlayer" :class="_number === 2 ? 'checked' : ''">
          <input type="radio" id="twoPlayer" value="2" v-model="_number">
          2
        </label>
        <label for="threePlayer" :class="_number === 3 ? 'checked' : ''">
          <input type="radio" id="threePlayer" value="3" v-model="_number">
          3
        </label>
        <label for="fourPlayer" :class="_number === 4 ? 'checked' : ''">
          <input type="radio" id="fourPlayer" value="4" v-model="_number">
          4
        </label>
      </div>
      <button class="btn" @click="check(); $event.preventDefault();">Play!</button>
    </div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      alertMsg: '',
    }
  },
  props: ['content'],
  created() {
  },
  computed: {
    _number: {
      get() {
        return this.content._number
      },
      set(val) {
        this.$emit('changed', this.objectReturn(parseInt(val), '_number'))
      }
    },
    _name: {
      get() {
        return this.content._name
      },
      set(val) {
        this.$emit('changed', this.objectReturn(val, '_name'))
      }
    }
  },
  methods: {
    objectReturn(val, index) {
      let model = this.content
      model[index] = val
      return model
    },
    check() {
      if (this._name === '') {
        this.alertMsg = 'Player name is required!'
      } else if ([2,3,4].indexOf(this._number) === -1) {
        this.alertMsg = 'Number of player is required!'
      } else {
        this.alertMsg = '';
        this.$emit('starting', true)
      }
    }
  }
}
</script>