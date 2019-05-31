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
import explosion from '../assets/explosion.png'

let score = 0
let scoreText
let lives = 3
let livesText


function takeCoin (player, coin) {
  //  TODO -> millorar en una animacio

  this.tweens.add({
    targets: coin,
    y: 50,
    scaleX: 0,
    ease: 'Linear',
    duration: 160,
    yoyo: false,
    repeat: 0,
    onComplete: () => { coin.disableBody(true, true) }
  })

  coin.disableBody(true)

  score = score + 10
  scoreText.setText('Score: ' + score)
  this.sound.play('soundCoin')
}

function die (player, enemy) {
  this.explosion = this.physics.add.sprite(
    player.body.x + player.body.height / 2,
    player.body.y + player.body.width / 2,
    'explosion'
  )
  this.explosion.anims.play('explosion', false)
  player.scene.cameras.main.shake(2000)
  player.disableBody(true, true)
  setTimeout(() => {
    this.sound.play('soundDead')
    this.scene.restart()
    lives = lives - 1
    if (lives == 0) {
      this.scene.stop()
    }
  }, 200)
}

function onWorldBoundsCollide () {
  this.explosion = this.physics.add.sprite(
    this.player.body.x + this.player.body.height / 2,
    this.player.body.y + this.player.body.width / 2,
    'explosion'
  )
  this.explosion.anims.play('explosion', false)
  this.player.disableBody(true, true)
  setTimeout(() => {
    this.sound.play('soundDead')
    this.scene.restart()
    lives = lives - 1
    if (lives == 0) {
      this.scene.stop()
    }
  }, 200)
}

export default {
  name: 'Game',
  mounted () {
    // Phaser 3.0 -> phaser
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
      scene: {
        preload () {
          console.log('preload')
          this.load.image('wall', wall)
          this.load.image('ground', ground)
          this.load.image('coin', coin)
          this.load.image('enemy', enemy)
          this.load.spritesheet('explosion', explosion, {
            frameWidth: 262,
            frameHeight: 262
          })
          this.load.spritesheet('player', player, { frameWidth: 28, frameHeight: 22 })

          // audio
          this.load.audio('soundDead', soundDead)
          this.load.audio('soundCoin', soundCoin)
        },
        create () {
          this.sound.add('soundDead')
          this.sound.add('soundCoin')
          console.log('created')
          this.cameras.main.backgroundColor.setTo(52, 152, 219)
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
          this.player.setCollideWorldBounds(true)
          this.player.body.onWorldBounds = true

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

          this.anims.create({
            key: 'explosion',
            frames: this.anims.generateFrameNumbers('explosion', {
              start: 0,
              end: 6
            }),
            frameRate: 60,
            repeat: -1
          })

          this.coins = this.physics.add.group()
          this.coins.create(140, 200 / 2, 'coin')
          this.coins.create(170, 200 / 2, 'coin')
          this.coins.create(200, 200 / 2, 'coin')

          this.physics.add.collider(this.coins, this.level)
          this.physics.add.overlap(this.player, this.coins, takeCoin, null, this)

          // score
          scoreText = this.add.text(20, 20, 'Score: ' + score, { fontSize: '12px', fill: '#000' })
          livesText = this.add.text(20, 40, 'Lives: ' + lives, { fontSize: '12px', fill: '#000' })

          //  Enemies

          this.enemies = this.physics.add.group()
          this.enemies.create(500 / 2 + 120, 200 / 2, 'enemy')
          this.physics.add.collider(this.enemies, this.level)

          this.physics.add.overlap(this.player, this.enemies, die, null, this)
          this.physics.world.on('worldbounds', onWorldBoundsCollide, this)

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
            this.player.setVelocityY(-170)
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
