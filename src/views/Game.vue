<template>
  <div id="game"></div>
</template>
  <script>
    /* eslint-disable */
    import Phaser from 'phaser'
    import _TILESET_SET from '../assets/tileSets/tilesets.png'
    import _JSON_MAP from '../assets/tileSets/map.json'
    import _PLAYER_BOMB from '../assets/tileSets/dude.png'

    var player;
    var platforms;
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
              this.load.tilemapTiledJSON("map",_JSON_MAP)
              this.load.spritesheet('player',_PLAYER_BOMB,{frameWidth:280,frameHeight:220})
            },
            create() {
              console.log("CREATED");
              // this.cameras.main.backgroundColor.setTo(52,152,219)

              let map = this.make.tilemap({ key: "map" })
              let tileset = map.addTilesetImage("tilesets", "tileset")
              map.createStaticLayer("background", tileset, 0, 0)
              map.createStaticLayer("floor", tileset, 0, 0)
              this.physics.add.staticGroup();

              player = this.physics.add.sprite(200, 850, 'player');
              player.setBounce(0.2);
              player.setCollideWorldBounds(true);

              this.anims.create({
                key: 'left',
                frames: this.anims.generateFrameNumbers('player', { start: 0, end: 3 }),
                frameRate: 10,
                repeat: -1
              });

              this.anims.create({
                key: 'turn',
                frames: [ { key: 'player', frame: 4 } ],
                frameRate: 20
              });

              this.anims.create({
                key: 'right',
                frames: this.anims.generateFrameNumbers('player', { start: 5, end: 8 }),
                frameRate: 10,
                repeat: -1
              });

              this.cameras.main.setZoom(0.5)
              this.cameras.main.startFollow(player)
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

