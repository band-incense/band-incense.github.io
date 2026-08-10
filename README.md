<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>INCENSE</title>

<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Italiana&family=JetBrains+Mono:wght@300;400&display=swap" rel="stylesheet">
<link rel="stylesheet" as="style" crossorigin href="https://cdn.jsdelivr.net/gh/orioncactus/pretendard@v1.3.9/dist/web/variable/pretendardvariable-dynamic-subset.min.css" />

<style>
  :root{
    --ink:       #08070a;   /* 배경 - 완전한 검정 대신 아주 옅은 보랏빛 */
    --ash:       #17151b;   /* 연기 그림자 */
    --bone:      #d8d3cb;   /* 연기 하이라이트 / 본문 텍스트 */
    --bone-dim:  #79736c;   /* 보조 텍스트 */
    --coal:      #e0561c;   /* 잔불 - 아주 작은 면적에만 */

    --display: 'Italiana', serif;
    --body: 'Pretendard Variable', Pretendard, -apple-system, sans-serif;
    --mono: 'JetBrains Mono', monospace;
  }

  *{ margin:0; padding:0; box-sizing:border-box; }

  html,body{ height:100%; background:var(--ink); }

  body{
    font-family:var(--body);
    color:var(--bone);
    overflow-x:hidden;
    -webkit-font-smoothing:antialiased;
  }

  /* ── 캔버스 ─────────────────────────────── */
  #smoke{
    position:fixed;
    inset:0;
    width:100%;
    height:100%;
    display:block;
    z-index:0;
  }

  .hero{
    position:relative;
    z-index:1;
    height:100svh;
    display:grid;
    grid-template-rows:auto 1fr auto;
    padding:clamp(20px,3.5vw,40px);
    pointer-events:none;
  }
  .hero a, .hero button{ pointer-events:auto; }

  /* ── 상단 바 ────────────────────────────── */
  .topbar{
    display:flex;
    justify-content:space-between;
    align-items:flex-start;
    gap:24px;
    font-family:var(--mono);
    font-size:11px;
    font-weight:300;
    letter-spacing:.16em;
    text-transform:uppercase;
    color:var(--bone-dim);
  }

  .nav{ display:flex; gap:clamp(16px,2.4vw,34px); flex-wrap:wrap; }
  .nav a{
    color:var(--bone-dim);
    text-decoration:none;
    transition:color .45s ease;
  }
  .nav a:hover, .nav a:focus-visible{ color:var(--bone); }

  .mark{ white-space:nowrap; }

  /* ── 워드마크 ───────────────────────────── */
  .wordmark-block{
    align-self:center;
    justify-self:center;
    text-align:center;
    width:100%;
  }

  .wordmark{
    font-family:var(--display);
    font-size:clamp(60px,13vw,190px);
    line-height:.95;
    letter-spacing:clamp(.06em,1.1vw,.16em);
    text-indent:clamp(.06em,1.1vw,.16em); /* 자간 때문에 생기는 우측 쏠림 보정 */
    color:var(--bone);
    display:flex;
    justify-content:center;
    flex-wrap:nowrap;
  }

  .wordmark span{
    display:inline-block;
    opacity:0;
    filter:blur(14px);
    transform:translateY(18px);
    animation:emerge 2.4s cubic-bezier(.16,.8,.3,1) forwards;
  }

  @keyframes emerge{
    to{ opacity:1; filter:blur(0); transform:translateY(0); }
  }

  .tagline{
    margin-top:clamp(10px,1.6vw,20px);
    font-family:var(--mono);
    font-size:clamp(9px,1vw,11px);
    font-weight:300;
    letter-spacing:.42em;
    text-indent:.42em;
    text-transform:uppercase;
    color:var(--bone);
    text-shadow:0 0 18px rgba(8,7,10,.9);
    opacity:0;
    animation:fadeUp 1.6s ease 2.5s forwards;
  }

  @keyframes fadeUp{ to{ opacity:1; } }

  /* ── 하단 ───────────────────────────────── */
  .footer{
    display:flex;
    justify-content:space-between;
    align-items:flex-end;
    gap:24px;
    opacity:0;
    animation:fadeUp 1.6s ease 3s forwards;
  }

  .next-live{
    font-family:var(--mono);
    font-size:11px;
    font-weight:300;
    letter-spacing:.1em;
    line-height:1.9;
    color:var(--bone-dim);
  }
  .next-live .label{
    display:block;
    font-size:9px;
    letter-spacing:.3em;
    color:#4a453f;
  }
  .next-live .venue{ color:var(--bone); }

  .scroll-cue{
    font-family:var(--mono);
    font-size:9px;
    letter-spacing:.3em;
    text-transform:uppercase;
    color:#4a453f;
    display:flex;
    align-items:center;
    gap:10px;
  }
  .scroll-cue::after{
    content:"";
    width:1px;
    height:34px;
    background:linear-gradient(to bottom, #4a453f, transparent);
    animation:drift 2.8s ease-in-out infinite;
  }
  @keyframes drift{
    0%,100%{ transform:scaleY(1); transform-origin:top; opacity:.9; }
    50%{ transform:scaleY(.55); opacity:.4; }
  }

  @media (max-width:640px){
    .topbar{ flex-direction:column; gap:14px; }
    .footer{ flex-direction:column; align-items:flex-start; gap:18px; }
    .scroll-cue::after{ height:24px; }
  }

  @media (prefers-reduced-motion:reduce){
    .wordmark span, .tagline, .footer{
      animation-duration:.01ms !important;
      animation-delay:0ms !important;
      opacity:1; filter:none; transform:none;
    }
    .scroll-cue::after{ animation:none; }
  }
</style>
</head>
<body>

<canvas id="smoke" aria-hidden="true"></canvas>

<main class="hero">

  <header class="topbar">
    <nav class="nav">
      <a href="#live">Live</a>
      <a href="#members">Members</a>
      <a href="#discography">Discography</a>
      <a href="#goods">Goods</a>
      <a href="#contact">Contact</a>
    </nav>
    <div class="mark">Est. 2026 — Seoul</div>
  </header>

  <div class="wordmark-block">
    <h1 class="wordmark" aria-label="INCENSE">
      <span>I</span><span>N</span><span>C</span><span>E</span><span>N</span><span>S</span><span>E</span>
    </h1>
    <!-- 아래 문구는 밴드 소개에 맞게 교체하세요 -->
    <p class="tagline">5-piece band</p>
  </div>

  <footer class="footer">
    <!-- 캘린더 연동 전까지의 임시 데이터입니다 -->
    <div class="next-live">
      <span class="label">Next Live</span>
      2026.09.12 FRI 19:00<br>
      <span class="venue">홍대 브이홀</span>
    </div>
    <div class="scroll-cue">Scroll</div>
  </footer>

</main>

<script id="vert" type="x-shader/x-vertex">
attribute vec2 aPos;
void main(){ gl_Position = vec4(aPos, 0.0, 1.0); }
</script>

<script id="frag" type="x-shader/x-fragment">
precision highp float;

uniform vec2  uRes;
uniform float uTime;
uniform vec2  uMouse;
uniform float uIgnite;   // 0 → 1, 로드 직후 잔불이 붙는 연출
uniform float uOct;      // 노이즈 옥타브 수 (기기 성능에 따라 조절)

/* ── 3D Simplex Noise (Ashima Arts / Stefan Gustavson, MIT) ── */
vec3 mod289(vec3 x){ return x - floor(x * (1.0/289.0)) * 289.0; }
vec4 mod289(vec4 x){ return x - floor(x * (1.0/289.0)) * 289.0; }
vec4 permute(vec4 x){ return mod289(((x*34.0)+1.0)*x); }
vec4 taylorInvSqrt(vec4 r){ return 1.79284291400159 - 0.85373472095314 * r; }

float snoise(vec3 v){
  const vec2 C = vec2(1.0/6.0, 1.0/3.0);
  const vec4 D = vec4(0.0, 0.5, 1.0, 2.0);

  vec3 i  = floor(v + dot(v, C.yyy));
  vec3 x0 = v - i + dot(i, C.xxx);

  vec3 g = step(x0.yzx, x0.xyz);
  vec3 l = 1.0 - g;
  vec3 i1 = min(g.xyz, l.zxy);
  vec3 i2 = max(g.xyz, l.zxy);

  vec3 x1 = x0 - i1 + C.xxx;
  vec3 x2 = x0 - i2 + C.yyy;
  vec3 x3 = x0 - D.yyy;

  i = mod289(i);
  vec4 p = permute(permute(permute(
             i.z + vec4(0.0, i1.z, i2.z, 1.0))
           + i.y + vec4(0.0, i1.y, i2.y, 1.0))
           + i.x + vec4(0.0, i1.x, i2.x, 1.0));

  float n_ = 0.142857142857;
  vec3 ns = n_ * D.wyz - D.xzx;

  vec4 j = p - 49.0 * floor(p * ns.z * ns.z);

  vec4 x_ = floor(j * ns.z);
  vec4 y_ = floor(j - 7.0 * x_);

  vec4 x = x_ * ns.x + ns.yyyy;
  vec4 y = y_ * ns.x + ns.yyyy;
  vec4 h = 1.0 - abs(x) - abs(y);

  vec4 b0 = vec4(x.xy, y.xy);
  vec4 b1 = vec4(x.zw, y.zw);

  vec4 s0 = floor(b0) * 2.0 + 1.0;
  vec4 s1 = floor(b1) * 2.0 + 1.0;
  vec4 sh = -step(h, vec4(0.0));

  vec4 a0 = b0.xzyw + s0.xzyw * sh.xxyy;
  vec4 a1 = b1.xzyw + s1.xzyw * sh.zzww;

  vec3 p0 = vec3(a0.xy, h.x);
  vec3 p1 = vec3(a0.zw, h.y);
  vec3 p2 = vec3(a1.xy, h.z);
  vec3 p3 = vec3(a1.zw, h.w);

  vec4 norm = taylorInvSqrt(vec4(dot(p0,p0), dot(p1,p1), dot(p2,p2), dot(p3,p3)));
  p0 *= norm.x; p1 *= norm.y; p2 *= norm.z; p3 *= norm.w;

  vec4 m = max(0.6 - vec4(dot(x0,x0), dot(x1,x1), dot(x2,x2), dot(x3,x3)), 0.0);
  m = m * m;
  return 42.0 * dot(m*m, vec4(dot(p0,x0), dot(p1,x1), dot(p2,x2), dot(p3,x3)));
}

float fbm(vec3 p){
  float sum = 0.0;
  float amp = 0.5;
  for(int i = 0; i < 5; i++){
    if(float(i) >= uOct) break;
    sum += amp * snoise(p);
    p *= 2.03;
    amp *= 0.5;
  }
  return sum;
}

float hash(vec2 p){
  return fract(sin(dot(p, vec2(12.9898, 78.233))) * 43758.5453);
}

void main(){
  vec2 uv = (gl_FragCoord.xy - 0.5 * uRes) / uRes.y;

  /* 향의 발화점을 화면 아래쪽에 둔다 */
  vec2 p = uv;
  p.y += 0.40;

  float h = max(p.y, 0.0);

  /* 세로로 긴 화면에서는 기둥을 좁혀 형태가 남게 한다 */
  float narrow = clamp((uRes.x / uRes.y) / 1.6, 0.44, 1.0);

  /* 마우스는 연기를 아주 약하게 밀어낸다 */
  float push = uMouse.x * 0.10;

  /* 상승하면서 좌우로 흔들리는 중심선 */
  float sway = fbm(vec3(0.0, p.y * 1.25 - uTime * 0.19, uTime * 0.06)) * 0.16;
  float centre = (sway * h * 1.5 + push * h) * narrow;

  /* 위로 갈수록 넓게 퍼진다 */
  float width = (0.040 + h * h * 0.72 + h * 0.20) * narrow;
  float lateral = (p.x - centre) / width;

  /* 연기 좌표: 시간에 따라 위로 흘러간다 */
  vec3 q = vec3(p.x * 2.4, p.y * 1.05 - uTime * 0.17, uTime * 0.045);

  /* 도메인 워핑 — 높이가 올라갈수록 난류가 강해진다 */
  float turb = 0.10 + h * 1.35;
  vec3 w = vec3(
    fbm(q * 1.25),
    fbm(q * 1.25 + vec3(5.2, 1.3, 2.8)),
    fbm(q * 1.25 + vec3(9.1, 7.4, 4.6))
  );
  float d = fbm((q + w * turb) * 1.55);
  d = d * 0.5 + 0.5;

  /* 기둥 형태로 잘라낸다 */
  float column = exp(-lateral * lateral * (1.35 - h * 0.32));
  float base   = smoothstep(-0.03, 0.10, p.y);
  float top    = 1.0 - smoothstep(0.26, 0.90, p.y);
  float mask   = column * base * top;

  float dens = smoothstep(0.30, 0.88, d) * mask;
  dens = pow(dens, 0.92) * uIgnite;

  /* ── 색 ── */
  vec3 ink       = vec3(0.031, 0.027, 0.039);
  vec3 smokeDeep = vec3(0.175, 0.163, 0.188);
  vec3 smokePale = vec3(0.880, 0.862, 0.828);
  vec3 coal      = vec3(1.000, 0.360, 0.110);

  vec3 col = ink;
  col = mix(col, smokeDeep, clamp(dens * 2.4, 0.0, 1.0));
  col = mix(col, smokePale, pow(dens, 1.9) * 1.0);

  /* 발화점 근처에만 따뜻한 기운을 남긴다 */
  col += coal * 0.115 * dens * exp(-p.y * 4.2);
  col += coal * 0.030 * exp(-length(vec2((p.x - centre) * 1.6, p.y * 2.2)) * 4.5);

  /* 잔불 — 아주 작게, 미세하게 숨쉬듯 */
  float ember = exp(-length(vec2(p.x - centre, p.y * 1.7)) * 30.0);
  float pulse = 0.72 + 0.28 * sin(uTime * 1.9 + snoise(vec3(uTime * 0.7)) * 2.4);
  col += coal * ember * pulse * 1.9 * uIgnite;

  /* 비네트 + 필름 그레인 */
  col *= 1.0 - 0.42 * length(uv * vec2(0.75, 1.0));
  col += (hash(gl_FragCoord.xy + fract(uTime) * 91.7) - 0.5) * 0.022;

  gl_FragColor = vec4(col, 1.0);
}
</script>

<script>
(function(){
  var canvas = document.getElementById('smoke');
  var gl = canvas.getContext('webgl', { antialias:false, alpha:false, powerPreference:'high-performance' })
        || canvas.getContext('experimental-webgl');

  if(!gl){
    // WebGL을 못 쓰는 환경에서는 조용히 배경색만 남긴다
    canvas.style.background = 'radial-gradient(ellipse at 50% 78%, #241d1a 0%, #08070a 62%)';
    return;
  }

  function compile(type, src){
    var s = gl.createShader(type);
    gl.shaderSource(s, src);
    gl.compileShader(s);
    if(!gl.getShaderParameter(s, gl.COMPILE_STATUS)){
      console.error(gl.getShaderInfoLog(s));
      return null;
    }
    return s;
  }

  var vs = compile(gl.VERTEX_SHADER, document.getElementById('vert').textContent);
  var fs = compile(gl.FRAGMENT_SHADER, document.getElementById('frag').textContent);
  if(!vs || !fs) return;

  var prog = gl.createProgram();
  gl.attachShader(prog, vs);
  gl.attachShader(prog, fs);
  gl.linkProgram(prog);
  gl.useProgram(prog);

  var buf = gl.createBuffer();
  gl.bindBuffer(gl.ARRAY_BUFFER, buf);
  gl.bufferData(gl.ARRAY_BUFFER, new Float32Array([-1,-1, 3,-1, -1,3]), gl.STATIC_DRAW);
  var aPos = gl.getAttribLocation(prog, 'aPos');
  gl.enableVertexAttribArray(aPos);
  gl.vertexAttribPointer(aPos, 2, gl.FLOAT, false, 0, 0);

  var uRes    = gl.getUniformLocation(prog, 'uRes');
  var uTime   = gl.getUniformLocation(prog, 'uTime');
  var uMouse  = gl.getUniformLocation(prog, 'uMouse');
  var uIgnite = gl.getUniformLocation(prog, 'uIgnite');
  var uOct    = gl.getUniformLocation(prog, 'uOct');

  /* 연기는 형태가 부드러워서 저해상도로 그려도 티가 나지 않는다.
     해상도를 낮춰 모바일 발열과 프레임 드랍을 막는다. */
  var isMobile = window.matchMedia('(max-width: 820px)').matches;
  var scale = isMobile ? 0.50 : 0.68;
  var octaves = isMobile ? 3.0 : 5.0;
  gl.uniform1f(uOct, octaves);

  function resize(){
    var dpr = Math.min(window.devicePixelRatio || 1, 2);
    var w = Math.max(1, Math.round(window.innerWidth  * dpr * scale));
    var h = Math.max(1, Math.round(window.innerHeight * dpr * scale));
    if(canvas.width !== w || canvas.height !== h){
      canvas.width = w;
      canvas.height = h;
      gl.viewport(0, 0, w, h);
      gl.uniform2f(uRes, w, h);
    }
  }
  window.addEventListener('resize', resize);
  resize();

  var mx = 0, tmx = 0;
  window.addEventListener('pointermove', function(e){
    tmx = (e.clientX / window.innerWidth) * 2.0 - 1.0;
  });

  var reduced = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
  var start = performance.now();

  function frame(now){
    var t = (now - start) / 1000;
    resize();

    mx += (tmx - mx) * 0.035;

    /* 로드 직후 4초에 걸쳐 잔불이 붙고 연기가 올라온다 */
    var ignite = Math.min(t / 4.0, 1.0);
    ignite = ignite * ignite * (3.0 - 2.0 * ignite);

    gl.uniform1f(uTime, reduced ? 12.0 : t);
    gl.uniform1f(uIgnite, reduced ? 1.0 : ignite);
    gl.uniform2f(uMouse, mx, 0.0);
    gl.drawArrays(gl.TRIANGLES, 0, 3);

    if(!reduced) requestAnimationFrame(frame);
  }
  requestAnimationFrame(frame);
})();

/* 워드마크 글자를 순차적으로 피어오르게 한다 */
(function(){
  var letters = document.querySelectorAll('.wordmark span');
  for(var i = 0; i < letters.length; i++){
    letters[i].style.animationDelay = (0.9 + i * 0.14) + 's';
  }
})();
</script>

</body>
</html>
