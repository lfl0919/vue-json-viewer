<script>
import JsonBox from '../json-box'

export default {
  name: 'JsonObject',
  props: {
    jsonValue: {
      type: Object,
      required: true
    },
    keyName: {
      type: String,
      default: ''
    },
    depth: {
      type: Number,
      default: 0
    },
    expand: Boolean,
    forceExpand: Boolean,
    sort: Boolean,
    previewMode: Boolean,
    showArrayIndex: Boolean,
  },
  data() {
    return {
      value: {}
    }
  },
  computed: {
    ordered () {
      if (!this.sort) {
        return this.value
      }

      const ordered = {}
      Object.keys(this.value).sort().forEach(key => {
        ordered[key] = this.value[key]
      })
      return ordered
    }
  },
  watch: {
    jsonValue(newVal) {
      this.setValue(newVal);
    }
  },
  mounted() {
    this.setValue(this.jsonValue);
  },
  methods: {
    setValue(val) {
      setTimeout(() => {
        this.value = val;
      }, 0);
    },
    toggle() {
      this.$emit('update:expand', !this.expand)
      this.dispatchEvent();
    },
    toggleAll() {
      this.$emit('update:expandAll', !this.expand)
      this.dispatchEvent();
    },
    dispatchEvent() {
      try {
        this.$el.dispatchEvent(new Event('resized'))
      } catch (e) {
        // handle IE not supporting Event constructor
        var evt = document.createEvent('Event')
        evt.initEvent('resized', true, false)
        this.$el.dispatchEvent(evt)
      }
    }
  },
  render (h) {
    let elements = []

    if (!this.previewMode && !this.keyName) {
      elements.push(h('span', {
        class: {
          'jv-toggle': true,
          'open': !!this.expand,
        },
        on: {
          click: (event) => {
            if (event.altKey) {
              this.toggleAll()
            } else {
              this.toggle()
            }
          }
        }
      }))
    }

    elements.push(h('span', {
      class: {
        'jv-item': true,
        'jv-object': true,
      },
      domProps: {
        innerText: '{'
      }
    }))

    if (this.expand) {
      for (let key in this.ordered) {
        if (this.ordered.hasOwnProperty(key)) {
          let value = this.ordered[key]

          elements.push(h(JsonBox, {
            key,
            props: {
              sort: this.sort,
              keyName: key,
              depth: this.depth + 1,
              value,
              previewMode: this.previewMode,
              forceExpand: this.forceExpand,
              showArrayIndex: this.showArrayIndex,
            }
          }))
        }
      }
    }

    if (!this.expand && Object.keys(this.value).length) {
      elements.push(h('span', {
        class: {
          'jv-ellipsis': true,
        },
        on: {
          click: (event) => {
            if (event.altKey) {
              this.toggleAll()
            } else {
              this.toggle()
            }
          }
        },
        attrs: {
          title: `click to reveal object content (keys: ${Object.keys(this.ordered).join(', ')})`
        },
        domProps: {
          innerText: '...'
        }
      }))
    }

    elements.push(h('span', {
      class: {
        'jv-item': true,
        'jv-object': true,
      },
      domProps: {
        innerText: '}'
      }
    }))

    return h('span', elements)
  }
}
</script>
