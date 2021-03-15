<template>
    <div id="app">
        <img src="./assets/dragon.png">
        <div class="container">
            <div class="player">
                <h2>Jogador</h2>
                <div>
                    <div class="hp" :style="playerLifeStyle"></div>
                </div>
                <p>HP: {{ playerLife }} / 100</p>
            </div>
            <div class="monster">
                <h2>Monstro</h2>
                <div>
                    <div class="hp" :style="monsterLifeStyle"></div>
                </div>
                <p>HP: {{ monsterLife }} / 100</p>
            </div>
        </div>
        <div v-if="lossMessage || winMessage || surrenderMessage" class="win-condition">
            <span v-if="lossMessage" class="loss">Você perdeu! ☔</span>
            <span v-else-if="winMessage" class="win">Você ganhou! ✨</span>
            <span v-else-if="surrenderMessage" class="surrender">Você desistiu 🏳️</span>
        </div>
        <div class="skills">
            <button @click="startGame" v-if="onGame">Iniciar novo jogo</button>
            <button v-if="tryAgain" @click="playAgain">Jogar novamente</button>
            <div v-else-if="skills">
                <Attack @emitAttack="attack"/>
                <SpecialAttack @emitSpecialAttack="specialAttack"/>
                <Heal @emitHeal="heal" :disabled="isDisabled"/>
                <Surrender @emitSurrender="surrender"/>
            </div>
        </div>
        <div class="logs" v-if="logs != false">
            <h2>Historico de batalha</h2>
            <ul>
                <li v-for="(log, index) in logs" :key="index">{{ log }}</li>
            </ul>
        </div>
    </div>
</template>

<script>

import Attack from './components/Attack'
import SpecialAttack from './components/SpecialAttack'
import Heal from './components/Heal'
import Surrender from './components/Surrender'

export default {
    name: 'App',
    components: {
        Attack,
        SpecialAttack,
        Heal,
        Surrender
    },
    data() {
        return {
            onGame: true,
            playerLife: 100,
            monsterLife: 100,
            playerLifeStyle: `width: ${ 100 }%`,
            monsterLifeStyle: `width: ${ 100 }%`,
            skills: false,
            disabledStatus: true,
            lossMessage: false,
            winMessage: false,
            surrenderMessage: false,
            tryAgain: false,
            logs: [],
        }
    },
    computed: {
        isDisabled() {
            this.healControl()
            return this.disabledStatus
        },
    },
    watch: {
        playerLife() {
            if (this.playerLife >= 100)
                this.disabledStatus = true
            if (this.playerLife == 100)
                this.playerLifeStyle = this.playerLife
        },
        monsterLife() {
            if (this.monsterLife == 100)
                this.monsterLifeStyle = this.monsterLife
        },
    },
    methods: {
        // Métodos que fazem executar o início de jogo
        startGame() {
            this.skills = true
            this.onGame = !this.onGame
            if (this.surrenderMessage)
                this.surrenderMessage = !this.surrenderMessage
        },
        playAgain() {
            this.skills = true
            if (this.tryAgain) {
                this.tryAgain = !this.tryAgain
                this.playerLife = 100
                this.monsterLife = 100
                this.logs = []
            }
            if (this.winMessage)
                this.winMessage = !this.winMessage
            else if (this.lossMessage)
                this.lossMessage = !this.lossMessage
        },

        // Métodos que serão chamados por outros métodos
        playerLosesLife(maxPlayerLife = 10, minPlayerLife = 7) { // Valores padrões max e min = (10, 7)
            this.playerLife -= Math.floor(Math.random() * (maxPlayerLife - minPlayerLife) + minPlayerLife)
        },
        monsterLosesLife(maxMonsterLife = 9, minMonsterLife = 6) { // Valores padrões max e min = (9, 6)
            this.monsterLife -= Math.floor(Math.random() * (maxMonsterLife - minMonsterLife) + minMonsterLife)
        },
        validateWinCondition(player, monster) {
            if (this.playerLife < 0) {
                if (this.monsterLife < 0)
                    this.monsterLife = 0

                this.playerLife = 0
                this.lossMessage = !this.lossMessage
                this.skills = !this.skills
                this.tryAgain = !this.tryAgain
            } else if (this.monsterLife < 0) {
                this.monsterLife = 0
                this.skills = !this.skills
                this.tryAgain = !this.tryAgain
                this.winMessage = !this.winMessage
            } else {
                player
                monster
            }
        },
        healControl() {
            if (this.playerLife < 100)
                this.disabledStatus = false
        },
        lifeControl() {
            this.playerLifeStyle = `width: ${ this.playerLife }%`
            this.monsterLifeStyle = `width: ${ this.monsterLife }%`
            
            if (this.playerLife <= 20)
                this.playerLifeStyle = `background: linear-gradient(0deg, rgb(180, 50, 50) 0%, rgb(100, 0, 0) 100%);
                    width: ${ this.playerLife }%;`
            if (this.monsterLife <= 20)
                this.monsterLifeStyle = `background: linear-gradient(0deg, rgb(180, 50, 50) 0%, rgb(100, 0, 0) 100%);
                    width: ${ this.monsterLife }%;`
        },
        pushToLogs(aux, aux2) {
            this.logs.push(`Monstro deu ${ aux - this.playerLife } de dano no Jogador`,
                `Jogador deu ${ aux2 - this.monsterLife } de dano no Monstro`)
            return this.logs
        },

        // Métodos das habilidades
        attack() {
            let oldPlayerLife = this.playerLife
            let oldMonsterLife = this.monsterLife
            
            this.validateWinCondition(this.playerLosesLife(), this.monsterLosesLife())
            this.pushToLogs(oldPlayerLife, oldMonsterLife)
            this.lifeControl()
        },
        specialAttack() {
            let oldPlayerLife = this.playerLife
            let oldMonsterLife = this.monsterLife

            this.validateWinCondition(this.playerLosesLife(), this.monsterLosesLife(12, 8))
            this.pushToLogs(oldPlayerLife, oldMonsterLife)
            this.lifeControl()
        },
        heal() {
            let helper = this.playerLife + Math.floor(Math.random() * (12 - 5) + 5)

            if (helper > 100) {
                this.playerLife = 100
                this.playerLosesLife(12, 0)
                this.lifeControl()
            } else {
                this.playerLife = helper
                this.playerLosesLife(12, 0)
                // Ainda não consegui implementar um lógica para isso.
                // this.logs.push(``) 
                this.lifeControl()
            }
        },
        surrender() {
            this.playerLife = 100
            this.monsterLife = 100
            this.surrenderMessage = !this.surrenderMessage
            this.onGame = !this.onGame
            this.skills = !this.skills
            this.logs = []
        },
    },
}
</script>

<style lang="postcss">

* {
    @apply m-0 p-0 box-border;
}

body {
    background-color: #191A21;
}

#app {
    @apply m-5 grid;
}

#app > img {
    @apply w-20 xl:w-24 m-0 mb-5 justify-self-center;
}

.container {
    background-color: #21222C;
    padding: 50px;
    border-radius: 10px;
    display: grid;
    grid-template-columns: 1fr;
    grid-gap: 50px;
}

.player, .monster {
    text-align: center;
    color: #fff;
    display: flex;
    flex-direction: column;
    gap: 10px;
}

.player > div, .monster > div {
    width: 100%;
    height: 80px;
    border: 3px solid #343746;
    border-radius: 15px;
    background-color: #343746;
}

.player > div > .hp, .monster > div > .hp {
    width: 100%;
    height: 100%;
    border-radius: 13px;
    background: linear-gradient(0deg, rgb(50, 180, 100) 0%, rgb(0, 100, 100) 100%);
}

.player > h2, .monster > h2 {
    margin-bottom: 10px;
    font-size: 2.5rem;
    text-transform: uppercase;
    letter-spacing: 2px;
}

.player > p, .monster > p {
    font-size: 1.5rem;
    font-weight: 600;
}

.skills, .win-condition, .logs {
    background-color: #21222C;
    padding: 20px;
    margin: 15px 0;
    border-radius: 10px;
    display: flex;
    justify-content: center;
}

.skills > div {
    display: flex;
    gap: 20px;
}

.skills > button, .skills > div > button {
    padding: 30px;
    border-radius: 15px;
    border: none;
    background-color: #343746;
    cursor: pointer;
    border: 2px dashed #525356;
    outline: none;
    color: #ccc;
    font-size: 1.5rem;
    font-weight: 600;
}

.skills > div > button {
    border-radius: 30px;
}

.skills > div > button > img {
    user-select: none;
    -webkit-user-drag: none;
    width: 64px;
}

.skills > div > button:disabled {
    background: lightcoral;
    border: 2px dashed rgb(255, 159, 159);
    transform: translate(0) scale(1);
}


.win-condition {
    font-weight: 600;
    width: 100%;
    text-align: center;
    margin-bottom: 0;
    font-size: 2rem;
}

.win {
    color: rgb(50, 180, 100);
}

.loss {
    color: rgb(180, 50, 50);
}

.logs {
    display: flex;
    flex-direction: column;
    color: #ccc;
    margin: 0;
    list-style: none;
    line-height: 25px;
    font-weight: 600;
    text-align: center;
}

.logs > h2 {
    margin: 20px 0 50px;
    font-size: 2.5rem;
}

.logs > ul {
    list-style: none;
    display: grid;
    gap: 10px;
    font-size: 2rem;
}

.logs > ul > li {
    padding: 30px 0;
    margin: 5px 0;
    border-radius: 15px;
}

.logs > ul > li:nth-child(2n+1) {
    background-color: rgb(180, 50, 50);
}

.logs > ul > li:nth-child(2n+2) {
    background-color: rgb(131, 141, 199);
}

@media only screen and (min-width: 1024px) {
    #app {
        width: 75%;
        margin: 20px auto;
    }

    .container {
        grid-template-columns: 1fr 1fr;
        padding: 50px;
    }

    .player > div, .monster > div {
        height: 40px;
        border-radius: 8px;
    }

    .player > div > .hp, .monster > div > .hp {
        border-radius: 5px;
    }

    .player > h2, .monster > h2 {
        font-size: 1.4rem;
    }

    .player > p, .monster > p {
        font-size: 1rem;
    }

    .win-condition {
        font-size: 1.2rem;
    }

    .skills > button, .skills > div > button {
        padding: 15px;
        font-size: 1rem;
        border-radius: 5px;
        transition: .3s;
    }

    .skills > div > button:hover, .skills > button:hover {
        transform: translateY(2px);
    }

    .skills > div > button:active, .skills > button:active {
        transform: translateY(0);
        transform: scale(0.95);
    }

    .skills > div > button:disabled {
        transform: translate(0) scale(1);
    }

    .skills > div > button > img {
        user-select: none;
        -webkit-user-drag: none;
        width: 32px;
    }

    .logs > h2 {
        font-size: 1.4rem;
        margin: 5px 0 25px;
    }

    .logs > ul {
        font-size: 1rem;
    }

    .logs > ul > li {
        padding: 15px 0;
        border-radius: 8px;
        margin: 0px;
    }
}
</style>
