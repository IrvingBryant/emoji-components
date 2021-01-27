<template>
  <div class="emoji">
    <ul class="emoji-container">
      <li 
        v-for="(emojiGroup, index) in emojiGroupList" 
        style="padding: 0;" 
        :key="index"
       >
       <span v-if="index === activeIndex">
          <a 
            href="javascript:;" 
            v-for="(emoji, index2) in emojiGroup.emojiArr"  
            :key="index2" @click.stop="selectItem(emoji,emojiGroup.type)">
            <span 
              v-if="emojiGroup.type === '1'"
              class="emoji-item"
              :class="'sprite-' + getPureName(emoji)"  
              @click.stop="selectItem(emoji,'1')"
            >
            </span>
            <span 
              v-if="emojiGroup.type === '2'"
              class="emoji-item"
              @click.stop="selectItem(emoji,'2')">
              {{emoji}}
            </span>
          </a>
        </span>
      </li>
    </ul>
    <ul class="emoji-controller">
      <li 
        v-for="(pannel,index) in pannels_text"
        :key="index"
        @click="changeActive(index)"
        :class="{'active': index === activeIndex}">
        <img :src="pannel.cnt" alt="" v-if="pannel.type==='1'"/> 
        <span v-if="pannel.type==='2'">{{pannel.cnt}}</span>
      </li>
    </ul>
  </div>
</template>
<script>
import data from './emoji-data.json'
import emojiTxt from './emoji-text-data.json'
import pannelsSmile from '@/assets/pannels-smile.png'
export default {
  name: 'emoji',
  data () {
    return {
      emojiData: data,
      emojiTxtData:emojiTxt,
      emojiImgpannels:['表情'],
      emojiTxtPannels:['People','Nature','Objects','Places','Symbols'],
      pannels_text:[
        {
          type:'1',
          cnt:pannelsSmile
        },
        {
          type:'2',
          cnt:'People'
        },
        {
          type:'2',
          cnt:'Nature'
        },
        {
          type:'2',
          cnt:'Objects'
        },
        {
          type:'2',
          cnt:'Places'
        },
        {
          type:'2',
          cnt:'Symbols'
        },
      ],
      activeIndex: 0
    }
  },
  methods: {
    changeActive (index) {
      this.activeIndex = index
    },
    getPureName (name) {
      return name.replace(/:/g, '')
    },
    /**
     * emoji:内容 
     * type：类型  1表示emojiImg 2表示emojiTxt
    */
    selectItem (emoji,type) {
      this.$emit('selectEmoji',emoji,type)
    }
  },
  computed: {
    emojiImg () {
      return this.emojiImgpannels.map(item => {
        return {
          type:'1',
          emojiArr:Object.keys(this.emojiData[item])
        }
      })
    },
    emojiTxt(){
      return this.emojiTxtPannels.map(item => {
        return {
            type:'2',
            emojiArr:this.emojiTxtData[item]
          }
      })
    },
    emojiGroupList(){
      return this.emojiImg.concat(this.emojiTxt)
    }
  }
}
</script>

<style lang='scss' scoped>
@import 'emoji-sprite';

.emoji {
  bottom: 30px;
  background: #fff;
  z-index: 10;
  position: relative;
  top:0;
  border: 1px solid rgb(139, 104, 104);
  border-radius: 4px;
  .emoji-controller {
    height: 50px;
    background: #F5F6F9;
    box-shadow: 0px 6px 16px 0px rgba(0, 0, 0, 0.08);
    height: 36px;
    overflow: hidden;
    margin-bottom: 0;
    padding-left:0 ;
    li {
      display: inline-flex;
      height: 36px;
      align-items: center;
      justify-content: center;
      float: left;
      width: 80px;
      list-style-type: none;
      font-size: 12px;
      line-height: 36px;
      cursor: pointer;
      text-align: center;
      position: relative;
      img{
        width: 22px;
        height: 22px;
      }
      &.active{
        background: #fff;
      }
    }
  }
  .emoji-container {
    height: 140px;
    overflow-y: auto;
    overflow-x: hidden;
    position: relative;
    padding-left:0 ;
    padding: 10px;
    display: flex;
    li {
      list-style-type:none;
      padding: 5px;
      a {
        float: left;
        overflow: hidden;
        height: 35px;
        transition: all ease-out .2s;
        border-radius: 4px;
        &:hover {
          background-color: #d8d8d8;
          border-color: #d8d8d8;
        }
        span {
          width: 25px;
          height: 25px;
          display: inline-block;
          border: 1px solid transparent;
          cursor: pointer;
        }
      }
    }
  }
}
</style>
