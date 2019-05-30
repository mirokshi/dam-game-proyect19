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
    import _PARTICLES_ARROW from '../assets/particles/arrow.png'
    import _PARTICLES_JSON from '../assets/particles/game/shapes.json'
    import _PARTICLES_COLLAPSE from '../assets/particles/game/shapes.png'
    import _BUTTONS from '../assets/objects/button.png'
    import _START from '../assets/img/game.png'
    import _END from '../assets/img/gameOver.png'
    //sounds
    import _GAME_MUSIC from '../assets/sounds/wapons.mp3'
    import _SOUND_EAT from '../assets/sounds/eat.mp3'


    let camera, player;
    let coins,CoinLayer,coinScore = 0;
    let EnemyLayer, enemies;
    let DemonLayer, demons;
    let scoreText;
    let emitter;
    let soundEat
    let lives = 3,livesText;
    let button, bgStart, bgEnd;

    function actionOnClick() {
      this.scene.start('inGame')
    }

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
        obj.speedX = 70

      })
    }
    function createDemons () {
      DemonLayer.forEach(object => {
        let obj = demons.create(object.x, object.y, 'demon',1)
        obj.setScale(1.5)
        // obj.setScale(object.width / 64, object.height / 64)
        // obj.setOrigin(0)
        obj.body.width = object.width
        obj.body.height = object.height
        obj.speedX = -100

      })
    }

    function lesslive (player, enemy) {
      lives = lives - 1
      player.scene.cameras.main.shake(250)
      player.x =150
      player.y=150
      // player.disableBody(true, true)
      livesText.setText('Lives : '+lives)
      if (lives === 0){
        this.scene.start('gameOver')
      }
    }
    function winGame1(){
      this.scene.start('inGame2')
    }

    export default {
      name: 'Game',
      created() {
        var controls = false
        // PHASER 2.6 -> phaser-ce
        // PHASER 3.0 -> phaser
        let config = {
          type: Phaser.AUTO,
          width: window.innerWidth,
          height: window.innerHeight,
          physics: {
            default: 'arcade',
            arcade: {
              gravity: {y: 300},
              debug :true
            }
          },
          // NO HI HA STATES A 3.0 -> SCENES
          scene: [loader,gameStart, inGame, inGame2,gameOver]
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

        //btn
        this.load.image('bgStart',_START)
        this.load.image('bgEnd',_END)
        this.load.spritesheet('buttons',_BUTTONS,{frameWidth:16,frameHeight:16})
        //pa
        this.load.atlas('collapse',_PARTICLES_COLLAPSE,_PARTICLES_JSON)
        this.load.image('arrow',_PARTICLES_ARROW)
        //so
        this.load.audio('sound',_GAME_MUSIC)
        this.load.audio('eat',_SOUND_EAT)

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
          bgStart = this.add.tileSprite(window.x/2,window.y/2,1390,920,'bgStart').setOrigin(0)
                // button = this.add.button(300,400,'buttons',actionOnClick,this,2,1,0)
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
        bgEnd = this.add.tileSprite(window.x/2,window.y/2,718,479,'bgEnd').setOrigin(0)
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


        //musica
        let music = this.sound.add('sound', { loop: true })
        //Souns
        soundEat = this.sound.add('eat', {loop: false})
        // sound.play()

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

        camera.setZoom(2)
        camera.startFollow(player)

        scoreText=this.add.text(camera.centerX-350,camera.centerY-170,'Score: 0',{fontSize:'18px',fill:'#000'})
        scoreText.setColor('#ffffff')
        scoreText.setScrollFactor(0)

        livesText=this.add.text(camera.centerX-350,camera.centerY-150,'Lives : 3',{fontSize:'18px',fill:'#000'})
        livesText.setColor('#ffffff')
        livesText.setScrollFactor(0)
        // music.play()

      }
      update(){
        if (enemies){
          enemies.children.entries.forEach(function (enemy) {
            enemy.setVelocityX(enemy.speedX)
            if (enemy.speedX>0){
              enemy.anims.play('walk_enemy', true)
              enemy.flipX = false
            }else{
              enemy.flipX = true
            }
              // enemy.setVelocityX(-200)

          })
        }
        if (this.cursors.left.isDown) {
          player.anims.play('walk', true)
          player.setVelocityX(-200)
          player.flipX = true
        }else if (this.cursors.right.isDown) {
          player.anims.play('walk', true)
          player.setVelocityX(200)
          player.flipX = false
        } else {
          player.anims.play('idle',true)
          player.setVelocityX(0)
        }

        if (this.cursors.up.isDown && player.body.onFloor()) {
          player.setFrame(14)
          player.setVelocityY(-300)
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
    class inGame2 extends Phaser.Scene{
      constructor (){
        super({key:'inGame2'})
      }
      preload(){
        console.log('PRELOAD');
      }
      create(){
        camera = this.cameras.main;
        console.log('CREATED');


        //musica
        let sound = this.sound.add('sound', { loop: true })
        //Souns
        soundEat = this.sound.add('eat', {loop: false})
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
        // enemies.enableBody=true;

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
        this.physics.add.collider(player,end)
        this.physics.add.overlap(player,coins,takeCoin,null,this)
        this.physics.add.collider(enemies,walls)
        this.physics.add.collider(enemies,collision,changeDirection)
        this.physics.add.collider(demons,walls)
        this.physics.add.collider(demons,player)
        this.physics.add.collider(demons,collision,changeDirection)

        camera.setZoom(2)
        camera.startFollow(player)

        scoreText=this.add.text(camera.centerX-350,camera.centerY-170,'Score: 0',{fontSize:'18px',fill:'#000'})
        scoreText.setColor('#ffffff')
        scoreText.setScrollFactor(0)

        livesText=this.add.text(camera.centerX-350,camera.centerY-150,'Lives : 3',{fontSize:'18px',fill:'#000'})
        livesText.setColor('#ffffff')
        livesText.setScrollFactor(0)
        // music.play()

      }
      update(){
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
            if (demon.speedX>0){
              demon.anims.play('idleDemon', true)
              demon.flipX = false
            }else{
              demon.flipX = true
            }
          })
        }
        if (this.cursors.left.isDown) {
          player.anims.play('walk', true)
          player.setVelocityX(-200)
          player.flipX = true
        }else if (this.cursors.right.isDown) {
          player.anims.play('walk', true)
          player.setVelocityX(200)
          player.flipX = false
        } else {
          player.anims.play('idle',true)
          player.setVelocityX(0)
        }

        if (this.cursors.up.isDown && player.body.onFloor()) {
          player.setFrame(14)
          player.setVelocityY(-300)
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

