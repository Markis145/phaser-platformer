<template>
    <div id="game">

    </div>
</template>

<script>
import Phaser from 'phaser'
import wall from '../assets/wall.png'
import ground from '../assets/ground.png'
import player from '../assets/player.png'
import coin from '../assets/coin.png'
import enemy from '../assets/enemy.png'
import soundDead from '../assets/dead.mp3'
import soundCoin from '../assets/coin.mp3'

let score = 0
let scoreText

//  Todo -> detectar quan l'usuari cau, ha de morir

function takeCoin (player, coin) {
  //  TODO -> millorar en una animacio
  coin.disableBody(true, true)

  score = score + 10

  // this.add.text(20, 20, 'score: ' + score, { fontSize: '12px', fill: '#000' })
  scoreText.setText('Score: ' + score)
  // TODO -> executar so que pertoca (coin.mp3)

  // TODO -> actualitzar comptador de punts
  this.sound.play('soundCoin')
}

function die (player, enemy) {
//  TODO -> Millorar en una animació
//  TODO -> Afegir audio
//  TODO -> Shake
  player.scene.cameras.main.shake(500)
  //  TODO -> Reiniciar nivell
  //  TODO -> Numero de vides i si les hem gastat aturat el joc

  player.disableBody(true, true)
  this.sound.play('soundDead')
}

export default {
  name: 'Game',
  mounted () {
    // Phaser 3.0 -> phaser
    let config = {
      type: Phaser.AUTO,
      width: 400,
      height: 300,
      physics: {
        default: 'arcade',
        arcade: {
          gravity: { y: 200 }
        }
      },
      // No hi ha states a phaser 3.0 -> scenes (son com les pantalles d'un joc)
      scene: {
        preload () {
          console.log('preload')
          // carregar assets pero encara no s'utilitzen-> imatges, jugadors, audios, videos
          this.load.image('wall', wall)
          this.load.image('ground', ground)
          this.load.image('coin', coin)
          this.load.image('enemy', enemy)
          this.load.spritesheet('player', player, { frameWidth: 28, frameHeight: 22 })

          // audio
          this.load.audio('soundDead', soundDead)
          this.load.audio('soundCoin', soundCoin)
        },
        create () {
          this.sound.add('soundDead')
          this.sound.add('soundCoin')
          console.log('created')
          //  Initialize del nivell -> Afegirem tiles (parets, terra, afegir jugadors, colectibles, enemics)
          this.cameras.main.backgroundColor.setTo(52, 152, 219)
          // group per defecte es dinamic
          this.level = this.physics.add.staticGroup()

          // this.ground = this.physics.add.image(500 / 2, 200 / 2 + 30, 'ground')
          // this.add.image(500 / 2 - 160, 200 / 2, 'wall')
          // this.add.image(500 / 2 + 160, 200 / 2, 'wall')

          this.level.create(500 / 2 - 160, 200 / 2, 'wall')
          this.level.create(500 / 2 + 160, 200 / 2, 'wall')
          this.level.create(500 / 2, 200 / 2 + 30, 'ground')

          // this.ground.body.allowGravity = false
          //  player -> fisiques
          this.player = this.physics.add.sprite(500 / 2, 200 / 2 - 50, 'player')
          this.player.setBounce(0.2)
          // this.player.setCollideWorldBounds(true)

          this.physics.add.collider(this.player, this.level)

          //  define sounds
          // this.dustSound = this.add.audio('dust', 0.1)

          // cursors
          this.cursors = this.input.keyboard.createCursorKeys()

          this.anims.create({
            key: 'idle',
            frames: this.anims.generateFrameNumbers('player', { start: 3, end: 5 }),
            frameRate: 5,
            repeat: -1
          })

          this.coins = this.physics.add.group()
          this.coins.create(140, 200 / 2, 'coin')
          this.coins.create(170, 200 / 2, 'coin')
          this.coins.create(200, 200 / 2, 'coin')

          this.physics.add.collider(this.coins, this.level)
          this.physics.add.overlap(this.player, this.coins, takeCoin, null, this)

          // score
          scoreText = this.add.text(20, 20, 'Score: 0', { fontSize: '12px', fill: '#000' })

          //  Enemies

          this.enemies = this.physics.add.group()
          this.enemies.create(500 / 2 + 120, 200 / 2, 'enemy')
          this.physics.add.collider(this.enemies, this.level)

          this.physics.add.overlap(this.player, this.enemies, die, null, this)

          //  prepare loser txt
          this.loserText = this.add.text(500 / 2, 200 / 2 - 50, 'Looser!', { fontSize: '24px', fill: '#000' }).setVisible(false)

          this.cameras.main.on('camerashakestart', () => {
            this.loserText.setVisible(true)
          })

          this.cameras.main.on('camerashakecomplete', () => {
            this.loserText.setVisible(false)
          })
        },
        update () {
          this.player.anims.play('idle', true)
          // INPUT EVENTS
          if (this.cursors.left.isDown) {
            this.player.setVelocityX(-160)
            this.player.setFrame(2)
          } else if (this.cursors.right.isDown) {
            this.player.setVelocityX(160)
            this.player.setFrame(1)
          } else {
            this.player.setVelocityX(0)
          }
          // JUMP
          if (this.cursors.up.isDown && this.player.body.touching.down) {
            this.player.setVelocityY(-200)
          }

          // console.log('updated')

          // S'executa continuament al game loop (60 vegades per segon)

          // if (this.player.body.touching.down && this.player.y > 10) {
          //   this.dustSound.play()
          // }
        }
      }
    }
    /* eslint-disable */
    new Phaser.Game(config)
  }
}
</script>
