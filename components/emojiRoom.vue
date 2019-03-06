<template>
  <div class="ly-emotion" >
    <slot></slot>
  </div>
</template>

<script>
import emojiData from '@/mixins/emoji'
export default {
  name: 'ly-emotion',
  mounted() {
    console.log(this.$el)
    const name = this.$el.innerHTML
    const list = emojiData.emojiFontData
    const index = list.indexOf(name)
    const imgHTML = `<img src="https://res.wx.qq.com/mpres/htmledition/images/icon/emotion/${index}.gif">`
    this.$nextTick(() => {
      this.$el.innerHTML = imgHTML
    })
  }
}
</script>
<style scoped>
.ly-emotion {
  display: inline-block
}
.ly-static-emotion {
  width: 24px;
  height: 24px;
  display: inline-block;
}
</style>
