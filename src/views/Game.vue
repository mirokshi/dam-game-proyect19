
<template>
  <div id="game"></div>
</template>
  <script>
    /* eslint-disable */
    import Phaser from 'phaser'
    import _TILESET_SET from '../assets/tileset.png'
    import _JSON_MAP from '../assets/map.json'

    export default {
      name: 'Game',
      created() {
        var controls = false
        // PHASER 2.6 -> phaser-ce
        // PHASER 3.0 -> phaser
        let config = {
          type: Phaser.AUTO,
          width: window.innerWidth -25,
          height: window.innerHeight -20,
          physics: {
            default: 'arcade',
            arcade: {
              gravity: {y: 200}
            }
          },
          // NO HI HA STATES A 3.0 -> SCENES
          scene: {
            preload() {
              this.load.image('tileset', _TILESET_SET)
              this.load.tilemapTiledJSON("map",_JSON_MAP)
            },
            create() {
              // let map = this.make.tilemap({ key: "_JSON_MAP" })
              // let tileset = map.addTilesetImage("game", "tileset")
              // map.createStaticLayer("Background", tileset, 0, 0)
              // map.createStaticLayer("walls", tileset, 0, 0)




            },
            update() {

            }
          }
        }
        // eslint-disable-next-line
        new Phaser.Game(config)
      }
    }
  </script>

