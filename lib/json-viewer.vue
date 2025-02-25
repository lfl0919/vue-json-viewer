<template>
  <div
    ref="viewer"
    :class="jvClass"
  >
    <div 
      v-if="copyable"
      :class="`jv-tooltip ${copyText.align || 'right'}`"
    >
      <span  
        ref="clip" 
        class="jv-button"
        :class="{copied}"
      >
        <slot
          name="copy"
          :copied="copied"
        >
          {{ copied ? copyText.copiedText : copyText.copyText }}
        </slot>
      </span>
    </div>
    <div 
      class="jv-code" 
      :class="{'open': expandCode, boxed}"
    >
      <json-box
        ref="jsonBox"
        :value="value"
        :sort="sort"
        :preview-mode="previewMode"
        :show-array-index="showArrayIndex"
      />
    </div>
    <div 
      v-if="expandableCode && boxed" 
      class="jv-more" 
      @click="toggleExpandCode"
    >
      <span 
        class="jv-toggle" 
        :class="{open: !!expandCode}"
      />
    </div>
  </div>
</template>

<script>
import Vue from 'vue'
import JsonBox from './json-box'
import Clipboard from 'clipboard'
import {debounce} from './utils';

export default {
  name: 'JsonViewer',
  components: {
    JsonBox
  },
  props: {
    value: {
      type: [Object, Array, String, Number, Boolean, Function],
      required: true
    },
    expanded: {
      type: Boolean,
      default: false
    },
    expandDepth: {
      type: Number,
      default: 1
    },
    copyable: {
      type: [Boolean, Object],
      default: false
    },
    sort: {
      type: Boolean,
      default: false
    },
    boxed: {
      type: Boolean,
      default: false
    },
    theme: {
      type: String,
      default: 'jv-light'
    },
    timeformat: {
      type: Function,
      default: value => value.toLocaleString(),
    },
    previewMode: {
      type: Boolean,
      default: false,
    },
    showArrayIndex: {
      type: Boolean,
      default: true,
    }
  },
  provide () {
    return {
      expandDepth: this.expandDepth,
      timeformat: this.timeformat,
    }
  },
  data () {
    return {
      copied: false,
      expandableCode: false,
      expandCode: this.expanded
    }
  },
  computed: {
    jvClass() {
      return 'jv-container ' + this.theme + (this.boxed ? ' boxed' : '')
    },
    copyText() {
      const { copyText, copiedText, timeout, align } = this.copyable

      return {
        copyText: copyText || 'copy',
        copiedText: copiedText || 'copied!',
        timeout: timeout || 2000,
        align,
      }
    }
  },
  watch: {
    value() {
      this.onResized()
    }
  },
  mounted: function () {
    this.debounceResized = debounce(this.debResized.bind(this), 200);
    if (this.boxed && this.$refs.jsonBox) {
      this.onResized()
      this.$refs.jsonBox.$el.addEventListener("resized", this.onResized, true)
    }
    if (this.copyable) {
      const clipBoard = new Clipboard(this.$refs.clip, {
        container: this.$refs.viewer,
        text: () => {
          return JSON.stringify(this.value, null, 2)
        }
      });
      clipBoard.on('success', (e) => {
        this.onCopied(e)
      })
    }
  },
  methods: {
    onResized () {
      this.debounceResized();
    },
    debResized() {
      this.$nextTick(() => {
        if (!this.$refs.jsonBox) return;
        if (this.$refs.jsonBox.$el.clientHeight >= 250) {
          this.expandableCode = true
        } else {
          this.expandableCode = false
        }
      })
    },
    onCopied(copyEvent) {
      if (this.copied) {
        return;
      }
      this.copied = true
      setTimeout(() => {
        this.copied = false
      }, this.copyText.timeout)
      this.$emit('copied', copyEvent)
    },
    toggleExpandCode () {
      this.expandCode = !this.expandCode
    }
  }
}
</script>

<style lang="scss">
.jv-container {
  box-sizing: border-box;
  position: relative;

  &.boxed {
    border: 1px solid #eee;
    border-radius: 6px;

    &:hover {
      box-shadow: 0 2px 7px rgba(0, 0, 0, 0.15);
      border-color: transparent;
      position: relative;
    }
  }

  &.jv-light {
    background: #fff;
    white-space: nowrap;
    color: #525252;
    font-size: 14px;
    font-family: Consolas, Menlo, Courier, monospace;

    .jv-ellipsis {
      color: #999;
      background-color: #eee;
      display: inline-block;
      line-height: 0.9;
      font-size: 0.9em;
      padding: 0px 4px 2px 4px;
      margin: 0 4px;
      border-radius: 3px;
      vertical-align: 2px;
      cursor: pointer;
      user-select: none;
    }
    .jv-button {
      color: #49b3ff;
    }
    .jv-key {
      color: #111111;
      margin-right: 4px;
    }
    .jv-item {
      &.jv-array {
        color: #111111;
      }
      &.jv-boolean {
        color: #fc1e70;
      }
      &.jv-function {
        color: #067bca;
      }
      &.jv-number {
        color: #fc1e70;
      }
      &.jv-object {
        color: #111111;
      }
      &.jv-undefined {
        color: #e08331;
      }
      &.jv-string {
        color: #42b983;
        word-break: break-word;
        white-space: normal;

        .jv-link {
          color: #0366d6;
        }
      }
    }
    .jv-code {
      .jv-toggle {
        &:before {
          padding: 0px 2px;
          border-radius: 2px;
        }
        &:hover {
          &:before {
            background: #eee;
          }
        }
      }
    }
  }

  .jv-code {
    overflow: hidden;
    padding: 30px 20px;

    &.boxed {
      max-height: 300px;
    }

    &.open {
      max-height: initial !important;
      overflow: visible;
      overflow-x: auto;
      padding-bottom: 45px;
    }
  }

  .jv-toggle {
    background-image: url(./icon.svg);
    background-repeat: no-repeat;
    background-size: contain;
    background-position: center center;
    cursor: pointer;
    width: 10px;
    height: 10px;
    margin-right: 2px;
    display: inline-block;
    transition: transform 0.1s;

    &.open {
      transform: rotate(90deg)
    }
  }

  .jv-more {
    position: absolute;
    z-index: 1;
    bottom: 0;
    left: 0;
    right: 0;
    height: 40px;
    width: 100%;
    text-align: center;
    cursor: pointer;

    .jv-toggle {
      position: relative;
      top: 40%;
      z-index: 2;
      color: #888;
      transition: all 0.1s;
      transform: rotate(90deg);

      &.open {
        transform: rotate(-90deg)
      }
    }

    &:after {
      content: "";
      width: 100%;
      height: 100%;
      position: absolute;
      bottom: 0;
      left: 0;
      z-index: 1;
      background: linear-gradient(
        to bottom,
        rgba(0, 0, 0, 0) 20%,
        rgba(230, 230, 230, 0.3) 100%
      );
      transition: all 0.1s;
    }

    &:hover {
      .jv-toggle {
        top: 50%; 
        color: #111;
      }

      &:after {
        background: linear-gradient(
          to bottom,
          rgba(0, 0, 0, 0) 20%,
          rgba(230, 230, 230, 0.3) 100%
        );
      }
    }
  }

  .jv-button {
    position: relative;
    cursor: pointer;
    display: inline-block;
    padding: 5px;
    z-index: 5;

    &.copied {
      opacity: 0.4;
      cursor: default;
    }
  }

  .jv-tooltip {
    position: absolute;

    &.right {
      right: 15px;
    }
    &.left {
      left: 15px;
    }
  }

  .j-icon {
    font-size: 12px;
  }
}
</style>
