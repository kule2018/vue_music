<template>
    <section class="lyric" ref="container">
        <p class="no" v-if="noLyric">该歌曲没有歌词</p>
        <section id="lrc" v-else>
            <p v-for="(item, i) in lrc" :key="i" :class="['line', i === cur? 'line-cur': '']">
                {{item.content === ''? '\n': item.content}}
            </p>
        </section>
    </section>
</template>

<script>
/**
 * 全屏滚动歌词
 * @prop {Array} lrc 歌词信息数组，数组元素为每行歌词（对象）
 * @prop {Number} cur 当前播放进度对应哪一句歌词的索引
 * @prop {Boolean} isShow 外层与v-show绑定的值，便于一些DOM操作
 */
import {print} from '../util/debug'
import BScroll from 'better-scroll'

export default {
    name: 'FullLyric',
    props: ['lrc', 'cur', 'isShow'],
    data() {
        return {
            container: null,
            lrcBox: null,
            scroll: null,
            pad: 0
        }
    },
    computed: {
        noLyric() {
            return !this.lrc.length
        }
    },
    methods: {
        toCurLine() {
            this.scroll.scrollToElement(this.lrcBox.querySelector('.line-cur'), 500, 0, true)
            // this.scroll.scrollTo(0, this.pad - this.lrcBox.querySelector('.line-cur').offsetTop, 300)
            // print(this.lrcBox.querySelector('.line-cur').offsetTop - this.pad)
            // this.container.scrollTop = this.lrcBox.querySelector('.line-cur').offsetTop - this.pad
            // print(this.container.scrollTop)
        },
        init() {
            this.lrcBox = document.getElementById('lrc')
            this.container = this.lrcBox.parentNode
            this.pad = this.container.clientHeight / 2
            this.lrcBox.style.padding = `${this.pad}px 0`
            if(this.scroll === null) {
                this.scroll = new BScroll(this.$refs.container, {
                    bounce: {
                        top: false,
                        bottom: false,
                        left: false,
                        right: false
                    },
                    stopPropagation: true,
                    bindToWrapper: true
                    //probeType: 2 // scroll事件相关设置
                })
                // this.scroll.on('scroll', pos => {
                //     print(pos)
                // })
                this.$nextTick(() => {
                    this.toCurLine()
                })
            }
            //print(this.lrcBox, this.container, this.pad)
        }
    },
    mounted() {
        if(!this.noLyric){
            this.init()
            print('init')
        }
        print('mounted')
    },
    updated() { // prop值改变时触发子组件更新，而watch无法监听到
        if(!this.container){ // v-if切换时可能之前#lrc没有渲染！
            this.init()
        }
        this.toCurLine()
        print('updated')
    },
    watch: {
        cur() {
            if(!this.container){ // v-if切换时可能之前#lrc没有渲染！
                this.container = document.getElementById('lrc')
            }
            this.toCurLine()
        },
        isShow(val) { // v-show切换时需要重新获取DOM（前提是有歌词）
            if(val && !this.noLyric) {
                this.init()
                this.toCurLine()
            }
        },
        lrc(val) {
            if(this.scroll !== null) this.scroll.refresh()
        }
    }
}
</script>

<style lang="scss" scoped>
    .lyric{
        position: relative;
        box-sizing: border-box;
        height: 100%;
        padding: 0 rem(12);
        overflow-x: hidden;
        overflow-y: auto;
    }
    .no{
        position: absolute;
        top: 50%;
        left: 0;
        margin: 0;
        width: 100%;
        text-align: center;
        color: #fff;
    }
    #lrc{
        position: absolute;
        left: rem(5);
        right: rem(5);
        transition: all 0.3s ease-in-out;
    }
    .line{
        margin: 0;
        padding: rem(8) rem(5);
        text-align: center;
        color: #ccc;
    }
    .line-cur{
        font-size: 16px;
        color: #fff;
    }
</style>