<template>
    <div class="app-full-page">
        <div class="chars-all">
            <div class="chars-Mask" :style="{'z-index':contentInputMask}">
                <span> <img :src="'./static/chars/no-data.png'" alt=""></span>
            </div>
             <div class="chars-Mask-top" :style="{'z-index':contentInputMask}">
            </div>
            <charsLeft :customerInfor="customerInfor" :contentInputMask="contentInputMask" :customerIndex.sync="customerIndex" ref="myCharsLeft"></charsLeft>
            <div class="chars-center" :style="{right:barDrag[1].top +'px'}">
                <div>
                    <onlineCenter @sendReview="sendReview" @sendTransfer="sendTransfer" @sendClose="sendClose" @agreeTransfer="agreeTransfer"></onlineCenter>
                </div>
                <div>
                    <history  ref="chatContentLiTop"   :customerInfor="customerInfor" :barDrag="barDrag"  :customerIndex="customerIndex" :moreHistoryDataLoading="moreHistoryDataLoading" @moreHistory="moreHistory"></history>
                </div>
                <div class="customer-bottom" :style="{height:barDrag[0].top +'px'}">
                    <div class="dragBar" ref="barAcross"></div>
                    <div class="rich-text">
                        <!-- <img @click="openEmoji" src="@/assets/chars/richText.png" alt=""> -->
                        <span @click="openEmoji">
                            <svg-icon icon-class="emoji"></svg-icon>
                        </span>
                        <span class="common-language">
                            <uploud @sendImg="sendImg"></uploud>
                        </span>
                        <emoji class="emoji" v-if="emojiVisible" @emotion="handleEmotion"></emoji>
                    </div>
                    <div class="textarea">
                        <textarea  @mouseenter="changeConnect" :style="{height:barDrag[0].top-90 +'px' ,resize:'none'}" @input="usefulInput"  v-model="textValue"  maxlength ="200"  placeholder="请输入......" rows="10" cols="30" >
                        </textarea> 
                        <!-- <div ref="txtarea" v-html="txtdata" class="txtarea"  @input="usefulInput" contenteditable="true" :style="{height:barDrag[0].top-90 +'px' ,resize:'none'}" @click="cursorClick" @keyup="cursorKeyup"></div> -->
                        <el-button class="send-btn" type="primary" @click="sendBtn">发送<img src="@/assets/chars/airplane-open.png" alt=""></el-button>
                        <dialogUseful :usefulData="usefulData" :showData="showData" :usefulFlag.sync="usefulFlag"></dialogUseful>
                    </div>
                </div>
            </div>
            <div class="chars-right" :style="{width:barDrag[1].top +'px'}">
                <div class="bar-right" ref="barVertical"></div>
                <charTabs></charTabs>
                <div class="iframeMask" :style="{'z-index':iframeMask}"></div>
            </div>
        </div>
    </div>
</template>
<script>
// import { getAgentCurrentSessions } from '@/api/ajax/chars'
import { websockets, dragBar, historyMessage } from '@/mixins/index'
import { mapState, mapActions } from 'vuex'
import { SessionStorage } from '@/utils/storage.js'
import rules from '@/utils/validate.js'
import charTabs from './tabsRight'
import charsLeft from './charsLeft.vue'
import onlineCenter from './indexCenter/onlineCenter'
import dialogUseful from './indexCenter/dialogUseful'
import uploud from './indexCenter/uploud'
import history from './indexCenter/history'
import emojiData from '@/mixins/emoji'
import emoji from '../components/emoji'
export default {
  name: 'charsJs',
  mixins: [rules, historyMessage, websockets, dragBar],
  components: {
    charTabs,
    charsLeft,
    onlineCenter,
    dialogUseful,
    uploud,
    emoji,
    history
  },
  data() {
    return {
      ruleForm: {
        billCode: ''
      },
      txtdata: null,
      showData: [],
      usefulData: [],
      usefulFlag: false,
      timer: '',
      emojiVisible: '',
      getAllChannelExpressionData: ''

      // 操作模态框里的数据
    }
  },
  computed: {
    ...mapState(['user'])
  },
  watch: {},
  methods: {
    ...mapActions([
      'queryCommonPhraseByTenantId',
      'getAgentCurrentSessions'
    ]),
    // 点击表情将emoji插入输入框
    handleEmotion(i) {
      this.textValue += i
      this.emojiVisible = false
      // this.innerEmo(this.emotion(i))
    },
    // 点击打开表情列表
    openEmoji() {
      if (this.emojiVisible) {
        this.emojiVisible = false
      } else {
        this.emojiVisible = true
      }
    },
    // 输入框按键事件-获取光标位置
    cursorKeyup() {
      const selection = getSelection()
      this.lastEditRange = selection.getRangeAt(0)
    },
    // 输入框点击事件-获取光标位置
    cursorClick() {
      const selection = getSelection()
      this.lastEditRange = selection.getRangeAt(0)
      console.log(this.lastEditRange)
    },
    // 在光标位置插入emoji
    innerEmo(str) {
      const selection = window.getSelection
        ? window.getSelection()
        : document.selection
      this.$refs.txtarea.focus()
      if (this.lastEditRange) {
        // 存在最后光标对象，选定对象清除所有光标并添加最后光标还原之前的状态
        selection.removeAllRanges()
        selection.addRange(this.lastEditRange)
      }
      const range = selection.createRange
        ? selection.createRange()
        : selection.getRangeAt(0)
      if (!window.getSelection) {
        const selection = window.getSelection
          ? window.getSelection()
          : document.selection
        const range = selection.createRange
          ? selection.createRange()
          : selection.getRangeAt(0)
        range.pasteHTML(str)
        range.collapse(false)
        range.select()
      } else {
        const hasR = range.createContextualFragment(str)
        let hasR_lastChild = hasR.lastChild
        while (
          hasR_lastChild &&
          hasR_lastChild.nodeName.toLowerCase() === 'br' &&
          hasR_lastChild.previousSibling &&
          hasR_lastChild.previousSibling.nodeName.toLowerCase() === 'br'
        ) {
          const e = hasR_lastChild
          hasR_lastChild = hasR_lastChild.previousSibling
          hasR.removeChild(e)
        }
        range.insertNode(hasR)
        if (hasR_lastChild) {
          range.setEndAfter(hasR_lastChild)
          range.setStartAfter(hasR_lastChild)
        }
        range.collapse(false)
        selection.removeAllRanges()
        selection.addRange(range)
      }
      this.lastEditRange = selection.getRangeAt(0)
      this.usefulInput()
    },
    // 将会话信息转义为表情gif
    emotion(res) {
      const word = res.replace(/\[|\]/gi, '')
      const list = emojiData.emojiFontData
      const index = list.indexOf(word)
      return `<img class="emoji-item" src="https://res.wx.qq.com/mpres/htmledition/images/icon/emotion/${index}.gif" align="middle">`
    },
    // 将会话信息转义为表情标识(#微笑；)
    // getTextValue() {
    //   let font = ''
    //   let keyList = []
    //   let textValue = ''
    //   const reg = /#.*?\;/gi
    //   const label = this.$refs.txtarea.innerHTML.toString()
    //   const emojiList = emojiData.emojiFontData
    //   const fontList = emojiData.emojiFontString
    //   font = label.split('<img class="emoji-item" src="https://res.wx.qq.com/mpres/htmledition/images/icon/emotion/').join('#')
    //   font = font.split('.gif" align="middle">').join(';')
    //   keyList = font.match(reg)
    //   console.log(keyList, 'keyList')
    //   if (keyList !== null) {
    //     for (let i = 0; i < keyList.length; i++) {
    //       font = font.split(keyList[i]).join('#' + emojiList[fontList.indexOf(keyList[i].replace(/\#|\;/gi, '').toString())] + ';')
    //     }
    //   }
    //   textValue = font.split('&nbsp;').join(' ')
    //   console.log(textValue, 'textValue')
    //   this.textValue = textValue
    // },
    // 输入框常用语
    usefulInput() {
      const data = {
        content: this.textValue, // 常用语内容
        isType: 0, // 0代表常用于内容
        pageNo: 0, // 页码
        pageSize: 0 // 当页数量
      }
      clearTimeout(this.timer)
      this.timer = setTimeout(() => {
        this.queryCommonPhraseByTenantId(data)
          .then(res => {
            console.log(res.result)
            this.usefulFlag = true
            if (res.result.data.length === 0 || this.textValue === '') {
              this.usefulFlag = false
            }
            this.usefulData = res.result.data.slice(0, 5)

            const data = JSON.stringify(this.usefulData)
            const data2 = data
            this.showData = JSON.parse(data2)
            for (let i = 0; i < this.usefulData.length; i++) {
              // console.log(this.textValue)
              // console.log(this.usefulData[i])
              this.usefulData[i].content = this.usefulData[i].content.replace(
                this.textValue,
                function(a) {
                  return "<span style='color:red'>" + a + '</span>'
                }
              )
              console.log(
                this.usefulData[i].content.indexOf("<span style='color:red'>")
              )
              const stringLength =
                this.usefulData[i].content.indexOf("<span style='color:red'>") -
                20
              if (stringLength > 0) {
                this.usefulData[i].content =
                  '...' + this.usefulData[i].content.substring(stringLength)
              }
            }
          })
          .catch(() => {})
      }, 100)
    }
  },
  created() {
    this.getAllChannelExpression().then((res) => {
      console.log(res.result)
      this.getAllChannelExpressionData = res.result
    }).catch(() => {})
  },
  mounted() {
    const getAgent = {
      agentId: this.user.userInfo.reception.id
    }
    // const getAgent = '?agentId=' + 1
    // 会话列表
    this.getAgentCurrentSessions(getAgent)
      .then(res => {
        // 会话列表
        const customerInforArr = res.result
        for (let i = 0; i < customerInforArr.length; i++) {
          this.contentInputMask = -1
          this.customerInfor.push({
            customerId: customerInforArr[i].customerId,
            conversationId: customerInforArr[i].conversationId,
            firstLinkServer: false,
            firstLinkVisitor: false,
            lastInfor: '',
            timer: -1,
            timerLabel: '',
            activeColor: '#66CC02',
            customerTimer: '',
            charChannel: 'CHANNEL_WECHAT',
            content: []
          })
          const customerInforSession = SessionStorage.get('customerInfor')
          if (customerInforSession) {
            for (let j = 0; j < customerInforSession.length; j++) {
              if (customerInforArr[i].conversationId === customerInforSession[j].conversationId) {
                this.customerInfor[i].timer = customerInforSession[j].timer
                this.customerInfor[i].firstLinkServer = customerInforSession[j].firstLinkServer
                this.customerInfor[i].firstLinkVisitor = customerInforSession[j].firstLinkVisitor
              }
            }
          }
          this.customerIndex = 0
          this.$refs.myCharsLeft.activeBackgrond()
          console.log(this)
          this.getMoreHistory(customerInforArr[i], i)
          this.customerTimers()
        }
      })
      .catch()
    this.connectBtn() // 初次进入会话页面
  }
}
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
@import '../../scss/chars/index';
</style>
