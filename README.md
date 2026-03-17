<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sabri Emir AI Mirror - V3</title>
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/pose/pose.js" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/camera_utils.js" crossorigin="anonymous"></script>
    <style>
        body { margin: 0; background: #000; overflow: hidden; font-family: sans-serif; color: #fff; }
        #canvas_output { position: absolute; width: 100vw; height: 100vh; transform: scaleX(-1); object-fit: cover; display: none; }
        
        /* Başlatma Ekranı */
        #overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.9); display: flex; flex-direction: column; justify-content: center; align-items: center; z-index: 1000; }
        #start-btn { padding: 20px 40px; font-size: 24px; font-weight: bold; color: #000; background: #00f5ff; border: none; border-radius: 50px; cursor: pointer; box-shadow: 0 0 20px #00f5ff; transition: 0.3s; }
        #start-btn:hover { transform: scale(1.1); box-shadow: 0 0 40px #00f5ff; }
        
        #ui { position: absolute; top: 20px; left: 20px; z-index: 100; padding: 15px; background: rgba(0,0,0,0.7); border-radius: 15px; border: 2px solid #00f5ff; display: none; }
        #menu { position: absolute; right: 20px; top: 20px; bottom: 20px; width: 200px; background: rgba(0,0,0,0.85); border: 2px solid #00f5ff; border-radius: 20px; overflow-y: auto; padding: 15px; z-index: 200; display: none; }
        .item { padding: 12px; margin-bottom: 10px; background: rgba(0, 245, 255, 0.1); border-radius: 10px; cursor: pointer; text-align: center; border: 1px solid transparent; }
        .active { background: #00f5ff; color: #000; font-weight: bold; }
    </style>
</head>
<body>

<div id="overlay">
    <h1 style="color: #00f5ff; margin-bottom: 30px;">SABRI EMIR AI MIRROR</h1>
    <button id="start-btn">AYNAYI BAŞLAT 🚀</button>
    <p style="margin-top: 20px; color: #aaa;">Kamerayı açmak için butona tıkla kanka</p>
</div>

<div id="ui">
    <b style="color: #00f5ff;">SABRI EMIR AI</b><br>
    <small id="status">Hazır...</small>
</div>

<canvas id="canvas_output"></canvas>

<div id="menu">
    <h3 style="color:#00f5ff; text-align:center;">KOMBİNLER</h3>
    <div id="list"></div>
</div>

<script>
    const btn = document.getElementById('start-btn');
    const overlay = document.getElementById('overlay');
    const canvas = document.getElementById('canvas_output');
    const ui = document.getElementById('ui');
    const menu = document.getElementById('menu');
    const ctx = canvas.getContext('2d');
    
    let currentStyle = { color: "rgba(0, 245, 255, 0.4)", secondary: "#fff" };

    // 17 Kombin Menüsü
    for(let i=1; i<=17; i++) {
        const d = document.createElement('div');
        d.className = 'item' + (i===1 ? ' active' : '');
        d.innerText = i + ". Kombin Tarzı";
        d.onclick = () => {
            document.querySelectorAll('.item').forEach(el => el.classList.remove('active'));
            d.classList.add('active');
            // Her tıklandığında renk değişsin (örnek)
            currentStyle.color = `hsla(${i * 20}, 70%, 50%, 0.5)`;
        };
        document.getElementById('list').appendChild(d);
    }

    function onResults(results) {
        ctx.save();
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        ctx.drawImage(results.image, 0, 0, canvas.width, canvas.height);

        if (results.poseLandmarks) {
            const p = results.poseLandmarks;
            const sL = p[11], sR = p[12], hL = p[23], hR = p[24];
            ctx.beginPath();
            ctx.moveTo(sL.x * canvas.width, sL.y * canvas.height);
            ctx.lineTo(sR.x * canvas.width, sR.y * canvas.height);
            ctx.lineTo(hR.x * canvas.width, hR.y * canvas.height);
            ctx.lineTo(hL.x * canvas.width, hL.y * canvas.height);
            ctx.closePath();
            ctx.fillStyle = currentStyle.color;
            ctx.fill();
            ctx.strokeStyle = currentStyle.secondary;
            ctx.lineWidth = 5;
            ctx.stroke();
        }
        ctx.restore();
    }

    const pose = new Pose({locateFile: (file) => `https://cdn.jsdelivr.net/npm/@mediapipe/pose/${file}`});
    pose.setOptions({ modelComplexity: 1, smoothLandmarks: true, minDetectionConfidence: 0.5 });
    pose.onResults(onResults);

    btn.onclick = async () => {
        try {
            const stream = await navigator.mediaDevices.getUserMedia({ video: true });
            const video = document.createElement('video');
            video.srcObject = stream;
            await video.play();
            
            overlay.style.display = 'none';
            canvas.style.display = 'block';
            ui.style.display = 'block';
            menu.style.display = 'block';

            const camera = new Camera(video, {
                onFrame: async () => { 
                    canvas.width = video.videoWidth;
                    canvas.height = video.videoHeight;
                    await pose.send({image: video}); 
                }
            });
            camera.start();
        } catch (e) {
            alert("Kanka kameraya izin vermen lazım!");
        }
    };
</script>
</body>
</html>
