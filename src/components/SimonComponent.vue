<template>
    <div class="simon-component">
        <audio ref="sound1" src="../assets/sounds/1.mp3"></audio>
        <audio ref="sound2" src="../assets/sounds/2.mp3"></audio>
        <audio ref="sound3" src="../assets/sounds/3.mp3"></audio>
        <audio ref="sound4" src="../assets/sounds/4.mp3"></audio>

        <h1>Simons The Game</h1>

        <div id="simon">
            <div class="counter">
                <span>{{ showError ? 'Конец игры' : 'Раунд: ' + displayCount }}</span>
            </div>

            <div class="simon__board">
                <div class="board__btn green" :class="{ glow: activeGreen }" @click="setInput(1)" />
                <div class="board__btn red" :class="{ glow: activeRed }" @click="setInput(2)" />
                <div class="board__btn yellow" :class="{ glow: activeYellow }" @click="setInput(3)" />
                <div class="board__btn blue" :class="{ glow: activeBlue }" @click="setInput(4)" />
            </div>
        </div>

        <div class="btns-wrapper">
            <button class="btn green-btn" @click="start">Старт</button>
            <button class="btn red-btn" @click="reset">Рестарт</button>
        </div>

        <div class="level">
            <p>Сложность</p>
            <input type="radio" value="easy" v-model="level" />
            <label for="easy">Легкий</label>
            <input type="radio" value="normal" v-model="level" />
            <label for="normal">Средний</label>
            <input type="radio" value="hard" v-model="level" />
            <label for="hard">Сложный</label>
        </div>
    </div>
</template>
  
<script>

export default {
    name: 'SimonComponent',
    components: {
    },
    computed: {
        displayCount() {
            if (this.roundsCount == 0) return "----";
            else return this.roundsCount;
        },
    },
    data() {
        return {
            activeGreen: false,
            activeRed: false,
            activeYellow: false,
            activeBlue: false,

            level: "easy",
            started: false,
            roundsCount: 0,
            series: [],
            playingSeries: false,
            userInput: [],

            allowInput: false,
            showError: false,
        }
    },
    methods: {
        setInput(input) {
            if (!this.allowInput) return;

            this.playSoundAndSetGlow(input);
            this.userInput.push(input);

            // compare user input to correct input
            if (this.userInput[this.userInput.length - 1] !== this.series[this.userInput.length - 1]) {
                this.userInput = [];
                this.allowInput = false;
                this.showError = true;
                // if wrong, end the game
                setTimeout(this.reset, 2000);
                return;
            }

            // last user input in serie
            if (this.userInput.length == this.series.length) {
                this.allowInput = false
                this.userInput = []

                setTimeout(() => {
                    this.addRound()
                    this.playSeries()
                }, 1000);
            }
        },
        playSoundAndSetGlow(input) {
            switch (input) {
                case 1:
                    this.$refs.sound1.play();
                    this.$refs.sound1.currentTime = 0;
                    this.activeGreen = true;
                    break;
                case 2:
                    this.$refs.sound2.play();
                    this.$refs.sound1.currentTime = 0;
                    this.activeRed = true;
                    break;
                case 3:
                    this.$refs.sound3.play();
                    this.$refs.sound1.currentTime = 0;
                    this.activeYellow = true;
                    break;
                case 4:
                    this.$refs.sound4.play();
                    this.$refs.sound1.currentTime = 0;
                    this.activeBlue = true;
                    break;
            }
            setTimeout(this.clearGlow, 300);
        },
        clearGlow() {
            this.activeGreen = false;
            this.activeRed = false;
            this.activeYellow = false;
            this.activeBlue = false;
        },
        reset() {
            this.activeRed = false,
                this.activeGreen = false,
                this.activeYellow = false,
                this.activeBlue = false,

                this.started = false,
                this.roundsCount = 0,
                this.series = [],
                this.playingSeries = false,
                this.userInput = [],

                this.allowInput = false,
                this.showError = false
        },
        start() {
            if (this.roundsCount == 0) {
                this.started = true
                this.addRound()
                this.playSeries()
            }
        },
        addRound() {
            this.roundsCount++
            this.series.push(this.randomInput())
        },
        playSeries() {
            this.allowInput = false; // input is restricted
            this.playingSeries = true;
            let serieDelay = 500;
            let inputDelay = 1500;

            switch (this.level) {
                case "easy":
                    inputDelay = 1500;
                    break;
                case "normal":
                    inputDelay = 1000;
                    break;
                case "hard":
                    inputDelay = 400;
                    break;
            }

            this.series.forEach((input, index, array) => {
                if (index == array.length - 1) {
                    setTimeout(() => {
                        if (this.started) {
                            this.playSoundAndSetGlow(input);
                            this.playingSeries = false;
                            this.allowInput = true;
                        }
                    }, serieDelay);
                }

                else
                    setTimeout(() => {
                        this.playSoundAndSetGlow(input);
                    }, serieDelay);

                serieDelay += inputDelay;
            });
            this.playingSeries = false;
        },
        randomInput() {
            return Math.floor(Math.random() * 4) + 1
        },
    }
}
</script>
  
<style scoped>
.simon__board {
    display: flex;
    flex-wrap: wrap;
    width: 305px;
    height: 310px;
    justify-content: space-between;
    align-items: center;
    margin: 0 auto;
}

.board__btn {
    cursor: pointer;
    width: 150px;
    height: 150px;
    border-radius: 10px;
}

.green {
    background-color: #2c771d;
}

.red {
    background-color: #c91943;
}

.yellow {
    background-color: #eedc54;
}

.blue {
    background-color: #4756de;
}

.green.glow {
    background-color: #81da6f;
}

.red.glow {
    background-color: #ff6c8e;
}

.yellow.glow {
    background-color: #fffbbd;
}

.blue.glow {
    background-color: #a7b0ff;
}

.btns-wrapper {
    display: flex;
    justify-content: center;
    gap: 10px;
}

.btn {
    cursor: pointer;
    margin-top: 12px;
    margin-bottom: 24px;
    outline: none;
    border: 2px solid #2c3e50;
    border-radius: 8px;
    width: 100px;
    padding: 8px;
    transition: background-color 0.1s linear;
}

.btn {
    transition: all 0.1s linear;
}

.green-btn:hover {
    color: white;
    background-color: green;
}

.red-btn:hover {
    color: white;
    background-color: red;
}

.counter {
    margin-bottom: 1rem;
}

input[type="radio"] {
    margin-right: 5px;
}

input[type="radio"]+label {
    cursor: pointer;
}
</style>
  