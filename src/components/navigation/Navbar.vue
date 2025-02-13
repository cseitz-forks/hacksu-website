<template>
  <div class="navbar"
    :class="{ 'scrolled': scrolled, 'fixed': !scrollNav, 'animated': !scrollNav && from }"
    :style="{ 'padding': padding, 'background-color': color, 'box-shadow': shadow, }"
  >
    <span class="background" v-on:click="hideMenu"></span>
    <button class="hamburger" v-on:click="showMenu">Menu</button>
    <span class="buttons">
      <button class="navbtn navbtn-back"></button>
      <slot></slot>
    </span>
  </div>
</template>

<script>
import { showingNavigationMenu } from "../../globals.js";
export default {
  data() {
    return {
      scrolled: false,
      offset: 0,
      distance: 100,
      progress: 0,
      scrollNav: false,
      from: false,
    }
  },
  computed: {
    padding() {
      let scaleFactor = 1;
      if (typeof window !== "undefined" && window.innerHeight && window.innerHeight < 700) {
        scaleFactor = 0.5;
      }
      let n = (30 * scaleFactor) * (1 - this.progress) + (10 * scaleFactor) * this.progress
      return `${n}px 10px ${10 + n}px 10px`;
    },
    color() {
      if (this.$el) {
        let bgc = "rgb(20, 32, 39)";
        let a = `rgba(${bgc.substr(4).split(')')[0]}, ${this.progress})`;
        return a;
      } else {
        return `rgba(${this.progress}, 0, 0, 0)`;
      }
    },
    shadow() {
      return `0 5px 15px rgba(0,0,0,${0.25 * (this.progress)})`;
    },
    dropdownOpen() {
      if (this.$el) {
        return this.$el.querySelector('.navbtn-dropdown.open') ? true : false;
      }
      return false;
    },
  },
  methods: {
    showMenu() {
      showingNavigationMenu.value = true;
    },
    hideMenu() {
      showingNavigationMenu.value = false;
    },
    computeScrolled() {
      this.offset = document.firstElementChild.scrollTop;
      this.scrolled = this.offset != 0;
      let scrollNav = this.$route.meta.scrollNav || false;
      if (scrollNav && !this.scrollNav) {
        let this_ = this;
        setTimeout(function() {
          this_.scrollNav = this_.$route.meta.scrollNav || false;
        }, 1000)
      } else {
        this.scrollNav = scrollNav;
      }
      if (scrollNav) {
        this.progress = Math.min(1, (this.offset / this.distance));
      } else {
        this.progress = 1;
      }
      return this.scrolled;
    }
  },
  watch: {
    '$route': function(route) {
      if (!this.from) {
        let _this = this;
        setTimeout(function() {
          if (!_this.from) {
            _this.from = route;
          }
        }, 300)
      } else {
        this.from = route;
      }
      this.computeScrolled();
    }
  },
  mounted() {
    let _this = this;
    let compute = function() {
      _this.computeScrolled();
    }
    document.addEventListener('scroll', compute)
    document.addEventListener('resize', compute)
    this.computeScrolled();
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss">

.navbar {
  position: fixed;
  width: 100vw;
  height: 55px;
  @include mobile {
    text-align: left;
  }
  //height: 4vh;
  /*@include mobile {
    height: 9vw;
  }*/
  top: 0px;
  left: 0px;
  padding: 30px 10px;
  z-index: 100;
  &.animated {
    //transition: background 0.25s, padding 0.25s;
    transition: padding 0.25s;
  }

  .hamburger {
    @include display-hide('tablet');
    @include display-hide('desktop');
    @include background(url('../../assets/hamburger.svg'), contain);
    background-color: transparent;
    background-position: left center !important;
    width: 175px;
    font-size: 15px;
    //color: transparent;
    margin-left: 10px;
    margin-right: 20px;
    border-radius: 0px;
    transition: opacity 0.05s;
  }

  @include mobile {
    .buttons {
      @include rounded;
      @include white-bg;
      pointer-events: none;
      //display: none;
      opacity: 0;
      transition: opacity 0.15s;
      z-index: 10000;
      display: block;
      position: fixed;
      width: calc(100vw - 50px);
      top: 25px;
      left: 25px;
      padding: 10px 0px 10px 0px;
      .navbtn {
        display: block;
        text-align: center;
        width: 100%;
        padding: 20px;
        color: black!important;
      }
    }
    .background {
      transition: opacity 0.15s;
      pointer-events: none;
      display: block;
      position: fixed;
      opacity: 0;
      top: 0px;
      left: 0px;
      background-color: black;
      width: 100vw;
      height: 100vh;
    }
  }

  &:not(.side) {
    @include display-not(mobile) {
      .navbtn-dropdown-content {
        margin-top: 2.2rem;
      }
    }
  }

}

@include mobile {
  #navigation.menu {
    .buttons {
      opacity: 1;
      pointer-events: all;
      .navbtn.underline {
        &:after {
          background: none!important;
        }
      }
    }
    .navbar {
      .hamburger {
        opacity: 0;
      }
      .background {
        opacity: 0.5;
        pointer-events: all;
      }
      .navbtn-dropdown.open {
        .navbtn-dropdown-content {
          position: relative;
          margin-left: inherit!important;
          display: block!important;
          .navbtn-dropdown-box {
            background-color: initial!important;
          }
        }
      }
    }
  }
}

.navbar.side {
  @include display-not(mobile) {
    position: fixed;
    height: 100vh;
    width: 300px;
    .buttons {
      .navbtn {
        display: block;
        width: 100%;
        height: 4vh;
        text-align: center;
      }
      .navbtn-dropdown-content {
        position: relative;
        margin-left: inherit!important;
      }
    }
  }
}

.navbar {
  .buttons {
    @include display-not(mobile) {
      width: 100%;
      display: flex;
      padding: 0 30px;
      box-sizing: border-box;
      .navbtn-back {
        display: none;
      }
    }
    @include mobile {
      &.dropdown-open > .navbtn {
        display: none;
      }
      &:not(.dropdown-open) {
        .navbtn-back {
          display: none;
        }
      }
    }
  }
}

</style>
