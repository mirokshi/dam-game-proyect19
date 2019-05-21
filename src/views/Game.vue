<template>
  <div id="game"></div>
</template>
  <script>
    /* eslint-disable */
    import Phaser from 'phaser'
    import _IMG_LOADING from '../assets/gamepad.png'
    import _TILESET_SET from '../assets/tileSets/tileset.png'
    import _JSON_MAP from '../assets/tileSets/mapa.json'
    import _PLAYER from '../assets/characters/cyclops.png'
    import _ENEMIES from '../assets/characters/imp.png'
    import _OBJECTS from '../assets/objects/objects.png'
    //sounds
    import _DANCIN from '../assets/sounds/dancin.mp3'

    let camera;
    let progressBar;
    let progressBox;
    let player;
    let CoinLayer;
    let coins;
    let EnemyLayer;
    let enemies;

    //
    // function drawOctopus(game, map, num_octopus) {
    //   game.octopus = game.physics.add.group();
    //   game.octopus.enableBody = true;
    //
    //   for (let i  = 1; i<= num_octopus; i++){
    //     let octupusPosition = map.findObject("coins", obj => obj.name ==="coin"+i);
    //     let octopus = game.octopus.create(octupusPosition.x, octupusPosition.y-50,'objects',265);
    //     game.physics.add.existing(octopus)
    //     octopus.body.allowGravity = false;
    //   }
    // }
    function changeDirection (enemy) {
      enemy.speedX*=-1
    }
    function createCoins () {
      CoinLayer.forEach(object => {
        let obj = coins.create(object.x, object.y, 'objects',199)
        obj.body.width = object.width
        obj.body.height = object.height
        // obj.anims.play('spin', coins)
      })
    }
    function createEnemies () {
      EnemyLayer.forEach(object => {
        let obj = enemies.create(object.x, object.y, 'enemy')
        obj.setScale(1.5)
        // obj.setScale(object.width / 64, object.height / 64)
        // obj.setOrigin(0)
        obj.body.width = object.width
        obj.body.height = object.height
        obj.speedX = -50

      })
    }

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
          scene: [loader, inGame]
        }
        // eslint-disable-next-line
        new Phaser.Game(config)
      }
    }
    class loader extends Phaser.Scene{
      constructor (){
        super({key: 'loader'})
      }
      preload () {
        var progressBar = this.add.graphics()
        var progressBox = this.add.graphics()
        progressBox.fillStyle(0x222222, 0.8)
        progressBox.fillRect(240, 270, 320, 50)
        var width = this.cameras.main.width
        var height = this.cameras.main.height
        var loadingText = this.make.text({
          x: width / 2,
          y: height / 2 - 50,
          text: 'Loading...',
          style: {
            font: '20px monospace',
            fill: '#ffffff'
          }
        })
        loadingText.setOrigin(0.5, 0.5)
        var percentText = this.make.text({
          x: width / 2,
          y: height / 2 - 5,
          text: '0%',
          style: {
            font: '18px monospace',
            fill: '#ffffff'
          }
        })
        percentText.setOrigin(0.5, 0.5)
        var assetText = this.make.text({
          x: width / 2,
          y: height / 2 + 50,
          text: '',
          style: {
            font: '18px monospace',
            fill: '#ffffff'
          }
        })
        assetText.setOrigin(0.5, 0.5)
        this.load.on('progress', function (value) {
          percentText.setText(parseInt(value * 100) + '%')
          progressBar.clear()
          progressBar.fillStyle(0xffffff, 1)
          progressBar.fillRect(250, 280, 300 * value, 30)
        })
        this.load.on('fileprogress', function (file) {
          assetText.setText('Loading asset: ' + file.key)
        })
        this.load.on('complete', function () {
          progressBar.destroy()
          progressBox.destroy()
          loadingText.destroy()
          percentText.destroy()
          assetText.destroy()
        })
        this.load.image('tileset', _TILESET_SET)
        this.load.tilemapTiledJSON('map', _JSON_MAP)
        this.load.spritesheet('player',_PLAYER ,{ frameWidth: 64, frameHeight: 64 })
        this.load.spritesheet('objects',_OBJECTS ,{ frameWidth: 32, frameHeight: 32 })
        this.load.spritesheet('enemy',_ENEMIES ,{ frameWidth: 32, frameHeight: 32 })
        // this.load.image('spark', Spark)

        this.load.audio('sound',_DANCIN)
      }
      create () {
        this.scene.start('inGame')
      }
    }
    class inGame extends Phaser.Scene{
      constructor (){
        super({key:'inGame'})
      }
      preload(){
        console.log('PRELOAD');
      }
      create(){
        camera = this.cameras.main;
        console.log("CREATED");
        let logo = this.add.image(300,300,'logo')

        //musica
        let sound = this.sound.add('sound', { loop: true })
        // sound.play()

        let map = this.make.tilemap({ key: "map" })
        let tileset = map.addTilesetImage("tileset", "tileset")
        // let tilest_object = map.addTilesetImage("objects", "objects")
        let backgroundP =map.createStaticLayer("backgroundP", tileset, 0, 0)
        let backgroundS =map.createStaticLayer("backgroundS", tileset, 0, 0)
        let details = map.createStaticLayer("details", tileset, 0, 0)
        let walls = map.createStaticLayer("walls",tileset,0,0)
        CoinLayer = map.getObjectLayer('coins')['objects']
        EnemyLayer = map.getObjectLayer('enemies')['objects']

        coins = this.physics.add.staticGroup()
        enemies = this.physics.add.group()
        // enemies.enableBody=true;

        backgroundP.setCollisionByExclusion([-1]);
        backgroundS.setCollisionByExclusion([-1]);
        details.setCollisionByExclusion([-1]);
        walls.setCollisionByExclusion([-1]);
        // walls.setCollisionBetween(0,1999)

        // Habilitem un cursor per a les fletxes del teclat
        this.cursors = this.input.keyboard.createCursorKeys();

        // La càmara no sortirà del món
        camera.setBounds(0, 0, map.widthInPixels, map.heightInPixels);

        //Aparicion del personaje
        let spawnpoint = map.findObject('player', obj => obj.name === 'spawnpoint');
        player = this.physics.add.sprite(spawnpoint.x, spawnpoint.y, 'player');

        player.body.setSize(24, 64, 40, 0)
        // player.setScale(2);
        player.lives = 3;

        //Animacion parado
        this.anims.create({
          key:'idle',
          frames:this.anims.generateFrameNames('player',{
            frames: [0,1,2,3,4,5,6,7,8,9,10,11,12,13,14]
          }),
          frameRate:7,
          repeat:-1
        });
        // Animación de correr del player
        this.anims.create({
          key: 'walk',
          frames: this.anims.generateFrameNames('player', {
            frames: [15,16,17,18,19,20,21,22,23,24,25, 26]
          }),
          frameRate: 20,
          repeat: -1
        });
        //Animacion de correr enemy
        this.anims.create({
          key:'walk_enemy',
          frames:this.anims.generateFrameNumbers('enemy',{start:8,end:15}),
          frameRate: 12,
          repeat: -1
        });

        createCoins()
        createEnemies()

        this.physics.add.collider(player,walls)
        // this.physics.add.collider(player,coins)
        this.physics.add.collider(enemies,walls)

        // drawOctopus(this,map,14)
        camera.setZoom(2)
        camera.startFollow(player)


      }
      update(){
        if (enemies){
          enemies.children.entries.forEach(function (enemy) {
            enemy.setVelocityX(enemy.speedX)
            if (enemy.speedX>0){
              enemy.anims.play('walk_enemy', true)

            }
              // enemy.setVelocityX(-200)
              // enemy.flipX = false
          })
        }
        if (this.cursors.left.isDown) {
          player.anims.play('walk', true)
          player.setVelocityX(-500)
          player.flipX = true
        }else if (this.cursors.right.isDown) {
          player.anims.play('walk', true)
          player.setVelocityX(500)
          player.flipX = false
        } else {
          player.anims.play('idle',true)
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

  </script>

