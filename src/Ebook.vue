<template>
    <div class="ebook">
        <title-bar :ifTitleAndMenuShow="ifTitleAndMenuShow"></title-bar>
        <div class="read-warpper">
            <div id="read">
                <div class="mask">
                    <div class="left" @click="prevPage"></div>
                    <div class="center" @click="toggleTitleAndMenu"></div>
                    <div class="right" @click="nextPage"></div>
                </div>
            </div>
        </div>
        <menu-bar :ifTitleAndMenuShow="ifTitleAndMenuShow"
        :fontSizeList="fontSizeList"
        :defaultFontSize="defaultFontSize"
        @setFontSize="setFontSize"
        :themeList="themeList"
        :defaultTheme="defaultTheme"
        @setTheme="setTheme"
        :bookAvailable="bookAvailable"
        @onProgressChange="onProgressChange"
        ref="menuBar"></menu-bar>
    </div>
</template>
<script>
import Epub from 'epubjs'
import TitleBar from '@/components/TitleBar.vue'
import MenuBar from '@/components/MenuBar.vue'


const DOWNLOAD_URL = '/static/internetBook.epub'
global.epub = Epub
/* eslint-disable */
export default {
    components: {
        TitleBar,
        MenuBar
    },
    data(){
        return{
            ifTitleAndMenuShow: false,
            fontSizeList: [
                {fontSize: 12 },
                {fontSize: 14 },
                {fontSize: 16 },
                {fontSize: 18 },
                {fontSize: 20 },
                {fontSize: 22 },
                {fontSize: 24 }
            ],
            defaultFontSize: 16,
            themeList: [
            {
                name: 'default',
                style: {
                    body: {
                        'color': '#000', background: '#fff'
                    }
                }
            },
            {
                name: 'eye',
                style: {
                    body: {
                        'color': '#000', background: '#ceeaba'
                    }
                }
            },
            {
                name: 'night',
                style: {
                    body: {
                        'color': '#fff', background: '#000'
                    }
                }
            },
            {
                name: 'gold',
                style: {
                    body: {
                        'color': '#000', background: 'rgb(241, 236, 226)'
                    }
                }
            }
        ],
        defaultTheme: 0,
        bookAvailable: false
        }
    },
    methods: {
        // jumpTo(href) {
        //     this.rendition.display(href)
        //     this.hideTitleAndMenu()
        // },
        // hideTitleAndMenu() {
        //     this.ifTitleAndMenuShow = false
        //     this.$refs.menuBar.hideSetting()
        //     this.$refs.menuBar.hideContent()
        // },
        onProgressChange(progress) {
            const percentage = progress / 100
            const location = percentage > 0 ? this.locations.cfiFromPercentage(percentage) : 0
            this.rendition.display(location)
        },
        setTheme(index){
            this.themes.select(this.themeList[index].name)
            this.defaultTheme = index
        },
        registerTheme(){
            this.themeList.forEach(theme => {
                this.themes.register(theme.name, theme.style)
            })
        },
        setFontSize(fontSize) {
            this.defaultFontSize = fontSize
            if(this.themes) {
                this.themes.fontSize(fontSize + 'px')
            }
        },
        toggleTitleAndMenu() {
            this.ifTitleAndMenuShow = !this.ifTitleAndMenuShow
            if(!this.ifTitleAndMenuShow){
                this.$refs.menuBar.hideSetting()
            }
        },
        prevPage() {
            this.rendition.prev()
        },
        nextPage(){
            this.rendition.next()
        },
        showEpub() {
            this.book = new Epub(DOWNLOAD_URL)
            this.rendition = this.book.renderTo('read',{
            width: window.innerWidth,
            height: window.innerHeight
            })
            this.rendition.display()
            this.themes = this.rendition.themes
            this.setFontSize(this.defaultFontSize)
            this.registerTheme()
            this.setTheme(this.defaultTheme)
            this.book.ready.then(() => {
                // this.navigation = this.book.navigation
                // console.log(this.navigation)
                return this.book.locations.generate()
            }).then(result => {
                this.locations = this.book.locations
                // this.onProgressChange(50)
                this.bookAvailable = true
            })
            },
        },

    mounted() {
        this.showEpub()
    }
        
        
}
</script>
<style lang='scss' scoped>
@import 'assets/styles/global';

.ebook{
    overflow: hidden;
    position: relative;
    .read-warpper{
        .mask{
        position: absolute;
        top: 0;
        left: 0;
        z-index: 100;
        display: flex;
        width: 100%;
        height: 100%;
        .left{
            flex: 0 0 px2rem(100);
        }
        .center{
            flex: 1;
        }
        .right{
            flex: 0 0 px2rem(100);
        }
        }
    }
}
</style>
