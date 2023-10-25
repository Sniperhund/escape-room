<script setup>
import { Webhook } from "discord-webhook-node"

const hook = new Webhook('https://discord.com/api/webhooks/1163742970785321000/37oAk5jp5cYlLqL55Dus-QFvrrMWtgRDemBsdgrrRQSCIwbTrbhYXRGJlN1h-NXusbuu')

/* ------- OPTIONS ------- */
const successCode = '1111'
const hintCode = '2222'
/* ------- OPTIONS ------- */

const hasStarted = useState('hasStarted', () => false)
const hasShowedStartVideo = useState('hasShowedStartVideo', () => false)
const showingHintVideo = useState('showingHintVideo', () => false)
const timeLeft = useState('timeLeft', () => 900)
const code = useState('code', () => [-1, -1, -1, -1])
const backgroundColor = useState('backgroundColor', () => 'black')

let currentCodeIndex = 0
let pressedKeys = {}

const keyPress = (key) => {
	if (showingHintVideo.value || backgroundColor == 'red')
		return
	if ((key >= '1' && key <= '9') || key == '0') {
		code.value[currentCodeIndex] = key
		currentCodeIndex++

		if (currentCodeIndex == 4) {
			if (getCodeInDisplayMode() == successCode) {
				hook.send("Someone guessed the correct code...")
				navigateTo({ path: "/done" })
			} else if (getCodeInDisplayMode() == hintCode) {
				code.value = [-1, -1, -1, -1]
				currentCodeIndex = 0
				showingHintVideo.value = true
			} else {
				console.log("You suck...")

				code.value = [-1, -1, -1, -1]
				currentCodeIndex = 0

				backgroundColor.value = 'red'
				setTimeout(() => backgroundColor.value = 'black', 1000)
			}
		}
	}
}

function showStartVideo() {
	hasStarted.value = true
}

function startVideoEnded() {
	hasShowedStartVideo.value = true

	setInterval(() => {
		if (timeLeft.value > 0) {
			timeLeft.value--
		}
	}, 1000)

	window.addEventListener('keydown', (key) => {
		var key = key.key
		if (pressedKeys[key])
			return
		if ((key >= '1' && key <= '9') || key == '0') {
			pressedKeys[key] = true
			keyPress(key)
		}
	});

	window.addEventListener('keyup', (key) => {
		var key = key.key
		pressedKeys[key] = false
	})
}

function getTimeLeft() {
	const timeLeftInMinutes = Math.floor(timeLeft.value / 60)
	const timeLeftInSeconds = timeLeft.value % 60
	return `${timeLeftInMinutes}:${timeLeftInSeconds.toString().padStart(2, '0')}`
}

function getCodeInDisplayMode() {
	let stringCode = ''
	for (const number of code.value) {
		if (number == -1) {
			stringCode = stringCode + "_"
		} else {
			stringCode = stringCode + number.toString()
		}
	}
	return stringCode
}
</script>

<template>
    <div class="background" :style="{backgroundColor: backgroundColor}">
		<div v-if="backgroundColor == 'black'">
			<button v-if="!hasStarted" @click="showStartVideo()">Start</button>
			<div v-else v-if="!hasShowedStartVideo">
				<video autoplay @ended="startVideoEnded()">
					<source src="sample.mp4" />
				</video>
			</div>
			<div class="hintVideoDiv" v-if="showingHintVideo">
				<video autoplay @ended="showingHintVideo = false">
					<source src="sample.mp4" />
				</video>
			</div>
			<div v-if="hasShowedStartVideo" class="flex">
				<h2>Time left: {{getTimeLeft()}}</h2>
				<h2 class="code">{{getCodeInDisplayMode()}}</h2>
			</div>
		</div>
    </div>
</template>

<style scoped>
.background {
    width: 100vw;
    height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
    color: white;
    font-family: Courier, monospace;
    font-size: 3rem;
	overflow: hidden;
}

.hintVideoDiv {
	position: absolute;
	width: 100vw;
	height: 100vh;
	overflow: hidden;
	top: 0;
	bottom: 0;
	left: 0;
	right: 0;
}

.flex {
	display: flex;
	align-items: center;
	flex-direction: column;
}

h2.code {
	letter-spacing: 3rem;
	padding-left: 1.5rem;
}

button {
	scale: 4;
}

video {
	width: 100vw;
	height: 100vh;
}
</style>