<template>
    <div id="app">
        <vue-fade-scrollbar :visible="mediaQuery" class="fade-scrollbar">
            <div class="content">
                <div class="brick"></div>
                <div class="queries-list">
                    <label v-for="[name, val] in Object.entries(mediaQueries)" :key="name">
                        <input
                            type="radio"
                            name="mediaQuery"
                            :value="name"
                            :checked="name === 's-down'"
                            @input="handleInput"
                        />
                        {{ name }}: {{ val }}
                    </label>
                </div>
                <div class="brick"></div>
            </div>
        </vue-fade-scrollbar>
    </div>
</template>

<script>
import VueFadeScrollbar from './components/VueFadeScrollbar'

export default {
    name: 'App',
    components: {
        VueFadeScrollbar,
    },
    data() {
        return {
            mediaQuery: 's-down',
            mediaQueries: {
                xs: 'min-width: 375px',
                s: 'min-width: 480px',
                m: 'min-width: 768px',
                l: 'min-width: 1024px',
                xl: 'min-width: 1440px',
                'xs-down': 'max-width: 374px',
                's-down': 'max-width: 479px',
                'm-down': 'max-width: 767px',
                'l-down': 'max-width: 1023px',
                'xl-down': 'max-width: 1439px',
            },
        }
    },
    methods: {
        handleInput(e) {
            this.mediaQuery = e.target.value
        },
    },
}
</script>

<style>
html,
body,
#app {
    padding: 0;
    margin: 0;
    width: 100%;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
}
</style>

<style>
#app {
    position: relative;
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    color: #2c3e50;
}

.fade-scrollbar {
    width: 60%;
}

.content {
    text-align: center;
    border: 1px solid green;
    padding: 1rem;
    max-height: 300px;
    overflow-y: auto;
    box-sizing: border-box;
}

.queries-list {
    display: flex;
    align-items: center;
    flex-direction: column;
    justify-content: center;
    font-size: 24px;
    line-height: 28px;
}

.queries-list > *:not(:last-child) {
    margin-bottom: 12px;
}

.brick {
    display: block;
    height: 100px;
    background-color: blue;
    flex: 0 0 auto;
}
</style>
