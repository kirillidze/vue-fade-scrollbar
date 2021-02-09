<template>
    <div ref="main" :class="mainClasses">
        <div class="vue-fade-scrollbar__main">
            <!-- @slot slot for <strong>single (!)</strong> element which has own scrollbar -->
            <slot />
        </div>
        <div
            ref="track"
            :class="trackClasses"
            @mousedown.stop="onFakeScrollBarTrackMouseDown"
            @click.stop="onFakeScrollBarTrackClick"
        >
            <div
                ref="thumb"
                class="vue-fade-scrollbar__thumb"
                :class="thumbClasses"
                :style="thumbStyles"
                @mousedown.stop="onFakeScrollBarMouseDown"
            ></div>
        </div>
    </div>
</template>

<script>
export default {
    name: 'VueFadeScrollbar',
    props: {
        /**
         * `xl`, `xl-down`, `l`, `l-down`, `m`, `m-down` etc.
         */
        visible: {
            type: String,
            default: '',
        },
        /**
         * { thikness, left, right, radius, color, hoverColor, activeColor }
         */
        thumb: {
            type: Object,
            default: () => {},
        },
        thumbClasses: {
            type: [String, Array, Object],
            default: '',
        },
        trackColor: {
            type: String,
            default: 'transparent',
        },
    },
    data() {
        return {
            hasScroll: false,
            thumbBox: null,
            trackBox: null,
            thumbHeight: 0,
            scrollRatio: 1,
            trackIsClicked: false,
            dragOptions: {
                startClientPos: 0,
                distance: 0,
                mouseIsDown: false,
            },
            scrollElContentHeight: null,
        }
    },
    computed: {
        thumbStyles() {
            return {
                height: `${this.thumbHeight}px`,
            }
        },

        scrollEl() {
            return this.$slots.default ? this.$slots.default[0].elm : null
        },
        mainClasses() {
            return {
                'vue-fade-scrollbar': true,
                [`vue-fade-scrollbar--visible${this.visible && `-${this.visible}`}`]: this.hasScroll,
            }
        },
        trackClasses() {
            return {
                'vue-fade-scrollbar__track': true,
                'vue-fade-scrollbar__track--visible': this.hasScroll,
            }
        },
        computedThumb: {
            get() {
                return {
                    thikness: '10px',
                    left: '1px',
                    right: '1px',
                    radius: 0,
                    color: '#c7ccd1',
                    hoverColor: '#8f99a3',
                    activeColor: '#666666',
                    ...this.thumb,
                }
            },
        },
    },
    watch: {
        trackColor: {
            handler(newValue) {
                this.setTrackColor(newValue)
            },
        },
        thumb: {
            handler(newValue) {
                this.setThumbProperties(newValue)
            },
            deep: true,
        },
    },
    mounted() {
        this.setTrackColor(this.trackColor)
        this.setThumbProperties(this.computedThumb)
        this.scrollEl && this.scrollEl.addEventListener('scroll', this.onFakeScrollBarContentScroll)

        this.$nextTick(() => {
            this.updateData()
        })
        window.addEventListener('resize', this.resize)

        document.documentElement.addEventListener('mousemove', this.onFakeScrollBarMouseMove)
        document.documentElement.addEventListener('mouseup', this.onFakeScrollBarMouseUp)
    },

    beforeDestroy() {
        this.scrollEl && this.scrollEl.removeEventListener('scroll', this.onFakeScrollBarContentScroll)
        document.documentElement.removeEventListener('mousemove', this.onFakeScrollBarMouseMove)
        document.documentElement.removeEventListener('mouseup', this.onFakeScrollBarMouseUp)

        window.removeEventListener('resize', this.resize)
    },
    methods: {
        resize() {
            this.updateData()
        },
        updateData() {
            this.checkScroll()
            this.setBoxes()
            this.setThumbHeight()
            this.setScrollRatio()
            this.scrollElContentHeight = this.scrollEl.scrollHeight
        },
        resetData() {
            this.scrollEl.scrollTo(0, 0)
            this.scrollElContentHeight = this.scrollEl.scrollHeight
            setTimeout(() => {
                this.updateData()
            }, 0)
        },
        checkScroll() {
            this.hasScroll = false
            if (this.scrollEl) {
                this.hasScroll = this.scrollEl.clientHeight < this.scrollEl.scrollHeight
            }
        },
        setThumbHeight() {
            if (this.scrollEl) {
                const thumbRatio = Math.min(this.scrollEl.clientHeight / this.scrollEl.scrollHeight, 1.0)
                this.thumbHeight = this.scrollEl.clientHeight * thumbRatio
                if (this.$refs.thumb) {
                    this.thumbBox.height = this.$refs.thumb.getBoundingClientRect().height
                }
            }
        },
        setScrollRatio() {
            if (this.thumbBox && this.trackBox) {
                const availableTrackWay = this.trackBox.height - this.thumbHeight
                const availableContentWay = this.scrollEl.scrollHeight - this.scrollEl.clientHeight
                this.scrollRatio = Math.min(availableTrackWay / availableContentWay, 1.0)
            }
        },
        setBoxes() {
            this.thumbBox = this.getBoxPos(this.$refs.thumb)
            this.trackBox = this.getBoxPos(this.$refs.track)
        },
        getBoxPos(el) {
            if (!el) return null
            const box = el.getBoundingClientRect()
            return {
                height: box.height,
                bottom: box.bottom,
                top: box.top + window.pageYOffset,
            }
        },

        onFakeScrollBarTrackMouseDown(e) {
            this.updateData()
            let pos = e.clientY - this.trackBox.top
            if (pos > this.trackBox.height - this.thumbBox.height * 0.5) {
                pos -= this.thumbBox.height
            } else if (pos > this.thumbBox.height * 0.5) {
                pos -= this.thumbBox.height * 0.5
            }

            this.trackIsClicked = true
            this.scrollEl.scrollTo({
                top: pos / this.scrollRatio,
                behavior: 'smooth',
            })
        },
        onFakeScrollBarTrackClick() {
            this.trackIsClicked = false
        },
        onFakeScrollBarMouseDown(e) {
            this.dragOptions.startClientPos = e.clientY
            this.dragOptions.mouseIsDown = true
            this.setBoxes()
            document.body.classList.add('no-user-select')
        },
        onFakeScrollBarMouseMove(e) {
            if (!this.dragOptions.mouseIsDown) return

            this.setThumbHeight()
            this.setScrollRatio()

            this.dragOptions.distance = e.clientY - this.dragOptions.startClientPos

            let calculateTopPos = this.thumbBox.top - this.trackBox.top + this.dragOptions.distance
            this.setThumbPos(calculateTopPos)
            this.scrollEl.scrollTo(0, calculateTopPos / this.scrollRatio)
        },
        setThumbPos(pos) {
            if (pos < 0) {
                this.$refs.thumb.style.top = '0px'
            } else if (pos + this.thumbBox.height > this.trackBox.height) {
                this.$refs.thumb.style.top = `${this.trackBox.height - this.thumbBox.height}px`
            } else {
                this.$refs.thumb.style.top = `${pos}px`
            }
        },
        onFakeScrollBarMouseUp() {
            this.dragOptions.mouseIsDown = false
            document.body.classList.remove('no-user-select')
        },
        onFakeScrollBarContentScroll(e) {
            const startScrollElContentHeight = this.scrollElContentHeight
            if (this.dragOptions.mouseIsDown || this.trackIsClicked) return

            this.updateData()
            const topPos = e.target.scrollTop
            this.setThumbPos(topPos * this.scrollRatio)

            if (startScrollElContentHeight !== e.target.scrollHeight) {
                this.scrollEl.scrollTo(0, e.target.scrollHeight * this.scrollRatio)
            }
        },
        setThumbProperties(val) {
            if (this.$refs.main) {
                this.$refs.main.style.setProperty('--thumb-thikness', val.thikness)
                this.$refs.main.style.setProperty('--thumb-left', val.left)
                this.$refs.main.style.setProperty('--thumb-right', val.right)
                this.$refs.main.style.setProperty('--thumb-radius', val.radius)
                this.$refs.main.style.setProperty('--thumb-main-color', val.color)
                this.$refs.main.style.setProperty('--thumb-hover-color', val.hoverColor)
                this.$refs.main.style.setProperty('--thumb-active-color', val.activeColor)
            }
        },
        setTrackColor(val) {
            if (this.$refs.main) {
                this.$refs.main.style.setProperty('--track-color', val)
            }
        },
    },
}
</script>

<style lang="scss">
@import '../styles/media-queries';
@import '../styles/mixins';

.vue-fade-scrollbar {
    --thumb-thikness: 5px;
    --thumb-left: 1px;
    --thumb-right: 1px;
    --thumb-opacity: 1;
    --thumb-radius: 8px;

    --thumb-main-color: #c7ccd1;
    --thumb-hover-color: #8f99a3;
    --thumb-active-color: #666666;
    --track-color: 'transparent';

    --track-thikness: calc(var(--thumb-thikness) + var(--thumb-left) + var(--thumb-right));

    $block-name: &;
    &__main {
        > * {
            @include hide-scrollbar();
        }
    }
    position: relative;
    &__track {
        @include position(absolute, 0 0 0 null);
        width: 0;
        opacity: 0;
        background-color: var(--track-color);
        z-index: 101;
        display: block;
    }
    &__thumb {
        @include position(absolute, 0 var(--thumb-right) null var(--thumb-left));
        background-color: var(--thumb-main-color);
        height: 100px;
        border-radius: var(--thumb-radius);
        &:hover {
            background-color: var(--thumb-hover-color);
        }
        &:active {
            background-color: var(--thumb-active-color);
        }
    }
    &:hover {
        #{$block-name}__track--visible {
            @include track-visibility(var(--track-thikness));
        }
    }
    &__track--visible:active {
        @include track-visibility(var(--track-thikness));
    }

    &--visible {
        #{$block-name}__track {
            @include track-visibility(var(--track-thikness));
        }
        @each $name, $value in $token-media-queries {
            &-#{$name} {
                @media #{$value} {
                    #{$block-name}__track {
                        @include track-visibility(var(--track-thikness));
                    }
                }
            }
        }
    }
}
.no-user-select {
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
}
</style>
