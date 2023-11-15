<template>
    <div class="simon-component">
        <audio ref="green" src="../assets/sounds/1.mp3"></audio>
        <audio ref="red" src="../assets/sounds/2.mp3"></audio>
        <audio ref="yellow" src="../assets/sounds/3.mp3"></audio>
        <audio ref="blue" src="../assets/sounds/4.mp3"></audio>

        <h1>Simons The Game</h1>

        <div id="simon">
            <div class="counter">
                <span>{{ showError ? 'Конец игры' : 'Раунд: ' + displayCount }}</span>
            </div>

            <div class="simon__board">
                <div class="board__btn green" :class="{ glow: soundSectors['green'] }" @click="setInput('green')" />
                <div class="board__btn red" :class="{ glow: soundSectors['red'] }" @click="setInput('red')" />
                <div class="board__btn yellow" :class="{ glow: soundSectors['yellow'] }" @click="setInput('yellow')" />
                <div class="board__btn blue" :class="{ glow: soundSectors['blue'] }" @click="setInput('blue')" />
            </div>
        </div>

        <div class="btns-wrapper">
            <button class="btn green-btn" @click="start">Старт</button>
            <button class="btn red-btn" @click="reset">Рестарт</button>
        </div>

        <div class="level">
            <p>Сложность</p>
            <div class="level__input-wrapper">
                <div>
                    <input type="radio" value="easy" v-model="level" />
                    <label for="easy">Легкий</label>
                </div>
                <div>
                    <input type="radio" value="normal" v-model="level" />
                    <label for="normal">Средний</label>
                </div>
                <div>
                    <input type="radio" value="hard" v-model="level" />
                    <label for="hard">Сложный</label>
                </div>
            </div>
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
            soundSectors: { green: false, red: false, yellow: false, blue: false },
            delayMap: {
                easy: 1500,
                normal: 100,
                hard: 400
            },

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

            const lastUserInput = this.userInput[this.userInput.length - 1];
            const lastSeriesInput = this.series[this.userInput.length - 1];

            // Проверка правильности ввода
            if (lastUserInput !== lastSeriesInput) {
                this.resetGameOnError();
                return;
            }

            // Последний ввод пользователя в серии
            if (this.userInput.length === this.series.length) {
                this.completeRound();
            }
        },

        resetGameOnError() {
            this.userInput = [];
            this.allowInput = false;
            this.showError = true;

            // Если ошибочный ввод, завершаем игру
            setTimeout(this.reset, 2000);
        },

        completeRound() {
            this.allowInput = false;
            this.userInput = [];

            setTimeout(() => {
                this.addRound();
                this.playSeries();
            }, 1000);
        },
        playSoundAndSetGlow(input) {
            this.$refs[input].play()
            this.soundSectors[input] = true
            this.$refs[input].currentTime = 0

            setTimeout(this.clearGlow, 300)
        },
        clearGlow() {
            Object.keys(this.soundSectors).forEach(key => {
                this.soundSectors[key] = false;
            })
        },
        reset() {
            this.clearGlow()

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

            const inputDelay = this.delayMap[this.level] || this.delayMap.easy;
            let serieDelay = 500;

            const playSoundAndSetGlow = (input) => {
                setTimeout(() => {
                    if (this.started) {
                        this.playSoundAndSetGlow(input);
                        this.playingSeries = false;
                        this.allowInput = true;
                    }
                }, serieDelay);
            };

            this.series.forEach((input, index, array) => {
                setTimeout(() => {
                    this.playSoundAndSetGlow(input);
                }, serieDelay);

                if (index === array.length - 1) {
                    playSoundAndSetGlow(input);
                }

                serieDelay += inputDelay;
            });
        },

        randomInput() {
            const colorKeys = Object.keys(this.soundSectors)
            const randomIndex = Math.floor(Math.random() * colorKeys.length)
            return colorKeys[randomIndex]
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
    background-color: transparent;
    font-size: 14px;
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

.level__input-wrapper {
    display: flex;
    justify-content: center;
    gap: 0.5rem;
}
</style>
  