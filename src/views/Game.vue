
<template>
  <div id="game"></div>
</template>
  <script>
    /* eslint-disable */
    import Phaser from 'phaser'
    import _TILESET_SET from '../assets/tileSets/tilesets.png'
    import _JSON_MAP from '../assets/tileSets/map.json'
    import _PLAYER_BOMB from '../assets/tileSets/playerBomb/idle/26.png'

    export default {
      name: 'Game',
      created() {
        var controls = false
        // PHASER 2.6 -> phaser-ce
        // PHASER 3.0 -> phaser
        let config = {
          type: Phaser.AUTO,
          width: window.innerWidth-25,
          height: window.innerHeight-20,
          physics: {
            default: 'arcade',
            arcade: {
              gravity: {y: 200}
            }
          },
          // NO HI HA STATES A 3.0 -> SCENES
          scene: {
            preload() {
              console.log("PRELOAD");
              this.load.image('tileset', _TILESET_SET)
              this.load.spritesheet('player',_PLAYER_BOMB,{frameWidth:28,frameHeight:22})
              this.load.tilemapTiledJSON("map",_JSON_MAP)
            },
            create() {
              console.log("CREATED");
              this.cameras.main.backgroundColor.setTo(52,152,219)
              this.level = this.physics.add.staticGroup()
              let map = this.make.tilemap({ key: "map" })
              let tileset = map.addTilesetImage("tilesets", "tileset")
              map.createStaticLayer("background", tileset, 0, 0)
              map.createStaticLayer("floor", tileset, 0, 0)

              // this.cameras.main.setZoom(0.5)
              this.cameras.main.startFollow()
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

