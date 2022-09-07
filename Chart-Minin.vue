<template>
    <div class="black-box">
        <canvas ref="canvas" @mousemove="fun($event)" @mouseleave="mouseleave">
        </canvas>
        <div v-if="dataX" class="black-box__text">
            <span>Данные:</span>
            <span>Ось Х: {{dataX}}</span>
            <span>Ось Y: {{dataY}}</span>
        </div>

        <div class="black-box__slider" @mousedown="mousedown($event)">
            <canvas ref="slider" class="black-box__slider__canvas"></canvas>
            <div ref="arrowLeft" class="black-box__slider__left">
                <div ref="left" class="left"></div>
            </div>
            <div ref="window" class="black-box__slider__window"></div>
            <div ref="arrowRight" class="black-box__slider__right">
                <div ref="right" class="right"></div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    data: () => ({
        heightCanvas: 200,
        weightCanvas: 500,
        dpiHeightCanvas: 400,
        dpiWeightCanvas: 1000,
        dataLine: [[0,0], [40, 100], [80, 50], [120, 150], [160, 150], [200, 200], [240,0], [280, 100], [320, 50], [360, 150], [400, 250], [440, 200], [480,200], [520, 100], [560, 50], [600, 150], [640, 50],
            [680, 200], [720, 100], [760, 50], [800, 150], [840, 50], [880, 200], [920, 100], [960, 50]],
        rowsCount: 5,
        colsCount: 8,
        padding: 40,
        clientCursor: null,
        raf: null,
        dataX: null,
        dataY: 0,
        clientCursorY: null,
        defaultWeight: 300,
    }),
    mounted() {
        const canvas = this.$refs.canvas;

        canvas.style.width = this.weightCanvas + 'px';
        canvas.style.height = this.heightCanvas + 'px';
        canvas.width =  this.dpiWeightCanvas;
        canvas.height = this.dpiHeightCanvas;
        this.paint()

        this.sliderChart()

        this.setPosition(0, this.defaultWeight)

    },
    watch: {
        clientCursor() {
            this.raf = requestAnimationFrame(this.paint);
        }
    },
    beforeDestroy() {
        cancelAnimationFrame(this.raf)
    },
    methods: {
        computedBoundaries(data) {
            let min
            let max

            for (const [, y] of data) {
                if (typeof min !== 'number') {min = y}
                if (typeof max !== 'number') {max = y}

                if (min > y) min = y
                if (max < y) max = y
            }

            return [min, max]
        },
        fun(event) {
            const { left } = this.$refs.canvas.getBoundingClientRect()
            this.clientCursor =( event.clientX - left) * 2;
            this.clientCursorY = event.clientY
        },
        paint() {
            const canvas = this.$refs.canvas;
            const ctx = canvas.getContext('2d');
            const viewHeight = this.dpiHeightCanvas - this.padding * 2
            const [yMin, yMax] = this.computedBoundaries(this.dataLine);
            const yRatio = viewHeight / (yMax - yMin)
            const xRatio = this.dpiWeightCanvas / this.dataLine.length

            this.clear(ctx)

            // ось y
            const step = viewHeight / this.rowsCount
            const textStep = (yMax - yMin) / this.rowsCount

            ctx.beginPath()
            ctx.lineWidth = 1;
            ctx.strokeStyle = '#bbb'
            for (let i = 1; i <= this.rowsCount; i++) {
                const y = step * i;
                const text = yMax - textStep * i;
                ctx.font = 'normal 20px sans-serif'
                ctx.fillText(text.toString(), 5, y  + this.padding - 10);
                ctx.moveTo(0, y + this.padding);
                ctx.lineTo(this.dpiWeightCanvas, y + this.padding);

            }
            ctx.stroke();
            ctx.closePath();


            // ось x

            ctx.beginPath();

            const stepX = Math.round(this.dataLine.length / this.colsCount);

            for (let i = 1; i <= this.dataLine.length; i++) {
                const x = Math.round(xRatio * i);

                if ((i - 1) % stepX === 0) {
                    ctx.fillText(x.toString(), x - 15, this.dpiHeightCanvas - 10);
                }
                if (this.isOver(this.clientCursor, x, this.dataLine.length)) {
                    this.dataX = x;
                    this.dataY = this.dataLine[i][1]
                    ctx.save()
                    ctx.moveTo(x, this.padding)
                    ctx.lineTo(x,this.dpiHeightCanvas - this.padding)
                    ctx.restore()
                }
            }
            ctx.stroke();
            ctx.closePath()

            this.line(ctx, this.dataLine, yRatio, this.dpiHeightCanvas, this.padding);
            this.circle(ctx, this.dataLine, yRatio);

        },
        clear(ctx) {
            ctx.clearRect(0, 0, this.dpiWeightCanvas, this.dpiHeightCanvas)
        },
        isOver(mouse, x, length) {
            if (!mouse) { return false }

            const width = this.dpiWeightCanvas / length

            return Math.abs(x - mouse) < width / 2
        },
        mouseleave() {
            this.clientCursor = null;
            this.dataX = null;
            this.dataY = null;
        },
        circle(ctx,  data, ratio) {
            for(const [x, y] of data) {
                if (this.isOver(this.clientCursor, x, this.dataLine.length)) {
                    ctx.beginPath();
                    ctx.strokeStyle = '#ff0000';
                    // ctx.fillStyle = '#fff';
                    ctx.arc(x, this.dpiHeightCanvas - this.padding - y * ratio, 10, 0, Math.PI*2);
                    ctx.fill();
                    ctx.stroke();
                    ctx.closePath();
                }
            }
        },
        line(ctx, data, ratio, dpiHeightCanvas, padding) {
            ctx.beginPath();
            ctx.lineWidth = 4;
            ctx.strokeStyle = '#ff0000'
            for(const [x, y] of data) {
                ctx.lineTo(x, dpiHeightCanvas - padding - y * ratio);
            }
            ctx.stroke();
            ctx.closePath();
        },
        sliderChart() {
            const canvas = this.$refs.slider;
            const ctx = canvas.getContext('2d');
            const width = 500;

            canvas.style.width = 500 + 'px';
            canvas.style.height = 40 + 'px';
            canvas.width =  500 * 2;
            canvas.height = 80;

            const dpiHeightCanvas = 80
            const dpiWeightCanvas = 1000
            const padding = 10

            const viewHeight = dpiHeightCanvas - padding * 2
            const [yMin, yMax] = this.computedBoundaries(this.dataLine);
            const yRatio = viewHeight / (yMax - yMin)

            this.line(ctx, this.dataLine, yRatio, dpiHeightCanvas, padding);

            const $left = this.$refs.arrowLeft;
            const $window = this.$refs.window;
            const $right = this.$refs.arrowRight;
        },
        mousedown(event) {
            const type = event.target.dataset.type
        },
        setPosition(left, right){
            const $left = this.$refs.arrowLeft;
            const $window = this.$refs.window;
            const $right = this.$refs.arrowRight;
            const minWidth = this.weightCanvas * 0.05
            const w = this.weightCanvas - right - left
            console.log(w)

            if (w < minWidth) {
                $window.style.width = minWidth + 'px'
            }
            if (left < 0) {
                $window.style.left = '0px'
                $left.style.width = '0px'
            }
            if (right < 0) {
                $window.style.right = '0px'
                $right.style.width = '0px'
            }

            $window.style.width = w + 'px'
            $window.style.left = left + 'px'
            $window.style.right = right + 'px'

            $right.style.width = right + 'px'
            $left.style.width = left + 'px'
        }
    }

}
</script>

<style scoped>
.black-box {
    padding: 30px;
}

.black-box__text {
    display: flex;
    flex-direction: column;
    margin-top: 50px;
}

.black-box__slider {
    width: 500px;
    position: relative;
    display: flex;
    justify-content: space-between;
    margin-top: 100px;
}

.black-box__slider__canvas {

}

.black-box__slider__left {
    position: absolute;
    top: 0;
    bottom: 0;
    left: 0;
    opacity: .8;
    transition: background 0.22s ease-in-out;
    background: #ddeaf3;
}

.black-box__slider__right {
    position: absolute;
    top: 0;
    bottom: 0;
    right: 0;
    opacity: .8;
    transition: background 0.22s ease-in-out;
    background: #ddeaf3;
}

.black-box__slider__window {
    position: absolute;
    background: transparent;
    top: 0;
    bottom: 0;
}

.left, .right {
    position: absolute;
    top: 0;
    bottom: 0;
    transition: background 0.22s ease-in-out;
    background: #ddeaf3;
    width: 4px;
    cursor: pointer;
}


</style>
