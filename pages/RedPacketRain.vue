<template>
  <div>
    <canvas id="rain" @click="judgePack($event)"></canvas>
    <img
      src="https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1578976684864&di=7e439e579a0e8bab15605e2d5cc9f978&imgtype=0&src=http%3A%2F%2Fimg.mp.sohu.com%2Fupload%2F20170715%2F49a52c12cdd542e185a11ae9765dc253_th.png"
      id="hotbag"
      hidden="true"
    />
    <button class="button--green" @click="init">开始</button>
  </div>
</template>

<script>
export default {
  name: "",
  data() {
    return {
      rainArray: [],
      W: 100,
      H: 100,
      hotbagW: 30, // 1/2红包的宽度
      hotbagH: 45, // 1/2红包的高度
      timeId: null,
      createTimeId: null
    };
  },
  mounted() {
    let can = document.querySelector("#rain");
    this.W = window.innerWidth; // 窗口大小
    this.H = window.innerHeight;
    can.width = window.innerWidth;
    can.height = window.innerHeight;
    window.onresize = function() {
      can.width = window.innerWidth;
      can.height = window.innerHeight;
    }
    window.canContent = can.getContext("2d");
  },
  methods: {
    init() {
      this.rainArray = [];
      clearTimeout(this.timeId);
      clearTimeout(this.createTimeId);
      for (let i = 0; i < 20; i++) {
        this.createTimeId = setTimeout(this.createRain, 100 * i, i);
      }
      this.run();
    },
    // 创建雨滴
    createRain(i) {
      let rain = new Rain(this.W, this.H, this.hotbagW, this.hotbagH);
      rain.init(i);
      rain.draw();
      this.rainArray.push(rain);
    },
    // 移动
    moveRain() {
      canContent.fillStyle = "rgba(0,0,0,0)";
      canContent.clearRect(0, 0, this.W, this.H);
      for (let k = 0; k < this.rainArray.length; k++) {
        this.rainArray[k].move();
      }
    },
    // 开始移动
    run() {
      this.moveRain();
      this.timeId = setTimeout(this.run, 1000 / 60);
    },
    // 判断点击
    judgePack(event) {
      let mouseX = event.clientX;
      let mouseY = event.clientY;
      this.rainArray.forEach((item, index) => {
        if (
          item.alive &&
          item.x <= mouseX &&
          mouseX - item.x < hotbagW * 2 &&
          item.y - hotbagH <= mouseY &&
          mouseY - item.y < hotbagH * 2
        ) {
          item.destory();
        }
      });
    }
  }
};

function random(min, max) {
  return Math.random() * (max - min) + min;
}
function Rain(windowW, windowH, hotbagW, hotbagH) {
  (this.init = function(i) {
    this.x = (i % 3) * windowW * 0.1 + random(0.3 * windowW, 0.5 * windowW);
    if (canContent.isPointInPath(this.x, this.y)) {
      console.log(i);
    }
    this.y = -(i * 2 * hotbagH) - 100;
    this.v = 2; //下落加速度
    this.h = 60;
    this.maxH = random(1.2 * windowH, 1.3 * windowH);
    this.opacity = 1; //透明度
    this.vopacity = 0.9;
    this.src = document.querySelector("#hotbag");
    this.alive = true;
    this.rotate = 0; //摇摆
    this.vrotate = 0.01; //摇摆
    this.scale = 0.1;
    this.vscale = 0.1;
    this.vnum = 1;
  }),
    (this.draw = function() {
      canContent.save();
      if (this.y < this.maxH && this.alive) {
        canContent.fillStyle = "#ff0000";
        canContent.translate(this.x + hotbagW, this.y + hotbagH);
        this.rotate += this.vrotate;
        canContent.rotate(this.rotate);
        if (this.rotate >= 0.15) {
          this.vrotate = -0.01;
        } else if (this.rotate <= -0.15) {
          this.vrotate = 0.01;
        }
        canContent.translate(-this.x - hotbagW, -this.y - hotbagH);
        canContent.beginPath();
        canContent.drawImage(
          this.src,
          this.x,
          this.y,
          2 * hotbagW,
          2 * hotbagH
        );
        canContent.stroke();
        canContent.closePath(); // 关闭路径
        canContent.restore();
      } else {
        if (this.opacity > 0.02) {
          canContent.beginPath();
          canContent.globalAlpha = this.opacity;
          canContent.drawImage(
            this.src,
            this.x - this.scale * hotbagW + hotbagW,
            this.y,
            hotbagW * 2 * this.scale,
            hotbagH * 2 * this.scale
          );
          if (this.vnum > 1) {
            canContent.fillStyle = "rgb(247,223,0)";
            canContent.font = "20px Arial";
            canContent.fillText("+1", this.x + 18, this.y + 20 - this.vnum);
          }
          canContent.stroke();
          canContent.closePath();
          canContent.restore();
        }
      }
    }),
    (this.destory = function() {
      this.alive = false;
    }),
    (this.move = function() {
      if (this.y < this.maxH && this.alive) {
        this.y += this.v;
      } else {
        if (this.opacity > 0.02) {
          if (this.scale < 0.9 && this.vnum == 1) {
            this.scale += this.vscale;
          } else {
            if (this.vnum > 20) {
              this.opacity *= this.vopacity;
              if (this.scale >= 0.2) {
                this.scale *= 0.96;
              }
            }
            this.vnum++;
          }
        } else {
        }
      }
      this.draw();
    });
}
</script>

<style scoped>
div {
  margin: 0;
  background-color: #000000;
  /*background: url('https://img.dpm.org.cn/Uploads/Picture/2020/01/07/s5e13e102dcfbb.jpg') no-repeat;*/
}

#rain {
  display: block;
  margin: 0 auto;
}
button {
  position: absolute;
  right: 50px;
  bottom: 50px;
  cursor: pointer;
}
</style>
