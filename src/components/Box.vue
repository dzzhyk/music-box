<template>
    <div id="music-box-container" class="shadow">
        <div style="flex: 2; margin: 0;" id="music-box-needle" class="shadow-needle"></div>
        <div style="flex: 2; margin: 0;" :id="playing ? 'music-box-needle-bar-play' : 'music-box-needle-bar-pause'" class="shadow-needle-bar"></div>
        <div style="flex: 3; margin: 0; justify-content: space-around">
            <div style="position: relative">
                <div id="music-box-time">{{ duration2String(now) }} / {{ duration2String(duration) }}</div>
                <el-slider
                    id="music-box-progress"
                    v-model="percentage"
                    @input="handleProgress"
                    @change="handleProgressChange"
                    :format-tooltip="formatProgress"
                ></el-slider>
                <div id="music-box-album" class="shadow"></div>
                <img id="music-box-photo" v-if="this.fileObj != null" :src="fileUrl" :class="this.playing ? 'spin' : ''">
                <div id="music-box-hook" class="shadow-hook"></div>
                <a id="music-box-input-a" @click="chooseFile"></a>
                <input id="music-box-input" style="display: none" type="file" @change="handleChange" accept=".mp3" />
            </div>
        </div>
        <div style="flex: 2">
            <div style="position: relative">
                <el-button
                    id="music-box-play-btn"
                    style="flex: 1"
                    :icon="playing ? 'el-icon-video-pause' : 'el-icon-video-play'"
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
let buf;

var jsmediatags = window.jsmediatags;

export default {
    data() {
        return {
            now: 0,
            duration: 0,
            fileObj: null,
            fileUrl: "",
            title: "Select MP3 File",
            percentage: 0,
            playing: false,
            timer: 0,
        };
    },
    methods: {
        play() {
            source = audioContext.createBufferSource();
            source.buffer = buf;
            source.loop = false;
            source.start(0, this.now);
            source.connect(audioContext.destination);

            this.timer = setInterval(() => {
                if (this.fileObj === null || audioContext === null) return;
                if (this.playing) {
                    if (this.fileObj === null || audioContext === null) return;
                    this.now += 1;
                    this.percentage = Math.floor((this.now / this.duration) * 100);
                    if (this.percentage >= 100) {
                        this.playing = false;
                        source.disconnect();
                        source.stop();
                        this.now = 0;
                        this.percentage = 0;
                        clearInterval(this.timer);
                    }
                }
            }, 1000);

            this.playing = true;
        },
        pause() {
            source.disconnect();
            if (this.timer !== null) clearInterval(this.timer);
            source.stop();
            this.playing = false;
        },
        chooseFile() {
            let input = document.getElementById("music-box-input");
            input.click();
        },
        handleChange(e) {
            let _this = this;
            if (e.target.files === null || e.target.files === undefined) return;
            this.fileObj = e.target.files[0];
            reader.readAsArrayBuffer(this.fileObj);

            jsmediatags.read(this.fileObj,{
                    onSuccess: function(tag) {
                        var imageData = tag.tags.picture.data;
                        var base64String = "";
                        for (var i = 0; i < imageData.length; i++) {
                            base64String += String.fromCharCode(imageData[i]);
                        }
                        _this.fileUrl = "data:" + tag.tags.picture.format + ";base64," + window.btoa(base64String);
                    },
                    onError: function(error) {
                    console.log(":(", error.type, error.info);
                    }
                }
            );

        },
        handlePlay() {
            if (this.fileObj === null) {
                this.chooseFile();
                return;
            }

            if (this.playing) {
                this.pause();
            } else {
                this.play();
            }
        },
        handleProgress(value) {
            this.percentage = value;
            this.now = (this.duration * this.percentage) / 100;
        },
        handleProgressChange(value) {
            this.percentage = value;
            this.now = (this.duration * this.percentage) / 100;
            if (this.fileObj === null || audioContext === null || source === null) return;
            if (this.playing) {
                this.pause();
            }
        },
        formatProgress(value) {
            return this.duration2String(this.now);
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
        let _this = this;
        reader.onload = function (e) {
            audioContext.decodeAudioData(e.target.result, function (buffer) {
                _this.duration = buffer.duration;
                _this.now = 0;
                _this.percentage = 0;
                _this.title = _this.fileObj.name;
                buf = buffer;
                
            });
        };
    },
    beforeDestroy() {
        clearInterval(this.timer);
        source.disconnect();
        audioContext.close();
    },
};
</script>

<style>
body {
    background-color: rgb(243, 210, 229);
}

#music-box-needle {
    position: absolute;
    left: 50%;
    top: 5%;
    width: 10px;
    height: 10px;
    border-radius: 50%;
    background-color: rgba(0, 0, 0);
    z-index: 3;
}

#music-box-needle-bar-play {
    position: absolute;
    left: 51.5%;
    top: 10%;
    width: 45px;
    height: 2px;
    border-radius: 20%;
    background-color: rgb(255, 255, 255);
    z-index: 2;
    transform: rotate(345deg);
    transform-origin: 0 0;
    transition: 1s;
}

#music-box-needle-bar-pause {
    position: absolute;
    left: 51.5%;
    top: 10%;
    width: 45px;
    height: 2px;
    border-radius: 20%;
    background-color: rgb(255, 255, 255);
    z-index: 2;
    transform: rotate(326deg);
    transform-origin: 0 0;
    transition: 1s;
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

.shadow-hook {
    box-shadow: 0px 0px 20px 3px rgb(255 255 255);
}

.shadow-needle {
    box-shadow: 0px 0px 3px 1px #999;
}

.shadow-needle-bar {
    box-shadow: 1px 1px 3px 0px #999
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

#music-box-album {
    position: absolute;
    height: 80px;
    width: 80px;
    margin-left: 130px;
    border-radius: 50%;
    margin-top: -20px;
    background-color: rgb(48, 48, 48);
    background-position: center;
    z-index: 1;
}

#music-box-hook {
    position: absolute;
    height: 14px;
    width: 14px;
    margin-left: 163px;
    border-radius: 50%;
    margin-top: 13px;
    background-color: rgb(255, 255, 255);
    background-position: center;
    z-index: 4;
}

#music-box-photo {
    position: absolute;
    height: 60px;
    width: 60px;
    margin-left: 140px;
    border-radius: 50%;
    margin-top: -10px;
    background-position: center;
    border: 0.3px solid black;
    z-index: 3;
}

#music-box-input-a {
    position: absolute;
    height: 80px;
    width: 80px;
    margin-left: 130px;
    margin-top: -20px;
    border-radius: 50%;
    opacity: 0;
    z-index: 5;
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

@-webkit-keyframes rotate{from{-webkit-transform: rotate(0deg)}
    to{-webkit-transform: rotate(360deg)}
}
@-moz-keyframes rotate{from{-moz-transform: rotate(0deg)}
    to{-moz-transform: rotate(359deg)}
}
@-o-keyframes rotate{from{-o-transform: rotate(0deg)}
    to{-o-transform: rotate(359deg)}
}
@keyframes rotate{from{transform: rotate(0deg)}
    to{transform: rotate(359deg)}
}

.spin {
    -webkit-transition-property: -webkit-transform;
    -webkit-transition-duration: 30s;
    -moz-transition-property: -moz-transform;
    -moz-transition-duration: 30s;
    -webkit-animation: rotate 30s linear infinite;
    -moz-animation: rotate 30s linear infinite;
    -o-animation: rotate 30s linear infinite;
    animation: rotate 30s linear infinite;
}

</style>
