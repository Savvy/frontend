<template lang="html">
  <div class="demo">
    <no-ssr>
      <CodeBlock
        v-model="code"
        class="editor"
        :mode="'syntek'"
      />
    </no-ssr>

    <canvas
      ref="canvas"
      class="canvas"
      width="400px"
      height="400px"
    />
  </div>
</template>

<script>
import SnakeSnippet from '~/assets/snippets/snake.txt';
import Game from '~/assets/snake/Game.js';

let CodeBlock;
let syntek;
if (process.browser) {
  // eslint-disable-next-line global-require
  syntek = require('../node_modules/syntek/build/bundle.js');

  // eslint-disable-next-line global-require
  CodeBlock = require('@syntek/vue-editor').CodeBlock;
}

export default {
  components: {
    CodeBlock,
  },
  data() {
    return {
      code: SnakeSnippet,
      game: null,
    };
  },
  watch: {
    code() {
      this.run();
    },
  },
  mounted() {
    // Declare functions
    syntek.interpreter.globalContext.declare('moveUp', syntek.FunctionStruct, new syntek.FunctionStruct(syntek.interpreter.globalContext, [], () => {
      this.game.snake.moveUp();
    }));

    syntek.interpreter.globalContext.declare('moveRight', syntek.FunctionStruct, new syntek.FunctionStruct(syntek.interpreter.globalContext, [], () => {
      this.game.snake.moveRight();
    }));

    syntek.interpreter.globalContext.declare('moveDown', syntek.FunctionStruct, new syntek.FunctionStruct(syntek.interpreter.globalContext, [], () => {
      this.game.snake.moveDown();
    }));

    syntek.interpreter.globalContext.declare('moveLeft', syntek.FunctionStruct, new syntek.FunctionStruct(syntek.interpreter.globalContext, [], () => {
      this.game.snake.moveLeft();
    }));

    // Start the game
    this.run();

    document.addEventListener('keydown', (event) => {
      let fnName;

      switch (event.which) {
        case 37: fnName = 'arrowLeft'; break;
        case 38: fnName = 'arrowUp'; break;
        case 39: fnName = 'arrowRight'; break;
        case 40: fnName = 'arrowDown'; break;
        default: break;
      }

      if (!fnName) {
        return;
      }

      const fn = syntek.interpreter.context.variables[fnName];
      if (fn) {
        fn.value.exec([]);
      }

      event.preventDefault();
    });

    setInterval(() => {
      // Loop
      const loopFunction = syntek.interpreter.context.variables.loop;
      if (loopFunction) {
        loopFunction.value.exec([]);
      }

      // Draw the canvas
      requestAnimationFrame(() => this.game.draw());
    }, 50);
  },
  methods: {
    run() {
      this.game = new Game(this.$refs.canvas);

      try {
        const compiled = syntek.compile(this.code, 'snake');
        syntek.run(compiled);
      } catch (err) {
        console.error(err);
      }
    },
  },
};
</script>

<style lang="scss" scoped>
@import 'sass-mq';
@import '~/assets/css/_globals.scss';

.demo {
  margin-top: 40vh;
  border: $border-yellow;
  height: 800px;

  display: grid;

  // On mobile display editor and canvas underneath eachother
  @include mq($until: tablet) {
    grid-template-rows: 1fr 1fr;
    margin-right: 5px;
    margin-left: 5px;
  }

  // On tablet and desktop display them side by side
  @include mq($from: tablet) {
    grid-template-columns: 1fr 1fr;
    margin-right: 75px;
    margin-left: 75px;

    // Make margin at the sides slightly larger on desktop
    @include mq($from: desktop) {
      margin-right: 100px;
      margin-left: 100px;
    }
  }

  .editor {
    font-size: 1.1em;
    font-weight: normal;
    overflow-x: auto;
  }

  .canvas {
    background-color: $gray;
    color: $yellow;

    display: flex;
    justify-content: center;
    align-items: center;
  }

  // TODO: Remove this when the page continues further down
  margin-bottom: 100px;
}

</style>
