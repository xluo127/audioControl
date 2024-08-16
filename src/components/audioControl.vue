<template>
    <div>
        <audio ref="audio" :src="audioSrc" @timeupdate="onTimeUpdate" @ended="onEnded"></audio>
        <div class="controls">
            <img
                style="width: 100%"
                :src="audioGifSrc"
                alt=""
            />
            <input style="width: 100%" type="range" class="progress" min="0" max="100" :value="progress" @input="seek" />
            <div style="position: relative; height: 20px">
                <span class="current-time">{{ currentTime }}</span>
                <span class="duration">{{ duration }}</span>
            </div>
            <div style="position: relative; height: 30px">
                <img class="play-pause" v-if="!isPlaying" src="/audioPlay.svg" alt="播放" @click="togglePlayPause">
                <img class="play-pause" v-else  src="/audioPause.png" alt="暂停" @click="togglePlayPause">
                <div class="speed-set" @click="setSpeed"><span class="speed-set-text">{{ speed + 'x' }}</span></div>
            </div>
        </div>
    </div>
</template>

<script setup name="audioControlComponent">
    import { ref, onMounted } from 'vue'
	const props = defineProps({
        src: String
    })
	onMounted(() => {
        // 父组件import当前文件，更新audioSrc值
        audioSrc.value = props.src
		audio.value.addEventListener('loadedmetadata', () => {
			duration.value = formatTime(audio.value.duration);
		});
	})
	// 多模态数据展示--start
    const audioSrc = ref(null)
	// const audioSrc = ref('https://www.nivic.cn/cxzx/weixin/fast100per.mp3')
	const imgPreviewVisible = ref(false)
	const previewImg = () => {
		imgPreviewVisible.value = true
	}
	// 多模态数据展示--end
	// 音频播放--start
	const audio = ref(null);
	const isPlaying = ref(false);
	const progress = ref(0);
	const currentTime = ref('00:00');
	const duration = ref('00:00');
	const audioGifSrc = ref('/audioPlaying.jpg')
	const audioGifSrc0 = ref('/audioPlaying.jpg')
	const audioGifSrc1 = ref('/audioPlaying.gif')

	function togglePlayPause() {
	if (isPlaying.value) {
		audio.value.pause();
		audioGifSrc.value = audioGifSrc0.value
	} else {
		audio.value.play();
		audioGifSrc.value = audioGifSrc1.value
	}
	isPlaying.value = !isPlaying.value;
	}

	function onTimeUpdate(e) {
        const current = e.target.currentTime;
        const total = e.target.duration;
        const percent = (current / total) * 100;
        progress.value = percent.toFixed(2);

        // 更新当前时间和总时长
        currentTime.value = formatTime(current);
        duration.value = formatTime(total);

        // js控制渐变样式
        console.log('渐变比例', progress.value, ((1 - current / total) * 100).toFixed(0))
        document.documentElement.style.setProperty('--start-color', (percent).toFixed(0) + '%');
        document.documentElement.style.setProperty('--end-color', ((1 - current / total) * 100).toFixed(0) + '%')
	}

	function onEnded() {
		isPlaying.value = false;
		audioGifSrc.value = audioGifSrc0.value
	}

	function seek(e) {
		const percentage = e.target.value;
		const seekTo = (percentage * audio.value.duration) / 100;
		audio.value.currentTime = seekTo;
	}

	const speed = ref('1.0');
	const speedList = ref(['1.0', '1.5', '2.0']);
	function setSpeed() {
		const currentI = speedList.value.indexOf(speed.value);
		if (currentI === -1 || currentI >= speedList.value.length - 1) {
			speed.value = speedList.value[0];
		} else {
			speed.value = speedList.value[currentI + 1];
		}
		audio.value.playbackRate = parseFloat(speed.value);
	}

	function formatTime(time) {
        const minutes = Math.floor(time / 60);
        let seconds = Math.floor(time % 60);
        seconds = seconds < 10 ? '0' + seconds : seconds;
        return `${minutes}:${seconds}`;
	}
	
</script>

<style scoped>
.controls {
  	margin-top: 10px;
	width: 100%;
}

.play-pause {
    font-size: 20px;
    cursor: pointer;
    margin-right: 10px;
}

.progress {
    width: 100%;
    margin-right: 10px;
}

.current-time {
	position: absolute;
 	left: 0;
}
.duration {
  	position: absolute;
 	right: 0;
}
.play-pause {
	position: absolute;
 	left: 34%;
	height: 26px;
}
.speed-set {
  	position: absolute;
 	right: 34%;
	height: 26px;
	width: 26px;
	background: #eeeeee;
	border-radius: 50%;
	text-align: center;
	cursor: pointer;
}
.speed-set-text {
	height: 26px;
	font-size: 8px;
	font-family: PingFang SC;
	font-weight: 600;
	text-align: LEFT;
	color: #000000;
	line-height: 26px;
}
.speed {
  margin-left: 10px;
}

/* 进度条样式 */
.progress {
  	-webkit-appearance: none;
	height: 3px;
	background: #eeeeee;
	border-radius: 11px;
  	outline: none;
  	transition: opacity 0.2s;
}

.progress::-webkit-slider-thumb {
	-webkit-appearance: none;
	appearance: none;
	width: 1px;
	height: 3px;
	background: #eeeeee;
    cursor: pointer;
}

.progress::-moz-range-thumb {
  	width: 1px;
	height: 3px;
	background: #eeeeee;
    cursor: pointer;
}

/* 已播放区域颜色 */
.progress::-webkit-slider-runnable-track {
    background: linear-gradient(to right, #2196F3 0%, #2196F3 var(--start-color), #e7eaec var(--start-color), #e7eaec 100%);
}

.progress::-moz-range-track {
    background: linear-gradient(to right, #2196F3 0%, #2196F3 var(--start-color), #e7eaec var(--start-color), #e7eaec 100%);
}
</style>