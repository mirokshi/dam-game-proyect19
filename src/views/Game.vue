<template>
  <div id="game"></div>
</template>
  <script>
    /* eslint-disable */
    import Phaser from 'phaser'
    import _TILESET_SET from '../assets/tileSets/tilesets.png'
    import _JSON_MAP from '../assets/tileSets/map.json'
    import _PLAYER from '../assets/tileSets/metalslug_mummy37x45.png'
    var player;


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
              gravity: {y: 300},
              debug :true
            }
          },
          // NO HI HA STATES A 3.0 -> SCENES
          scene: {
            preload() {
              console.log("PRELOAD");
              this.load.image('tileset', _TILESET_SET)
              this.load.tilemapTiledJSON("map",_JSON_MAP)
              this.load.spritesheet('player',_PLAYER ,{ frameWidth: 37, frameHeight: 45 })

            },
            create() {
              const camera = this.cameras.main;
              console.log("CREATED");
              let map = this.make.tilemap({ key: "map" })
              let tileset = map.addTilesetImage("tilesets", "tileset")
              let background =map.createStaticLayer("background", tileset, 0, 0)
              let floor = map.createStaticLayer("floor", tileset, 0, 0)
              let elevation = map.createStaticLayer("elevation",tileset,0,0)

              background.setCollisionByExclusion([-1]);
              floor.setCollisionByExclusion([-1]);
              elevation.setCollisionByExclusion([-1]);


              // Habilitem un cursor per a les fletxes del teclat
              this.cursors = this.input.keyboard.createCursorKeys();

              // La càmara no sortirà del món
              camera.setBounds(0, 0, map.widthInPixels, map.heightInPixels);

              let spawnpoint = map.findObject('player', obj => obj.name === 'spanwPoint');

              player = this.physics.add.sprite(spawnpoint.x, spawnpoint.y, 'player');
              player.setScale(2);
              player.lives = 3;

              // Animació de correr del player
              this.anims.create({
                key: 'walk',
                frames: this.anims.generateFrameNames('player', {
                  frames: [0, 17]
                }),
                frameRate: 8,
                repeat: -1
              });

              this.physics.add.collider(player,floor)
              this.physics.add.collider(player,elevation)

              camera.startFollow(player)

            },
            update() {
              if (this.cursors.left.isDown) {
                player.anims.play('walk', true)
                player.setVelocityX(-500)
                player.flipX = true
              }else if (this.cursors.right.isDown) {
                player.anims.play('walk', true)
                player.setVelocityX(500)
                player.flipX = false
              } else {
                player.setFrame(9)
                player.setVelocityX(0)
              }

              if (this.cursors.up.isDown && player.body.onFloor()) {
                player.setVelocityY(-900)
              }
              if(player.body.velocity.y < 0){
                player.setFrame(14)
              }
              if(player.body.velocity.y > 0){
                player.setFrame(17)
                player.body.setMaxVelocity(500, 800);
              }

            }
          }
        }
        // eslint-disable-next-line
        new Phaser.Game(config)
      }
    }
  </script>

