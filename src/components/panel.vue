<template>
    <div id="panel">
        <form action="" class="form">
            <label for="upper">上</label><input type="text" name="upper" id="upper" v-model="upperText"><br>
            <label for="upper">QR</label><input type="text" name="qr" id="qr" v-model="qrText" @change="onChangeQrCode"><br>
            <label for="upper">下</label><input type="text" name="lower" id="lower" v-model="lowerText"><br>
            <label for="img-size">画像サイズ：</label>
            <label for="img-size-small"><input type="radio" name="img-size" id="img-size-small" value="small" @change="onChangeSize">小</label>&nbsp;
            <label for="img-size-normal"><input type="radio" name="img-size" id="img-size-normal" value="normal" @change="onChangeSize" checked>普通</label>&nbsp;
            <label for="img-size-large"><input type="radio" name="img-size" id="img-size-large" value="large" @change="onChangeSize">大</label>&nbsp;
        </form>
        <div class="result" id="result"></div>
        <canvas id="qr-code"></canvas>
    </div>
</template>

<style scoped>
#result {
    box-sizing: border-box;
    border: 1px solid #999;
}
</style>

<script>
//@see https://qiita.com/mitsuya_bauhaus/items/b6f3d1aec07a9e07bb3a
const ratio = {
    textSize: 0.1,
    containerHeight: 0.13
};
const imgSizeConst = {
    small: {width: 200, height: 300, qrW: 180},
    normal: {width: 354, height: 460, qrW: 320},
    large: {width: 450, height: 600, qrW: 410}
};
let qr = require("qrcode");

export default {
    name: "panel",
    data() {
        return {
            upperText: "hogehoge",
            qrText: "https://appeal.watts1985.jp/",
            lowerText: "hagehage",
            qrImg: null,
            imgSize: imgSizeConst.normal
        }
    },
    mounted: function () {
        this.drawResult();
        this.onChangeQrCode();
        //終了処理を実装する。
    },
    methods: {
        onChangeSize: function(e) {
            if (e.target.value == "small") {
                this.imgSize = imgSizeConst.small;
            } else if (e.target.value == "normal") {
                this.imgSize = imgSizeConst.normal;
            } else if (e.target.value == "large") {
                this.imgSize = imgSizeConst.large;
            }
            this.onChangeQrCode();
        },
        onChangeQrCode: function() {
            const options = {
                width: this.imgSize.qrW
            };
            qr.toDataURL(this.qrText, options, (err, url) => {
                if (err) console.error(err);
                let img = new Image();
                img.src = url;
                this.qrImg = img;
            });
        },
        drawResult: function () {
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

                    p.fill(255);
                    p.textSize(this.getOptimizedFontSize());
                    p.textAlign(p.CENTER, p.BOTTOM);
                    const containerPaddingBottom = Math.round(h * (ratio.containerHeight - ratio.textSize) / 2);
                    p.text(this.upperText, w / 2, h * ratio.containerHeight - containerPaddingBottom);
                    p.text(this.lowerText, w / 2, h - containerPaddingBottom);
                    //TODO 枠線
                    //TODO 配置時、中央に来るようにする。
                    if (this.qrImg) p.canvas.getContext("2d").drawImage(this.qrImg, w / 2 - this.imgSize.qrW / 2, h / 2 - this.imgSize.qrW / 2);
                };
            };
            const p5 = require("p5");
            new p5(s, "result");
        },
        //getOptimizedFontSize(text, fontSize, fontSet, containerWidth) {
        getOptimizedFontSize() {
            return Math.round(this.imgSize.width * ratio.textSize);
            //containerWidthに左右余白10px程度余裕をもたせたぐらいのフォントサイズを返す。
            //fontSizeが最初から余裕のある数値だったら、増やさずそのまま返す。
        }
    }
}
//QRCodeの書き出し方法 node-qrcode使用。
//@see https://github.com/soldair/node-qrcode
</script>