<template>
<div class='mdl-layout__container'>         
  <div class ='menu' @click="toggleMenu">
    <i class="material-icons" >&#xE5D2;</i>
    <ul v-if="menu">
      <li><a href="https://imgss.github.io/demo" target="_blank">demos</a></li>
      <li @click="$router.push('/resume')"><a>about</a></li>
    </ul>
    </div>

  <div class="demo-blog mdl-layout mdl-js-layout">
    <main class="mdl-layout__content">
        <div class='demo-blog__posts mdl-grid tagWrapper' :class='reverse ? "active" : ""' v-if='!this.$route.query.tag'>
          <timeline class='mdl-card mdl-cell mdl-cell--12-col mdl-cell--2-col-desktop' @yearClick = 'getPagesOfYear'></timeline>
          <!--标签云-->
          <tagcloud  class='mdl-card mdl-cell mdl-cell--12-col mdl-cell--4-col-desktop' width='200' height='200' r='80' @tagClick='getPagesOfTag' :tags='tags'></tagcloud>
          <!--关于，个人留言-->
          <div class='mdl-card mdl-cell mdl-cell--12-col mdl-cell--6-col-desktop meta about'><h1>about</h1><div v-html='about'></div></div>
        </div>
        <div class='demo-blog__posts mdl-grid tagWrapper' v-else>
          <div  class='mdl-card mdl-cell mdl-cell--8-col mdl-cell--12-col-desktop meta '><h1>{{this.$route.query.tag}}</h1></div>
        </div>
        <div class='demo-blog__posts mdl-grid'>
          <transition-group name='fade' appear @after-appear="removeDelay" v-if='hasArticles'>
            <!--文章摘要-->
            <div class="mdl-card mdl-cell mdl-cell--12-col-desktop mdl-cell--8-col" 
              :style='styles[index]' 
              @mouseenter = "hover(index)" 
              @mouseleave = "hover(index)" 
              @click="setCurrent(index)" 
              v-for='(card,index) in articles'
              v-bind:key="index"
              >
              <router-link :to="{path: card.id}">
                <div class="mdl-card__title title mdl-card__media mdl-color-text--grey-50" :style='{backgroundColor:colors[index]}'>{{card.title}}</div>
                <div class="mdl-card__supporting-text meta mdl-color-text--grey-600">{{card.text}}</div>
                <div class="mdl-card__supporting-text meta mdl-color-text--grey-600">
                  <svg style="width:24px;height:24px" viewBox="0 0 24 24">
                        <path fill="#000000" d="M14,14H7V16H14M19,19H5V8H19M19,3H18V1H16V3H8V1H6V3H5C3.89,3 3,3.9 3,5V19A2,2 0 0,0 5,21H19A2,2 0 0,0 21,19V5A2,2 0 0,0 19,3M17,10H7V12H17V10Z" />
                  </svg>{{card.postDate.replace('date:','')}}
                </div>
              </router-link>
            </div>
          </transition-group>
            <div v-else class="mdl-card mdl-cell mdl-cell--12-col-desktop mdl-cell--8-col">
                <div class="mdl-card__title title mdl-card__media mdl-color-text--grey-50" style='background:gray'>你来到了没有知识的荒原!</div>
                <div class="mdl-card__supporting-text meta mdl-color-text--grey-600">404 Not Found</div>
                <div class="mdl-card__supporting-text meta mdl-color-text--grey-600">
                  <svg style="width:24px;height:24px" viewBox="0 0 24 24">
                    <path fill="#000000" d="M14,14H7V16H14M19,19H5V8H19M19,3H18V1H16V3H8V1H6V3H5C3.89,3 3,3.9 3,5V19A2,2 0 0,0 5,21H19A2,2 0 0,0 21,19V5A2,2 0 0,0 19,3M17,10H7V12H17V10Z" />
                  </svg>1970-1-1
                </div>
            </div>
        </div>
        <foot :show='show'></foot>
    </main>

  </div>

</div>
</template>

<script>
import axios from 'axios'
import tagcloud from './tagCloud'
import { mapState } from 'vuex'
import timer from './timer'
import foot from './footer'
import timeline from './timeline'
import {root, about} from '../config.json'
export default {
  data () {
    return {
      show: false,
      menu: false,
      index: 0,
      tags: null,
      noDelay: false,
      about,
      reverse: false,
      articles: [],
      colors: [],
      styles: [],
      dates: []
    }
  },
  computed: {
    pixs () {
      return this.styles.map(style => style + 'px')
    },
    hasArticles () {
      return !!this.articles.length
    },
    ...mapState(['tags', 'articles'])
  },
  created () {
    if (!Object.keys(this.$route.query).length) {
      this.getPages()
    } else {
      this.getPagesOfTag()
    }
  },
  mounted () {
    let main = document.querySelector('main')
    let timeId
    main.onscroll = () => {
      clearTimeout(timeId)
      timeId = setTimeout(() => {
        if (main.scrollTop > 280) {
          this.reverse = true
        } else if (main.scrollTop < 280) {
          this.reverse = false
        }
        let index = Math.floor((main.scrollTop) / 180)
        this.index = index > 0 ? index : 0
      }, 100)
    }
  },
  methods: {
    removeDelay () {
      this.styles.forEach(style => { style.transitionDelay = '0s' })
      this.noDelay = true
    },
    hover (index) {
      this.styles[index].transitionDelay = '0s' // 将之前的过渡延迟去掉，防止影响hover效果
      let top = parseInt(this.styles[index].top)
      if (top % 100 === 0) {
        this.styles[index].top = top - 80 + 'px'
      } else {
        this.styles[index].top = top + 80 + 'px'
      }
    },
    getColor () {
      return '#' + Math.random().toString(16).slice(2, 8)
    },
    getPages () {
      axios.get(`${root}index.json`).then((data) => {
        let articles = data.data.values
        for (let i = 0, len = articles.length; i < len; i++) {
          this.colors.push(this.getColor())
          this.styles.push({transitionDelay: 0.1 * i + 's'})
        }
        this.$store.commit('saveColors', this.colors)
        this.articles = articles
        this.dates = articles.map(article => article.postDate)
        this.tags = data.data.allTags
        this.$store.commit('saveArticles', this.articles)
        this.$store.commit('saveTags', data.data.allTags)
        this.show = true
      })
    },
    getPagesOfTag (tag = this.$route.query.tag) { // 响应tag点击事件
      !this.noDelay && this.removeDelay()
      this.articles = this.$store.state.articles.filter(article => {
        if (article.tags.indexOf(tag) !== -1) {
          return article
        }
      })
      this.show = true
    },
    getPagesOfYear (year) {
      !this.noDelay && this.removeDelay()
      this.articles = this.$store.state.articles.filter(article => {
        let re = new RegExp(year)
        if (re.test(article.postDate)) {
          return article
        }
      })
    },
    setCurrent (index) {
      this.$store.commit('setCurrent', index)
    },
    toggleMenu () {
      this.menu = !this.menu
    }
  },
  components: {
    tagcloud,
    timeline,
    timer,
    foot
  }
}

</script>

<style scoped lang='stylus'>
a
  text-decoration : none
.menu
  cursor :pointer
  z-index: 10
  position :fixed
  display :block
  left:20px
  top:20px
  i
    font-size:2em
  ul
    list-style : none
    padding : 10px 0px
    margin-top: -5px
    background : #fff
    font-size: 24px
    line-height: 30px
    li
      padding: 10px 30px
      border-bottom 1px solid #eee
      a
        color #000
      &:hover
        background : #888
        a
          color : #fff
svg
  width:25px
  height:20px
.active  div
.mdl-card__supporting-text.meta.mdl-color-text--grey-600
  text-align: left
  line-height: 2em
.mdl-layout__container
  background: #eeeeee
  /*background:url('../assets/main_bg.jpg');*/
  /*background:-webkit-linear-gradient(top,#3c3382 20%,#79547d 80%,#f39746)*/
  background-size: cover
  background-repeat: no-repeat
.title
  font-size: 1em
.mdl-card.mdl-cell.mdl-cell--12-col
  box-shadow: 0px 0px 20px #aaa
.posts
  display: flex
  justify-content: center
  width: 100%
.tagWrapper
  justify-content: center
  transition:all .3s ease
  transform-origin : 0% 0%
.mdl-card.mdl-cell.mdl-cell--12-col
  transition: all .3s
.fade-enter-active {
  transition: all .3s ease-out
}
.fade-leave-active {
  transition: all .01s ease-out
} 
.fade-enter {
  transform: translateX(800px);
}
</style>
