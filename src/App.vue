<script setup>
import Uno from './components/Uno.vue'
</script>
<template>
  <Uno />
  <div class="offcanvas offcanvas-start" tabindex="-1" id="about" aria-labelledby="aboutLabel">
    <div class="offcanvas-header">
      <h5 class="offcanvas-title" id="aboutLabel">ABOUT VUNO</h5>
      <button type="button" class="btn-close text-reset" data-bs-dismiss="offcanvas" aria-label="Close"></button>
    </div>
    <div class="offcanvas-body">
      <div>
        <p class="lead">VUNO; It is a game developed for hobby purposes with Vue.</p>
      </div>
      <div>
        <h6>Credits</h6>
        <ul>
          <li>
            <a href="https://vuejs.org/" target="_blank" rel="noopener noreferrer">Vue.js</a>
          </li>
          <li>
            <a href="https://getbootstrap.com/" target="_blank" rel="noopener noreferrer">Bootstrap</a>
          </li>
          <li>
            <a href="https://bootswatch.com/" target="_blank" rel="noopener noreferrer">Bootswatch</a>
          </li>
          <li><a href="https://fonts.google.com/specimen/JetBrains+Mono" target="_blank" rel="noopener noreferrer">"JetBrains Mono" through Google Font</a></li>
          <li><a href="https://fonts.google.com/specimen/Press+Start+2P" target="_blank" rel="noopener noreferrer">"Press Start 2P" through Google Font</a></li>
          <li><a href="https://materialdesignicons.com/" target="_blank" rel="noopener noreferrer">Material Design Icons</a></li>
        </ul>
        <small class="copyright">UNO game and format is owned by <a href="https://mattel.com" target="_blank" rel="noopener noreferrer">Mattel.</a></small>
      </div>
      <div>
        <small class="copyright">This game is an open source project. <a target="_blank" href="https://github.com/halillusion/vueno">Contribute.</a></small>
      </div>
    </div>
  </div>
</template>
<style lang="scss">
$main-color: #0C4740;
$main-color-dark: #092522;
$accent-color: #00FFA3;
$accent-color-dark: #0f9262;
$font-primary: 'Press Start 2P', cursive;
$font-secondary: 'JetBrains Mono', monospace;
$red-color: #ca2b2b;
$green-color: #68b313;
$blue-color: #1f52b1;
$yellow-color: #edbc01;
$transition: 0.2s ease-out;

@keyframes textShadow {
  0% {
    text-shadow: 0 0 8px currentColor;
  }

  50% {
    text-shadow: 0 0 20px currentColor;
  }

  100% {
    text-shadow: 0 0 8px currentColor;
  }
}

@keyframes spinLeft { 
  100% { 
    transform: rotateZ(360deg);
  }
}

@keyframes spinRight { 
  100% { 
    transform: rotateZ(-360deg);
  }
  
}

@keyframes downToScene { 
  0% { 
    opacity: 0;
    transform: scale(2);
  }
  100% { 
    opacity: 1;
    transform: scale(1);
  }
}

@keyframes opacityHalf { 
  0% { 
    opacity: 1;
  }
  50% { 
    opacity: 0.5;
  }
  100% { 
    opacity: 1;
  }
}

html {
  // Main Template
  body {
    font-family: $font-primary;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    background: #e7e7e7;
    color: #0C4740;
    transition: $transition;

    h1, h2, h3, h4, h5, h6 {
      font-family: $font-secondary
    }
    img {
      image-rendering: pixelated !important;
    }
    .form-control,
    .form-select {
      background: $accent-color-dark;
      font-family: $font-secondary;
      border-color: $main-color;
      color: $main-color;
      ~ label {
        color: $accent-color;
      }
      &:focus {
        background: $accent-color;
        ~ label {
          color: $accent-color-dark;
        }
      }
      
    }
    .navbar {
      background: $accent-color;
      &.focus-game {
        top: -3rem;
        transition: 0.2s ease-out 0.2s;
        &:hover {
          opacity: 1;
          top: 0rem;
        }
      }
      .navbar-brand {
        transition: $transition;
        font-family: $font-primary;
        padding: 0;
        &.hidden-logo {
          opacity: 0;
        }
        img {
          height: 2rem;
          width: auto;
        }
      }
      .navbar-toggler {
        color: $main-color;
      }
      .navbar-nav {
        .nav-item {
          .nav-link {
            color: $accent-color-dark;
            font-size: 0.8rem;
            &:hover, &:focus, &.active {
              color: $main-color;
            }
          }
        }
      }
    }

    .modal {
      .modal-content {
        background: $main-color-dark;
        color: $accent-color;
        border-color: $accent-color;
        .modal-header {
          border-bottom-color: $accent-color;
        }
      }
    }
    .modal-backdrop {
      background: $main-color;
    }

    .btn {
      font-family: $font-primary;
      &.btn-play {
        background: #0C4740;
        color: #00FFA3;
        border-color: #0C4740;
      }
      &.btn-success {
        background: $green-color;
        color: #fff;
        border-color: $green-color;
      }
      &.btn-danger {
        background: $red-color;
        color: #fff;
        border-color: $red-color;
      }
      &.btn-primary {
        background: $blue-color;
        color: #fff;
        border-color: $blue-color;
      }
      &.btn-warning {
        background: $yellow-color;
        color: #fff;
        border-color: $yellow-color;
      }
    }

    .offcanvas {

      &.offcanvas-start {
        border-right: 2px solid $accent-color;
      }
      background: $main-color;
      color: $accent-color;
      .offcanvas-header {
        .offcanvas-title {
          font-family: $font-primary
        }
        .btn-close {
          image-rendering: pixelated;
        }
      }
      .offcanvas-body {
        font-family: $font-secondary;
        font-weight: 200;
        display: flex;
        flex-direction: column;
        flex-wrap: wrap;
        align-content: center;
        justify-content: space-between;
        align-items: center;

        a {
          color: $accent-color;
          transition: 0.2s ease-out;
          font-weight: 400;
          text-decoration: none;
          &:hover, &:focus {
            color: #ffffff;
          }
        }

        ul {
          list-style-type: square;
        }

        .copyright {
          font-size: 0.7rem
        }
      }
    }

    .offcanvas-backdrop {
      background: $main-color;
      &.show {
        opacity: 0.9;
      }
    }

    .tooltip {
      font-family: $font-secondary;
    }

    .wrap {
      display: block;
      width: 100vw;
      height: 100vh;
      position: relative;
      transition: $transition;

      .start-screen {
        transition: $transition;
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        background: $main-color;
        z-index: 2;
        .logo {
          width: 15rem;
        }
      }

      .game-screen {
        transition: $transition;
        position: relative;
        background: $main-color;
        z-index: 2;
        height: 100%;
        width: 100%;

        .game-area {
          position: absolute;
          transition: $transition;
          top: 0;
          left: 0;
          right: 0;
          bottom: 0;
          width: 100%;
          height: 100%;
        }

        .counter {
          display: none;
        }

        &.count-active {
          .game-area {
            opacity: 0.5;
            pointer-events: none;
            overflow: hidden;
          }

          .counter {
            position: absolute;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            display: flex;
            flex-direction: column;
            flex-wrap: nowrap;
            align-content: center;
            align-items: center;
            justify-content: center;
            text-align: center;
            font-size: 5rem;
            height: 100%;
            width: 100%;
            animation: textShadow 1s infinite ;
            z-index: 3;
            background: rgba($main-color, 0.8);
          }
        }

        .cards {
          display: flex;
          flex-direction: row;
          flex-wrap: nowrap;
          align-content: center;
          justify-content: center;
          align-items: center;
          .game-card {
            height: 8rem;
            width: 5rem;
            background: rgb(255, 255, 255);
            box-shadow: 0 0 0.4rem #092522;
            display: block;
            transition: 0.2s ease-out;
            border-radius: 0.5rem;
            border: 2px solid #fff;
            position: relative;
            cursor: pointer;
            font-family: $font-secondary;
            animation: downToScene 1s;

            &:hover {
              transform: rotateZ(-5deg) translateY(-20%);
              box-shadow: 0 0 0.8rem #092522;
            }
            & + .game-card {
              margin-left: -3rem;
            }

            &:not(.rival)[data-card-type*="red"] {
              background: $red-color;
              color: #fff;
            }
            &:not(.rival)[data-card-type*="green"] {
              background: $green-color;
              color: #fff;
            }
            &:not(.rival)[data-card-type*="blue"] {
              background: $blue-color;
              color: #fff;
            }
            &:not(.rival)[data-card-type*="yellow"] {
              background: $yellow-color;
              color: #fff;
            }
            &:not(.rival)[data-card-type*="joker"] {
              background: #000;
              color: #fff;
            }

            &:not(.rival)[data-card-type] {
              &:before {
                pointer-events: none;
                position: absolute;
                top: -0.1rem;
                left: 0.2rem;
                font-size: 1.2rem;
              }
              &:after {
                pointer-events: none;
                position: absolute;
                top: 0rem;
                left: 0rem;
                bottom: 0rem;
                right: 0rem;
                font-size: 3rem;
                display: flex;
                flex-direction: column;
                flex-wrap: nowrap;
                align-content: center;
                justify-content: center;
                align-items: center;
              }
            }

            &:not(.rival)[data-card-type*=":block"] {
              &:before {
                left: 0.1rem;
                top: 0rem;
                content: url(data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0iVVRGLTgiPz48IURPQ1RZUEUgc3ZnIFBVQkxJQyAiLS8vVzNDLy9EVEQgU1ZHIDEuMS8vRU4iICJodHRwOi8vd3d3LnczLm9yZy9HcmFwaGljcy9TVkcvMS4xL0RURC9zdmcxMS5kdGQiPjxzdmcgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIiB4bWxuczp4bGluaz0iaHR0cDovL3d3dy53My5vcmcvMTk5OS94bGluayIgdmVyc2lvbj0iMS4xIiB3aWR0aD0iMjQiIGhlaWdodD0iMjQiIHZpZXdCb3g9IjAgMCAyNCAyNCI+PHBhdGggZmlsbD0iI2ZmZiIgZD0iTTEyIDJDMTcuNSAyIDIyIDYuNSAyMiAxMlMxNy41IDIyIDEyIDIyIDIgMTcuNSAyIDEyIDYuNSAyIDEyIDJNMTIgNEMxMC4xIDQgOC40IDQuNiA3LjEgNS43TDE4LjMgMTYuOUMxOS4zIDE1LjUgMjAgMTMuOCAyMCAxMkMyMCA3LjYgMTYuNCA0IDEyIDRNMTYuOSAxOC4zTDUuNyA3LjFDNC42IDguNCA0IDEwLjEgNCAxMkM0IDE2LjQgNy42IDIwIDEyIDIwQzEzLjkgMjAgMTUuNiAxOS40IDE2LjkgMTguM1oiIC8+PC9zdmc+);
              }
              &:after {
                transform: scale(2.5);
                top: -1.5rem;
                left: 0.2rem;
                content: url(data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0iVVRGLTgiPz48IURPQ1RZUEUgc3ZnIFBVQkxJQyAiLS8vVzNDLy9EVEQgU1ZHIDEuMS8vRU4iICJodHRwOi8vd3d3LnczLm9yZy9HcmFwaGljcy9TVkcvMS4xL0RURC9zdmcxMS5kdGQiPjxzdmcgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIiB4bWxuczp4bGluaz0iaHR0cDovL3d3dy53My5vcmcvMTk5OS94bGluayIgdmVyc2lvbj0iMS4xIiB3aWR0aD0iMjQiIGhlaWdodD0iMjQiIHZpZXdCb3g9IjAgMCAyNCAyNCI+PHBhdGggZmlsbD0iI2ZmZiIgZD0iTTEyIDJDMTcuNSAyIDIyIDYuNSAyMiAxMlMxNy41IDIyIDEyIDIyIDIgMTcuNSAyIDEyIDYuNSAyIDEyIDJNMTIgNEMxMC4xIDQgOC40IDQuNiA3LjEgNS43TDE4LjMgMTYuOUMxOS4zIDE1LjUgMjAgMTMuOCAyMCAxMkMyMCA3LjYgMTYuNCA0IDEyIDRNMTYuOSAxOC4zTDUuNyA3LjFDNC42IDguNCA0IDEwLjEgNCAxMkM0IDE2LjQgNy42IDIwIDEyIDIwQzEzLjkgMjAgMTUuNiAxOS40IDE2LjkgMTguM1oiIC8+PC9zdmc+);
              }
            }

            &:not(.rival)[data-card-type*=":direction"] {
              &:before {
                transform: rotate(135deg);
                left: -0.4rem;
                top: -0.4rem;
                content: url(data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0iVVRGLTgiPz48IURPQ1RZUEUgc3ZnIFBVQkxJQyAiLS8vVzNDLy9EVEQgU1ZHIDEuMS8vRU4iICJodHRwOi8vd3d3LnczLm9yZy9HcmFwaGljcy9TVkcvMS4xL0RURC9zdmcxMS5kdGQiPjxzdmcgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIiB4bWxuczp4bGluaz0iaHR0cDovL3d3dy53My5vcmcvMTk5OS94bGluayIgdmVyc2lvbj0iMS4xIiB3aWR0aD0iMjQiIGhlaWdodD0iMjQiIHZpZXdCb3g9IjAgMCAyNCAyNCI+PHBhdGggZmlsbD0iI2ZmZmZmZiIgZD0iTTIxLDlMMTcsNVY4SDEwVjEwSDE3VjEzTTcsMTFMMywxNUw3LDE5VjE2SDE0VjE0SDdWMTFaIiAvPjwvc3ZnPg==);
              }
              &:after {
                transform: rotate(135deg) scale(3);
                top: 1.5rem;
                left: 1.5rem;
                content: url(data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0iVVRGLTgiPz48IURPQ1RZUEUgc3ZnIFBVQkxJQyAiLS8vVzNDLy9EVEQgU1ZHIDEuMS8vRU4iICJodHRwOi8vd3d3LnczLm9yZy9HcmFwaGljcy9TVkcvMS4xL0RURC9zdmcxMS5kdGQiPjxzdmcgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIiB4bWxuczp4bGluaz0iaHR0cDovL3d3dy53My5vcmcvMTk5OS94bGluayIgdmVyc2lvbj0iMS4xIiB3aWR0aD0iMjQiIGhlaWdodD0iMjQiIHZpZXdCb3g9IjAgMCAyNCAyNCI+PHBhdGggZmlsbD0iI2ZmZmZmZiIgZD0iTTIxLDlMMTcsNVY4SDEwVjEwSDE3VjEzTTcsMTFMMywxNUw3LDE5VjE2SDE0VjE0SDdWMTFaIiAvPjwvc3ZnPg==);
              }
            }

            &:not(.rival)[data-card-type*=":color"] {
              &:before {
                content: '';
                width: 1.5rem;
                height: 1.5rem;
                border-radius: 50%;
                top: 0.2rem;
                background: #fff;
                background: linear-gradient(90deg, $red-color 0%, $red-color 25%, $green-color 25%, $green-color 50%, $blue-color 50%, $blue-color 75%, $yellow-color 75%, $yellow-color 100%);
              }
              &:after {
                content: '';
                width: 3rem;
                height: 3rem;
                top: 2.4rem;
                left: 0.9rem;
                border-radius: 50%;
                background: #fff;
                background: linear-gradient(90deg, $red-color 0%, $red-color 25%, $green-color 25%, $green-color 50%, $blue-color 50%, $blue-color 75%, $yellow-color 75%, $yellow-color 100%);
              }
            }

            &:not(.rival)[data-card-type*="+2"] {
              &:before {
                content: "+2"
              }
              &:after {
                content: "+2";
              }
            }

            &:not(.rival)[data-card-type*="+4"] {
              &:before {
                content: "+4"
              }
              &:after {
                content: "+4";
              }
            }

            @for $i from 0 through 9 {
                &:not(.rival)[data-card-type*=":#{$i}"] {
                &:before {
                  content: "#{$i}"
                }
                &:after {
                  content: "#{$i}";
                }
              }
            }
          }
        }

        .game-area {
          min-height: calc(100vh - 12rem);
          overflow: hidden;
          
          .game-table {
            transition: $transition;
            display: flex;
            width: 20rem;
            height: 20rem;
            flex-direction: row;
            flex-wrap: nowrap;
            align-content: center;
            justify-content: center;
            align-items: center;
            position: relative;
            border: 0.5rem solid rgba($main-color-dark, 0.8);
            border-radius: 50%;

            &:before {
              position: absolute;
              top: 0;
              left: 0;
              right: 0;
              bottom: 0;
              content: '';
              display: block;
              width: 100%;
              height: 100%;
              border-radius: 50%;
              background-clip: content-box;
              border: 0.3rem dashed rgb(255, 255, 255);
            }

            &.direction-left {
              &:before {
                animation: spinLeft 15s linear infinite;
              }
            }

            &.direction-right {
              &:before {
                animation: spinRight 15s linear infinite;
              }
            }

            &.top {
              border-top-color: currentColor;
            }
            &.bottom {
              border-bottom-color: currentColor;
            }
            &.left {
              border-left-color: currentColor;
            }
            &.right {
              border-right-color: currentColor;
            }

            &.blue {
              background: $blue-color;
              color: $blue-color;
            }

            &.red {
              background: $red-color;
              color: $red-color;
            }

            &.green {
              background: $green-color;
              color: $green-color;
            }

            &.yellow {
              background: $yellow-color;
              color: $yellow-color;
            }

            div {
              & + div {
                margin-left: 0.5rem;
              }
            }
            
            .other-cards {
              cursor: pointer;
              background: $accent-color-dark;
              border: 3px solid #fff;
              border-radius: 0.4rem;
              height: 8rem;
              width: 5rem;
              transition: 0.2s ease-out;
              display: flex;
              flex-direction: column;
              flex-wrap: nowrap;
              align-content: center;
              justify-content: center;
              align-items: center;
              overflow: hidden;
              &:hover {
                transform: scale(1.1);
                z-index: 1;
                border-style: dotted;
              }
              img {
                transform: rotate(315deg);
                width: 12rem;
              }
            }

            .current-card {
              .cards {
                position: relative;
                width: 5rem;
                height: 8rem;
                .game-card {
                  position: absolute;
                  top: 0;
                  left: 0;
                  opacity: 0;
                  & + .game-card {
                    margin: 0;
                  }
                  &:nth-last-child(-n+3) {
                    opacity: 1;
                  }
                  &:nth-child(even) {
                    transform: rotate(2deg);
                  }
                  &:nth-child(odd) {
                    transform: rotate(-2deg);
                  }
                  &:hover {
                    transform: none;
                  }
                }
              }
            }
          }
          .player-side {
            opacity: 0.5;
            transition: 0.2s ease-out;
            pointer-events: none;
            width: 100%;
            display: flex;
            flex-direction: column;
            flex-wrap: nowrap;
            align-content: center;
            justify-content: center;
            align-items: center;

            .player-info {
              margin-bottom: 1rem;
              font-size: 1rem;
              display: flex;
              width: 100%;
              flex-direction: row;
              flex-wrap: nowrap;
              align-content: center;
              justify-content: space-between;
              align-items: center;
              padding: 0 1rem;
              
              .score-badge {
                background: $main-color;
                color: $accent-color;
                padding: 0.2rem 0.5rem;
                border-radius: 0.2rem;
              }

              .name {
                margin: 0 2rem;
              }

              .action-buttons {
                button {
                  border: none;
                  background: none;
                  color: $accent-color-dark;
                  font-family: $font-primary;
                  font-size: 0.8rem;
                  opacity: 0.5;
                  pointer-events: none;
                  &.active {
                    pointer-events: all;
                    opacity: 1;
                  }
                  &:hover {
                    color: $accent-color;
                  }
                }
              }
            }
          }

          .current {
            &.bottom {
              .player-side {
                pointer-events: all;
              }
            }
            .player-side {
              opacity: 1;
              .name {
                animation: 1s opacityHalf infinite
              }
              .cards {
                .game-card {
                  & + .game-card {
                    margin-left: -1rem;
                  }
                }
              }
            }
          }

          .left {
            .player-side {
              transform: rotate(90deg);
            }
          }

          .right {
            .player-side {
              transform: rotate(270deg);
            }
          }

          .top {
            .player-side {
              flex-direction: column-reverse;
              .player-info {
                margin-bottom: 0;
                margin-top: 1rem;
              }
            }
          }
        }
      }
    }

    .wrap {
      position: relative;
      .wrap--game {
        margin-top: 3.4rem;
        margin-bottom: 0;
        height: calc(100vh - 3.4rem);
        width: 100vw;
        overflow: hidden;

        .wrap--game-counter {
          overflow: hidden;
          position: absolute;
          top: 0;
          left: 0;
          right: 0;
          bottom: 0;
          opacity: 0;
          pointer-events: none;

          &.active {
            pointer-events: all;
            opacity: 1;
          }

          .count {
            display: flex;
            flex-direction: column;
            flex-wrap: nowrap;
            align-content: center;
            align-items: center;
            justify-content: center;
            font-size: 5rem;
            animation: textShadow 1s infinite 
          }
        }
      }
    }
  }
  // Dark Theme
  &[data-theme=dark] {
    body {
      background: $main-color-dark;
      color: $accent-color;
      .form-control,
      .form-select {
        border-color: $accent-color;
        color: #fff;
        &:focus {
          color: $main-color-dark;
        }
        ~ label {
          font-size: 0.8rem;
        }
      }
      .navbar {
        background: $main-color;
        .navbar-toggler {
          color: $accent-color;
        }
        .navbar-nav {
          .nav-item {
            .nav-link {
              color: $accent-color-dark;
              font-size: 0.8rem;
              &:hover, &:focus, &.active {
                color: $accent-color;
              }
            }
          }
        }
      }
    }
  }
  // First Load
  &[data-theme=loading] {
    body {
      transition: 0.8s ease-out;
      background: $accent-color;
      * {
        display: none;
      }
    }
  }
}
</style>
