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

    --display: 'Italiana', serif;
    --body: 'Pretendard Variable', Pretendard, -apple-system, sans-serif;
    --mono: 'JetBrains Mono', monospace;
  }

  *{ margin:0; padding:0; border:0; box-sizing:border-box; }

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
    height:100vh;
    height:100svh;
    min-height:100vh;
    min-height:100svh;
    display:grid;
    grid-template-rows:auto 1fr auto;
    padding:clamp(20px,3.5vw,40px);
    pointer-events:none;
  }
  .hero a, .hero button{ pointer-events:auto; }

  /* ── 상단 바 ────────────────────────────── */
  .topbar{
    grid-row:1;
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

  .mark{ white-space:nowrap; text-align:right; line-height:2; }

  /* ── 로고 아래 태그라인 ──────────────────── */
  .visually-hidden{
    position:absolute; width:1px; height:1px;
    overflow:hidden; clip:rect(0 0 0 0); white-space:nowrap;
  }

  @keyframes fadeUp{ to{ opacity:1; } }

  /* ── 하단 ───────────────────────────────── */
  .footer{
    grid-row:3;
    align-self:end;
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


  @media (max-width:640px){
    .topbar{ flex-direction:column; gap:14px; }
    .mark{ text-align:left; line-height:1.9; }
    .footer{ flex-direction:column; align-items:flex-start; gap:18px; }
  
  @media (prefers-reduced-motion:reduce){
    .footer{
      animation-duration:.01ms !important;
      animation-delay:0ms !important;
      opacity:1;
    }
  </style>
</head>
<body>

<canvas id="smoke" aria-hidden="true"></canvas>

<main class="hero">

  <header class="topbar">
    <nav class="nav">
      <a href="#schedule">Schedule</a>
      <a href="#members">Members</a>
      <a href="#discography">Discography</a>
      <a href="#goods">Goods</a>
      <a href="#contact">Contact</a>
    </nav>
    <div class="mark">Est. 2026 — Seoul<br>5-piece band</div>
  </header>

  <h1 class="visually-hidden">INCENSE</h1>

  <footer class="footer">
    <!-- data/shows.json 의 다음 일정이 자동으로 채워집니다 -->
    <div class="next-live" id="nextLive" hidden>
      <span class="label">Next Live</span>
      <span id="nextLiveDate"></span><br>
      <span class="venue" id="nextLiveVenue"></span>
    </div>
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
uniform sampler2D uLogo; // 로고 알파 텍스처
uniform vec2  uLogoSize; // 로고의 가로/세로 크기 (uv 단위)
uniform float uLogoY;    // 로고 중심의 세로 위치
uniform float uGather;   // 0 = 연기, 1 = 로고로 응결 완료
uniform float uHasLogo;  // 텍스처 로드 완료 여부

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

  /* 향의 시작점을 화면 아래 가장자리에 둔다 */
  vec2 p = uv;
  p.y += 0.47;

  float h = max(p.y, 0.0);
  float t = uTime;

  /* 세로로 긴 화면에서는 기둥을 좁혀 형태가 프레임 안에 남게 한다 */
  float narrow = clamp((uRes.x / uRes.y) / 1.6, 0.42, 1.0);

  /* 층류 → 난류 전이.
     실제 향 연기는 한동안 곧게 오르다 특정 높이에서 갑자기 흐트러진다. */
  float lam = smoothstep(0.26, 0.82, h);

  /* 중심선을 흔드는 파동. 위상이 높이에 따라 달라지므로
     좌우로 밀리는 게 아니라 S자로 굽이치며 위로 전달된다. */
  float wave =
      sin(p.y *  3.4 - t * 0.78)              * 1.00
    + sin(p.y *  5.9 - t * 1.18 + 1.9)        * 0.40
    + sin(p.y *  9.3 - t * 1.72 + 4.1)        * 0.15;

  /* 불규칙한 표류 — 파동만 있으면 기계적으로 보인다 */
  float wander = fbm(vec3(0.0, p.y * 1.10 - t * 0.22, t * 0.08)) * 0.72;

  /* 흔들림의 폭은 위로 갈수록 커진다 */
  float amp  = smoothstep(0.015, 0.42, h) * (0.030 + h * h * 0.235);
  float push = uMouse.x * 0.055 * h * h;
  /* 폭은 세로 화면에서 좁히되, 굽이까지 같이 줄이면 곧은 바늘이 된다 */
  float swayScale = mix(1.0, narrow, 0.70);
  float centre = ((wave * 0.42 + wander) * amp + push) * swayScale;

  /* 연기 폭: 아래는 실처럼 가늘고, 전이 후 빠르게 확산 */
  float width = (0.009 + h * 0.026 + lam * lam * 0.22) * narrow;
  float lateral = (p.x - centre) / width;

  /* 노이즈를 중심선 기준으로 뽑는다 → 결이 굽이를 따라 흐른다 */
  vec3 q = vec3((p.x - centre) * 2.6, p.y * 1.05 - t * 0.20, t * 0.05);

  /* 도메인 워핑 — 층류 구간에서는 거의 걸지 않는다 */
  float turb = 0.015 + lam * lam * 0.85;
  vec3 w = vec3(
    fbm(q * 1.30),
    fbm(q * 1.30 + vec3(5.2, 1.3, 2.8)),
    fbm(q * 1.30 + vec3(9.1, 7.4, 4.6))
  );
  float d = fbm((q + w * turb) * 1.60);
  d = d * 0.5 + 0.5;

  /* 리본 단면 — 위로 갈수록 가장자리가 무르게 풀린다 */
  float column = exp(-lateral * lateral * (2.60 - lam * 1.15));
  float base   = smoothstep(-0.015, 0.045, p.y);
  float top    = 1.0 - smoothstep(0.46, 0.99, p.y);
  float mask   = column * base * top;

  /* 층류 구간은 노이즈를 거의 안 먹여 매끈한 실로 남긴다 */
  float grainy = smoothstep(0.29, 0.86, d);
  float body   = mix(0.86, grainy, lam);

  float volume = 0.60 + h * 0.75;
  float plume = clamp(body * mask * volume, 0.0, 1.0) * uIgnite;

  /* ── 연기가 로고 형태로 응결한다 ────────────────── */
  float gather = uGather * uHasLogo;

  /* 로고 좌표계로 옮긴다 */
  vec2 luv = (uv - vec2(0.0, uLogoY)) / uLogoSize + 0.5;

  /* 알파를 여러 번 떠서 부드러운 윤곽을 만든다 (연기처럼 번지게) */
  vec2 e = 1.6 / uLogoSize / uRes.y;
  float soft =
      texture2D(uLogo, luv).a * 0.28
    + texture2D(uLogo, luv + vec2( e.x, 0.0)).a * 0.12
    + texture2D(uLogo, luv + vec2(-e.x, 0.0)).a * 0.12
    + texture2D(uLogo, luv + vec2(0.0,  e.y)).a * 0.12
    + texture2D(uLogo, luv + vec2(0.0, -e.y)).a * 0.12
    + texture2D(uLogo, luv + e * 2.2).a * 0.06
    + texture2D(uLogo, luv - e * 2.2).a * 0.06
    + texture2D(uLogo, luv + vec2(e.x, -e.y) * 2.2).a * 0.06
    + texture2D(uLogo, luv - vec2(e.x, -e.y) * 2.2).a * 0.06;

  /* 화면 밖은 로고가 없다 */
  float inRect = step(0.0, luv.x) * step(luv.x, 1.0)
               * step(0.0, luv.y) * step(luv.y, 1.0);
  soft *= inRect;

  /* 로고 안쪽에도 연기 결이 흐른다 */
  float ln = fbm(vec3(uv * 3.1, t * 0.16)) * 0.5 + 0.5;

  /* 응결 진행에 따라 문턱값이 내려가며 형태가 드러난다 */
  float thr = mix(1.30, 0.26, gather);
  float logoD = smoothstep(thr, thr + 0.30, soft + (ln - 0.5) * 0.70 * (1.0 - gather * 0.55));
  logoD *= 0.58 + 0.42 * ln;
  logoD += soft * 0.10 * ln * gather;   /* 옅은 잔무리 */
  logoD *= gather;

  /* 응결이 끝나면 위쪽 연기는 걷히고 로고로 향하는 실만 남는다 */
  float logoBottom = uLogoY - uLogoSize.y * 0.5;
  float belowLogo = 1.0 - smoothstep(logoBottom - 0.04, logoBottom + 0.12, uv.y);
  float thread = plume * belowLogo * (1.0 - 0.30 * gather);

  float dens = clamp(max(mix(plume, thread, gather), logoD), 0.0, 1.0);

  /* ── 색 ── */
  vec3 ink       = vec3(0.031, 0.027, 0.039);
  vec3 smokeDeep = vec3(0.175, 0.163, 0.188);
  vec3 smokePale = vec3(0.880, 0.862, 0.828);

  vec3 col = ink;
  col = mix(col, smokeDeep, clamp(dens * 2.7, 0.0, 1.0));
  col = mix(col, smokePale, pow(dens, 1.7));

  /* 비네트 + 필름 그레인 */
  col *= 1.0 - 0.34 * length(uv * vec2(0.70, 1.0));
  col += (hash(gl_FragCoord.xy + fract(t) * 91.7) - 0.5) * 0.022;

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
  var uOct      = gl.getUniformLocation(prog, 'uOct');
  var uLogo     = gl.getUniformLocation(prog, 'uLogo');
  var uLogoSize = gl.getUniformLocation(prog, 'uLogoSize');
  var uLogoY    = gl.getUniformLocation(prog, 'uLogoY');
  var uGather   = gl.getUniformLocation(prog, 'uGather');
  var uHasLogo  = gl.getUniformLocation(prog, 'uHasLogo');

  /* ── 로고 텍스처 ───────────────────────────────
     assets/logo.png 의 알파 채널만 사용한다. */
  var LOGO_AR = 0.8564;     /* 로고 가로 / 세로 */
  var LOGO_CY = 0.055;      /* 화면 중앙 기준 로고 중심 (uv 단위) */
  var logoReady = 0.0;
  var logoLoadedAt = 0;

  var tex = gl.createTexture();
  gl.bindTexture(gl.TEXTURE_2D, tex);
  gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, 1, 1, 0, gl.RGBA, gl.UNSIGNED_BYTE,
                new Uint8Array([0, 0, 0, 0]));
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_S, gl.CLAMP_TO_EDGE);
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_T, gl.CLAMP_TO_EDGE);
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.LINEAR);
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MAG_FILTER, gl.LINEAR);
  gl.activeTexture(gl.TEXTURE0);
  gl.uniform1i(uLogo, 0);
  gl.uniform1f(uLogoY, LOGO_CY);

  var logoImg = new Image();
  logoImg.onload = function(){
    gl.bindTexture(gl.TEXTURE_2D, tex);
    gl.pixelStorei(gl.UNPACK_FLIP_Y_WEBGL, true);
    gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.RGBA, gl.UNSIGNED_BYTE, logoImg);
    LOGO_AR = logoImg.width / logoImg.height;
    logoReady = 1.0;
    logoLoadedAt = performance.now();
    layout();
  };
  logoImg.onerror = function(){ console.warn('assets/logo.png 를 불러오지 못했습니다.'); };
  logoImg.src = 'assets/logo.png';

  /* 로고 크기: 세로가 넉넉하면 높이 기준, 가로가 좁으면 폭 기준으로 맞춘다.
     어느 화면에서도 여백이 뜨거나 잘리지 않게 하는 부분. */
  function logoDims(){
    var ar = window.innerWidth / window.innerHeight;
    var byHeight = 0.70;                 /* 화면 높이의 70% */
    var byWidth  = (0.82 * ar) / LOGO_AR;/* 화면 폭의 82% */
    var lh = Math.min(byHeight, byWidth);
    return { w: lh * LOGO_AR, h: lh };
  }

  function layout(){
    var d = logoDims();
    gl.uniform2f(uLogoSize, d.w, d.h);
  }
  window.addEventListener('resize', layout);
  layout();

  /* 연기는 형태가 부드러워서 저해상도로 그려도 티가 나지 않는다.
     해상도를 낮춰 모바일 발열과 프레임 드랍을 막는다. */
  var isMobile = window.matchMedia('(max-width: 820px)').matches;
  var scale = isMobile ? 0.50 : 0.68;
  var octaves = isMobile ? 3.0 : 4.0;
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

    /* 연기가 다 올라온 뒤 로고로 응결한다 */
    var gather = 0.0;
    if(logoReady){
      var g = ((now - logoLoadedAt) / 1000 - 2.2) / 3.4;
      gather = Math.max(0.0, Math.min(1.0, g));
      gather = gather * gather * (3.0 - 2.0 * gather);
    }

    gl.uniform1f(uTime, reduced ? 12.0 : t);
    gl.uniform1f(uIgnite, reduced ? 1.0 : ignite);
    gl.uniform1f(uHasLogo, logoReady);
    gl.uniform1f(uGather, reduced ? logoReady : gather);
    gl.uniform2f(uMouse, mx, 0.0);
    gl.drawArrays(gl.TRIANGLES, 0, 3);

    if(!reduced || !logoReady) requestAnimationFrame(frame);
  }
  requestAnimationFrame(frame);
})();

/* data/shows.json 에서 다가오는 가장 이른 공연을 찾아 표시한다.
   일정을 추가하려면 shows.json 만 수정하면 된다. */
(function(){
  var box   = document.getElementById('nextLive');
  var dateEl= document.getElementById('nextLiveDate');
  var venEl = document.getElementById('nextLiveVenue');
  if(!box) return;

  var DAYS = ['SUN','MON','TUE','WED','THU','FRI','SAT'];

  fetch('data/shows.json', { cache:'no-store' })
    .then(function(r){
      if(!r.ok) throw new Error('shows.json ' + r.status);
      return r.json();
    })
    .then(function(shows){
      var today = new Date();
      today.setHours(0,0,0,0);

      var next = shows
        .filter(function(s){
          var d = new Date(s.date + 'T00:00:00');
          return !isNaN(d) && d >= today;
        })
        .sort(function(a,b){ return a.date < b.date ? -1 : 1; })[0];

      if(!next) return;   // 예정된 공연이 없으면 블록을 숨긴 채로 둔다

      var d = new Date(next.date + 'T00:00:00');
      dateEl.textContent =
        next.date.replace(/-/g, '.') + ' ' + DAYS[d.getDay()] +
        (next.time ? ' ' + next.time : '');
      venEl.textContent = next.venue || '';
      box.hidden = false;
    })
    .catch(function(err){
      console.warn('다음 공연 정보를 불러오지 못했습니다:', err.message);
    });
})();

</script>

</body>
</html>
