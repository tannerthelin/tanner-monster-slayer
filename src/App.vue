<template>
  <div id="app">
    <section class="row">
        <div class="small-6 columns">
            <h1 class="text-center">You</h1>
            <div class="healthbar">
                <div class="healthbar text-center" style="background-color: #12B475; margin: 0; color: white;"
                :style="{width: playerHealth + '%'}">
                    {{playerHealth}}
                </div>
            </div>
        </div>
        <div class="small-6 columns">
            <h1 class="text-center">Monster</h1>
            <div class="healthbar">
                <div class="healthbar text-center" style="background-color: #FEB04F; margin: 0; color: white;"
                :style="{width: monsterHealth + '%'}">
                    {{monsterHealth}}
                </div>
            </div>
        </div>
    </section>
    <section class="row controls" v-if="!gameIsRunning">
        <div class="small-12 columns">
            <button id="start-game" @click="startGame">Start New Game</button>
        </div>
    </section>
    <section class="row controls" v-else>
        <div class="small-12 columns">
            <button id="attack" @click="attack">Attack</button>
            <button id="special-attack" @click="specialAttack">Special Attack ({{specialAttacks}})</button>
            <button id="heal" @click="heal">Heal  ({{heals}})</button>
            <button id="give-up" @click="giveUp">Give Up</button>
        </div>
    </section>
    <section class="row log" v-if="turns.length > 0">
        <div class="small-12 columns">
            <ul>
                <li 
                    v-for="turn in turns"
                    v-bind:key = "turn.id"
                    :class="{'player-turn': turn.isPlayer, 'monster-turn': !turn.isPlayer, 'special-attack': turn.isSpecialAttack, 'heal-message': turn.isHealing, 'gave-up-message': turn.gaveUp}">
                    {{ turn.text }}
                </li>
            </ul>
        </div>
    </section>
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'

export default {
  name: 'app',
  components: {
    HelloWorld
  },
  data: function () {
  return {
    playerHealth: 100,
    monsterHealth: 100,
    gameIsRunning: false,
    turns: [],
    heals: 5,
    specialAttacks: 2
    }
  },
    methods: {
        startGame: function() {
            this.gameIsRunning = true;
            this.playerHealth = 100;
            this.monsterHealth = 100;
            this.turns = [];
            this.heals = 5;
            this.specialAttacks = 2;
        },
        attack: function() {
            var damage = this.calculateDamage(5, 12);
            this.monsterHealth -= damage;
            this.turns.unshift({
                isPlayer: true,
                text: 'Player hits Monster for ' + damage
            });
            if (this.checkWin()) {
                return;
            }

            this.monsterAttack();
            this.checkWin();
        },
        specialAttack: function() {
            if (this.specialAttacks >= 1) {
                this.specialAttacks = this.specialAttacks - 1;
            } else {
                this.turns.unshift({
                    gaveUp: true,
                    text: 'No more Special Attacks left!'     
                }); return;
            }
            var damage = this.calculateDamage(10, 20);
            this.monsterHealth -= damage;
            this.turns.unshift({
                isSpecialAttack: true,
                text: 'Player hits Monster hard for ' + damage
            });
            if (this.checkWin()) {
                return;
            }
            this.monsterAttack();
            this.checkWin();
        },
        heal: function() {
            if (this.heals >= 1) {
                this.heals = this.heals - 1;
            } else {
                this.turns.unshift({
                    gaveUp: true,
                    text: 'No more heals left!'
                    
                }); return;
            }
            
            if(this.playerHealth <= 90) {
            this.playerHealth += 10;
            } else {
                this.playerHealth = 100;
            }
            this.turns.unshift({
                isHealing: true,
                text: 'Player heals for 10.'
            });
            this.monsterAttack();
        },
        giveUp: function() {
            this.gameIsRunning = false;
            this.turns.unshift({
                gaveUp: true,
                text: 'Player gives up.'
            });
        },
        monsterAttack: function() {
            var damage = this.calculateDamage(5, 12);
            this.playerHealth -= damage;
            this.checkWin;
            this.turns.unshift({
                isPlayer: false,
                text: 'Monster hits Player for ' + damage
            });
        },
        calculateDamage: function(min, max) {
            return Math.max(Math.floor(Math.random() * max) + 1, min);
        },
        checkWin: function() {
            if (this.monsterHealth <= 0) {
                if (confirm('You won! New Game?')) {
                    this.startGame();
                } else {
                    this.gameIsRunning = false;
                }
                return true;
            } else if (this.playerHealth <= 0) {
                if (confirm('You Lost! New Game?')) {
                    this.startGame();
                } else {
                    this.gameIsRunning = false;
                }
                return true;
            } return false;
        }
    }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
@font-face {
    font-family: "Apercu";
    src: url("/fonts/Apercu-Regular.otf") format("otf"),
  }

body {
    font-family: 'Apercu', sans-serif;
}

h1 {
    font-weight: 300;
}

.text-center {
    text-align: center;
}

.healthbar {
    width: 80%;
    height: 25px;
    background-color: #eee;
    margin: auto;
    transition: width 500ms;
    border-radius: 100px;
}

.controls, .log {
    margin-top: 30px;
    text-align: center;
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 4px;
}

.turn {
    margin-top: 20px;
    margin-bottom: 20px;
    font-size: 22px;
}

.log ul {
    list-style: none;
}

.log ul li {
    margin: 5px;
}

.log ul .player-turn {
    color: #12B475;
    background-color: #DDF8ED;
    border-radius: 100px;
}

.log ul .monster-turn {
    color: #FF3A39;
    background-color: #FFDEDE;
    border-radius: 100px;
}

.log ul .special-attack {
    color: #4A90E2;
    background-color: #DCEAFC;
    border-radius: 100px;
}

.log ul .heal-message {
    color: #A755EF;
    background-color: #F2E6FD;
    border-radius: 100px;
}

.log ul .gave-up-message {
    color: #939393;
    background-color: #F5F5F5;
    border-radius: 100px;
}

button {
    font-size: 20px;
    background-color: #F5F5F5;
    color: #939393;
    padding: 16px 25px;
    margin: 10px;
    border-radius: 100px;
}

button:focus {
    outline: 0;
}

#start-game {
    background-color: #F5F5F5;
}

#start-game:hover {
    background-color: #EFEDED;
}

#attack {
    background-color: #FFDEDE;
    color: #FF5C5C;
}

#attack:hover {
    background-color: #FACFCF;
}

#special-attack {
    background-color: #DCEAFC;
    color: #4A90E2;
}

#special-attack:hover {
    background-color: #D1E3FB;
}

#heal {
    background-color: #F2E6FD;
    color: #A755EF;
}

#heal:hover {
    background-color: #E9D9F8;
}

#give-up {
    background-color: #F5F5F5;
}

#give-up:hover {
    background-color: #EFEDED;
}
</style>
