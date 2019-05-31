<template>
  <div id="game"></div>
</template>
  <script>
    /* eslint-disable */
    import Phaser from 'phaser'
    import _TILESET_SET from '../assets/tileSets/tileset.png'
    import _JSON_MAP from '../assets/tileSets/mapa.json'
    import _JSON_MAP2 from '../assets/tileSets/mapa2.json'
    import _PLAYER from '../assets/characters/cyclops.png'
    import _ENEMIES from '../assets/characters/imp.png'
    import _OBJECTS from '../assets/objects/objects.png'
    import _DEMON from '../assets/characters/demon160x128.png'
    import _BUTTONS from '../assets/objects/button.png'
    import _START from '../assets/img/game.png'
    import _END from '../assets/img/lose.png'
    import _WIN from '../assets/img/win.png'
    import _DUNGEON  from '../assets/img/dungeon.png'
    //particles
    import _PARTICLES_ARROW from '../assets/particles/arrow.png'
    import _PARTICLES_JSON from '../assets/particles/game/shapes.json'
    import _PARTICLES from '../assets/particles/game/shapes.png'


    //sounds
    import _GAME_MUSIC from '../assets/sounds/wapons.mp3'
    import _SOUND_EAT from '../assets/sounds/eat.mp3'
    import _SOUND_JUMP from '../assets/sounds/jump.wav'
    import _SOUND_DAMAGE from '../assets/sounds/damage.wav'
    import _SOUND_LOSE from '../assets/sounds/lose.wav'
    import _SOUND_WIN from '../assets/sounds/win.mp3'

    let camera, player;
    let coins,CoinLayer,coinScore = 0;
    let EnemyLayer, enemies;
    let DemonLayer, demons;
    let scoreText;
    let emitter;
    let soundEat,soundJump,soundDamage,soundLose,soundWin,music;
    let musicOn=true;
    let livesText;
    let bgStart, bgEnd,bgWin,bgdungeon;

    function changeDirection (enemy) {
      enemy.speedX*=-1
    }
    function createCoins () {
      CoinLayer.forEach(object => {
        let obj = coins.create(object.x, object.y, 'objects',243)
        obj.body.width = object.width
        obj.body.height = object.height
      })
    }
    function  takeCoin(player, coin) {
      coin.disableBody(true,true)
      coinScore+=10
      scoreText.setText('Score: '+coinScore)
      soundEat.play()
      // console.log(scoreText);

    }
    function createEnemies () {
      EnemyLayer.forEach(object => {
        let obj = enemies.create(object.x, object.y, 'enemy',1)
        obj.setScale(1.5)
        // obj.setScale(object.width / 64, object.height / 64)
        // obj.setOrigin(0)
        obj.body.width = object.width
        obj.body.height = object.height
        obj.speedX = 110

      })
    }
    function createDemons () {
      DemonLayer.forEach(object => {
        let obj = demons.create(object.x, object.y, 'demon',1)
        obj.body.setSize(70, 100, 90, 28)
        // frameWidth: 160, frameHeight: 128
        obj.setScale(1.3)

        // obj.setOrigin(0)
        obj.body.width = object.width
        obj.body.height = object.height
        obj.staticY = object.y
        obj.speedX = 100
        obj.body.allowGravity = false

      })
    }

    function lesslive (player, enemy) {
      player.lives = player.lives - 1;
      player.scene.cameras.main.shake(250);
      player.x =190;
      player.y=150;
      // player.disableBody(true, true)
      livesText.setText('Lives : '+player.lives);
      soundDamage.play();
    }
    function winGame1(){
      music.pause()
      this.scene.start('inGame2');
    }
    function winGame(){
      music.pause();
      this.scene.start('win');

    }
    export default {
      name: 'Game',
      created() {
        var controls = false
        // PHASER 2.6 -> phaser-ce
        // PHASER 3.0 -> phaser
        let config = {
          type: Phaser.AUTO,
          scale: {
            mode: Phaser.Scale.FIT,
            parent: 'game',
            autoCenter: Phaser.Scale.CENTER_BOTH,
            width: window.innerWidth,
            height: window.innerHeight
          },
          physics: {
            default: 'arcade',
            arcade: {
              gravity: {y: 300},
              debug :false
            }
          },
          // NO HI HA STATES A 3.0 -> SCENES
          scene: [loader,gameStart, inGame, inGame2,win,gameOver]
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
        var width = this.cameras.main.width
        var height = this.cameras.main.height
        progressBox.fillRect(window.x = width / 2 - 165, window.y = height / 2 - 75, 320, 50)
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
          progressBar.fillRect(window.x = width / 2 - 165, window.y = height / 2 - 65, 300 * value, 30)
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
        this.load.tilemapTiledJSON('map2', _JSON_MAP2)
        this.load.spritesheet('player',_PLAYER ,{ frameWidth: 64, frameHeight: 64 })
        this.load.spritesheet('objects',_OBJECTS ,{ frameWidth: 32, frameHeight: 32 })
        this.load.spritesheet('enemy',_ENEMIES ,{ frameWidth: 32, frameHeight: 32 })
        this.load.spritesheet('demon',_DEMON ,{ frameWidth: 160, frameHeight: 128 })

        //joystick
        this.load.plugin('rexvirtualjoystickplugin', 'https://raw.githubusercontent.com/rexrainbow/phaser3-rex-notes/master/plugins/dist/rexvirtualjoystickplugin.min.js', true)

        //bg
        this.load.image('bgStart',_START)
        this.load.image('bgEnd',_END)
        this.load.image('bgWin',_WIN)
        this.load.image('dungeon',_DUNGEON)
        //btn
        this.load.spritesheet('buttons',_BUTTONS,{frameWidth:16,frameHeight:16})
        //pa
        this.load.atlas('collapse',_PARTICLES,_PARTICLES_JSON)
        this.load.atlas('winParticles',_PARTICLES,_PARTICLES_JSON)
        this.load.image('arrow',_PARTICLES_ARROW)
        //so
        this.load.audio('sound',_GAME_MUSIC)
        this.load.audio('eat',_SOUND_EAT)
        this.load.audio('jump',_SOUND_JUMP)
        this.load.audio('damage',_SOUND_DAMAGE)
        this.load.audio('lose',_SOUND_LOSE)
        this.load.audio('win',_SOUND_WIN)

      }
      create () {
        this.scene.start('gameStart')
      }
    }

    class gameStart extends Phaser.Scene{
      constructor(){
        super({key:'gameStart'})
      }
      preload(){
        console.log('PRELOAD GAME START');
      }
      create(){
        console.log('CREATED GAME START');

        bgStart = this.add.image(0,0,'bgStart').setOrigin(0).setDepth()
        bgStart.displayHeight = this.game.renderer.height
        bgStart.displayWidth = this.game.renderer.width

        bgdungeon = this.add.image(this.game.renderer.width / 2-290, this.game.renderer.height / 2-150,'dungeon').setOrigin(0).setDepth()

        const startButton = this.add.sprite(this.game.renderer.width / 2 - 100, this.game.renderer.height / 2,'buttons',1)
                .setInteractive()
                .on('pointerdown', () => this.scene.start('inGame'));
          startButton.displayHeight =40
          startButton.displayWidth =40

        const soundButton = this.add.sprite(this.game.renderer.width / 2 + 100, this.game.renderer.height / 2,'buttons',13)
                .setInteractive()
                .on('pointerdown',() =>{
                          musicOn=false
                })
        soundButton.displayHeight =40
        soundButton.displayWidth =40

        if (this.sys.game.device.os.desktop) {
          startButton.setScale(4)
          soundButton.setScale(4)
        }

      }

    }
    class gameOver extends Phaser.Scene{
      constructor(){
        super({key:'gameOver'})
      }
      preload(){
        console.log('PRELOAD GAME OVER');
      }
      create(){
        console.log('CREATED GAME OVER');
        soundLose = this.sound.add('lose',{loop:false})
        soundLose.play()
        bgEnd = this.add.image(0,0,'bgEnd').setOrigin(0).setDepth()
        bgEnd.displayHeight = this.game.renderer.height
        bgEnd.displayWidth = this.game.renderer.width

        const restartButton = this.add.sprite(this.game.renderer.width / 2, this.game.renderer.height / 2+50,'buttons',3)
                .setInteractive()
                .on('pointerdown', () => this.scene.start('inGame'));
        // restartButton.setVisible(false)
        restartButton.displayWidth=40
        restartButton.displayHeight=40

        if (this.sys.game.device.os.desktop) {
          restartButton.setScale(4)
        }
      }
    }
    class win extends Phaser.Scene{
      constructor(){
        super({key:'win'})
      }
      preload(){
        console.log('PRELOAD WIN');
      }
      create() {
        console.log('CREATED WIN');
        soundWin = this.sound.add('win',{loop:false})
        soundWin.play()

        bgWin = this.add.image(0,0,'bgWin').setOrigin(0).setDepth()
        bgWin.displayHeight = this.game.renderer.height
        bgWin.displayWidth = this.game.renderer.width

        const restartButton = this.add.sprite(this.game.renderer.width / 2, this.game.renderer.height / 2+100,'buttons',3)
                .setInteractive()
                .on('pointerdown', () => {
                  soundWin.pause()
                  this.scene.start('inGame')
                });

        restartButton.displayWidth=40
        restartButton.displayHeight=40
        if (this.sys.game.device.os.desktop) {
          restartButton.setScale(4)
        }
        let particleswin = this.add.particles('winParticles')
        particleswin.createEmitter({
          "active":true,
          "visible":true,
          "collideBottom":true,
          "collideLeft":true,
          "collideRight":true,
          "collideTop":true,
          "on":true,
          "particleBringToTop":true,
          "radial":true,
          "frame":{"frames":["symbol_02"],
            "cycle":false,
            "quantity":1},
          "frequency":0,
          "gravityX":0,
          "gravityY":0,
          "maxParticles":0,
          "timeScale":0.5,
          "blendMode":3,
          "accelerationX":0,
          "accelerationY":0,
          "alpha":1,
          "angle":{"min":0,
            "max":360,
            "ease":"Linear"},
          "bounce":{"ease":"Linear",
            "min":1,
            "max":500},
          "delay":{"ease":"Linear",
            "min":1,
            "max":500},
          "lifespan":{"ease":"Linear",
            "min":1,
            "max":50000},
          "maxVelocityX":{"start":0,
            "end":0,
            "ease":"Linear"},
          "maxVelocityY":10000,
          "moveToX":0,
          "moveToY":0,
          "quantity":1,
          "rotate":0,
          "scale":{"ease":"Linear",
            "min":1,
            "max":2},
          "speed":{"ease":"Linear",
            "min":1,
            "max":500},
          "x":{"ease":"Linear",
            "min":0,
            "max":3000},
          "y":{"ease":"Linear",
            "min":1,
            "max":3000},
          "tint":[11142573,
            7441958,
            3893608,
            12013648],
          "emitZone":{"source":new Phaser.Geom.Circle(0,0,600),"type":"random"}})
      }
    }
    class inGame extends Phaser.Scene{
      constructor (){
        super({key:'inGame'})
      }
      preload(){
        console.log('PRELOAD IN GAME');
      }
      create(){
        camera = this.cameras.main;
        console.log("CREATED IN GAME");

        //musica
        music = this.sound.add('sound', { loop: true })
        //Souns
        soundEat = this.sound.add('eat', {loop: false})
        soundJump = this.sound.add('jump',{loop:false})
        soundDamage = this.sound.add('damage',{loop:false})


        let map = this.make.tilemap({ key: "map" })
        let tileset = map.addTilesetImage("tileset", "tileset")
        let backgroundP =map.createStaticLayer("backgroundP", tileset, 0, 0)
        let end = map.createStaticLayer("end",tileset,0,0)
        let collision = map.createStaticLayer("collisions",tileset,0,0)
        let backgroundS =map.createStaticLayer("backgroundS", tileset, 0, 0)
        let details = map.createStaticLayer("details", tileset, 0, 0)
        let walls = map.createStaticLayer("walls",tileset,0,0)
        CoinLayer = map.getObjectLayer('coins')['objects']
        EnemyLayer = map.getObjectLayer('enemies')['objects']

        coins = this.physics.add.staticGroup()
        enemies = this.physics.add.group()

        backgroundP.setCollisionByExclusion([-1]);
        end.setCollisionByExclusion([-1]);
        collision.setCollisionByExclusion([-1]);
        backgroundS.setCollisionByExclusion([-1]);
        details.setCollisionByExclusion([-1]);
        walls.setCollisionByExclusion([-1]);

        // La càmara no sortirà del món
        camera.setBounds(0, 0, map.widthInPixels, map.heightInPixels);

        //Aparicion del personaje
        let spawnpoint = map.findObject('player', obj => obj.name === 'spawnpoint');
        player = this.physics.add.sprite(spawnpoint.x, spawnpoint.y, 'player');
        player.body.setSize(24, 64, 40, 0)
        player.lives = 3;

        // Habilitar flechas
        this.cursors = this.input.keyboard.createCursorKeys();

        var particlescollapse1 = this.add.particles('collapse')
          particlescollapse1.createEmitter({
            "active":true,
            "visible":true,
            "collideBottom":true,
            "collideLeft":true,
            "collideRight":true,
            "collideTop":true,
            "on":true,
            "particleBringToTop":true,
            "radial":true,
            "frame":{
              "frames":["dirt_01"],
              "cycle":false,"quantity":1
            },
            "frequency":1,
            "gravityX":0,
            "gravityY":200,
            "maxParticles":0,
            "timeScale":1,
            "blendMode":1,
            "accelerationX":0,
            "accelerationY":0,
            "alpha":1,
            "angle":{"min":0,"max":360,"ease":"Linear"},
            "bounce":0,
            "delay":0,
            "lifespan":1000,
            "maxVelocityX":10000,
            "maxVelocityY":10000,
            "moveToX":0,
            "moveToY":0,
            "quantity":1,
            "rotate":0,
            "scale":1,
            "speed":0,
            "x":150,
            "y":110,
            "tint":[3091757],
            "emitZone":{"source":new Phaser.Geom.Rectangle(0,0,7000,50),"type":"random"},
            "deathZone":{"source":new Phaser.Geom.Rectangle(0,300,7000,50),"type":"onEnter"}
          });
        var particlescollapse2 = this.add.particles('collapse')
        particlescollapse2.createEmitter({
          "active":true,
          "visible":true,
          "collideBottom":true,
          "collideLeft":true,
          "collideRight":true,
          "collideTop":true,
          "on":true,
          "particleBringToTop":true,
          "radial":true,
          "frame":{
            "frames":["dirt_01"],
            "cycle":false,"quantity":1
          },
          "frequency":1,
          "gravityX":0,
          "gravityY":240,
          "maxParticles":0,
          "timeScale":1,
          "blendMode":1,
          "accelerationX":0,
          "accelerationY":0,
          "alpha":1,
          "angle":{"min":0,"max":360,"ease":"Linear"},
          "bounce":0,
          "delay":0,
          "lifespan":1000,
          "maxVelocityX":10000,
          "maxVelocityY":10000,
          "moveToX":0,
          "moveToY":0,
          "quantity":1,
          "rotate":0,
          "scale":1,
          "speed":0,
          "x":50,
          "y":740,
          "tint":[3091757],
          "emitZone":{"source":new Phaser.Geom.Rectangle(0,0,7000,50),"type":"random"},
          "deathZone":{"source":new Phaser.Geom.Rectangle(0,100,7000,50),"type":"onEnter"}
        });
        var particlescollapse3 = this.add.particles('collapse')
        particlescollapse3.createEmitter({
          "active":true,
          "visible":true,
          "collideBottom":true,
          "collideLeft":true,
          "collideRight":true,
          "collideTop":true,
          "on":true,
          "particleBringToTop":true,
          "radial":true,
          "frame":{
            "frames":["dirt_01"],
            "cycle":false,"quantity":1
          },
          "frequency":1,
          "gravityX":0,
          "gravityY":240,
          "maxParticles":0,
          "timeScale":1,
          "blendMode":1,
          "accelerationX":0,
          "accelerationY":0,
          "alpha":1,
          "angle":{"min":0,"max":360,"ease":"Linear"},
          "bounce":0,
          "delay":0,
          "lifespan":1000,
          "maxVelocityX":10000,
          "maxVelocityY":10000,
          "moveToX":0,
          "moveToY":0,
          "quantity":1,
          "rotate":0,
          "scale":1,
          "speed":0,
          "x":150,
          "y":1110,
          "tint":[3091757],
          "emitZone":{"source":new Phaser.Geom.Rectangle(0,0,7000,50),"type":"random"},
          "deathZone":{"source":new Phaser.Geom.Rectangle(0,100,7000,50),"type":"onEnter"}
        });



        var particles = this.add.particles('arrow')
        emitter = particles.createEmitter({
          x: 1080,
          y: 1400,
          speed: 180,
          lifespan: 400,
          rotate: 90 ,
          scale:0.1
        })
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
        this.physics.add.collider(player,enemies,lesslive)
        this.physics.add.collider(player,end,winGame1,null,this)
        this.physics.add.overlap(player,coins,takeCoin,null,this)
        this.physics.add.collider(enemies,walls)
        this.physics.add.collider(enemies,collision,changeDirection)

        if (this.sys.game.device.os.desktop) {
          camera.setZoom(2)
        }
        camera.startFollow(player)

        if (!this.sys.game.device.os.desktop) {
          this.joyStick = this.plugins.get('rexvirtualjoystickplugin').add(this, {
            x: this.cameras.main.centerX - 250,
            y: this.cameras.main.centerY + 100,
            radius: 40,
            base: this.add.graphics().fillStyle(0x150000).fillCircle(0, 0, 50),
            thumb: this.add.graphics().fillStyle(0xB45F06).fillCircle(0, 0, 30)
          })
          this.cursorKeysVirtual = this.joyStick.createCursorKeys()
        }

          if (!this.sys.game.device.os.desktop) {
          //movil
          scoreText=this.add.text(100,10,'Score: 0',{fontSize:'18px',fill:'#000'}).setScrollFactor(0).setDepth(200)
          scoreText.setColor('#ffffff')
          scoreText.setScrollFactor(0)
          livesText=this.add.text(100,40,'Lives : 3',{fontSize:'18px',fill:'#000'}).setScrollFactor(0).setDepth(200)
          livesText.setColor('#ffffff')
          livesText.setScrollFactor(0)
          }else{
          scoreText=this.add.text((window.innerWidth / 3) - 100, (window.innerHeight / 4),'Score: 0',{fontSize:'18px',fill:'#000'}).setScrollFactor(0).setDepth(200)
          scoreText.setColor('#ffffff')
          scoreText.setScrollFactor(0)
          livesText=this.add.text((window.innerWidth / 3) -100, (window.innerHeight / 4)+30,'Lives : 3',{fontSize:'18px',fill:'#000'}).setScrollFactor(0).setDepth(200)
          livesText.setColor('#ffffff')
          livesText.setScrollFactor(0)
        }

        if (musicOn === false){
          music.pause()
        } else{
          music.play()
        }

      }
      update() {
        if (player.lives === 0) {
          console.log(player.lives);
          this.scene.start('gameOver')
          music.pause();
        }
        if (enemies) {
          enemies.children.entries.forEach(function (enemy) {
            enemy.setVelocityX(enemy.speedX)
            if (enemy.speedX > 0) {
              enemy.anims.play('walk_enemy', true)
              enemy.flipX = false
            } else {
              enemy.flipX = true
            }
            // enemy.setVelocityX(-200)

          })
        }
        if (this.sys.game.device.os.desktop) {
          if (this.cursors.left.isDown) {
            player.anims.play('walk', true)
            player.setVelocityX(-160)
            player.flipX = true
          } else if (this.cursors.right.isDown) {
            player.anims.play('walk', true)
            player.setVelocityX(160)
            player.flipX = false
          } else {
            player.anims.play('idle', true)
            player.setVelocityX(0)
          }

          if (this.cursors.up.isDown && player.body.onFloor()) {
            player.setFrame(14)
            player.setVelocityY(-290)
            soundJump.play()
          }
          if (player.body.velocity.y < 0) {
            player.setFrame(14)
          }
          if (player.body.velocity.y > 0) {
            player.setFrame(17)
            player.body.setMaxVelocity(400, 550);
          }
        }else {
          if (this.cursorKeysVirtual.left.isDown) {
            player.anims.play('walk', true)
            player.setVelocityX(-160)
            player.flipX = true
          } else if (this.cursorKeysVirtual.right.isDown) {
            player.setVelocityX(160)
            player.anims.play('walk', true)
            player.flipX = false
          } else {
            player.anims.play('idle',true)
            player.setVelocityX(0)

          }
          if (this.cursorKeysVirtual.up.isDown && player.body.onFloor()) {
            player.setFrame(14)
            player.setVelocityY(-290) // 340
            soundJump.play();
          }
        }
      }
    }
    class inGame2 extends Phaser.Scene{
      constructor (){
        super({key:'inGame2'})
      }
      preload(){
        console.log('PRELOAD GAME 2');
      }
      create(){
        camera = this.cameras.main;
        console.log('CREATED GAME 2');


        //musica
        music = this.sound.add('sound', { loop: true })
        //Souns
        soundEat = this.sound.add('eat', {loop: false})
        soundJump = this.sound.add('jump',{loop:false})
        soundDamage = this.sound.add('damage',{loop:false})
        soundLose = this.sound.add('lose',{loop:false})
        // sound.play()

        let map = this.make.tilemap({ key: "map2" })
        let tileset = map.addTilesetImage("tileset", "tileset")
        let backgroundP =map.createStaticLayer("backgroundP", tileset, 0, 0)
        let end = map.createStaticLayer("end",tileset,0,0)
        let collision = map.createStaticLayer("collisions",tileset,0,0)
        let backgroundS =map.createStaticLayer("backgroundS", tileset, 0, 0)
        let details = map.createStaticLayer("details", tileset, 0, 0)
        let walls = map.createStaticLayer("walls",tileset,0,0)
        CoinLayer = map.getObjectLayer('coins')['objects']
        EnemyLayer = map.getObjectLayer('enemies')['objects']
        DemonLayer = map.getObjectLayer('demons')['objects']

        coins = this.physics.add.staticGroup()
        enemies = this.physics.add.group()
        demons = this.physics.add.group()

        backgroundP.setCollisionByExclusion([-1]);
        end.setCollisionByExclusion([-1]);
        collision.setCollisionByExclusion([-1]);
        backgroundS.setCollisionByExclusion([-1]);
        details.setCollisionByExclusion([-1]);
        walls.setCollisionByExclusion([-1]);


        // La càmara no sortirà del món
        camera.setBounds(0, 0, map.widthInPixels, map.heightInPixels);

        //Aparicion del personaje
        let spawnpoint = map.findObject('player', obj => obj.name === 'spawnpoint');
        player = this.physics.add.sprite(spawnpoint.x, spawnpoint.y, 'player');
        player.body.setSize(24, 64, 40, 0)
        player.lives = 3;


        // Habilitar flechas
        this.cursors = this.input.keyboard.createCursorKeys();

        var particlescollapse1 = this.add.particles('collapse')
        particlescollapse1.createEmitter({
          "active":true,
          "visible":true,
          "collideBottom":true,
          "collideLeft":true,
          "collideRight":true,
          "collideTop":true,
          "on":true,
          "particleBringToTop":true,
          "radial":true,
          "frame":{
            "frames":["dirt_01"],
            "cycle":false,"quantity":1
          },
          "frequency":1,
          "gravityX":0,
          "gravityY":200,
          "maxParticles":0,
          "timeScale":1,
          "blendMode":1,
          "accelerationX":0,
          "accelerationY":0,
          "alpha":1,
          "angle":{"min":0,"max":360,"ease":"Linear"},
          "bounce":0,
          "delay":0,
          "lifespan":1000,
          "maxVelocityX":10000,
          "maxVelocityY":10000,
          "moveToX":0,
          "moveToY":0,
          "quantity":1,
          "rotate":0,
          "scale":1,
          "speed":0,
          "x":50,
          "y":1000,
          "tint":[3091757],
          "emitZone":{"source":new Phaser.Geom.Rectangle(0,0,1000,50),"type":"random"},
          "deathZone":{"source":new Phaser.Geom.Rectangle(0,300,1000,50),"type":"onEnter"}
        });
        var particlescollapse2 = this.add.particles('collapse')
        particlescollapse2.createEmitter({
          "active":true,
          "visible":true,
          "collideBottom":true,
          "collideLeft":true,
          "collideRight":true,
          "collideTop":true,
          "on":true,
          "particleBringToTop":true,
          "radial":true,
          "frame":{
            "frames":["dirt_01"],
            "cycle":false,"quantity":1
          },
          "frequency":1,
          "gravityX":0,
          "gravityY":300,
          "maxParticles":0,
          "timeScale":1,
          "blendMode":1,
          "accelerationX":0,
          "accelerationY":0,
          "alpha":1,
          "angle":{"min":0,"max":360,"ease":"Linear"},
          "bounce":0,
          "delay":0,
          "lifespan":1000,
          "maxVelocityX":10000,
          "maxVelocityY":10000,
          "moveToX":0,
          "moveToY":0,
          "quantity":1,
          "rotate":0,
          "scale":1,
          "speed":0,
          "x":350,
          "y":300,
          "tint":[3091757],
          "emitZone":{"source":new Phaser.Geom.Rectangle(0,0,1500,50),"type":"random"},
          "deathZone":{"source":new Phaser.Geom.Rectangle(0,1100,1500,50),"type":"onEnter"}
        });
        var particlesEnd = this.add.particles('collapse')
        particlesEnd.createEmitter({"active":true,
          "visible":true,
          "collideBottom":true,
          "collideLeft":true,
          "collideRight":true,
          "collideTop":true,
          "on":true,
          "particleBringToTop":true,
          "radial":true,
          "frame":{"frames":["circle_05"],"cycle":false,"quantity":1},
          "frequency":0,
          "gravityX":0,
          "gravityY":0,
          "maxParticles":0,
          "timeScale":0.5,
          "blendMode":1,
          "accelerationX":0,
          "accelerationY":0,
          "alpha":{"ease":"Linear","min":1,"max":5},
          "angle":{"min":0,"max":360,"ease":"Expo.easeIn"},
          "bounce":{"ease":"Linear","min":1,"max":2},
          "delay":{"ease":"Linear","min":1,"max":2000},
          "lifespan":{"ease":"Stepped","min":1,"max":1.5},
          "maxVelocityX":10000,
          "maxVelocityY":10000,
          "moveToX":0,
          "moveToY":0,
          "quantity":{"ease":"Linear","min":0,"max":1.5},
          "rotate":0,
          "scale":{"ease":"Linear","min":0,"max":1.5},
          "speed":{"ease":"Stepped","min":1,"max":2},
          "x":2600,
          "y":150,
          "bounds":{"x":0,"y":0,"width":0,"height":0},
          "tint":[11911517,4960712,4537989,16777215,0,6474180,6441763,10372243],
          "emitZone":{"source":new Phaser.Geom.Circle(0,0,70),"type":"random"}})

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
        this.anims.create({
          key:'idleDemon',
          frames:this.anims.generateFrameNames('demon',{
            frames:[0,1,2,3,4,5]
          }),
          frameRate:1,
          repeat: -1
        })
        createCoins()
        createEnemies()
        createDemons()

        this.physics.add.collider(player,walls)
        this.physics.add.collider(player,enemies,lesslive)
        this.physics.add.collider(player,demons,lesslive)
        this.physics.add.overlap(player,coins,takeCoin,null,this)
        this.physics.add.collider(player,end,winGame,null,this)
        this.physics.add.collider(enemies,walls)
        this.physics.add.collider(enemies,collision,changeDirection)
        this.physics.add.collider(demons,player)
        this.physics.add.collider(demons,collision,changeDirection)

        if (this.sys.game.device.os.desktop) {
          camera.setZoom(2)
        }
        camera.startFollow(player)

        if (!this.sys.game.device.os.desktop) {
          this.joyStick = this.plugins.get('rexvirtualjoystickplugin').add(this, {
            x: this.cameras.main.centerX - 250,
            y: this.cameras.main.centerY + 100,
            radius: 40,
            base: this.add.graphics().fillStyle(0x150000).fillCircle(0, 0, 50),
            thumb: this.add.graphics().fillStyle(0xB45F06).fillCircle(0, 0, 30)
          })
          this.cursorKeysVirtual = this.joyStick.createCursorKeys()
        }

        if (!this.sys.game.device.os.desktop) {
          //movil
          scoreText=this.add.text(100,10,'Score: 0',{fontSize:'18px',fill:'#000'}).setScrollFactor(0).setDepth(200)
          scoreText.setColor('#ffffff')
          scoreText.setScrollFactor(0)
          livesText=this.add.text(100,40,'Lives : 3',{fontSize:'18px',fill:'#000'}).setScrollFactor(0).setDepth(200)
          livesText.setColor('#ffffff')
          livesText.setScrollFactor(0)
        }else{
          scoreText=this.add.text((window.innerWidth / 3) - 100, (window.innerHeight / 4),'Score: 0',{fontSize:'18px',fill:'#000'}).setScrollFactor(0).setDepth(200)
          scoreText.setColor('#ffffff')
          scoreText.setScrollFactor(0)
          livesText=this.add.text((window.innerWidth / 3) -100, (window.innerHeight / 4)+30,'Lives : 3',{fontSize:'18px',fill:'#000'}).setScrollFactor(0).setDepth(200)
          livesText.setColor('#ffffff')
          livesText.setScrollFactor(0)
        }

        if (musicOn === false){
          music.pause()
        } else{
          music.play()
        }

      }
      update(){
        if (player.lives ===0) {
          console.log(player.lives);
          this.scene.start('gameOver')
          music.pause();
        }
        if (enemies){
          enemies.children.entries.forEach(function (enemy) {
            enemy.setVelocityX(enemy.speedX)
            if (enemy.speedX>0){
              enemy.anims.play('walk_enemy', true)
              enemy.flipX = false
            }else{
              enemy.flipX = true
            }
          })
        }
        if (demons){
          demons.children.entries.forEach(function (demon) {
            demon.setVelocityX(demon.speedX)
            if (demon.speedX<0){
              demon.anims.play('idleDemon', true)
              demon.flipX = false
            }else{
              demon.flipX = true
            }
          })
        }
        if (this.sys.game.device.os.desktop) {
          if (this.cursors.left.isDown) {
            player.anims.play('walk', true)
            player.setVelocityX(-160)
            player.flipX = true
          } else if (this.cursors.right.isDown) {
            player.anims.play('walk', true)
            player.setVelocityX(160)
            player.flipX = false
          } else {
            player.anims.play('idle', true)
            player.setVelocityX(0)
          }

          if (this.cursors.up.isDown && player.body.onFloor()) {
            soundJump.play()
            player.setFrame(14)
            player.setVelocityY(-295)
          }
          if (player.body.velocity.y < 0) {
            player.setFrame(14)
          }
          if (player.body.velocity.y > 0) {
            player.setFrame(17)
            player.body.setMaxVelocity(400, 550);
          }
        }else {
          if (this.cursorKeysVirtual.left.isDown) {
            player.anims.play('walk', true)
            player.setVelocityX(-160)
            player.flipX = true
          } else if (this.cursorKeysVirtual.right.isDown) {
            player.anims.play('walk', true)
            player.setVelocityX(160)
            player.flipX = false
          } else {
            player.anims.play('idle',true)
            player.setVelocityX(0)

          }
          if (this.cursorKeysVirtual.up.isDown && player.body.onFloor()) {
            soundJump.play();
            player.setFrame(14)
            player.setVelocityY(-295)
          }
          if (player.body.velocity.y < 0) {
            player.setFrame(14)
          }
          if (player.body.velocity.y > 0) {
            player.setFrame(17)
            player.body.setMaxVelocity(400, 550);
          }
        }
      }
    }
  </script>

