<template>
    <div>
        <ul class="join-list">
            <li
                v-for="message in joinMessages"
                :key="message"
                class="join-message"
            >
                {{ message }}
            </li>
        </ul>
        <template v-if="readyToPlay">
            <div class="current-card">
                <label>
                    {{ currentCard }}
                </label>
            </div>
            <div class="board">
                <div 
                    v-for="(block, i) in boardBlocks"
                    :key="i"
                    class="block"
                    :class="{ marked: block.marked }"
                >
                    <label>
                        {{ block.index }}
                    </label>
                </div>
            </div>
        </template>
        <template v-else>
            <div>
                Enter your name <input v-model="name" type="text" />
            </div>
            <button @click="submitName">Submit Name</button>
        </template>
    </div>
</template>

<script>
import { io } from 'socket.io-client';
import { shuffle } from 'lodash';
const socket = io('ws://localhost:3500');
// const socket = new WebSocket('ws://localhost:3500');
export default {
    data () {
        return {
            boardBlocks: shuffle([...new Array(100).keys()].map(item => ({ index: item, marked: false }))),
            currentCard: null,
            name: '',
            nameSubmitted: false,
            sessionId: `session-${Math.random().toString().split('.').join('')}-${Math.random().toString().split('.').join('')}-${Math.random().toString().split('.').join('')}`,
            joinMessages: []
        };
    },
    computed: {
        readyToPlay () {
            return this.name && this.nameSubmitted;
        },
    },
    methods: {
        submitName () {
            socket.emit('message', `playerName|${this.name}|${this.sessionId}`);
            this.nameSubmitted = true;
        },
        showNewPlayer (player) {
            this.joinMessages.push(`${player} has joined the game`);
        },
    },
    mounted() {
        console.log(socket);
        socket.on("message", (data) => {
            if (this.readyToPlay && data?.type === 'card') {
                this.currentCard = data.value;
                this.boardBlocks.forEach(item => {
                    if (data.value === item.index) {
                        item.marked = true;
                    }
                });
            }
            else if (data) {

                const dataList = data.toString().split('|');
                if (dataList[0].includes('playerName') && dataList[2] !== this.sessionId) {
                    this.showNewPlayer(dataList[1]);
                }
            }
            
        });
        // socket.on('updates', () => { console.log('update happened')});
    },
}
</script>

<style scoped>
.board{
    display: flex;
    flex-wrap: wrap;
    width: 500px;
    height: 500px;
}
.board .block{
    width: 10%;
    height: 10%;
    display: flex;
    justify-content: center;
    align-items: center;
    box-shadow:  0 0 0 2px #ff0000;
    font-size: 18px;
}
.board .block.marked{
    background-color: #999999;
    color: #ffffff;
}
.current-card{
    width: 200px;
    height: 200px;
    display: flex;
    justify-content: center;
    align-items: center;
    box-shadow:  0 0 0 2px #ff0000;
    font-size: 50px;
}
.join-list{
    margin: 0;
    padding: 0;
    pointer-events: none;
}
@keyframes join-fade {
    0%{
        opacity: 1;
        height: 40px;
    }
    75%{
        opacity: 1;
        height: 40px;
    }
    100%{
        opacity: 0;
        height: 0;
    }
}
.join-list .join-message{
    opacity: 0;
    height: 0;
    animation: join-fade;
    animation-duration: 2s;
    animation-iteration-count: 1;
}
</style>