<template>
  <div class="code-copy" title="copy">
    <svg title="Copy Code" @click="copyToClipboard" width="20"
         :class="iconClass"
         :style="alignStyle"
         height="20" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
            d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2"></path>
    </svg>
    <span :class="success ? 'success' : ''" :style="alignStyle">
            {{ options.successText }}
        </span>
  </div>
</template>

<script>
export default {
  props: {
    parent: Object,
    code: String,
    options: {
      align: String,
      color: String,
      backgroundTransition: Boolean,
      backgroundColor: String,
      successText: String,
      staticIcon: Boolean,
    },
  },
  data() {
    return {
      success: false,
      originalBackground: null,
      originalTransition: null,
    };
  },
  computed: {
    alignStyle() {
      let style = {};
      style[this.options.align] = '7.5px';
      return style;
    },
    iconClass() {
      return this.options.staticIcon ? '' : 'hover';
    },
  },
  mounted() {
    this.originalTransition = this.parent.style.transition;
    this.originalBackground = this.parent.style.background;
  },
  beforeDestroy() {
    this.parent.style.transition = this.originalTransition;
    this.parent.style.background = this.originalBackground;
  },
  methods: {
    // From: https://stackoverflow.com/a/5624139
    hexToRgb(hex) {
      let result = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(hex);
      return result
          ? {
            r: parseInt(result[1], 16),
            g: parseInt(result[2], 16),
            b: parseInt(result[3], 16),
          }
          : null;
    },
    copyToClipboard(el) {
      if (navigator.clipboard) {
        navigator.clipboard.writeText(this.code).then(
            () => {
              this.setSuccessTransitions();
            },
            () => {},
        );
      } else {
        let copyelement = document.createElement('textarea');
        document.body.appendChild(copyelement);
        copyelement.value = this.code;
        copyelement.select();
        document.execCommand('Copy');
        copyelement.remove();

        this.setSuccessTransitions();
      }
    },
    setSuccessTransitions() {
      clearTimeout(this.successTimeout);

      if (this.options.backgroundTransition) {
        this.parent.style.transition = 'background 350ms';

        let color = this.hexToRgb(this.options.backgroundColor);
        this.parent.style.background = `rgba(${color.r}, ${color.g}, ${color.b}, 0.1)`;
      }

      this.success = true;
      this.successTimeout = setTimeout(() => {
        if (this.options.backgroundTransition) {
          this.parent.style.background = this.originalBackground;
          this.parent.style.transition = this.originalTransition;
        }
        this.success = false;
      }, 500);
    },
  },
};
</script>

<style scoped>
svg {
  position: absolute;
  right: 5px;
  opacity: 0.75;
  top: 35px;
  cursor: pointer;
}

svg.hover {
  opacity: 0;
}

svg:hover {
  opacity: 1 !important;
}

span {
  position: absolute;
  font-size: 0.85rem;
  line-height: 0.425rem;
  right: 50px;
  opacity: 0;
  transition: opacity 500ms;
}

.success {
  opacity: 1 !important;
}
</style>