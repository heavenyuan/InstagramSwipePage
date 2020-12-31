<template>
    <div class="pc-container" :style="isIosHeight">
        <div
            class="video-container"
            :style="isIosHeight"
            @touchstart.stop="gtouchstart($event)"
            @touchmove.stop="gtouchmove($event)"
            @touchend.stop="gtouchend($event)"
        >
            <keep-alive>
                <div class="camera">
                    <div ref="space" class="space" :style="isIosHeight">
                        <div class="box" :style="boxStyle">
                            <div
                                v-for="(n, i) in 10"
                                v-show="chkAct(i)"
                                :key="i"
                                class="item"
                                :style="getStyle(i)"
                            >
                                {{ n }}
                            </div>
                        </div>
                    </div>
                </div>
            </keep-alive>
        </div>
    </div>
</template>

<script>
export default {
    components: {},
    data() {
        return {
            screenWidth: 0,
            screenHeight: window.innerHeight,
            halfScreenWidth: 0,
            halfScreenHeight: 0,
            rate: 0.36,
            id: this.getId(),
            rotate: this.getRotate(this.getId()),
            translateZ: this.getTransZ(0),
            touchPointX: 0,
            tempTouchMove: 0,
            tempRotate: 0,
            lock: false,
            isIos: false,
            muted: false,
        }
    },
    computed: {
        boxStyle() {
            return {
                transform: `translateX(0px) translateY(0px) translateZ(${this.translateZ}px) rotateY(${this.rotate}deg)`,
            }
        },
        halfWidth() {
            return Math.floor(this.screenWidth / 2)
        },
        halfHeight() {
            return Math.floor(this.screenHeight / 2)
        },
        isIosHeight() {
            return {
                height: `${this.screenHeight}px`,
            }
        },
    },
    watch: {
        screenWidth(newWidth, oldWidth) {
            this.halfScreenWidth = Math.floor(newWidth / 2)
        },
        screenHeight(newHeight, oldHeight) {
            this.halfScreenHeight = Math.floor(newHeight / 2)
        },
        id(newId, oldId) {
            this.$router.replace({ path: `/${newId}` })
        },
    },
    created() {
        if (
            /Safari/.test(navigator.userAgent) &&
            !/Chrome/.test(navigator.userAgent)
        ) {
            this.isIos = true
        } else {
            this.isIos = false
        }
    },
    mounted() {
        this.screenWidth = this.$refs.space.offsetWidth
        this.screenHeight = window.innerHeight // this.$refs.space.offsetHeight
        this.halfScreenWidth = this.halfWidth
        this.halfScreenHeight = this.halfHeight
        this.$nextTick(() => {
            window.addEventListener('resize', this.onResize)
        })
    },
    beforeDestroy() {
        window.removeEventListener('resize', this.onResize, true)
    },
    methods: {
        getId() {
            const tid = parseInt(this.$route.params.id, 10) || 1
            return this.idLimit(tid)
        },
        idLimit(tid) {
            return Math.min(Math.max(tid, 1), 9)
        },
        getRotate(id) {
            return -90 * (id - 1)
        },
        onResize() {
            this.screenWidth = this.$refs.space.offsetWidth
            this.screenHeight = window.innerHeight // this.$refs.space.offsetHeight
        },
        randomDEC() {
            const color = Math.floor(Math.random() * 100) + 156
            const dec = color.toString(16)
            return dec.length > 1 ? dec : `0${dec}`
        },
        randomColor() {
            return `${this.randomDEC()}${this.randomDEC()}${this.randomDEC()}`
        },
        getStyle(n) {
            const v = (n % 4) + 1
            const style = {
                transform: false,
                // 'background-color': `#${this.randomColor()}`,
            }

            switch (v) {
                case 1:
                    style.transform = `translateZ(${this.halfScreenWidth}px)`
                    break
                case 2:
                    style.transform = `translateX(${this.halfScreenWidth}px) translateY(0px) rotateY(90deg)`
                    break
                case 3:
                    style.transform = `translateZ(-${this.halfScreenWidth}px) rotateY(180deg)`
                    break
                case 4:
                    style.transform = `translateX(-${this.halfScreenWidth}px) translateY(0px) rotateY(-90deg)`
                    break
            }

            return style
        },
        chkAct(n) {
            const v = this.id - 1
            return n === v || n === v - 1 || n === v + 1
        },
        gtouchstart(e) {
            const touch = e.targetTouches[0]
            this.touchPointX = touch.pageX || touch.clientX
            this.tempRotate = this.getRotate(this.getId())
            this.tempTouchMove = 0
            // console.warn(this.tempRotate)
        },
        gtouchmove(e) {
            // const sec = this.tempTouchMove === 0 ? 0 : 0.2
            const touch = e.targetTouches[0]
            const dx = touch.pageX || touch.clientX
            this.tempTouchMove = dx - this.touchPointX
            const r =
                (this.tempTouchMove / this.screenWidth) * 90 + this.tempRotate
            this.rotateCard(0, this.rotateLimit(r))
        },
        gtouchend(e) {
            if (this.tempTouchMove === 0) return
            if (Math.abs(this.tempTouchMove) > this.halfScreenWidth) {
                const plus = this.tempTouchMove > 0 ? 1 : -1
                this.id = this.idLimit(this.id - plus)
            }
            this.touchPointX = 0
            this.tempTouchMove = 0
            // this.rotate = this.getRotate(this.id)
            this.rotateCard(0.3, this.getRotate(this.id))
        },
        rotateLimit(rotate) {
            const limitS = 2
            const limitB = -((10 - 1) * 90) - 2
            let r = rotate
            if (rotate > limitS) {
                r = limitS
            } else if (rotate < limitB) {
                r = limitB
            }
            return r
        },
        getTransZ(rotationY) {
            const dz = ((45 - Math.abs((-rotationY % 90) - 45)) / 45) * 70

            return -this.halfScreenWidth - dz
        },
        rotateCard(duration, rotationY) {
            // console.warn(duration, rotationY)
            this.rotate = rotationY
            this.translateZ = this.getTransZ(rotationY)
        },
        rotateCardFromTo(duration, rotationY) {
            // gsap.fromTo(
            //     '.box',
            //     {
            //         rotationY: this.rotate
            //     },
            //     {
            //         duration,
            //         rotationY,
            //         translateZ: this.getTransZ(rotationY),
            //         ease: 'power1.inOut',
            //         overwrite: 'auto',
            //         onComplete: this.rotateComplete,
            //         onCompleteParams: [newId, rotationY]
            //     }
            // )
            // const dx = (rotationY - this.rotate) * 0.1
            // this.rotate += dx
            // console.warn(this.rotate)
            // if (Math.abs(this.rotate - rotationY) < 0.5) {
            //     this.rotate = rotationY
            // } else {
            //     requestAnimationFrame(() =>
            //         this.rotateCardFromTo(duration, rotationY)
            //     )
            // }
        },
        rotateComplete(newId, rotationY) {
            this.lock = false
            this.id = newId
            this.rotate = rotationY
        },
    },
}
</script>

<style lang="scss">
$screen-tablet: 768px !default;
$screen-desktop-xs: 992px !default;
$ios-screen-height: 100vh !default; //iphone6,7,8
$ios-screen-md-height: 100vh !default; //ipad

.video-container {
    width: 100vw;
    height: 100vh;
    overflow: hidden;
    // padding: 0 56px * $bw;
    background-color: #362520;
    margin: 0 auto;
    min-height: 100vh;
    color: #fff;
}

.camera {
    perspective-origin: center center;
    perspective: 500px;

    .space {
        // width: 638px * $bw;
        width: 100%;
        height: 100vh;
        transform-style: preserve-3d;

        .box {
            width: 100%;
            height: 100%;
            position: relative;
            transform: translateX(0px) translateY(0px) translateZ(-150px)
                rotateY(0deg);
            transition: all 0.3s;
            transform-style: preserve-3d;

            .item {
                width: 100%;
                height: 100%;
                background-color: #86c166;
                position: absolute;
                backface-visibility: hidden;
                display: flex;
                justify-content: center;
                align-items: center;
                font-size: 10rem;

                > div {
                    height: 100%;
                }

                &.i1 {
                    transform: translateZ(150px);
                }

                &.i2 {
                    transform: translateX(150px) translateY(0px) rotateY(90deg);
                }

                &.i3 {
                    transform: translateZ(-150px) rotateY(180deg);
                }

                &.i4 {
                    transform: translateX(-150px) translateY(0px)
                        rotateY(-90deg);
                }
            }
        }
    }
}

@media screen and (min-width: $screen-tablet) {
    .camera {
        .space {
            height: $ios-screen-md-height;
        }
    }
}

@media screen and (min-width: $screen-desktop-xs) {
    .video-container {
        height: 100vh !important;
        width: auto;
        display: inline-block;
        margin: auto;
        position: relative;
        overflow: visible;
        &:before {
            position: relative;
            content: '';
            display: inline-block;
            padding-left: calc(100vh * 0.5625);
        }
    }

    .camera {
        position: absolute;
        width: 100%;
        height: 100%;
        top: 0;
        left: 0;
        .space {
            position: absolute;
            width: 100%;
            height: 100%;
            top: 0;
            left: 0;
        }
    }
}
</style>
