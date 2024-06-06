<template>
  <v-container class="ma-5">
    <v-row justify="center">
      <v-col cols="12" class="text-center">
        <h1>FPGA 小车</h1>
      </v-col>
    </v-row>
    <v-row justify="center">
      <v-col cols="2" class="text-center">
        <v-btn icon @click="setRotate('front')">
          <v-icon>mdi-arrow-up</v-icon>
        </v-btn>
      </v-col>
    </v-row>
    <v-row justify="center">
      <v-col cols="2" class="text-center">
        <v-btn icon @mousedown="handleMouseDown('left')" @mouseup="handleMouseUp('none')"
          @touchstart="handleTouchStart('left')" @touchend="handleTouchEnd('none')">
          <v-icon>mdi-arrow-left</v-icon>
        </v-btn>
      </v-col>
      <v-col cols="2" class="text-center">
        <v-btn icon @click="isEnabled = false">
          <v-icon>mdi-stop</v-icon>
        </v-btn>
      </v-col>
      <v-col cols="2" class="text-center">
        <v-btn icon @mousedown="handleMouseDown('right')" @mouseup="handleMouseUp('none')"
          @touchstart="handleTouchStart('right')" @touchend="handleTouchEnd('none')">
          <v-icon>mdi-arrow-right</v-icon>
        </v-btn>
      </v-col>
    </v-row>
    <v-row justify="center">
      <v-col cols="2" class="text-center">
        <v-btn icon @click="setRotate('back')">
          <v-icon>mdi-arrow-down</v-icon>
        </v-btn>
      </v-col>
    </v-row>
    <v-row justify="center" class="mt-5">
      <v-col cols="12" class="text-center">
        <v-slider v-model="speed" @end="updateSpeed()" :max="3" :min="0" step="1" label="速度" class="mt-3"></v-slider>
      </v-col>
    </v-row>
    <v-row justify="center" class="mt-3">
      <v-col cols="12" class="text-center">
        <v-switch v-model="isEnabled" label="开关" class="mt-3"></v-switch>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import { createVuetify } from 'vuetify';

export default {
  data() {
    return {
      isTouching: false,
      rotate: 'front', // 'front' or 'back'
      direction: 'none', // 'right', 'left' or 'none'
      speed: 0, // 0, 1, 2 or 3
      isEnabled: false,
      flag: '00000'
    };
  },
  methods: {
    handleMouseDown(direction) {
      if (!this.isTouching) {
        this.setDirection(direction);
      }
    },
    handleMouseUp(direction) {
      if (!this.isTouching) {
        this.setDirection(direction);
      }
      this.isTouching = false;
    },
    handleTouchStart(direction) {
      this.isTouching = true;
      this.setDirection(direction);
    },
    handleTouchEnd(direction) {
      // this.isTouching = false;
      this.setDirection(direction);
    },
    setRotate(rotate) {
      let directionFlag = '0';
      switch (rotate) {
        case 'front':
          directionFlag = '0';
          break;
        case 'back':
          directionFlag = '1';
          break;
      }
      this.flag = directionFlag + this.flag.slice(1);
    },
    setDirection(direction) {
      let directionFlag = '00';
      switch (direction) {
        case 'right':
          directionFlag = '01';
          break;
        case 'left':
          directionFlag = '10';
          break;
        case 'none':
          directionFlag = '00';
          break;
      }
      this.flag = this.flag.slice(0, 1) + directionFlag + this.flag.slice(3);
    },
    updateSpeed() {
      this.flag = this.flag.slice(0, 3) + this.speed.toString(2).padStart(2, '0');
    },
    sendFlag(flagToSend) {
      fetch(`/status?value=${flagToSend}`)
        .then(response => response.text())
        .then(data => console.log('Current status:', data));
    }
  },
  watch: {
    flag: function (newFlag) {
      if (this.isEnabled) {
        this.sendFlag(newFlag);
      } else if (newFlag !== '00000') {
        this.sendFlag('00000');
      }
    },
    isEnabled: function (newIsEnabled) {
      if (!newIsEnabled) {
        this.sendFlag('00000');
      } else {
        this.sendFlag(this.flag);
      }
    }
  },
  vuetify: createVuetify()
}
</script>
