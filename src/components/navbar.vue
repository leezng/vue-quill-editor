<template>
  <ul class="navbar">
    <li
      v-for="(item, index) in config"
      class="navbar-button"
      :key="index"
      :data-label="item.label">
      <template v-if="!item.isLocal">
        <el-button @click="setStyle(item.value)" :icon="item.icon"></el-button>
      </template>

      <template v-else-if="item.value === 'foreColor' || item.value === 'backColor'">
        <el-button @click="beforeExecColor(item.value)" :icon="item.icon"></el-button>
        <compact-color-picker
          v-model="$data[`${item.value}`]"
          :class="{ 'hidden': $data[`${item.value}Hidden`], 'color-picker': true }"
        ></compact-color-picker>
      </template>

      <template v-else>
        <el-dropdown @command="setStyle(item.value, $event)" @visible-change="beforeExecFontSize" trigger="click">
          <el-button :icon="item.icon">
            <i class="el-icon-caret-bottom el-icon--right"></i>
          </el-button>

          <el-dropdown-menu slot="dropdown">
            <el-dropdown-item
              v-for="option in item.options"
              :key="option"
              :command="option">
              {{ option }}
            </el-dropdown-item>
          </el-dropdown-menu>
        </el-dropdown>
      </template>
    </li>
  </ul>
</template>

<script>
import basicConfig from '@/config'
import { Compact as CompactColorPicker } from 'vue-color'

export default {
  name: 'navbar',
  props: ['user-config'],
  components: {
    CompactColorPicker
  },
  data () {
    return {
      basicConfig: basicConfig.navbar,
      foreColorHidden: true,
      foreColor: '#333',
      backColorHidden: true,
      backColor: '#eee'
    }
  },
  computed: {
    config () {
      const basicConfig = this.basicConfig
      return !this.userConfig || !Array.isArray(this.userConfig)
        ? basicConfig
        : basicConfig.filter(item => this.userConfig.includes(item.value))
    }
  },
  methods: {
    getCurrentRange () {
      var sel = getSelection()
      if (sel.getRangeAt && sel.rangeCount) {
        return sel.getRangeAt(0)
      }
    },
    saveRange (range) {
      let isRange = Object.prototype.toString.call(range) === '[object Range]'
      this.selectedRange = isRange ? range : this.getCurrentRange()
    },
    restoreRange (range) {
      var selection = getSelection()
      if (range) {
        try {
          selection.removeAllRanges()
        } catch (ex) {
          document.body.createTextRange().select()
          document.selection.empty()
        }
        selection.addRange(range)
      }
    },
    beforeExecColor (colorAttr) {
      this.saveRange(this.getCurrentRange())
      this[`${colorAttr}Hidden`] = !this[`${colorAttr}Hidden`]
    },
    beforeExecFontSize (visible) {
      this.saveRange(this.getCurrentRange())
    },
    /**
     * 设置被选中的元素样式
     * @param {String} attr 样式属性, 驼峰写法, 例如 fontSize
     * @param {String} param 可选, 样式属性值, 默认为空字符串
     */
    setStyle (attr, param = '') {
      let range = this.selectedRange
      if (!range) {
        range = this.getCurrentRange()
      }
      if (!range || !range.toString()) return
      this.restoreRange(range)
      if (document.queryCommandSupported('styleWithCss')) {
        document.execCommand('styleWithCss', false, true)
      }
      if (typeof this[`${attr}Local`] === 'function') {
        this[`${attr}Local`](range, param)
      } else {
        document.execCommand(attr, false, param)
      }
      this.selectedRange = ''
    },
    fontSizeLocal (range, param = '') {
      document.execCommand('fontSize', false, 2)
      let container = range.commonAncestorContainer
      container.nodeType === 3 && (container = container.parentNode)
      container.tagName === 'SPAN' && (container = container.parentNode)
      Array.from(container.getElementsByTagName('span')).forEach(span => {
        if (span.style.fontSize.trim() === 'small') {
          span.style.fontSize = param
        }
        span.normalize()
      })
    }
  },
  watch: {
    foreColor (newVal) {
      this.setStyle('foreColor', newVal.hex)
      this.foreColorHidden = true
    },
    backColor (newVal) {
      this.setStyle('backColor', newVal.hex)
      this.backColorHidden = true
    }
  }
}
</script>

<style lang="less" scoped>
  @import "~@/assets/lib/iconfont/iconfont.css";

  a {
    text-decoration: none;
    color: inherit;
  }

  .hidden {
    display: none;
  }

  .navbar {
    padding: 0;
    margin: 0 0 30px 0;
  }

  .navbar > .navbar-button {
    display: inline-block;
    position: relative;
    margin: 5px 5px 5px 0;
    padding: 0;
    list-style: none;
    &:after {
      content: attr(data-label);
      position: absolute;
      left: 0;
      top: 100%;
      width: 100%;
      font-size: 12px;
      white-space: nowrap;
      text-align: center;
      opacity: 0;
      z-index: 10;
    }
    &:hover:after {
      opacity: 1;
    }
  }

  .color-picker {
    position: absolute;
  }
</style>
