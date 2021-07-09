<template>
    <div id="music-box-container" class="shadow">
        <div style="flex: 3; margin: 0; justify-content: space-around">
            <div style="position: relative">
                <div id="music-box-time">{{ duration2String(now) }} / {{ duration2String(duration) }}</div>
                <el-slider
                    id="music-box-progress"
                    :disabled="playing === 'false' ? true : false"
                    v-model="percentage"
                    :show-tooltip="false"
                    @change="handleProgress"
                ></el-slider>
                <div id="music-box-photo" class="shadow"></div>
                <a id="music-box-input-a" @click="chooseFile"></a>
                <input id="music-box-input" style="display: none" type="file" @change="handleChange" accept=".mp3" />
            </div>
        </div>
        <div style="flex: 2">
            <div style="position: relative">
                <el-button
                    id="music-box-play-btn"
                    style="flex: 1"
                    :icon="playing === 'true' ? 'el-icon-video-pause' : 'el-icon-video-play'"
                    aria-checked="false"
                    @click="handlePlay"
                ></el-button>
                <div id="music-box-title">{{ title }}</div>
            </div>
        </div>
    </div>
</template>

<script>
const AudioContext = window.AudioContext || window.webkitAudioContext;

const reader = new FileReader();
const audioContext = new AudioContext();
let source;

export default {
    data() {
        return {
            now: 0,
            duration: 0,
            fileObj: null,
            title: "请选择mp3文件",
            percentage: 0,
            playing: "false",
            timer: 0,
        };
    },
    methods: {
        chooseFile() {
            let input = document.getElementById("music-box-input");
            input.click();
        },
        handleChange(e) {
            if (e.target.files === null || e.target.files === undefined) return;
            this.fileObj = e.target.files[0];
            reader.readAsArrayBuffer(this.fileObj);
        },
        handlePlay() {
            if (this.fileObj === null) {
                this.chooseFile();
                return;
            }
            if (this.playing === "true") {
                audioContext.suspend();
                this.playing = "suspend";
            } else if (this.playing === "false") {
                source.start(this.now);
                this.playing = "true";
            } else if (this.playing === "suspend") {
                audioContext.resume();
                this.playing = "true";
            } else if (this.playing === "changed") {
                source.stop();
                source.start(this.now);
                this.playing = "true";
            }
        },
        handleProgress(value) {
            console.log(value);
            this.percentage = value;
            if (this.fileObj === null || audioContext === null) return;
            if (this.playing === "true") {
                audioContext.suspend();
            }
            this.now = (this.duration * this.percentage) / 100;
            this.playing = "changed";
        },
        duration2String(dur) {
            if (dur === 0) return "00:00";
            var date = new Date(0);
            date.setSeconds(Math.ceil(dur));
            if (isNaN(+date)) return "--:--";
            return date.toISOString().substr(14, 5);
        },
    },
    created() {
        this.timer = setInterval(() => {
            if (this.fileObj === null || audioContext === null) return;
            if (this.playing === "true") {            if (this.fileObj === null || audioContext === null) return;
            if (this.playing === "true") {
                this.now += 1;
                this.percentage = Math.floor((this.now / this.duration) * 100);
            }
            console.log(this.now, this.percentage, audioContext.currentTime);
        }}, 1000);
        let _this = this;
        reader.onload = function (e) {
            audioContext.decodeAudioData(e.target.result, function (buffer) {
                _this.duration = buffer.duration;
                _this.now = 0;
                _this.title = _this.fileObj.name;
                source = audioContext.createBufferSource();
                source.buffer = buffer;
                source.connect(audioContext.destination);
                source.loop = false;
            });
        };
    },
    beforeDestroy() {
        clearInterval(this.timer);
        audioContext.close();
    },
};
</script>

<style>
body {
    background-color: rgb(243, 210, 229);
}

#music-box-container {
    display: flex;
    flex-direction: column;
    height: 100px;
    width: 190px;
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    border-radius: 15%;
    background-color: whitesmoke;
}

.shadow {
    position: relative;
    max-width: 200px;
    box-shadow: 0px 8px 10px rgba(0, 0, 0, 0.4), 0px 3px 20px rgba(0, 0, 0, 0.1) inset;
}

#music-box-time {
    width: 100px;
    height: 14px;
    position: absolute;
    margin-left: 10px;
    margin-top: 20px;
    font-family: "Microsoft YaHei", Georgia, Times, "Times New Roman", serif;
    font-size: 12px;
}

#music-box-progress {
    position: absolute;
    width: 120px;
    margin-top: 30px;
    margin-left: 5px;
    border-radius: 0% !important;
}

#music-box-progress .el-slider__button {
    width: 10px;
    height: 10px;
    margin-left: 7px;
    border: 0.8px solid #f90000;
}

#music-box-progress .el-slider__bar {
    background-color: #f90000;
}

#music-box-photo {
    position: absolute;
    height: 20px;
    width: 20px;
    margin-top: -20px;
    margin-left: 130px;
    border: 30px solid rgb(48, 48, 48);
    border-radius: 50%;
    border-image: url("img/test.jpg") no-repeat;
    background-position: center;
}

#music-box-input-a {
    position: absolute;
    height: 80px;
    width: 80px;
    margin-left: 130px;
    margin-top: -20px;
    border-radius: 50%;
    opacity: 0;
}

#music-box-play-btn {
    position: absolute;
    border: none;
    border-radius: 50%;
    padding: 0;
    width: 30px;
    min-height: 30px;
    margin-left: 8px;
    font-size: 30px;
    opacity: 30;
    background: none;
}

.el-button:hover,
.el-button:active,
.el-button:focus {
    color: #606266 !important;
}

#music-box-title {
    width: 120px;
    height: 30px;
    position: absolute;
    margin-left: 50px;
    margin-top: 8px;
    font-size: 12px;
    font-family: "Microsoft YaHei", Georgia, Times, "Times New Roman", serif;
    text-align: center;
}
</style>
