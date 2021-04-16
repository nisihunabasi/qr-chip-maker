<template>
    <div id="panel" class="flex flex-row flex-wrap flex-auto justify-center mx-auto my-6">
        <form action="" class="form m-2 flex flex-col justify-around">
            <label class="block">
                <div class="input-label">上テキスト</div>
                <input type="text" name="upper" id="upper" v-model="upperText" class="input-form">
            </label>
            <label class="block">
                <div class="input-label">QRコード</div>
                <input type="text" name="qr" id="qr" v-model="qrText" @change="onChangeQrCode" class="input-form">
            </label>
            <label class="block">
                <div class="input-label">下テキスト</div>
                <input type="text" name="lower" id="lower" v-model="lowerText" class="input-form">
            </label>
            <label class="block">
                <div class="input-label">画像サイズ</div>
                <div class="input-form-container grid grid-cols-3 gap-0">
                    <label>
                        <input type="radio" name="img-size" id="img-size-small" value="small" v-model="imgSizeType" class="hidden">
                        <div class="input-radio-label rounded-bl-lg">小</div>
                    </label>
                    <label>
                        <input type="radio" name="img-size" id="img-size-normal" value="normal" v-model="imgSizeType" checked class="hidden">
                        <div class="input-radio-label border-gray-200 border-solid border-l border-r bg-blue-300">普通</div>
                    </label>
                    <label>
                        <input type="radio" name="img-size" id="img-size-large" value="large" v-model="imgSizeType" class="hidden">
                        <div for="img-size-large" class="input-radio-label rounded-br-lg">大</div>
                    </label>
                </div>
            </label>
        </form>
        <div id="result" class="result m-2 p-2 box-border border border-solid border-gray-700"></div>
        <canvas id="shadow" class="hidden"></canvas>
    </div>
</template>

<style scoped>
.input-label {
    @apply text-center py-2 border border-solid border-gray-200 border-b-0 rounded-t-lg bg-black text-white;
}
.input-form {
    @apply px-2 py-2 border border-solid border-gray-200 border-t-0 rounded-b-lg;
}
.input-form-container {
    @apply p-0 border border-solid border-gray-200 border-t-0 rounded-b-lg;
}
.input-radio-label {
    @apply text-center py-2;
}
.result {
    width: 468px;
    height: 606px;
}
</style>

<script>
//@see https://qiita.com/mitsuya_bauhaus/items/b6f3d1aec07a9e07bb3a
const ratio = {
    textSize: 0.1,
    containerHeight: 0.13
};
const imgSizeConst = {
    small: {width: 200, height: 260, qrW: 190},
    normal: {width: 354, height: 460, qrW: 334},
    large: {width: 450, height: 586, qrW: 430}
};
let qr = require("qrcode");

export default {
    name: "panel",
    data () {
        return {
            upperText: "lorem ipsum",
            qrText: "https://localhost:8080",
            lowerText: "localhost:8080",
            qrImg: null,
            imgSizeType: "normal"
        }
    },
    mounted () {
        this.drawResult();
        this.onChangeQrCode();
        //終了処理を実装する。
    },
    watch: {
        imgSizeType (val) {
            document.querySelectorAll("input[name='img-size'] + div").forEach(elm => elm.className = elm.className.replace(/\s?bg-blue-300/, ""));
            document.querySelector("input[name='img-size'][value='" + val + "'] + div").className += " bg-blue-300";
            this.onChangeQrCode();
        }
    },
    computed: {
        imgSize () {
            return imgSizeConst[this.imgSizeType];
        }
    },
    methods: {
        onChangeQrCode () {
            const options = {
                width: this.imgSize.qrW,
                margin: 2
            };
            qr.toDataURL(this.qrText, options, (err, url) => {
                if (err) console.error(err);
                let img = new Image();
                img.src = url;
                this.qrImg = img;
            });
        },
        drawResult () {
            const s = (p) => {
                p.setup = () => {
                    //no-op
                };
                p.draw = () => {
                    const w = this.imgSize.width;
                    const h = this.imgSize.height;
                    p.createCanvas(w, h);
                    p.background(255);
                    p.fill(0);
                    p.rect(0, 0, w, h * ratio.containerHeight);
                    p.rect(0, h - (h * ratio.containerHeight), w, h * ratio.containerHeight);

                    //枠線
                    p.stroke(0);
                    p.strokeWeight(2);
                    p.line(1, 0, 1, h);
                    p.line(w - 1, 0, w - 1, h);

                    p.fill(255);
                    p.textAlign(p.CENTER, p.CENTER);
                    p.textSize(this.getOptimizedFontSize(this.upperText, this.imgSize.width, Math.round(this.imgSize.width * ratio.textSize)));
                    p.text(this.upperText, w / 2, h * ratio.containerHeight / 2);
                    p.textSize(this.getOptimizedFontSize(this.lowerText, this.imgSize.width, Math.round(this.imgSize.width * ratio.textSize)));
                    p.text(this.lowerText, w / 2, h - (h * ratio.containerHeight / 2));
                    //TODO 枠線
                    if (this.qrImg) p.canvas.getContext("2d").drawImage(this.qrImg, w / 2 - this.imgSize.qrW / 2, h / 2 - this.imgSize.qrW / 2);
                };
            };
            const p5 = require("p5");
            new p5(s, "result");
        },
        getOptimizedFontSize(text, containerWidth, fontSize, fontSet = "sans-serif") {
            let nowFontSize = fontSize;
            let ctx = document.getElementById("shadow").getContext("2d");
            while (nowFontSize > 0) {
                //ここでfontの形式が間違っていると、代入が成立せず、かつエラーが起きずに進んでしまう。
                //"20px sans-serif"と言った形式を守る。
                ctx.font = nowFontSize + "px " + fontSet;
                const measure = ctx.measureText(text);
                if (containerWidth - measure.width > 20) {
                    break;
                } else {
                    nowFontSize--;
                    continue;
                }
            }
            return nowFontSize;
            //containerWidthに左右余白10px程度余裕をもたせたぐらいのフォントサイズを返す。
            //fontSizeが最初から余裕のある数値だったら、増やさずそのまま返す。
        }
    }
}
//@see https://github.com/soldair/node-qrcode
</script>