<template>
  <div>
    <div id="intro" v-if="showIntro">
      <div>
        <p>Which StraitsTimes interactive graphics team member are you?</p>
        <button @click="showIntro = false">Play</button>
      </div>
    </div>
    <div id="navigation" v-else>
      <div class="button" @click="backToPrevious"><img src="back.svg"></div>
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
      pathIds: [],
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
          this.pathIds.push(choice.id.replace('c.', 'p.'))
          this.drawPath(pathElement, pathLength / SPEED, 0)
          this.moveTo(choice.id, pathElement, pathLength, true)
        })
      })
    },
    drawPath (pathElement, duration, end) {
      TweenMax.to(pathElement, duration, {
        attr: { 'stroke-dashoffset': end },
        ease: Power1.easeInOut
      })
    },
    moveTo (choice, pathElement, pathLength, forwards) {
      const startNode = choice.slice(2).split('--')[0]
      const nextNode = choice.slice(2).split('--')[1]
      const s = this.flowchart.getElementById('n.' + startNode).getBBox()
      const n = this.flowchart.getElementById('n.' + nextNode).getBBox()

      const startX = s.x + (s.width / 2)
      const startY = s.y + (s.height / 2)

      const nextX = n.x + (n.width / 2)
      const nextY = n.y + (n.height / 2)

      // so that viewbox continues on from end of path to middle of node
      const startD = 'M' + startX + ',' + startY + 'L'
      const nextD = 'L' + nextX + ',' + nextY
      this.targetPath = pathElement.getAttribute('d').replace(/^M/, startD) + nextD

      let fromOffset
      let toOffset
      if (forwards) {
        fromOffset = 0
        toOffset = getPathLength(this.targetPath)
        this.x = nextX
        this.y = nextY
      } else {
        fromOffset = getPathLength(this.targetPath)
        toOffset = 0
        this.x = startX
        this.y = startY
      }

      this.travelling = true

      TweenMax.fromTo(this.$data, pathLength / SPEED, { offset: fromOffset }, {
        offset: toOffset,
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
      this.pathIds = []

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
    },
    backToPrevious () {
      if (this.pathIds.length !== 0) {
        // move path drawing backwards
        const pathId = this.pathIds[this.pathIds.length - 1]
        this.pathIds.pop()
        const pathElementBack = this.flowchart.getElementById(pathId)
        const pathElementBackLength = pathElementBack.getTotalLength()
        this.drawPath(pathElementBack, pathElementBackLength / SPEED, pathElementBackLength)

        // move viewport backwards
        this.moveTo(pathId, pathElementBack, pathElementBackLength, false)
      }
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
