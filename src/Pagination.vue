<template>
     <div  
      v-show="carousel.pageCount > 1"
       class="indicator"
        role="tablist" 
        >
      <div
        v-for="(page, index) in paginationCount"
        :key="`${page}_${index}`"
        class="bead"
        aria-hidden="false"
        role="tab"
        :title="getDotTitle(index)"
        :value="getDotTitle(index)"
        :aria-label="getDotTitle(index)"
        :aria-selected="isCurrentDot(index) ? 'true' : 'false'"
        v-on:click="goToPage(index)"
        :data-active="isCurrentDot(index) ? 'true' : 'false'"
        :data-prev="isPrevDot(index) ? 'true':'false'"
        :data-first="index === 0"
        :style="{'--index':carousel.currentPage}"
      ></div>
    </div>
</template>

<script>
export default {
  name: "pagination",
  inject: ["carousel"],
  computed: {
    paginationPositionModifierName() {
      const { paginationPosition } = this.carousel;
      // guard to add only required class modifiers
      if (paginationPosition.indexOf("overlay") < 0) return;
      return paginationPosition;
    },
    paginationPropertyBasedOnPosition() {
      return this.carousel.paginationPosition.indexOf("top") >= 0
        ? "bottom"
        : "top";
    },
    paginationCount() {
      return this.carousel && this.carousel.scrollPerPage
        ? this.carousel.pageCount
        : this.carousel.slideCount || 0;
    },
    dotContainerStyle() {
      const { carousel } = this;
      if (carousel.maxPaginationDotCount === -1)
        return {
          "margin-top": `${carousel.paginationPadding * 2}px`
        };
      const doublePadding = carousel.paginationPadding * 2;
      const containerWidth =
        carousel.maxPaginationDotCount *
        (carousel.paginationSize + doublePadding);
      return {
        "margin-top": `${carousel.paginationPadding * 2}px`,
        overflow: "hidden",
        width: `${containerWidth}px`,
        margin: "0 auto",
        "white-space": "nowrap"
      };
    },
    cssVar(index) {
      return { "--index": index };
    }
  },
  methods: {
    /**
     * Change page by index
     * @param {number} index
     * return {void}
     */
    goToPage(index) {
      /**
       * @event paginationclick
       * @type {number}
       */
      this.$emit("paginationclick", index);
    },

    /**
     * Check on current dot
     * @param {number} index - dot index
     * @return {boolean}
     */
    isCurrentDot(index) {
      return index === this.carousel.currentPage;
    },
    isPrevDot(index) {
      return index >= this.carousel.currentPage;
    },
    /**
     * Generate dot title
     * @param {number} index - dot index
     * @return {string}
     */
    getDotTitle(index) {
      return this.carousel.$children[index].title
        ? this.carousel.$children[index].title
        : `Item ${index}`;
    },
    /**
     * Control dots appear and disappear
     * @param {number} index - dot index
     * @return {Object} - dot(buttn) style
     */
    dotStyle(index) {
      const { carousel } = this;
      const basicBtnStyle = {};
      basicBtnStyle[
        `margin-${this.paginationPropertyBasedOnPosition}`
      ] = `${carousel.paginationPadding * 2}px`;

      Object.assign(basicBtnStyle, {
        padding: `${carousel.paginationPadding}px`,
        width: `${carousel.paginationSize}px`,
        height: `${carousel.paginationSize}px`,
        "background-color": `${
          this.isCurrentDot(index)
            ? carousel.paginationActiveColor
            : carousel.paginationColor
        }`
      });

      if (carousel.maxPaginationDotCount === -1) return basicBtnStyle;

      const eachDotsWidth =
        carousel.paginationSize + carousel.paginationPadding * 2;
      const maxReverse = carousel.pageCount - carousel.maxPaginationDotCount;
      const translateAmount =
        carousel.currentPage > maxReverse
          ? maxReverse
          : carousel.currentPage <= carousel.maxPaginationDotCount / 2
            ? 0
            : carousel.currentPage -
              Math.ceil(carousel.maxPaginationDotCount / 2) +
              1;
      const transformWidth = 0 - eachDotsWidth * translateAmount;
      return Object.assign(basicBtnStyle, {
        "-webkit-transform": `translate3d(${transformWidth}px,0,0)`,
        transform: `translate3d(${transformWidth}px,0,0)`,
        "-webkit-transition": `-webkit-transform ${carousel.speed / 1000}s`,
        transition: `transform ${carousel.speed / 1000}s`
      });
    }
  }
};
</script>

<style scoped>
.indicator {
  position: absolute;
  width: 100%;
  justify-content: center;
  height: 12px;
  transition: all 0.2s;
  display: flex;
  flex-direction: row;
  bottom: 3.125rem;
  z-index: 4;
}

@media only screen and (max-width: 600px) {
  .indicator {
    bottom: 1.75rem;
  }
}
.bead {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  z-index: 2;
  background-color: rgba(196, 196, 196, 0.7);
  margin-inline: 2.5px;
  transition: all 0.2s;
  visibility: visible;
  border: none;
}
.bead[data-active="true"] {
  width: 12px !important;
  height: 12px;
  background-color: white;
}
.bead[data-prev="true"] {
  opacity: 0;
  visibility: hidden;
  transform: translateX(calc(var(--index) * 0.5px));
}
.bead[data-first="true"] {
  width: calc(var(--index) * 25px);
  border-radius: 6px;
  opacity: 1 !important;
  visibility: visible !important;
}
</style>
