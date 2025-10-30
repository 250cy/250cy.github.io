# 250cy.github.io
<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8" />
    <meta
      name="viewport"
      content="width=device-width, initial-scale=1.0, user-scalable=no"
    />
    <title>新婚快乐</title>
    <style>
      * {
        margin: 0;
        padding: 0;
        box-sizing: border-box;
      }
      body {
        min-height: 100vh;
        background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%);
        overflow: hidden;
        position: relative;
      }

      /* 烟花Canvas容器（全屏覆盖，在最底层） */
      #fireworks {
        position: absolute;
        top: 0;
        left: 0;
        z-index: 1; /* 烟花在最底层 */
      }

      /* 花瓣容器（在烟花之上，文字之下） */
      #petals-container {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        z-index: 5;
      }

      /* 中心祝福文字 */
      .blessing {
        position: relative;
        z-index: 10; /* 文字在最上层 */
        text-align: center;
        padding-top: 30vh;
        color: #fff;
        font-family: "SimSun", serif;
        text-shadow: 0 0 10px rgba(255, 255, 255, 0.5);
      }
      .blessing h1 {
        font-size: 2.5rem;
        margin-bottom: 1rem;
        animation: fadeIn 2s ease-in-out;
      }
      .blessing p {
        font-size: 1.2rem;
        line-height: 1.8;
        animation: fadeIn 3s ease-in-out;
      }

      /* 跳动爱心 */
      .heart {
        position: relative;
        width: 80px;
        height: 80px;
        margin: 2rem auto;
        background-color: #ff416c;
        transform: rotate(45deg);
        animation: pulse 1.5s infinite;
        z-index: 10;
        box-shadow: 0 0 20px #ff416c;
      }
      .heart::before,
      .heart::after {
        content: "";
        position: absolute;
        width: 80px;
        height: 80px;
        background-color: inherit;
        border-radius: 50%;
        box-shadow: 0 0 20px #ff416c;
      }
      .heart::before {
        top: -40px;
        left: 0;
      }
      .heart::after {
        top: 0;
        left: -40px;
      }

      /* 花瓣样式 */
      .petal {
        position: absolute;
        background-color: #ffb6c1;
        border-radius: 80% 0 80% 0;
        opacity: 0.8;
        animation: fall linear infinite;
      }
      .petal.small {
        width: 15px;
        height: 15px;
      }
      .petal.medium {
        width: 25px;
        height: 25px;
      }
      .petal.large {
        width: 35px;
        height: 35px;
      }

      /* 动画定义 */
      @keyframes fall {
        0% {
          transform: translateY(-10px) rotate(0deg);
          opacity: 0.8;
        }
        100% {
          transform: translateY(100vh) rotate(360deg);
          opacity: 0.4;
        }
      }
      @keyframes pulse {
        0%,
        100% {
          transform: rotate(45deg) scale(1);
        }
        50% {
          transform: rotate(45deg) scale(1.2);
        }
      }
      @keyframes fadeIn {
        from {
          opacity: 0;
          transform: translateY(20px);
        }
        to {
          opacity: 1;
          transform: translateY(0);
        }
      }
    </style>
  </head>
  <body>
    <!-- 烟花Canvas -->
    <canvas id="fireworks"></canvas>

    <!-- 花瓣容器 -->
    <div id="petals-container"></div>

    <!-- 祝福内容 -->
    <div class="blessing">
      <h1>林小宇 邓贤梅</h1>
      <p>佳偶天成，百年琴瑟，<br />祝新婚快乐，爱意绵长</p>
      <div class="heart"></div>
    </div>

    <script>
      // 1. 花瓣动画逻辑
      function createPetals() {
        const container = document.getElementById("petals-container");
        const petalCount = window.innerWidth < 768 ? 30 : 50;

        for (let i = 0; i < petalCount; i++) {
          const petal = document.createElement("div");
          petal.classList.add("petal");
          const sizes = ["small", "medium", "large"];
          petal.classList.add(sizes[Math.floor(Math.random() * sizes.length)]);
          petal.style.left = `${Math.random() * 100}vw`;
          petal.style.top = `-${Math.random() * 20}px`;
          petal.style.transform = `rotate(${Math.random() * 360}deg)`;
          petal.style.animationDuration = `${5 + Math.random() * 10}s`;
          petal.style.animationDelay = `${Math.random() * 10}s`;
          const colors = [
            "#FFB6C1",
            "#FFC0CB",
            "#FFD1DC",
            "#FF9AA2",
            "#FF69B4",
          ];
          petal.style.backgroundColor =
            colors[Math.floor(Math.random() * colors.length)];
          container.appendChild(petal);
        }
      }

      // 2. 烟花动画逻辑
      class Firework {
        constructor() {
          // 烟花发射起点（屏幕底部随机位置）
          this.x = Math.random() * canvas.width;
          this.y = canvas.height;
          // 发射目标点（屏幕上方随机高度）
          this.targetY = Math.random() * canvas.height * 0.4 + 50;
          this.targetX = this.x + (Math.random() - 0.5) * canvas.width * 0.3;
          // 发射速度和颜色
          this.speed = 5 + Math.random() * 3;
          this.color = `hsl(${Math.random() * 360}, 100%, 60%)`; // 随机色相（五彩缤纷）
          this.radius = 2;
          this.reachedTarget = false;
          this.particles = []; // 爆炸后的粒子
        }

        // 更新烟花位置（发射阶段）
        update() {
          if (!this.reachedTarget) {
            // 计算到目标点的距离
            const dx = this.targetX - this.x;
            const dy = this.targetY - this.y;
            const distance = Math.sqrt(dx * dx + dy * dy);

            if (distance < this.speed) {
              this.reachedTarget = true;
              this.explode(); // 到达目标后爆炸
            } else {
              // 向目标点移动
              this.x += (dx / distance) * this.speed;
              this.y += (dy / distance) * this.speed;
            }
          } else {
            // 爆炸后更新粒子状态
            this.particles.forEach((particle) => {
              particle.update();
            });
            // 过滤已消失的粒子
            this.particles = this.particles.filter(
              (particle) => particle.alpha > 0
            );
          }
        }

        // 绘制烟花（发射阶段或爆炸阶段）
        draw() {
          if (!this.reachedTarget) {
            // 绘制发射轨迹
            ctx.beginPath();
            ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
            ctx.fillStyle = this.color;
            ctx.fill();
          } else {
            // 绘制爆炸粒子
            this.particles.forEach((particle) => {
              particle.draw();
            });
          }
        }

        // 爆炸生成粒子
        explode() {
          const particleCount = 80 + Math.floor(Math.random() * 40); // 80-120个粒子
          for (let i = 0; i < particleCount; i++) {
            this.particles.push(
              new Particle(
                this.x,
                this.y,
                this.color // 粒子继承烟花颜色
              )
            );
          }
        }
      }

      // 烟花爆炸后的粒子类
      class Particle {
        constructor(x, y, color) {
          this.x = x;
          this.y = y;
          this.color = color;
          // 随机速度和方向
          const angle = Math.random() * Math.PI * 2;
          const speed = 1 + Math.random() * 3;
          this.vx = Math.cos(angle) * speed;
          this.vy = Math.sin(angle) * speed;
          this.radius = 1 + Math.random() * 2;
          this.alpha = 1; // 透明度
          this.gravity = 0.03; // 重力效果（粒子下落）
        }

        update() {
          this.vy += this.gravity; // 受重力影响加速下落
          this.x += this.vx;
          this.y += this.vy;
          this.alpha -= 0.01; // 逐渐消失
          this.radius *= 0.98; // 逐渐缩小
        }

        draw() {
          ctx.beginPath();
          ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
          ctx.fillStyle = `${this.color}${Math.floor(this.alpha * 255)
            .toString(16)
            .padStart(2, "0")}`;
          ctx.fill();
        }
      }

      // 初始化烟花Canvas
      const canvas = document.getElementById("fireworks");
      const ctx = canvas.getContext("2d");

      // 适配屏幕大小
      function resizeCanvas() {
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;
      }
      resizeCanvas();
      window.addEventListener("resize", resizeCanvas);

      // 管理烟花实例
      const fireworks = [];
      // 每隔1-3秒发射一个烟花
      setInterval(() => {
        fireworks.push(new Firework());
      }, 1000 + Math.random() * 2000);

      // 烟花动画循环
      function animateFireworks() {
        ctx.fillStyle = "rgba(0, 0, 0, 0.05)"; // 半透明黑色覆盖，产生轨迹淡出效果
        ctx.fillRect(0, 0, canvas.width, canvas.height);

        fireworks.forEach((firework, index) => {
          firework.update();
          firework.draw();
          // 移除完全消失的烟花
          if (firework.reachedTarget && firework.particles.length === 0) {
            fireworks.splice(index, 1);
          }
        });

        requestAnimationFrame(animateFireworks);
      }

      // 启动所有动画
      window.onload = () => {
        createPetals();
        animateFireworks();
      };
    </script>
  </body>
</html>
