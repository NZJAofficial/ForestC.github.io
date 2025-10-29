<!doctype html>
<html lang="et">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Liikumise ja objektide tuvastus (HTML)</title>
  <style>
    body{font-family:system-ui,-apple-system,BlinkMacSystemFont,'Segoe UI',Roboto,Arial;margin:0;display:flex;flex-direction:column;align-items:center;gap:12px;padding:12px}
    .stage{position:relative;width:100%;max-width:720px;background:#111;color:#fff;border-radius:10px;overflow:hidden}
    video{width:100%;height:auto;display:block}
    canvas.overlay{position:absolute;left:0;top:0;width:100%;height:100%;pointer-events:none}
    .controls{display:flex;gap:8px;flex-wrap:wrap}
    button,select{padding:8px 12px;border-radius:8px;border:1px solid #ccc;background:#fff;color:#000}
    .status{font-size:14px}
    .hint{font-size:13px;color:#444;max-width:720px}
  </style>
</head>
<body>
  <h2>Kaamera — liikumise ja objektide tuvastus</h2>
  <div class="stage">
    <video id="video" autoplay playsinline></video>
    <canvas id="overlay" class="overlay"></canvas>
  </div>

  <div class="controls">
    <button id="startBtn">Alusta</button>
    <button id="stopBtn" disabled>Peata</button>
    <label>Kaamera: <select id="camSelect"></select></label>
    <label>Liikumise tundlikkus: <input id="sensitivity" type="range" min="0.5" max="10" step="0.1" value="2"> <span id="sensVal">2.0</span></label>
  </div>

  <div class="status" id="status">Staatus: ootab käivitamist</div>
  <div class="hint">Märkus: see näidis kasutab TensorFlow.js'i ja COCO-SSD mudelit. Töötamiseks peab leht olema laaditud <strong>https</strong>-i või <strong>localhost</strong> üle. Soovitatav kasutada tagumist kaamerat ("environment") telefonis.</div>

  <!-- TensorFlow.js ja COCO-SSD CDN-id -->
  <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@3.21.0/dist/tf.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/@tensorflow-models/coco-ssd@2.2.2/dist/coco-ssd.min.js"></script>

  <script>
    // DOM
    const video = document.getElementById('video');
    const overlay = document.getElementById('overlay');
    const ctx = overlay.getContext('2d');
    const startBtn = document.getElementById('startBtn');
    const stopBtn = document.getElementById('stopBtn');
    const statusEl = document.getElementById('status');
    const camSelect = document.getElementById('camSelect');
    const sensitivity = document.getElementById('sensitivity');
    const sensVal = document.getElementById('sensVal');

    let stream = null;
    let model = null;
    let animationId = null;
    let prevImageData = null;

    // motion detection params
    let motionThresholdRatio = parseFloat(sensitivity.value); // lower -> more sensitive
    sensVal.textContent = motionThresholdRatio.toFixed(1);

    sensitivity.addEventListener('input', () => {
      motionThresholdRatio = parseFloat(sensitivity.value);
      sensVal.textContent = motionThresholdRatio.toFixed(1);
    });

    // enumerate cameras
    async function listCameras(){
      try{
        const devices = await navigator.mediaDevices.enumerateDevices();
        const cams = devices.filter(d => d.kind === 'videoinput');
        camSelect.innerHTML = '';
        cams.forEach((c, i) => {
          const opt = document.createElement('option');
          opt.value = c.deviceId;
          opt.text = c.label || `Kaamera ${i+1}`;
          camSelect.appendChild(opt);
        });
      }catch(e){
        console.warn('Ei saanud seadmeid lugeda', e);
      }
    }

    async function startCamera(){
      stopCamera();
      const deviceId = camSelect.value || undefined;
      const constraints = { video: { deviceId: deviceId ? { exact: deviceId } : undefined, facingMode: 'environment', width: { ideal: 1280 }, height: { ideal: 720 } }, audio: false };
      try{
        stream = await navigator.mediaDevices.getUserMedia(constraints);
        video.srcObject = stream;
        await video.play();
        overlay.width = video.videoWidth;
        overlay.height = video.videoHeight;
        prevImageData = null;
        status('Kaamera käivitatud. Laadin mudelit...');

        if(!model){
          // laadime COCO-SSD detektori
          model = await cocoSsd.load();
          status('Mudeli laaditud. Otsin liikumist ja tuvastan objekte.');
        } else {
          status('Mudeli juba laetud. Otsin liikumist ja tuvastan objekte.');
        }

        startLoop();
        startBtn.disabled = true;
        stopBtn.disabled = false;
      }catch(err){
        console.error(err);
        status('Viga kaamera avamisel: ' + err.message);
      }
    }

    function stopCamera(){
      if(animationId) cancelAnimationFrame(animationId);
      animationId = null;
      if(stream){
        stream.getTracks().forEach(t => t.stop());
        stream = null;
      }
      startBtn.disabled = false;
      stopBtn.disabled = true;
      status('Peatatatud.');
      ctx.clearRect(0,0,overlay.width, overlay.height);
    }

    function status(text){ statusEl.textContent = 'Staatus: ' + text; }

    // lihtne liikumise tuvastus: võtame iga kaadri, teisendame halltooniks, leiame erinevuse eelmise kaadriga
    function detectMotion(imageData){
      if(!prevImageData){ prevImageData = imageData; return false; }
      const w = imageData.width, h = imageData.height;
      const curr = imageData.data;
      const prev = prevImageData.data;
      let diffCount = 0;
      // sample every 4th pixel to reduce load
      for(let i=0;i<curr.length;i+=16){
        // RGB to gray
        const r1 = curr[i], g1 = curr[i+1], b1 = curr[i+2];
        const r0 = prev[i], g0 = prev[i+1], b0 = prev[i+2];
        const gray1 = (r1+g1+b1)/3;
        const gray0 = (r0+g0+b0)/3;
        if(Math.abs(gray1-gray0) > 25) diffCount++;
      }
      prevImageData = imageData;
      const sampledPixels = (w*h)/4; // because step 16 covers 4 pixels roughly per loop iteration
      const ratio = diffCount / sampledPixels;
      // adapt threshold: motionThresholdRatio values ~ 0.5..10, map to ratio threshold
      const threshold = 0.01 * motionThresholdRatio; // empiric
      return ratio > threshold;
    }

    async function startLoop(){
      const off = document.createElement('canvas');
      const offCtx = off.getContext('2d');
      off.width = video.videoWidth;
      off.height = video.videoHeight;
      overlay.width = off.width;
      overlay.height = off.height;

      async function loop(){
        if(video.paused || video.ended){ animationId = requestAnimationFrame(loop); return; }
        // draw current frame to offscreen
        offCtx.drawImage(video, 0, 0, off.width, off.height);
        const imageData = offCtx.getImageData(0,0,off.width, off.height);
        const motion = detectMotion(imageData);

        // clear overlay
        ctx.clearRect(0,0,overlay.width,overlay.height);

        if(motion){
          status('Liikumine tuvastatud — käivitan objektituvastuse');
          try{
            // model.detect võib võtta videoelemendi otse
            const predictions = await model.detect(video);
            // joonista kastid ja sildid
            predictions.forEach(pred => {
              const [x,y,w,h] = pred.bbox;
              ctx.strokeStyle = 'lime';
              ctx.lineWidth = Math.max(2, Math.round(overlay.width/400));
              ctx.strokeRect(x,y,w,h);
              ctx.font = `${Math.max(12, Math.round(overlay.width/50))}px sans-serif`;
              ctx.fillStyle = 'rgba(0,0,0,0.5)';
              const text = `${pred.class} ${(pred.score*100).toFixed(0)}%`;
              const textW = ctx.measureText(text).width + 8;
              const textH = parseInt(ctx.font,10) + 6;
              ctx.fillRect(x, y - textH, textW, textH);
              ctx.fillStyle = 'white';
              ctx.fillText(text, x+4, y-4);
            });
            if(predictions.length===0){ status('Liikumine tuvastatud, aga objekti ei leitud.'); }
          }catch(e){ console.error('detect error', e); status('Objektituvastuse viga: ' + (e.message||e)); }
        } else {
          // kui ei ole liikumist, näita kerget infot
          status('Ootab liikumist...');
        }

        animationId = requestAnimationFrame(loop);
      }
      loop();
    }

    // tapahtised
    startBtn.addEventListener('click', startCamera);
    stopBtn.addEventListener('click', stopCamera);

    camSelect.addEventListener('change', async ()=>{
      // kui jooksvalt käib, taaskäivita uue kaameraga
      if(stream){ await startCamera(); }
    })

    // inits
    (async ()=>{
      if(!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia){
        status('Brauser ei toeta getUserMedia API-d. Proovi Chrome/Edge/Firefox mobiilis või töölaudadel.');
        return;
      }
      await listCameras();
      // kui on valik, püüa valida environment (tagumine)
      for(let i=0;i<camSelect.options.length;i++){
        const opt = camSelect.options[i];
        if(/back|rear|environment/i.test(opt.text)) { camSelect.selectedIndex = i; break; }
      }
    })();

  </script>
</body>
</html>
