<template>
  <div class="tinymce">
    <div class="tinymce__init-area" v-el:textarea></div>
  </div>
</template>

<script>
export default {
  props: {
    config: {
      type: Object,
      default: () => ({}),
    },
    value: {
      type: String,
      default: '',
    },
    initialValue: {
      type: String,
      default: '',
    },
  },
  
  data: () => ({
    instance: null,
  }),
  
  computed: {
    content: {
      cache: false,
      get () {
        return this.instance.getContent()
      },
    },
  },
  
  methods: {
    handleError (err) {
      this.$emit('error', err)
    },

    handleSuccess (editor) {
      this.instance = editor
      this.$emit('init', editor)

      const content = this.initialValue || this.value

      editor.setContent(content)
      editor.on('input change undo redo setcontent', this.handleInput)
      editor.on('change setcontent', this.handleChange)
    },

    setContent (content) {
      const instance = this.instance

      if (instance) {
        instance.setContent(content)

        return true
      }

      return false
    },
    
    handleInput () {
      this.$emit('input', this.instance.getContent())
    },
    
    handleChange () {
      this.$emit('change', this.instance.getContent())
    },
  },

  watch: {
    initialValue (newValue) {
      this.setContent(newValue)
    },
  },

  ready () {
    if (!window.tinymce) {
      return this.handleError(new Error("TinyMce wasn't found"))
    }

    const config = this.config
    const target = this.$els.textarea

    config.target = target
    config.init_instance_callback = this.handleSuccess

    window.tinymce.init(config).catch(this.handleError)
  },
  
  beforeDestroy () {
    this.$emit('destroy', this.instance)
    window.tinymce.remove(this.$els.textarea)
  }
}
</script>