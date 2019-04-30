<template>
  <div id="game"></div>
</template>

<script>
  import Phaser from 'phaser'
  import background from '../assets/background.jpg'
  export default {
    name: 'Game',
    mounted () {
      // PHASER 2.6 -> phaser-ce
      // PHASER 3.0 -> phaser
      let config = {
        type: Phaser.AUTO,
        width: 500,
        height: 200,
        physics: {
          default: 'arcade',
          arcade: {
            gravity: { y: 200 }
          }
        },
        // NO HI HA STATES A 3.0 -> SCENES
        scene: {
          preload () {
              this.load.image('background', background)
          },
          create () {

          },
          update () {

          }
        }
      }
      // eslint-disable-next-line
      new Phaser.Game(config)
    }
  }
</script>
