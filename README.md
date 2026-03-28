<!DOCTYPE html>
<html>
<head>
  <title>AI Hand Shooter (Thumb Shoot)</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">

  <script src="https://cdn.jsdelivr.net/npm/@mediapipe/hands/hands.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/camera_utils.js"></script>

  <style>
    body { margin:0; overflow:hidden; background:black; }
    canvas { position:absolute; top:0; left:0; }
    #score { position:absolute; top:10px; left:10px; color:white; font-size:20px; }
  </style>
</head>
<body>

<div id="score">Score: 0</div>
<video id="video" autoplay playsinline style="display:none;"></video>
<canvas id="canvas"></canvas>

<script>
const video = document.getElementById("video");
const canvas = document.getElementById("canvas");
const ctx = canvas.getContext("2d");

canvas.width = window.innerWidth;
canvas.height = window.innerHeight;

let score = 0;
let enemies = [];
let bullets = [];
let shootCooldown = 0;

let fingerX = canvas.width/2;
let fingerY = canvas.height/2;

// Spawn musuh
setInterval(() => {
    enemies.push({
        x: Math.random() * canvas.width,
        y: -50,
        size: 30
    });
}, 1000);

// Setup AI Hands
const hands = new Hands({
    locateFile: file => `https://cdn.jsdelivr.net/npm/@mediapipe/hands/${file}`
});

hands.setOptions({
    maxNumHands: 1,
    modelComplexity: 1,
    minDetectionConfidence: 0.7,
    minTrackingConfidence: 0.7
});

hands.onResults(results => {
    if (results.multiHandLandmarks.length > 0) {
        let hand = results.multiHandLandmarks[0];

        // 👉 AIM = telunjuk
        let index = hand[8];
        fingerX = index.x * canvas.width;
        fingerY = index.y * canvas.height;

        // 👍 DETEKSI JEMPOL BENGKOK
        let thumb_tip = hand[4];
        let thumb_ip = hand[3];

        let thumbBent = thumb_tip.y > thumb_ip.y;

        // 🔫 TEMBAK
        if (thumbBent && shootCooldown <= 0) {
            shoot();
            shootCooldown = 10;
        }
    }
});

// Kamera
const camera = new Camera(video, {
    onFrame: async () => {
        await hands.send({image: video});
    },
    width: 640,
    height: 480
});
camera.start();

// Tembak
function shoot() {
    bullets.push({
        x: fingerX,
        y: fingerY
    });
}

// Update game
function update() {
    ctx.clearRect(0,0,canvas.width,canvas.height);

    if (shootCooldown > 0) shootCooldown--;

    // 🎯 Crosshair
    ctx.fillStyle = "lime";
    ctx.beginPath();
    ctx.arc(fingerX, fingerY, 10, 0, Math.PI*2);
    ctx.fill();

    // 🔫 Peluru
    bullets.forEach((b, i) => {
        b.y -= 12;
        ctx.fillStyle = "yellow";
        ctx.fillRect(b.x, b.y, 5, 12);

        if (b.y < 0) bullets.splice(i,1);
    });

    // 👾 Musuh
    enemies.forEach((e, ei) => {
        e.y += 3;
        ctx.fillStyle = "red";
        ctx.fillRect(e.x, e.y, e.size, e.size);

        // 💥 Collision
        bullets.forEach((b, bi) => {
            if (b.x > e.x && b.x < e.x + e.size &&
                b.y > e.y && b.y < e.y + e.size) {

                enemies.splice(ei,1);
                bullets.splice(bi,1);
                score++;
                document.getElementById("score").innerText = "Score: " + score;
            }
        });

        if (e.y > canvas.height) enemies.splice(ei,1);
    });

    requestAnimationFrame(update);
}

update();
</script>

</body>
</html>
