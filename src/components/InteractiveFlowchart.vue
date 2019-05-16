<template>
  <div>
    <div id="intro" v-if="showIntro">
      <div>
        <p>Which StraitsTimes interactive graphics team member are you?</p>
        <button @click="showIntro = false">Play</button>
      </div>
    </div>
    <div id="navigation" v-else>
      <div class="button"><img src="back.svg"></div>
      <div class="button" @click="backToStart"><img src="restart.svg"></div>
    </div>
    <graphic :viewBox="viewBox"></graphic>
  </div>
</template>

<script>
import { TweenMax } from 'gsap'
import graphic from './Graphic.vue'
const SPEED = 1500

export default {
  components: { graphic },
  data () {
    return {
      showIntro: true,
      padding: 100,
      travelling: false,
      targetPath: null,
      offset: 0,
      x: null,
      y: null,
      width: null,
      height: null
    }
  },
  mounted () {
    this.initialView()
    this.clearPaths()
    this.initiateFlow()
  },
  computed: {
    viewBox () {
      if (this.travelling) {
        const {x, y} = getPointAtLength(this.targetPath, this.offset)
        return [
          x - (this.width / 2), // so that viewbox is topleft
          y - (this.height / 2),
          this.width,
          this.height
        ].join(',')
      } else {
        return [
          this.x - (this.width / 2),
          this.y - (this.height / 2),
          this.width,
          this.height
        ].join(',')
      }
    }
  },
  methods: {
    initialView () {
      this.flowchart = document.getElementById('flowchart')
      const start = this.flowchart.getElementById('n.start').getBBox()
      this.x = start.x + (start.width / 2) // center point of node
      this.y = start.y + (start.height / 2)// center point of node
      this.width = start.width + 2 * this.padding
      this.height = start.height + 2 * this.padding
    },
    clearPaths () {
      const paths = this.flowchart.querySelectorAll('*[id^="p."]')
      Array.prototype.forEach.call(paths, path => {
        const pathElement = this.flowchart.getElementById(path.id)
        const pathLength = pathElement.getTotalLength()

        // path not showing initially
        pathElement.setAttribute('stroke-dasharray', pathLength)
        pathElement.setAttribute('stroke-dashoffset', pathLength)
      })
    },
    initiateFlow () {
      const choices = this.flowchart.querySelectorAll('*[id^="c."]')
      // For IE compatibility
      Array.prototype.forEach.call(choices, choice => {
        const pathElement = this.flowchart.getElementById(choice.id.replace('c.', 'p.'))
        const pathLength = pathElement.getTotalLength()

        this.flowchart.getElementById(choice.id).addEventListener('click', () => {
          this.drawPath(pathElement, pathLength / SPEED)
          this.moveTo(choice.id, pathElement, pathLength)
        })
      })
    },
    drawPath (pathElement, duration) {
      TweenMax.to(pathElement, duration, {
        attr: { 'stroke-dashoffset': 0 },
        ease: Power1.easeInOut
      })
    },
    moveTo (choice, pathElement, pathLength, duration) {
      const nextNode = choice.slice(2).split('--')[1]
      const n = this.flowchart.getElementById('n.' + nextNode).getBBox()
      const finalPoint = pathElement.getPointAtLength(pathLength)

      this.x = n.x + (n.width / 2)
      this.y = n.y + (n.height / 2)

      // so that viewbox continues on from end of path to middle of node
      const difference = [this.x - finalPoint.x, this.y - finalPoint.y].join(',')
      this.targetPath = pathElement.getAttribute('d') + 'l' + difference

      this.travelling = true
      TweenMax.fromTo(this.$data, pathLength / SPEED, { offset: 0 }, {
        offset: getPathLength(this.targetPath),
        width: n.width + 2 * this.padding,
        height: n.height + 2 * this.padding,
        ease: Power1.easeInOut,
        onComplete: () => {
          this.travelling = false
        }
      })
    },
    backToStart () {
      this.travelling = true

      this.initialView()
      this.clearPaths()

      TweenMax.to(this.$data, 0, {
        x: this.x,
        y: this.y,
        width: this.width,
        height: this.height,
        ease: Power1.easeInOut,
        onComplete: () => {
          this.travelling = false
        }
      })
    }
  }
}

// because path has not been mounted yet
const $path = document.createElementNS('http://www.w3.org/2000/svg', 'path')
// duplicate the d of the path
function getPathLength (d) {
  $path.setAttribute('d', d)
  return Math.ceil($path.getTotalLength())
}

function getPointAtLength (d, offset) {
  $path.setAttribute('d', d)
  return $path.getPointAtLength(offset)
}
</script>

<style>
body {
  margin: 0;
  overflow: hidden;
}
div, svg {
  height: 100%;
  width: 100%;
}
#intro {
  display: table;
  position: absolute;
  width: 70vw;
  height: 70vh;
  left: 50%;
  margin-left: -35vw;
  top: 50%;
  margin-top: -35vh;
  background-color: #5A9DDC;
  border-radius: 30px;
  border: 1px solid rgba(12,43,87,.5);
  text-align: center;
  font-family: 'Courgette-Regular';
  font-size: 36px;
  color: white;
}
#intro div {
  display: table-cell;
  vertical-align: middle;
}
#intro p {
  padding: 0 10%;
}
#navigation {
  position: absolute;
  width: 100%;
  height: 50px;
}
#navigation .button {
  height: 40px;
  width: 40px;
  float: left;
  padding: 5px;
  cursor: pointer;
}
#navigation .button:first-child {
  float: left;
}
#navigation .button:last-child {
  float: right;
}
#navigation img {
  height: 100%;
  width: 100%;
}
button {
  display: block;
  margin: 0 auto;
  font-family: 'FiraSans-Regular';
  padding: 5px 25px;
  border-radius: 25px;
  outline: none;
  background-color: #0FA3B1;
  color: white;
  font-size: 20px;
  cursor: pointer;
}
/* all choices to look selectable */
g[id^="c."] {
  cursor: pointer;
}
g[id^="c."]:hover > path {
  transform-origin: center center;
  transform-box: fill-box;
  transform: scale(1.03);
}
g[id^="c."]:hover > text {
  font-size: 20.8px;
}
@font-face {
  font-family: Courgette-Regular;
  src: url(../assets/Courgette-Regular.ttf);
}
@font-face {
  font-family: FiraSans-Regular;
  src: url(../assets/FiraSans-Regular.ttf);
}
@media only screen and (max-width: 420px) {
  #intro {
    font-size: 30px;
  }
}
</style>
