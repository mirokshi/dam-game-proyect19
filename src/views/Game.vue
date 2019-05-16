<template>
  <div id="game"></div>
</template>
  <script>
    /* eslint-disable */
    import Phaser from 'phaser'
    import _TILESET_SET from '../assets/tileSets/tileset.png'
    import _JSON_MAP from '../assets/tileSets/mapa.json'
    import _PLAYER from '../assets/player/cyclops.png'
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
              this.load.spritesheet('player',_PLAYER ,{ frameWidth: 64, frameHeight: 64 })

            },
            create() {
              const camera = this.cameras.main;
              console.log("CREATED");
              let map = this.make.tilemap({ key: "map" })
              let tileset = map.addTilesetImage("tileset", "tileset")
              let backgroundP =map.createStaticLayer("backgroundP", tileset, 0, 0)
              let backgroundS =map.createStaticLayer("backgroundS", tileset, 0, 0)
              let details = map.createStaticLayer("details", tileset, 0, 0)
              let walls = map.createStaticLayer("walls",tileset,0,0)

              backgroundP.setCollisionByExclusion([-1]);
              backgroundS.setCollisionByExclusion([-1]);
              details.setCollisionByExclusion([-1]);
              walls.setCollisionByExclusion([-1]);


              // Habilitem un cursor per a les fletxes del teclat
              this.cursors = this.input.keyboard.createCursorKeys();

              // La càmara no sortirà del món
              camera.setBounds(0, 0, map.widthInPixels, map.heightInPixels);

              let spawnpoint = map.findObject('player', obj => obj.name === 'spawnpoint');

              player = this.physics.add.sprite(spawnpoint.x, spawnpoint.y, 'player');
              // player.setScale(0.5);
              player.lives = 3;

              //Animacion parado
              this.anims.create({
                key:'stop',
                frames:this.anims.generateFrameNames('player',{
                  frames: [0,1,2,3,4,5,6,7,8,9,10,11,12,13,14]
                }),
                frameRate:7,
                repeat:-1
              });
              // Animació de correr del player
              this.anims.create({
                key: 'walk',
                frames: this.anims.generateFrameNames('player', {
                  frames: [15,16,17,18,19,20,21,22,23,24,25, 26]
                }),
                frameRate: 20,
                repeat: -1
              });

              this.physics.add.collider(player,walls)
              this.physics.add.collider(player,walls)
              // this.physics.add.collider(player,elevation)
              // camera.setZoom(4)
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
                player.anims.play('stop',true)
                player.setVelocityX(0)
              }

              if (this.cursors.up.isDown && player.body.onFloor()) {
                player.setFrame()
                player.setVelocityY(-900)
              }
              if(player.body.velocity.y < 0){
                player.setFrame(14)
              }
              if(player.body.velocity.y > 0){
                player.setFrame(17)
                player.body.setMaxVelocity(400, 550);
              }

            }
          }
        }
        // eslint-disable-next-line
        new Phaser.Game(config)
      }
    }
  </script>

