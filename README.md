# GCPS-Game
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Jumping Ball Runner - Sponsor's Journey</title>
  <style>
    body { margin: 0; overflow: hidden; background: #87ceeb; font-family: 'Comic Sans MS', cursive, sans-serif; }
    canvas { display: block; background: #87ceeb; }
    #ui { position: absolute; top: 10px; left: 10px; color: #333; font-size: 20px; }
    #retryBtn { display: none; position: absolute; top: 60%; left: 50%; transform: translate(-50%, -50%); padding: 15px 30px; font-size: 20px; border-radius: 12px; border: none; background: #ff6f61; color: #fff; cursor: pointer; }
    #retryBtn:hover { background: #ff3b2e; }
    #winMsg { display:none; position:absolute; top:40%; left:50%; transform:translate(-50%, -50%); font-size:32px; color:#000; background:#ffff99; padding:20px; border-radius:12px; text-align:center; }
    #gameOverMsg { display:none; position:absolute; top:30%; left:50%; transform:translate(-50%, -50%); font-size:32px; color:#fff; background:#cc0000; padding:20px; border-radius:12px; text-align:center; }
  </style>
</head>
<body>
  <div id="ui">Score: <span id="score">0</span> | High Score: <span id="highScore">0</span> | Level: <span id="level">1</span></div>
  <div id="winMsg">Congratulations! Sponsor reached GCPS!</div>
  <div id="gameOverMsg">Game Over!</div>
  <button id="retryBtn">Retry</button>
  <canvas id="gameCanvas"></canvas>
  <script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;

    let gameOver = false;
    let obstacles = [];
    let score = 0;
    let highScore = localStorage.getItem('highScore') || 0;
    let level = 1;
    document.getElementById('highScore').textContent = highScore;
    document.getElementById('level').textContent = level;
    let speed = 4;
    let obstacleFrequency = 140;

    const sponsor = {
      x: 100,
      y: canvas.height - 160,
      width: 140,
      height: 140,
      dy: 0,
      dx: 0,
      gravity: 0.5,
      grounded: true,
      draw() {
        ctx.fillStyle = '#FFA500'; ctx.fillRect(this.x+20, this.y-100, 100, 40);
        ctx.fillStyle = '#1E90FF'; ctx.fillRect(this.x+15, this.y-70, 110, 80);
        ctx.fillStyle = '#ffff00'; ctx.fillRect(this.x+15, this.y, 40, 16); ctx.fillRect(this.x+75, this.y, 40, 16);
        ctx.fillStyle = '#fff'; ctx.font='20px Comic Sans MS'; ctx.textAlign='center'; ctx.fillText('Sponsor', this.x+this.width/2, this.y-40);
      },
      update() {
        this.dy += this.gravity;
        this.y += this.dy;
        this.x += this.dx;
        if (this.y >= canvas.height - 100) { this.dy = 0; this.y = canvas.height - 100; this.grounded = true; }
        if (this.x < 0) this.x = 0;
        this.draw();
      },
      jump() {
        let jumpAdjustment;
        if(level <= 3) jumpAdjustment = -18;
        else if(level <= 7) jumpAdjustment = -16;
        else jumpAdjustment = -16;
        if (this.grounded) { this.dy = jumpAdjustment; this.grounded=false; playFunnySound(); }
      }
    };

    const goal = {
      x: canvas.width*0.85,
      y: canvas.height - 180,
      width: 180,
      height: 180,
      draw() {
        ctx.fillStyle = '#ffcc00'; ctx.beginPath(); ctx.arc(this.x+this.width/2,this.y-this.height/2,this.width/2,0,Math.PI*2); ctx.fill();
        ctx.fillStyle = '#000'; ctx.font = '32px Comic Sans MS'; ctx.textAlign='center'; ctx.textBaseline='middle'; ctx.fillText('GCPS', this.x+this.width/2, this.y- this.height/2);
      }
    };

    const obstaclesList = [
      {text:'Regulatory', color:'#ff4d4d'}, {text:'eTMF', color:'#ff9933'},
      {text:'Site', color:'#4d94ff'}, {text:'Monitoring', color:'#66ff66'},
      {text:'Data', color:'#ff66cc'}, {text:'Safety', color:'#ffd11a'},
      {text:'Recruitment', color:'#cc99ff'}, {text:'Logistics', color:'#66ffff'},
      {text:'Training', color:'#ff99cc'}, {text:'Quality', color:'#99ff66'}, {text:'Budget', color:'#ffcc66'}
    ];

    class Obstacle {
      constructor(config) { this.width=80; this.height=40; this.x=canvas.width; this.y=canvas.height-100-this.height; this.text=config.text; this.color=config.color; }
      draw() {
        ctx.fillStyle=this.color; ctx.fillRect(this.x,this.y,this.width,this.height);
        ctx.fillStyle='#000'; ctx.font='14px Comic Sans MS'; ctx.textAlign='center'; ctx.textBaseline='middle'; ctx.fillText(this.text,this.x+this.width/2,this.y+this.height/2);
      }
      update() { this.x -= speed; this.draw(); }
    }

    function playFunnySound() {
      const ctxAudio = new (window.AudioContext||window.webkitAudioContext)();
      const o=ctxAudio.createOscillator(), g=ctxAudio.createGain(); o.connect(g); g.connect(ctxAudio.destination);
      o.type='square'; o.frequency.setValueAtTime(440+Math.random()*200, ctxAudio.currentTime); g.gain.setValueAtTime(0.1, ctxAudio.currentTime); o.start(); o.stop(ctxAudio.currentTime+0.2);
    }

    function playFanfare(){
      const audioCtx = new (window.AudioContext||window.webkitAudioContext)();
      const notes = [523, 659, 784, 1046];
      notes.forEach((freq, i) => {
        const o = audioCtx.createOscillator();
        const g = audioCtx.createGain();
        o.connect(g); g.connect(audioCtx.destination);
        o.type='sawtooth';
        o.frequency.setValueAtTime(freq, audioCtx.currentTime + i*0.25);
        g.gain.setValueAtTime(0.15, audioCtx.currentTime + i*0.25);
        o.start(audioCtx.currentTime + i*0.25);
        o.stop(audioCtx.currentTime + i*0.25 + 0.4);
      });
    }

    function playGameOverSound(){
      const audioCtx = new (window.AudioContext||window.webkitAudioContext)();
      const o = audioCtx.createOscillator();
      const g = audioCtx.createGain();
      o.connect(g); g.connect(audioCtx.destination);
      o.type = 'triangle';
      o.frequency.setValueAtTime(220, audioCtx.currentTime);
      g.gain.setValueAtTime(0.2, audioCtx.currentTime);
      o.start();
      o.frequency.exponentialRampToValueAtTime(60, audioCtx.currentTime + 1);
      o.stop(audioCtx.currentTime+1);
    }

    let usedObstacles = [];

    function spawnObstacle() {
      let availableObstacles = obstaclesList.filter(o => !usedObstacles.includes(o.text));
      if(availableObstacles.length === 0) { usedObstacles = []; availableObstacles = [...obstaclesList]; }
      const cfg = availableObstacles[Math.floor(Math.random() * availableObstacles.length)];
      obstacles.push(new Obstacle(cfg));
      usedObstacles.push(cfg.text);
    }

    function winGame() {
      gameOver = true;
      playFanfare();
      document.getElementById('winMsg').style.display = 'block';
      setTimeout(()=>{
        level++; 
        if(level > 10){
          document.getElementById('winMsg').textContent = 'Congratulations! You are part of our amazing GCPS family.';
          return;
        }
        speed += 0.5 + level*0.3;
        obstacleFrequency = Math.max(120, obstacleFrequency - 20);
        obstacles = [];
        usedObstacles = [];
        sponsor.x = 100;
        sponsor.y = canvas.height - 160;
        sponsor.dy = 0;
        gameOver = false;
        document.getElementById('level').textContent = level;
        document.getElementById('winMsg').style.display = 'none';
        animate();
      }, 2000);
    }

    let obstacleTimer=0;
    function animate() {
      ctx.clearRect(0,0,canvas.width,canvas.height);
      ctx.fillStyle='#8B4513'; ctx.fillRect(0,canvas.height-100,canvas.width,100);

      goal.draw();
      sponsor.update();
      obstacleTimer++; if(obstacleTimer % obstacleFrequency === 0) spawnObstacle();

      for(let i=obstacles.length-1;i>=0;i--){
        let obs=obstacles[i]; obs.update();
        if(obs.x+obs.width<0){ obstacles.splice(i,1); score++; document.getElementById('score').textContent=score; }
        if(sponsor.x+sponsor.width>obs.x && sponsor.x<obs.x+obs.width && sponsor.y>obs.y-obs.height){ endGame(); }
      }

      if(sponsor.x + sponsor.width >= goal.x && sponsor.y >= goal.y - goal.height){
        winGame();
      }

      if(!gameOver) requestAnimationFrame(animate);
    }

    function endGame(){
      gameOver=true;
      playGameOverSound();
      document.getElementById('gameOverMsg').style.display='block';
      document.getElementById('retryBtn').style.display='block';
      if(score>highScore){
        highScore=score;
        localStorage.setItem('highScore',highScore);
        document.getElementById('highScore').textContent=highScore;
      }
    }

    document.addEventListener('keydown',e=>{ if(e.code==='Space') sponsor.jump(); if(e.code==='ArrowRight') sponsor.dx=3; if(e.code==='ArrowLeft') sponsor.dx=-3; });
    document.addEventListener('keyup',e=>{ if(e.code==='ArrowRight'||e.code==='ArrowLeft') sponsor.dx=0; });
    document.addEventListener('click',()=>sponsor.jump());

    document.getElementById('retryBtn').addEventListener('click',()=>{
      score=0; document.getElementById('score').textContent=score;
      obstacles=[]; speed=4; obstacleFrequency=140; level=1;
      sponsor.x=100; sponsor.y=canvas.height-160; sponsor.dy=0; sponsor.dx=0;
      gameOver=false;
      document.getElementById('level').textContent=level;
      document.getElementById('winMsg').style.display='none';
      document.getElementById('gameOverMsg').style.display='none';
      document.getElementById('retryBtn').style.display='none';
      animate();
    });

    // Start the game loop
    animate();
  </script>
</body>
</html>
