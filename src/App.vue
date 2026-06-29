<script setup>
import { ref, reactive, computed, onMounted, onUnmounted, nextTick, watch } from 'vue'
import gsap from 'gsap'
import { ScrollTrigger } from 'gsap/ScrollTrigger'
import Lenis from '@studio-freight/lenis'

gsap.registerPlugin(ScrollTrigger)

/* ────────────── Mock Product Data ────────────── */
const productsData = Object.freeze([
  {
    id: 1,
    name: 'Elixir Primordial',
    slug: 'elixir-primordial',
    price: '128.00',
    priceRaw: 128,
    sale_price: null,
    sku: 'ALC-ELX-001',
    stock_status: 'instock',
    categories: [{ name: 'Serums' }],
    description: 'Regenerative night serum with bakuchiol, squalane, and centella asiatica. Restores elasticity while you sleep.',
    images: [{ src: '/images/product-elixir.jpg' }]
  },
  {
    id: 2,
    name: 'Aurora Mist',
    slug: 'aurora-mist',
    price: '68.00',
    priceRaw: 68,
    sale_price: null,
    sku: 'ALC-AUR-002',
    stock_status: 'instock',
    categories: [{ name: 'Mists' }],
    description: 'Hydrating facial toner with Bulgarian rose hydrosol and tremella mushroom. Instant dewy luminosity.',
    images: [{ src: '/images/product-aurora.jpg' }]
  },
  {
    id: 3,
    name: 'Terra Mask',
    slug: 'terra-mask',
    price: '86.00',
    priceRaw: 86,
    sale_price: null,
    sku: 'ALC-TER-003',
    stock_status: 'instock',
    categories: [{ name: 'Masks' }],
    description: 'Detoxifying clay treatment with French green clay, matcha, and spirulina. Draws impurities without stripping.',
    images: [{ src: '/images/product-terra.jpg' }]
  },
  {
    id: 4,
    name: 'Solstice Oil',
    slug: 'solstice-oil',
    price: '112.00',
    priceRaw: 112,
    sale_price: null,
    sku: 'ALC-SOL-004',
    stock_status: 'instock',
    categories: [{ name: 'Oils' }],
    description: 'Lightweight facial oil with prickly pear seed, marula, and sea buckthorn. Rich in omega fatty acids.',
    images: [{ src: '/images/product-solstice.jpg' }]
  },
  {
    id: 5,
    name: 'Nocturne Balm',
    slug: 'nocturne-balm',
    price: '94.00',
    priceRaw: 94,
    sale_price: null,
    sku: 'ALC-NOC-005',
    stock_status: 'instock',
    categories: [{ name: 'Masks' }],
    description: 'Restorative overnight mask with reishi mushroom, CBD isolate, and blue tansy. Calms inflammation.',
    images: [{ src: '/images/product-nocturne.jpg' }]
  },
  {
    id: 6,
    name: 'Luminescence',
    slug: 'luminescence',
    price: '145.00',
    priceRaw: 145,
    sale_price: null,
    sku: 'ALC-LUM-006',
    stock_status: 'instock',
    categories: [{ name: 'Serums' }],
    description: 'Vitamin C brightening concentrate with Kakadu plum, ferulic acid, and niacinamide. Fades discoloration.',
    images: [{ src: '/images/product-luminescence.jpg' }]
  }
])

/* ────────────── Cart State (Simulated Pinia) ────────────── */
const cart = reactive({
  items: [],
  open: false
})

const cartCount = computed(() => cart.items.reduce((sum, item) => sum + item.quantity, 0))
const cartTotal = computed(() =>
  cart.items.reduce((sum, item) => sum + item.priceRaw * item.quantity, 0).toFixed(2)
)

function addToCart(product) {
  const existing = cart.items.find(item => item.id === product.id)
  if (existing) {
    existing.quantity++
  } else {
    cart.items.push({
      id: product.id,
      name: product.name,
      price: product.price,
      priceRaw: product.priceRaw,
      quantity: 1,
      image: product.images[0].src
    })
  }
  cart.open = true
}

function removeFromCart(productId) {
  const idx = cart.items.findIndex(item => item.id === productId)
  if (idx > -1) {
    if (cart.items[idx].quantity > 1) {
      cart.items[idx].quantity--
    } else {
      cart.items.splice(idx, 1)
    }
  }
}

function updateQuantity(productId, delta) {
  const item = cart.items.find(i => i.id === productId)
  if (!item) return
  item.quantity += delta
  if (item.quantity <= 0) {
    const idx = cart.items.indexOf(item)
    cart.items.splice(idx, 1)
  }
}

function clearCart() {
  cart.items = []
}

/* ────────────── UI State ────────────── */
const navScrolled = ref(false)
const mobileMenuOpen = ref(false)
const currentTestimonial = ref(0)
const testimonialInterval = ref(null)
const heroCanvasRef = ref(null)
const oilCanvasRef = ref(null)
const philosophySectionRef = ref(null)

const testimonials = [
  {
    text: 'After three weeks with Elixir Primordial, I stopped wearing foundation. My skin has never been this calm, this luminous. This is not skincare \u2014 this is self-respect in a bottle.',
    name: 'Elena M.',
    location: 'Lisbon'
  },
  {
    text: 'I have reactive, rosacea-prone skin. Every product I\'ve tried either irritated or did nothing. The Nocturne Balm is the first thing that actually reduced my redness. I cry happy tears.',
    name: 'Sarah K.',
    location: 'Melbourne'
  },
  {
    text: 'The ritual of applying Solstice Oil each morning has become my meditation. The scent, the texture, the way it absorbs \u2014 everything about it feels intentional and sacred.',
    name: 'Yuki T.',
    location: 'Kyoto'
  }
]

const ingredients = [
  { name: 'Prickly Pear Seed Oil', origin: 'Morocco' },
  { name: 'Marula Oil', origin: 'Namibia' },
  { name: 'Bakuchiol', origin: 'India' },
  { name: 'Centella Asiatica', origin: 'Madagascar' }
]

const scatterImages = [
  { src: '/images/scatter-1.jpg', style: { top: '-20px', right: '-30px', rotation: 6 } },
  { src: '/images/scatter-2.jpg', style: { top: '60px', left: '-40px', rotation: -4 } },
  { src: '/images/scatter-3.jpg', style: { bottom: '40px', right: '-20px', rotation: 8 } },
  { src: '/images/scatter-4.jpg', style: { bottom: '-10px', left: '-25px', rotation: -7 } },
  { src: '/images/scatter-5.jpg', style: { top: '30%', right: '-45px', rotation: 3 } },
  { src: '/images/scatter-6.jpg', style: { bottom: '25%', left: '-35px', rotation: -5 } }
]

/* ────────────── Shader Code ────────────── */

/* Twilight Sky Shader */
const skyVertSrc = `
attribute vec2 a_pos;
varying vec2 v_pos;
void main() {
  gl_Position = vec4(a_pos, 0.0, 1.0);
  v_pos = a_pos;
}`

const skyFragSrc = `
precision highp float;
varying vec2 v_pos;
uniform float u_time;
uniform vec2 u_res;
uniform float u_sunSpeed;
uniform float u_moonGlow;
uniform float u_starDensity;
uniform float u_twilight;
uniform float u_cloudSpeed;
uniform float u_birdCount;
#define PI 3.14159265359
#define TAU 6.28318530718

float hash(float n) { return fract(sin(n) * 43758.5453123); }
float hash2(vec2 p) { return fract(sin(dot(p, vec2(127.1, 311.7))) * 43758.5453); }
vec2 hash2v(vec2 p) {
  return vec2(
    fract(sin(dot(p, vec2(127.1, 311.7))) * 43758.5453),
    fract(sin(dot(p, vec2(269.5, 183.3))) * 43758.5453)
  );
}

float noise(vec2 p) {
  vec2 i = floor(p);
  vec2 f = fract(p);
  f = f * f * (3.0 - 2.0 * f);
  return mix(
    mix(hash2(i), hash2(i + vec2(1.0, 0.0)), f.x),
    mix(hash2(i + vec2(0.0, 1.0)), hash2(i + vec2(1.0, 1.0)), f.x),
    f.y
  );
}

float fbm(vec2 p, int octaves) {
  float val = 0.0;
  float amp = 0.5;
  float freq = 1.0;
  for (int i = 0; i < 15; i++) {
    if (i >= octaves) break;
    val += amp * noise(p * freq);
    freq *= 2.03;
    amp *= 0.49;
    p += vec2(1.7, 9.2);
  }
  return val;
}

float getNoise(vec3 p, float t) {
  return fbm(p.xy + fbm(p.xy + p.z + t, 3) + t, 4);
}

float vnoise(vec3 p) {
  vec3 i = floor(p);
  vec3 f = fract(p);
  vec3 u = f * f * f * (f * (f * 6.0 - 15.0) + 10.0);
  float a000 = hash(i.x + i.y * 127.1 + i.z * 311.7);
  float a100 = hash(i.x + 1.0 + i.y * 127.1 + i.z * 311.7);
  float a010 = hash(i.x + (i.y + 1.0) * 127.1 + i.z * 311.7);
  float a110 = hash(i.x + 1.0 + (i.y + 1.0) * 127.1 + i.z * 311.7);
  float a001 = hash(i.x + i.y * 127.1 + (i.z + 1.0) * 311.7);
  float a101 = hash(i.x + 1.0 + i.y * 127.1 + (i.z + 1.0) * 311.7);
  float a011 = hash(i.x + (i.y + 1.0) * 127.1 + (i.z + 1.0) * 311.7);
  float a111 = hash(i.x + 1.0 + (i.y + 1.0) * 127.1 + (i.z + 1.0) * 311.7);
  float b00 = mix(a000, a100, u.x);
  float b10 = mix(a010, a110, u.x);
  float b01 = mix(a001, a101, u.x);
  float b11 = mix(a011, a111, u.x);
  float c0 = mix(b00, b10, u.y);
  float c1 = mix(b01, b11, u.y);
  return mix(c0, c1, u.z);
}

vec4 noised(vec3 p) {
  vec3 i = floor(p);
  vec3 f = fract(p);
  vec3 u = f * f * f * (f * (f * 6.0 - 15.0) + 10.0);
  vec3 du = 30.0 * f * f * (f * (f - 2.0) + 1.0);
  float a000 = hash(i.x + i.y * 127.1 + i.z * 311.7);
  float a100 = hash(i.x + 1.0 + i.y * 127.1 + i.z * 311.7);
  float a010 = hash(i.x + (i.y + 1.0) * 127.1 + i.z * 311.7);
  float a110 = hash(i.x + 1.0 + (i.y + 1.0) * 127.1 + i.z * 311.7);
  float a001 = hash(i.x + i.y * 127.1 + (i.z + 1.0) * 311.7);
  float a101 = hash(i.x + 1.0 + i.y * 127.1 + (i.z + 1.0) * 311.7);
  float a011 = hash(i.x + (i.y + 1.0) * 127.1 + (i.z + 1.0) * 311.7);
  float a111 = hash(i.x + 1.0 + (i.y + 1.0) * 127.1 + (i.z + 1.0) * 311.7);
  vec3 va000 = vec3(a000, a000, a000), va100 = vec3(a100, a100, a100);
  vec3 va010 = vec3(a010, a010, a010), va110 = vec3(a110, a110, a110);
  vec3 va001 = vec3(a001, a001, a001), va101 = vec3(a101, a101, a101);
  vec3 va011 = vec3(a011, a011, a011), va111 = vec3(a111, a111, a111);
  float b00 = mix(a000, a100, u.x);
  float b10 = mix(a010, a110, u.x);
  float b01 = mix(a001, a101, u.x);
  float b11 = mix(a011, a111, u.x);
  float c0 = mix(b00, b10, u.y);
  float c1 = mix(b01, b11, u.y);
  float v = mix(c0, c1, u.z);
  vec3 d = mix(
    mix(mix(va000, va100, u.x), mix(va010, va110, u.x), u.y),
    mix(mix(va001, va101, u.x), mix(va011, va111, u.x), u.y),
    u.z
  );
  return vec4(v, d.x, d.y, d.z);
}

float fbm3(vec3 p, float t) {
  float v = 0.0;
  float a = 0.5;
  mat2 rot = mat2(0.866, 0.5, -0.5, 0.866);
  for (int i = 0; i < 5; i++) {
    v += a * vnoise(p + t);
    p.xy = rot * p.xy * 2.0 + vec2(1.7, 9.2);
    p.z = p.z * 2.0;
    a *= 0.49;
  }
  return v;
}

vec3 getSkyColor(vec3 rd, float t) {
  float tlt = u_twilight / (u_twilight + 1.0);
  vec3 zenith = vec3(0.55, 0.55, 0.75) * (1.0 - tlt * 0.3);
  vec3 horizon = mix(vec3(0.95, 0.75, 0.6), vec3(0.35, 0.45, 0.75), tlt);
  vec3 ground = mix(vec3(0.75, 0.55, 0.4), vec3(0.2, 0.25, 0.45), tlt);
  float horizonWidth = 0.15;
  float dist = abs(rd.y);
  float blend = smoothstep(0.0, horizonWidth, dist);
  vec3 col = mix(horizon, zenith, blend);
  if (rd.y < 0.0) {
    col = mix(ground, horizon, smoothstep(0.0, -horizonWidth, rd.y));
  }
  float upness = smoothstep(0.0, 0.5, max(rd.y, 0.0));
  col = mix(col, zenith * 1.1, upness * (1.0 - tlt * 0.5));
  return col;
}

vec3 getCloudColor(vec3 ro, vec3 rd, float t) {
  vec3 mapP(vec3 p) { return p; }
  float ct = t * u_cloudSpeed;
  vec3 col = vec3(0.0);
  float d = 0.05;
  float amax = 10.0;
  float cnt = 40.0;
  for (float i = 0.0; i < cnt; i++) {
    if (i >= amax) break;
    vec3 p = ro + rd * d;
    p.z += ct * 0.5;
    float dens = fbm3(mapP(p), ct);
    dens = smoothstep(0.3, 0.7, dens);
    vec3 cColor = mix(vec3(0.9, 0.85, 0.8), vec3(0.7, 0.65, 0.7), dens);
    float alpha = dens * 0.03;
    col = mix(col, cColor, alpha);
    d += 0.05;
  }
  return col;
}

vec3 getSunPosition(float t) {
  float angle = t * 0.15 * u_sunSpeed;
  float sunY = sin(angle);
  float sunX = cos(angle) * 0.3;
  return vec3(sunX, sunY, -1.0);
}

vec3 getMoonPosition(float t) {
  float angle = t * 0.12 * u_sunSpeed + PI;
  float moonY = sin(angle);
  float moonX = cos(angle) * 0.3;
  return vec3(moonX, moonY, -1.0);
}

vec3 getCelestialGlow(vec3 rd, vec3 pos, vec3 color, float intensity, float size) {
  float dotProduct = max(dot(rd, normalize(pos)), 0.0);
  return color * intensity * pow(dotProduct, 1.0 / size);
}

float getStar(vec3 rd, float threshold) {
  vec3 starDirection = rd * 100.0;
  vec3 cellId = floor(starDirection);
  vec3 starCenter = cellId + hash2v(cellId.xy) * 0.9 + 0.05;
  float dist = length(starDirection - starCenter);
  return (1.0 - smoothstep(0.0, 1.0, dist)) * step(threshold, hash2(cellId.xy));
}

float bird(vec2 uv, float size, float state) {
  float wing = sin(state * 5.0) * 0.1;
  float b = abs(uv.x) + wing;
  float wingShape = smoothstep(0.02, 0.0, abs(b - uv.y) - 0.01 * size);
  float body = smoothstep(0.02, 0.0, length(uv) - 0.01 * size);
  return max(wingShape, body);
}

void main() {
  vec2 uv = (v_pos * u_res / min(u_res.x, u_res.y)) * 0.5;
  float t = u_time * 0.05;
  float animT = fract(t);
  float aspect = u_res.x / u_res.y;
  vec3 rd = normalize(vec3(uv.x, uv.y, -1.0 / tan(PI * 0.25)));
  rd.x *= aspect;
  vec3 ro = vec3(0.0, 0.0, 0.0);
  vec3 skyColor = getSkyColor(rd, t);

  vec3 sunPos = getSunPosition(t);
  vec3 sunDir = normalize(sunPos);
  skyColor += getCelestialGlow(rd, sunPos, vec3(1.0, 0.85, 0.5), 0.8 * u_twilight, 0.02);

  vec3 moonPos = getMoonPosition(t);
  vec3 moonDir = normalize(moonPos);
  skyColor += getCelestialGlow(rd, moonPos, vec3(0.8, 0.9, 1.0), u_moonGlow, 0.03);

  if (moonPos.y > -0.1) {
    float starThreshold = 0.04 + 0.06 * u_starDensity;
    float star = getStar(rd, starThreshold);
    skyColor += vec3(0.9, 0.95, 1.0) * star * moonPos.y;
  }

  vec3 cloudColor = getCloudColor(ro, rd, t);
  skyColor = mix(skyColor, cloudColor, smoothstep(0.0, 1.0, cloudColor.r + cloudColor.g + cloudColor.b));

  float sunAngle = atan(sunPos.y, sunPos.x);
  rd.xy = mat2(cos(-sunAngle), sin(-sunAngle), -sin(-sunAngle), cos(-sunAngle)) * rd.xy;

  for (int i = 0; i < 6; i++) {
    if (float(i) >= u_birdCount) break;
    float birdIdx = float(i);
    float birdX = -0.5 + fract(birdIdx * 0.37 + animT * 0.1) * 2.0;
    float birdY = 0.1 + hash(birdIdx * 13.0) * 0.3;
    float birdZ = -0.8 - hash(birdIdx * 7.0) * 0.5;
    vec3 birdPos = vec3(birdX, birdY, birdZ);
    float birdSize = 0.015;
    float state = t * (0.5 + hash(birdIdx * 3.0) * 0.5) + birdIdx;
    vec3 toBird = birdPos - ro;
    float birdDist = length(toBird);
    vec3 birdDir = normalize(toBird);
    float dotProduct = dot(rd, birdDir);
    float angle = acos(clamp(dotProduct, -1.0, 1.0));
    vec2 screenPos = vec2(dotProduct, 0.0);
    vec3 birdPlane = normalize(cross(birdDir, vec3(0.0, 1.0, 0.0)));
    vec3 birdPlane2 = cross(birdDir, birdPlane);
    vec2 birdUV = vec2(dot(rd - birdDir, birdPlane), dot(rd - birdDir, birdPlane2)) / max(birdSize * birdDist, 0.001);
    float birdShape = bird(birdUV, 1.0, state);
    vec3 birdColor = vec3(0.05, 0.05, 0.05) * (1.0 - smoothstep(0.0, 1.0, birdDist));
    skyColor += birdColor * birdShape;
  }

  skyColor = mix(skyColor, skyColor * vec3(1.05, 0.95, 0.85), 0.1);
  skyColor = max(skyColor, vec3(0.0));
  skyColor = pow(skyColor, vec3(0.9));
  skyColor = skyColor / (1.0 + skyColor * 0.2);
  skyColor = smoothstep(0.0, 1.0, skyColor);

  gl_FragColor = vec4(skyColor, 1.0);
}`

/* Oil Simulation Shaders */
const oilVertSrc = `
attribute vec2 a_pos;
varying vec2 v_uv;
varying vec2 v_pos;
void main() {
  v_uv = a_pos * 0.5 + 0.5;
  v_pos = a_pos;
  gl_Position = vec4(a_pos, 0.0, 1.0);
}`

const oilSplatFragSrc = `
precision highp float;
varying vec2 v_uv;
varying vec2 v_pos;
uniform vec2 u_center;
uniform float u_radius;
uniform vec3 u_color;
uniform float u_strength;
uniform float u_seed;
uniform float u_aspect;
uniform float u_time;
uniform sampler2D u_texture;
uniform int splatCount;

float encode(float v) { return v / 0.8 + 0.5; }
float decode(float v) { return (v - 0.5) * 0.8; }
float sdCircle(vec2 p, vec2 center, float radius) {
  return length((p - center) * vec2(u_aspect, 1.0)) - radius;
}

void main() {
  vec4 raw = texture2D(u_texture, v_uv);
  float existing = decode(raw.x);
  float existingNoise = decode(raw.y);
  float dist = existing;
  float noiseField = existingNoise;

  for (int i = 0; i < 8; i++) {
    if (i >= splatCount) break;
    float fi = float(i);
    vec2 offset = vec2(
      sin(u_time * (0.3 + fi * 0.1) + fi * 2.0 + u_seed),
      cos(u_time * (0.2 + fi * 0.07) + fi * 1.5 + u_seed)
    ) * 0.1;
    float idx = fi * 3.0;
    float cX = (sin(u_seed * 7.13 + fi * 3.7) * 0.5 + 0.5) * 0.6 + 0.2;
    float cY = (cos(u_seed * 3.77 + fi * 2.3) * 0.5 + 0.5) * 0.6 + 0.2;
    vec2 center = vec2(cX + offset.x, cY + offset.y);
    float radius = (0.04 + 0.03 * sin(u_seed * 11.17 + fi * 5.1)) * u_strength;
    float circle = sdCircle(v_uv, center, radius);
    dist = min(dist, circle);
    float falloff = smoothstep(radius, 0.0, circle);
    float colorHash = fract(sin(u_seed * 13.37 + fi * 7.77) * 43758.5453);
    float noiseVal = colorHash < 0.33 ? 0.1 : (colorHash < 0.66 ? 0.2 : 0.3);
    noiseField = mix(noiseField, noiseVal, falloff * 0.5);
  }

  gl_FragColor = vec4(encode(dist), encode(noiseField), raw.z, raw.w);
}`

const oilLevelFragSrc = `
precision highp float;
varying vec2 v_uv;
uniform sampler2D u_texture;
uniform float u_delta;
uniform float u_gridScale;
uniform float u_aspect;
uniform float u_time;

float encode(float v) { return v / 0.8 + 0.5; }
float decode(float v) { return (v - 0.5) * 0.8; }

void main() {
  vec2 dx = vec2(1.0 / u_gridScale, 0.0);
  vec2 dy = vec2(0.0, 1.0 / (u_gridScale * u_aspect));
  vec4 val = texture2D(u_texture, v_uv);
  float d = decode(val.x);
  vec4 valL = texture2D(u_texture, v_uv - dx);
  vec4 valR = texture2D(u_texture, v_uv + dx);
  vec4 valU = texture2D(u_texture, v_uv + dy);
  vec4 valD = texture2D(u_texture, v_uv - dy);
  float dL = decode(valL.x);
  float dR = decode(valR.x);
  float dU = decode(valU.x);
  float dD = decode(valD.x);
  float nX = dR - dL;
  float nY = (dU - dD) / u_aspect;
  float mag = sqrt(nX * nX + nY * nY + 1e-6);
  nX /= mag;
  nY /= mag;
  vec2 advUV = v_uv + vec2(nX, nY) * u_delta * 0.8;
  vec4 adv = texture2D(u_texture, advUV);
  float advD = decode(adv.x);
  float smoothD = (dL + dR + dU + dD) * 0.25;
  float d_new = mix(advD, smoothD, 0.35);
  d_new = clamp(d_new, -0.4, 0.4);
  float t = decode(val.y);
  gl_FragColor = vec4(encode(d_new), encode(t), val.z, val.w);
}`

const oilReinitFragSrc = `
precision highp float;
varying vec2 v_uv;
uniform sampler2D u_texture;
uniform float u_gridScale;
uniform float u_aspect;

float encode(float v) { return v / 0.8 + 0.5; }
float decode(float v) { return (v - 0.5) * 0.8; }

void main() {
  vec2 dx = vec2(1.0 / u_gridScale, 0.0);
  vec2 dy = vec2(0.0, 1.0 / (u_gridScale * u_aspect));
  vec4 val = texture2D(u_texture, v_uv);
  float d = decode(val.x);
  vec4 valL = texture2D(u_texture, v_uv - dx);
  vec4 valR = texture2D(u_texture, v_uv + dx);
  vec4 valU = texture2D(u_texture, v_uv + dy);
  vec4 valD = texture2D(u_texture, v_uv - dy);
  float d_new = d + 0.1 * ((dR - 0.5) + (dL - 0.5) + (dU - 0.5) + (dD - 0.5) - 4.0 * (d - 0.5));
  gl_FragColor = vec4(encode(d_new), val.y, val.z, val.w);
}`

const oilDisplayFragSrc = `
precision highp float;
varying vec2 v_uv;
uniform sampler2D u_texture;
uniform vec2 u_res;
uniform float u_thick;
uniform float u_visc;
uniform float u_time;
uniform vec3 u_colors[6];

float hash12(vec2 p) { return fract(sin(dot(p, vec2(127.1, 311.7))) * 43758.5453); }

float noise(vec2 p) {
  vec2 i = floor(p);
  vec2 f = fract(p);
  f = f * f * (3.0 - 2.0 * f);
  return mix(
    mix(hash12(i), hash12(i + vec2(1.0, 0.0)), f.x),
    mix(hash12(i + vec2(0.0, 1.0)), hash12(i + vec2(1.0, 1.0)), f.x),
    f.y
  );
}

void main() {
  vec4 raw = texture2D(u_texture, v_uv);
  float dist = (raw.x - 0.5) * 0.8;
  float noiseField = (raw.y - 0.5) * 0.8;
  float ratio = u_res.x / u_res.y;
  vec2 uv = vec2(v_uv.x * ratio, v_uv.y);
  float normDist = 1.0 - smoothstep(-0.05, 0.05, dist);
  vec3 displayColor = vec3(0.0);
  int colorIdx = 0;
  if (normDist > 0.01) {
    colorIdx = int(mod(noiseField * 10.0 + 100.0, 6.0));
    if (colorIdx == 0) displayColor = u_colors[0];
    else if (colorIdx == 1) displayColor = u_colors[1];
    else if (colorIdx == 2) displayColor = u_colors[2];
    else if (colorIdx == 3) displayColor = u_colors[3];
    else if (colorIdx == 4) displayColor = u_colors[4];
    else displayColor = u_colors[5];
  } else {
    displayColor = vec3(0.93, 0.92, 0.88);
  }
  float edge = 1.0 - smoothstep(0.0, 0.05, abs(dist));
  float thick = 0.5 + 0.5 * noise(uv * 5.0 + u_time * 0.1) * (1.0 - edge);
  float specular = 0.3 + 0.4 * noise(uv * 8.0 + vec2(u_time * 0.05, u_time * 0.03));
  vec3 highlight = vec3(thick * 0.9, thick * 0.8, thick * 0.7);
  vec3 finalColor = mix(displayColor * 0.7, displayColor, thick);
  finalColor = mix(finalColor, displayColor * 1.2, specular * edge);
  finalColor += vec3(1.0, 0.98, 0.95) * highlight * 0.3;
  float shadow = smoothstep(-0.05, 0.05, dist);
  finalColor *= mix(1.0, 0.85, shadow);
  gl_FragColor = vec4(finalColor, 1.0);
}`

/* ────────────── WebGL Helpers ────────────── */
function compileShader(gl, type, src) {
  const s = gl.createShader(type)
  gl.shaderSource(s, src)
  gl.compileShader(s)
  if (!gl.getShaderParameter(s, gl.COMPILE_STATUS)) {
    console.error('Shader compile error:', gl.getShaderInfoLog(s))
  }
  return s
}

function createProgram(gl, vsSrc, fsSrc) {
  const prog = gl.createProgram()
  gl.attachShader(prog, compileShader(gl, gl.VERTEX_SHADER, vsSrc))
  gl.attachShader(prog, compileShader(gl, gl.FRAGMENT_SHADER, fsSrc))
  gl.linkProgram(prog)
  if (!gl.getLinkParameter(prog, gl.LINK_STATUS)) {
    console.error('Program link error:', gl.getProgramInfoLog(prog))
  }
  return prog
}

function bindQuad(gl, prog) {
  const buf = gl.createBuffer()
  gl.bindBuffer(gl.ARRAY_BUFFER, buf)
  gl.bufferData(gl.ARRAY_BUFFER, new Float32Array([-1, -1, 1, -1, -1, 1, 1, 1]), gl.STATIC_DRAW)
  const aPos = gl.getAttribLocation(prog, 'a_pos')
  gl.enableVertexAttribArray(aPos)
  gl.vertexAttribPointer(aPos, 2, gl.FLOAT, false, 0, 0)
}

function createFBO(gl, w, h) {
  const data = new Float32Array(w * h * 4)
  const tex = gl.createTexture()
  gl.bindTexture(gl.TEXTURE_2D, tex)
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.LINEAR)
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MAG_FILTER, gl.LINEAR)
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_S, gl.CLAMP_TO_EDGE)
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_T, gl.CLAMP_TO_EDGE)
  gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, w, h, 0, gl.RGBA, gl.FLOAT, data)
  const fbo = gl.createFramebuffer()
  gl.bindFramebuffer(gl.FRAMEBUFFER, fbo)
  gl.framebufferTexture2D(gl.FRAMEBUFFER, gl.COLOR_ATTACHMENT0, gl.TEXTURE_2D, tex, 0)
  gl.bindFramebuffer(gl.FRAMEBUFFER, null)
  return { tex, fbo, w, h }
}

/* ────────────── Init Twilight Sky ────────────── */
function initTwilightSky(canvas) {
  const gl = canvas.getContext('webgl', { alpha: false, antialias: false, preserveDrawingBuffer: false })
  if (!gl) return null
  const prog = createProgram(gl, skyVertSrc, skyFragSrc)
  gl.useProgram(prog)
  bindQuad(gl, prog)

  const u = {
    time: gl.getUniformLocation(prog, 'u_time'),
    res: gl.getUniformLocation(prog, 'u_res'),
    sunSpeed: gl.getUniformLocation(prog, 'u_sunSpeed'),
    moonGlow: gl.getUniformLocation(prog, 'u_moonGlow'),
    starDensity: gl.getUniformLocation(prog, 'u_starDensity'),
    twilight: gl.getUniformLocation(prog, 'u_twilight'),
    cloudSpeed: gl.getUniformLocation(prog, 'u_cloudSpeed'),
    birdCount: gl.getUniformLocation(prog, 'u_birdCount')
  }

  let time = 0
  const state = { sunSpeed: 0.5, moonGlow: 1.0, starDensity: 0.5, twilight: 1.0, cloudSpeed: 0.15, birdCount: 3.0 }
  let running = true
  let rafId = null

  function resize() {
    const w = canvas.clientWidth
    const h = canvas.clientHeight
    if (canvas.width !== w || canvas.height !== h) {
      canvas.width = w
      canvas.height = h
      gl.viewport(0, 0, w, h)
    }
  }

  function render(now) {
    if (!running) return
    resize()
    time = now * 0.001
    gl.uniform1f(u.time, time)
    gl.uniform2f(u.res, canvas.width, canvas.height)
    gl.uniform1f(u.sunSpeed, state.sunSpeed)
    gl.uniform1f(u.moonGlow, state.moonGlow)
    gl.uniform1f(u.starDensity, state.starDensity)
    gl.uniform1f(u.twilight, state.twilight)
    gl.uniform1f(u.cloudSpeed, state.cloudSpeed)
    gl.uniform1f(u.birdCount, state.birdCount)
    gl.drawArrays(gl.TRIANGLE_STRIP, 0, 4)
    rafId = requestAnimationFrame(render)
  }

  window.addEventListener('resize', resize)
  rafId = requestAnimationFrame(render)

  return {
    updateParam(key, value) { state[key] = value },
    pause() { running = false; cancelAnimationFrame(rafId) },
    resume() { if (!running) { running = true; rafId = requestAnimationFrame(render) } }
  }
}

/* ────────────── Init Oil Simulation ────────────── */
function initOilSimulation(canvas, colorPalette) {
  const gl = canvas.getContext('webgl', { alpha: false, preserveDrawingBuffer: true })
  if (!gl) {
    /* OES_texture_float fallback */
    const ext = gl && gl.getExtension('OES_texture_float')
    if (!ext) return null
  }

  const progSplat = createProgram(gl, oilVertSrc, oilSplatFragSrc)
  const progLevel = createProgram(gl, oilVertSrc, oilLevelFragSrc)
  const progReinit = createProgram(gl, oilVertSrc, oilReinitFragSrc)
  const progDisplay = createProgram(gl, oilVertSrc, oilDisplayFragSrc)

  const simScale = 0.5
  let sW, sH, sIM
  let mainPtr = 0
  let reinitPtr = 0
  let time = 0
  const colors = colorPalette
  let running = true
  let rafId = null
  let mainPair = []
  let reinitPair = []

  function resize() {
    const w = canvas.clientWidth
    const h = canvas.clientHeight
    if (canvas.width !== w || canvas.height !== h) {
      canvas.width = w
      canvas.height = h
      sW = Math.floor(w * simScale)
      sH = Math.floor(h * simScale)
      sIM = Math.max(sW, sH)
      mainPair = [createFBO(gl, sIM, sIM), createFBO(gl, sIM, sIM)]
      reinitPair = [createFBO(gl, sIM, sIM), createFBO(gl, sIM, sIM)]
      mainPtr = 0
      reinitPtr = 0
      gl.viewport(0, 0, w, h)
    }
  }

  function doRender(now) {
    if (!running) return
    time = now * 0.001
    resize()
    if (sIM === undefined) { rafId = requestAnimationFrame(doRender); return }

    let src = mainPair[mainPtr]
    let dst = mainPtr === 0 ? mainPair[1] : mainPair[0]

    /* Pass 1: Splat */
    gl.useProgram(progSplat)
    gl.bindFramebuffer(gl.FRAMEBUFFER, src.fbo)
    gl.viewport(0, 0, sIM, sIM)
    gl.activeTexture(gl.TEXTURE0)
    gl.bindTexture(gl.TEXTURE_2D, src.tex)
    gl.uniform1f(gl.getUniformLocation(progSplat, 'u_strength'), 0.8)
    gl.uniform1f(gl.getUniformLocation(progSplat, 'u_seed'), 42.0)
    gl.uniform1f(gl.getUniformLocation(progSplat, 'u_aspect'), sW / sH)
    gl.uniform1f(gl.getUniformLocation(progSplat, 'u_time'), time)
    gl.uniform1i(gl.getUniformLocation(progSplat, 'u_texture'), 0)
    gl.uniform1i(gl.getUniformLocation(progSplat, 'splatCount'), 6)
    bindQuad(gl, progSplat)
    gl.drawArrays(gl.TRIANGLE_STRIP, 0, 4)

    /* Pass 2: Level Set */
    gl.useProgram(progLevel)
    gl.bindFramebuffer(gl.FRAMEBUFFER, dst.fbo)
    gl.viewport(0, 0, sIM, sIM)
    gl.activeTexture(gl.TEXTURE0)
    gl.bindTexture(gl.TEXTURE_2D, src.tex)
    gl.uniform1f(gl.getUniformLocation(progLevel, 'u_delta'), 0.016)
    gl.uniform1f(gl.getUniformLocation(progLevel, 'u_gridScale'), sIM)
    gl.uniform1f(gl.getUniformLocation(progLevel, 'u_aspect'), sW / sH)
    gl.uniform1f(gl.getUniformLocation(progLevel, 'u_time'), time)
    gl.uniform1i(gl.getUniformLocation(progLevel, 'u_texture'), 0)
    bindQuad(gl, progLevel)
    gl.drawArrays(gl.TRIANGLE_STRIP, 0, 4)
    mainPtr = mainPtr === 0 ? 1 : 0

    /* Pass 3: Reinitialize (6 iterations) */
    for (let r = 0; r < 6; r++) {
      gl.useProgram(progReinit)
      let rSrc = mainPair[mainPtr]
      let rDst = reinitPtr === 0 ? reinitPair[1] : reinitPair[0]
      gl.bindFramebuffer(gl.FRAMEBUFFER, rDst.fbo)
      gl.viewport(0, 0, sIM, sIM)
      gl.activeTexture(gl.TEXTURE0)
      gl.bindTexture(gl.TEXTURE_2D, rSrc.tex)
      gl.uniform1f(gl.getUniformLocation(progReinit, 'u_gridScale'), sIM)
      gl.uniform1f(gl.getUniformLocation(progReinit, 'u_aspect'), sW / sH)
      gl.uniform1i(gl.getUniformLocation(progReinit, 'u_texture'), 0)
      bindQuad(gl, progReinit)
      gl.drawArrays(gl.TRIANGLE_STRIP, 0, 4)
      mainPtr = mainPtr === 0 ? 1 : 0
      reinitPtr = reinitPtr === 0 ? 1 : 0
    }

    /* Pass 4: Display */
    gl.useProgram(progDisplay)
    gl.bindFramebuffer(gl.FRAMEBUFFER, null)
    gl.viewport(0, 0, canvas.width, canvas.height)
    let dispSrc = mainPair[mainPtr]
    gl.activeTexture(gl.TEXTURE0)
    gl.bindTexture(gl.TEXTURE_2D, dispSrc.tex)
    gl.uniform2f(gl.getUniformLocation(progDisplay, 'u_res'), canvas.width, canvas.height)
    gl.uniform1f(gl.getUniformLocation(progDisplay, 'u_thick'), 1.0)
    gl.uniform1f(gl.getUniformLocation(progDisplay, 'u_visc'), 0.5)
    gl.uniform1f(gl.getUniformLocation(progDisplay, 'u_time'), time)
    gl.uniform1i(gl.getUniformLocation(progDisplay, 'u_texture'), 0)
    for (let i = 0; i < 6; i++) {
      gl.uniform3f(gl.getUniformLocation(progDisplay, 'u_colors[' + i + ']'), colors[i][0], colors[i][1], colors[i][2])
    }
    bindQuad(gl, progDisplay)
    gl.drawArrays(gl.TRIANGLE_STRIP, 0, 4)

    rafId = requestAnimationFrame(doRender)
  }

  window.addEventListener('resize', resize)
  resize()
  rafId = requestAnimationFrame(doRender)

  return {
    pause() { running = false; cancelAnimationFrame(rafId) },
    resume() { if (!running) { running = true; rafId = requestAnimationFrame(doRender) } }
  }
}

/* ────────────── Lifecycle ────────────── */
let skyRenderer = null
let oilRenderer = null
let lenis = null

onMounted(() => {
  /* Init Lenis smooth scroll */
  lenis = new Lenis({ lerp: 0.08, smoothWheel: true })
  lenis.on('scroll', ScrollTrigger.update)
  function raf(time) {
    lenis.raf(time)
    requestAnimationFrame(raf)
  }
  requestAnimationFrame(raf)

  /* Init Twilight Sky */
  if (heroCanvasRef.value) {
    skyRenderer = initTwilightSky(heroCanvasRef.value)
    heroCanvasRef.value._renderer = skyRenderer
  }

  /* Init Oil Simulation */
  if (oilCanvasRef.value) {
    const oilColors = [
      [0.65, 0.35, 0.22],
      [0.28, 0.45, 0.35],
      [0.76, 0.82, 0.60],
      [0.55, 0.42, 0.35],
      [0.83, 0.72, 0.45],
      [0.42, 0.55, 0.50]
    ]
    oilRenderer = initOilSimulation(oilCanvasRef.value, oilColors)
    oilCanvasRef.value._renderer = oilRenderer
  }

  /* IntersectionObserver for shader pause/resume */
  const shaderObserver = new IntersectionObserver((entries) => {
    for (const entry of entries) {
      const canvas = entry.target
      if (entry.isIntersecting) {
        canvas._renderer?.resume()
      } else {
        canvas._renderer?.pause()
      }
    }
  }, { threshold: 0.1 })

  if (heroCanvasRef.value) shaderObserver.observe(heroCanvasRef.value)
  if (oilCanvasRef.value) shaderObserver.observe(oilCanvasRef.value)

  /* Nav scroll detection */
  const handleScroll = () => {
    navScrolled.value = window.scrollY > window.innerHeight * 0.5
  }
  window.addEventListener('scroll', handleScroll, { passive: true })

  /* GSAP entrance animations */
  nextTick(() => {
    /* Hero entrance */
    gsap.fromTo('.hero-headline', { y: 40, opacity: 0 }, { y: 0, opacity: 1, duration: 1.2, ease: 'cubic-bezier(0.25, 0.1, 0.25, 1)', delay: 0.5 })
    gsap.fromTo('.hero-sub', { y: 40, opacity: 0 }, { y: 0, opacity: 1, duration: 1.2, ease: 'cubic-bezier(0.25, 0.1, 0.25, 1)', delay: 0.8 })
    gsap.fromTo('.hero-cta', { y: 40, opacity: 0 }, { y: 0, opacity: 1, duration: 1.2, ease: 'cubic-bezier(0.25, 0.1, 0.25, 1)', delay: 1.0 })
    gsap.fromTo('.scroll-indicator', { opacity: 0 }, { opacity: 0.5, duration: 1, delay: 1.5 })

    /* Philosophy section */
    gsap.fromTo('.philosophy-image', { x: -60, opacity: 0 }, {
      x: 0, opacity: 1, duration: 1, ease: 'power3.out',
      scrollTrigger: { trigger: '.philosophy-section', start: 'top 75%', toggleActions: 'play none none none' }
    })
    gsap.fromTo('.philosophy-label', { y: 30, opacity: 0 }, {
      y: 0, opacity: 1, duration: 0.8, ease: 'power3.out',
      scrollTrigger: { trigger: '.philosophy-section', start: 'top 75%', toggleActions: 'play none none none' }
    })
    gsap.fromTo('.philosophy-headline', { y: 30, opacity: 0 }, {
      y: 0, opacity: 1, duration: 0.8, ease: 'power3.out', delay: 0.15,
      scrollTrigger: { trigger: '.philosophy-section', start: 'top 75%', toggleActions: 'play none none none' }
    })
    gsap.fromTo('.philosophy-body', { y: 30, opacity: 0 }, {
      y: 0, opacity: 1, duration: 0.8, ease: 'power3.out', stagger: 0.15,
      scrollTrigger: { trigger: '.philosophy-section', start: 'top 75%', toggleActions: 'play none none none' }
    })

    /* Scatter photos */
    const scatterPhotos = gsap.utils.toArray('.scatter-photo')
    if (scatterPhotos.length > 0) {
      const tl = gsap.timeline({
        scrollTrigger: { trigger: '.philosophy-section', start: 'top 80%', end: 'center center', scrub: 1 }
      })
      scatterPhotos.forEach((el) => {
        const finalRot = gsap.getProperty(el, 'rotation')
        const direction = Math.atan2(
          el.offsetTop + el.offsetHeight / 2 - window.innerHeight / 2,
          el.offsetLeft + el.offsetWidth / 2 - window.innerWidth / 2
        )
        const distance = 150 + Math.random() * 150
        const startX = Math.cos(direction) * distance
        const startY = Math.sin(direction) * distance
        tl.from(el, { x: startX, y: startY, rotation: finalRot + (Math.random() * 20 - 10), opacity: 0, ease: 'none' }, 0)
      })
    }

    /* Product grid */
    gsap.fromTo('.product-card', { y: 60, opacity: 0 }, {
      y: 0, opacity: 1, duration: 0.8, stagger: 0.1, ease: 'power3.out',
      scrollTrigger: { trigger: '.collection-section', start: 'top 80%', toggleActions: 'play none none none' }
    })

    /* Ingredient showcase */
    gsap.fromTo('.oil-canvas-wrap', { opacity: 0 }, {
      opacity: 1, duration: 1.5,
      scrollTrigger: { trigger: '.ingredient-section', start: 'top 70%', toggleActions: 'play none none none' }
    })
    gsap.fromTo('.ingredient-text > *', { x: -40, opacity: 0 }, {
      x: 0, opacity: 1, duration: 0.8, stagger: 0.12, ease: 'power3.out',
      scrollTrigger: { trigger: '.ingredient-section', start: 'top 70%', toggleActions: 'play none none none' }
    })

    /* Testimonials */
    gsap.fromTo('.testimonial-block', { y: 40, opacity: 0 }, {
      y: 0, opacity: 1, duration: 1, ease: 'power3.out',
      scrollTrigger: { trigger: '.testimonials-section', start: 'top 75%', toggleActions: 'play none none none' }
    })

    /* Newsletter */
    gsap.fromTo('.newsletter-content > *', { y: 30, opacity: 0 }, {
      y: 0, opacity: 1, duration: 0.8, stagger: 0.15, ease: 'power3.out',
      scrollTrigger: { trigger: '.newsletter-section', start: 'top 80%', toggleActions: 'play none none none' }
    })

    /* Footer */
    gsap.fromTo('.footer-column', { y: 20, opacity: 0 }, {
      y: 0, opacity: 1, duration: 0.6, stagger: 0.1, ease: 'power2.out',
      scrollTrigger: { trigger: '.footer-section', start: 'top 90%', toggleActions: 'play none none none' }
    })
  })

  /* Testimonial auto-advance */
  testimonialInterval.value = setInterval(() => {
    currentTestimonial.value = (currentTestimonial.value + 1) % testimonials.length
  }, 6000)
})

onUnmounted(() => {
  if (testimonialInterval.value) clearInterval(testimonialInterval.value)
  if (skyRenderer) skyRenderer.pause()
  if (oilRenderer) oilRenderer.pause()
})

/* ────────────── Scroll to section ────────────── */
function scrollToSection(id) {
  const el = document.querySelector(id)
  if (el && lenis) {
    lenis.scrollTo(el, { offset: -72 })
    mobileMenuOpen.value = false
  }
}

/* ────────────── Cart focus trap ────────────── */
function onCartKeydown(e) {
  if (e.key === 'Escape') {
    cart.open = false
  }
}

function onCartBackdropClick() {
  cart.open = false
}
</script>

<template>
  <div id="app-wrapper">
    <!-- Navigation -->
    <header
      class="fixed top-0 left-0 right-0 z-50 transition-all duration-500"
      :class="navScrolled ? 'bg-[rgba(245,240,232,0.92)] backdrop-blur-[20px]' : 'bg-transparent'"
      style="height: 72px;"
    >
      <nav class="flex items-center justify-between h-full max-w-[1400px] mx-auto px-4 md:px-8" aria-label="Main navigation">
        <!-- Brand -->
        <a
          href="#"
          class="font-display text-[13px] tracking-[0.18em] uppercase"
          style="color: var(--alchemy-forest);"
          @click.prevent="scrollToSection('#hero')"
        >
          SERUMA
        </a>

        <!-- Desktop Nav Links -->
        <div class="hidden md:flex items-center gap-8">
          <a
            v-for="link in [{label:'Shop',href:'#collection'},{label:'Philosophy',href:'#philosophy'},{label:'Ingredients',href:'#ingredients'},{label:'Ritual',href:'#testimonials'}]"
            :key="link.label"
            :href="link.href"
            class="relative text-[13px] uppercase tracking-[0.08em] font-body font-normal transition-colors duration-300 group"
            style="color: var(--alchemy-forest);"
            @click.prevent="scrollToSection(link.href)"
          >
            {{ link.label }}
            <span
              class="absolute bottom-0 left-0 w-full h-[1px] scale-x-0 group-hover:scale-x-100 transition-transform duration-[400ms] origin-left"
              style="background-color: var(--alchemy-terracotta);"
            />
          </a>
        </div>

        <!-- Right: Cart + Mobile Menu -->
        <div class="flex items-center gap-4">
          <button
            class="text-[13px] uppercase tracking-[0.08em] font-body font-normal relative"
            style="color: var(--alchemy-forest);"
            aria-label="Shopping cart"
            :aria-expanded="cart.open"
            aria-controls="cart-panel"
            @click="cart.open = !cart.open"
          >
            Cart
            <sup
              v-if="cartCount > 0"
              class="inline-flex items-center justify-center w-5 h-5 rounded-full text-[10px] ml-1 transition-transform duration-300"
              style="background-color: var(--alchemy-terracotta); color: var(--alchemy-cream);"
              aria-live="polite"
              aria-atomic="true"
            >
              {{ cartCount }}
            </sup>
          </button>

          <!-- Mobile hamburger -->
          <button
            class="md:hidden flex flex-col gap-[5px] p-2"
            aria-label="Toggle mobile menu"
            :aria-expanded="mobileMenuOpen"
            @click="mobileMenuOpen = !mobileMenuOpen"
          >
            <span
              class="block w-5 h-[1.5px] transition-all duration-300"
              :class="mobileMenuOpen ? 'rotate-45 translate-y-[6.5px]' : ''"
              style="background-color: var(--alchemy-forest);"
            />
            <span
              class="block w-5 h-[1.5px] transition-all duration-300"
              :class="mobileMenuOpen ? 'opacity-0' : ''"
              style="background-color: var(--alchemy-forest);"
            />
            <span
              class="block w-5 h-[1.5px] transition-all duration-300"
              :class="mobileMenuOpen ? '-rotate-45 -translate-y-[6.5px]' : ''"
              style="background-color: var(--alchemy-forest);"
            />
          </button>
        </div>
      </nav>

      <!-- Mobile Menu Overlay -->
      <div
        v-show="mobileMenuOpen"
        class="fixed inset-0 top-[72px] z-40 flex flex-col items-center justify-center gap-8 md:hidden transition-opacity duration-300"
        style="background-color: var(--alchemy-cream);"
      >
        <a
          v-for="link in [{label:'Shop',href:'#collection'},{label:'Philosophy',href:'#philosophy'},{label:'Ingredients',href:'#ingredients'},{label:'Ritual',href:'#testimonials'}]"
          :key="link.label"
          :href="link.href"
          class="font-display text-2xl tracking-[0.02em]"
          style="color: var(--alchemy-ink);"
          @click.prevent="scrollToSection(link.href)"
        >
          {{ link.label }}
        </a>
      </div>
    </header>

    <!-- Hero Section -->
    <section id="hero" class="relative w-full min-h-[100dvh] overflow-hidden">
      <canvas
        ref="heroCanvasRef"
        class="absolute inset-0 w-full h-full"
        role="img"
        aria-label="Animated twilight sky with drifting clouds, celestial bodies, and birds"
      />
      <div class="absolute inset-0 z-10 flex flex-col justify-end" style="padding: 0 0 10vh 5vw;">
        <h1
          class="hero-headline font-display text-[36px] md:text-[48px] lg:text-[72px] leading-[1.1] tracking-[-0.02em] max-w-[600px] opacity-0"
          style="color: var(--alchemy-ink); text-shadow: 0 2px 30px rgba(28, 43, 36, 0.3);"
        >
          <span class="block">Where Science</span>
          <span class="block">Meets the Sacred</span>
        </h1>
        <p
          class="hero-sub font-body text-base md:text-lg font-light max-w-[480px] mt-6 leading-[1.7] opacity-0"
          style="color: var(--alchemy-forest);"
        >
          Botanical formulations born from ancient alchemical wisdom, refined through modern dermatological science. Each product is a ritual of transformation.
        </p>
        <a
          href="#collection"
          class="hero-cta inline-flex items-center gap-2 mt-8 text-[13px] uppercase tracking-[0.12em] font-body font-normal opacity-0 group transition-colors duration-300"
          style="color: var(--alchemy-terracotta);"
          @click.prevent="scrollToSection('#collection')"
        >
          Discover the Collection
          <svg class="w-4 h-4 transition-transform duration-300 group-hover:translate-x-1" fill="none" stroke="currentColor" viewBox="0 0 24 24" stroke-width="1.5">
            <path stroke-linecap="round" stroke-linejoin="round" d="M17.25 8.25L21 12m0 0l-3.75 3.75M21 12H3" />
          </svg>
        </a>
      </div>
      <div
        class="scroll-indicator absolute bottom-8 left-1/2 -translate-x-1/2 z-10 flex flex-col items-center gap-2 opacity-0"
      >
        <span class="text-[10px] uppercase tracking-[0.2em] font-body font-normal" style="color: var(--alchemy-forest);">Scroll</span>
        <div class="w-[1px] h-8 bg-current opacity-40 animate-pulse" style="color: var(--alchemy-forest);" />
      </div>
    </section>

    <!-- Brand Philosophy Section -->
    <section
      id="philosophy"
      ref="philosophySectionRef"
      class="philosophy-section relative overflow-hidden"
      style="background-color: var(--alchemy-cream); padding: var(--space-xxl) var(--space-md);"
    >
      <div class="max-w-[1400px] mx-auto flex flex-col lg:flex-row gap-12 lg:gap-0">
        <!-- Left: Image Column -->
        <div class="w-full lg:w-[40%] relative">
          <div class="philosophy-image relative lg:mt-[var(--space-lg)]" style="aspect-ratio: 3/4;">
            <img
              src="/images/philosophy-main.jpg"
              alt="Hands holding dried botanical ingredients and an amber glass apothecary bottle"
              class="w-full h-full object-cover"
              style="border-radius: 4px;"
              loading="lazy"
            >
            <!-- Scattered photos (hidden on mobile) -->
            <div class="hidden lg:block">
              <img
                v-for="(img, idx) in scatterImages"
                :key="idx"
                :src="img.src"
                :alt="`Scattered detail photo ${idx + 1}`"
                class="scatter-photo absolute w-20 h-20 object-cover shadow-[0_4px_20px_rgba(0,0,0,0.08)]"
                :style="{
                  top: img.style.top,
                  left: img.style.left,
                  right: img.style.right,
                  bottom: img.style.bottom,
                  transform: `rotate(${img.style.rotation}deg)`,
                  borderRadius: '4px',
                  zIndex: idx % 5 + 1
                }"
                loading="lazy"
              >
            </div>
          </div>
        </div>

        <!-- Right: Text Column -->
        <div class="w-full lg:w-[60%] lg:pl-[var(--space-lg)] flex flex-col justify-center">
          <span
            class="philosophy-label text-[12px] uppercase tracking-[0.12em] font-body font-normal mb-8"
            style="color: var(--alchemy-terracotta);"
          >
            Our Philosophy
          </span>
          <h2
            class="philosophy-headline font-display text-[32px] md:text-[48px] leading-[1.2] tracking-[-0.01em] mb-8"
            style="color: var(--alchemy-ink);"
          >
            <span class="block">We believe skincare</span>
            <span class="block">is a sacred practice</span>
          </h2>
          <p
            class="philosophy-body font-body text-base font-light leading-[1.7] mb-6"
            style="color: var(--alchemy-forest);"
          >
            Every formula begins with a question: what does the skin truly need? Our apothecary draws from centuries of botanical knowledge \u2014 adaptogenic roots, cold-pressed seed oils, mineral-rich clays \u2014 each ingredient chosen not for trend, but for proven cellular affinity. We source directly from small-batch growers across six continents.
          </p>
          <p
            class="philosophy-body font-body text-base font-light leading-[1.7] mb-8"
            style="color: var(--alchemy-forest);"
          >
            No synthetic fragrances. No petroleum derivatives. No compromise. Just the pure intelligence of plants, preserved through minimal processing and delivered in bioavailable forms your skin recognizes as nourishment.
          </p>
          <a
            href="#"
            class="inline-flex items-center gap-2 text-[13px] uppercase tracking-[0.12em] font-body font-normal group transition-colors duration-300 w-fit"
            style="color: var(--alchemy-terracotta);"
          >
            Read Our Story
            <svg class="w-4 h-4 transition-transform duration-300 group-hover:translate-x-1" fill="none" stroke="currentColor" viewBox="0 0 24 24" stroke-width="1.5">
              <path stroke-linecap="round" stroke-linejoin="round" d="M17.25 8.25L21 12m0 0l-3.75 3.75M21 12H3" />
            </svg>
          </a>
        </div>
      </div>
    </section>

    <!-- Featured Collection Section -->
    <section
      id="collection"
      class="collection-section"
      style="background-color: var(--alchemy-cream); padding: var(--space-xl) var(--space-md);"
    >
      <div class="max-w-[1400px] mx-auto">
        <div class="text-center mb-16">
          <span
            class="text-[12px] uppercase tracking-[0.12em] font-body font-normal block mb-4"
            style="color: var(--alchemy-terracotta);"
          >
            The Collection
          </span>
          <h2
            class="font-display text-[32px] md:text-[48px] leading-[1.2] tracking-[-0.01em] mb-4"
            style="color: var(--alchemy-ink);"
          >
            Formulas for Transformation
          </h2>
          <p
            class="font-body text-base font-light max-w-[560px] mx-auto"
            style="color: var(--alchemy-forest);"
          >
            Each product is handcrafted in small batches at our atelier in Provence.
          </p>
        </div>

        <!-- Product Grid -->
        <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3" style="gap: var(--space-md);">
          <article
            v-for="product in productsData"
            :key="product.id"
            v-memo="[product]"
            class="product-card group"
          >
            <div class="overflow-hidden" style="aspect-ratio: 1/1;">
              <img
                :src="product.images[0].src"
                :alt="product.name"
                class="w-full h-full object-cover transition-transform duration-[600ms] ease-out group-hover:scale-[1.03]"
                loading="lazy"
              >
            </div>
            <div class="pt-4">
              <h3 class="font-body text-base font-medium" style="color: var(--alchemy-ink);">
                {{ product.name }}
              </h3>
              <p class="font-body text-sm font-normal mt-1" style="color: var(--alchemy-forest); opacity: 0.7;">
                ${{ product.price }}
              </p>
              <p
                class="font-body text-sm font-light mt-2 line-clamp-2"
                style="color: var(--alchemy-forest); line-height: 1.6;"
              >
                {{ product.description }}
              </p>
              <button
                class="w-full mt-4 text-[12px] uppercase tracking-[0.1em] font-body font-normal py-3 transition-all duration-300 hover:text-[var(--alchemy-cream)]"
                style="background: transparent; border: 1px solid var(--alchemy-forest); color: var(--alchemy-forest);"
                @mouseenter="$event.target.style.backgroundColor = 'var(--alchemy-forest)'"
                @mouseleave="$event.target.style.backgroundColor = 'transparent'"
                @click="addToCart(product)"
              >
                Add to Cart
              </button>
            </div>
          </article>
        </div>
      </div>
    </section>

    <!-- Ingredient Showcase Section -->
    <section
      id="ingredients"
      class="ingredient-section overflow-hidden"
      style="background: linear-gradient(180deg, var(--alchemy-cream) 0%, #EDE7DC 100%); padding: var(--space-xxl) var(--space-md);"
    >
      <div class="max-w-[1400px] mx-auto flex flex-col lg:flex-row gap-12 lg:gap-[var(--space-lg)]">
        <!-- Left: Text -->
        <div class="ingredient-text w-full lg:w-[45%] flex flex-col justify-center">
          <span
            class="text-[12px] uppercase tracking-[0.12em] font-body font-normal mb-6"
            style="color: var(--alchemy-terracotta);"
          >
            The Seruma
          </span>
          <h2
            class="font-display text-[32px] md:text-[48px] leading-[1.2] tracking-[-0.01em] mb-6"
            style="color: var(--alchemy-ink);"
          >
            <span class="block">Watch transformation</span>
            <span class="block">in real time</span>
          </h2>
          <p
            class="font-body text-base font-light leading-[1.7] mb-8"
            style="color: var(--alchemy-forest);"
          >
            Our proprietary cold-infusion process slowly releases active compounds from raw botanicals over 40 days. The result: oils that retain their full spectrum of vitamins, antioxidants, and fatty acids \u2014 no heat degradation, no solvent residue. This is the art of patience.
          </p>
          <div class="flex flex-col gap-4" style="margin-top: var(--space-lg);">
            <div
              v-for="ing in ingredients"
              :key="ing.name"
              class="flex items-center gap-3 group cursor-default"
            >
              <span
                class="block w-[6px] h-[6px] rounded-full transition-transform duration-300 group-hover:scale-[1.67]"
                style="background-color: var(--alchemy-terracotta);"
              />
              <span class="font-body text-sm font-normal" style="color: var(--alchemy-forest);">
                {{ ing.name }} \u2014 {{ ing.origin }}
              </span>
            </div>
          </div>
          <a
            href="#"
            class="inline-flex items-center gap-2 mt-8 text-[13px] uppercase tracking-[0.12em] font-body font-normal group transition-colors duration-300 w-fit"
            style="color: var(--alchemy-terracotta);"
          >
            Explore Our Process
            <svg class="w-4 h-4 transition-transform duration-300 group-hover:translate-x-1" fill="none" stroke="currentColor" viewBox="0 0 24 24" stroke-width="1.5">
              <path stroke-linecap="round" stroke-linejoin="round" d="M17.25 8.25L21 12m0 0l-3.75 3.75M21 12H3" />
            </svg>
          </a>
        </div>

        <!-- Right: Oil Canvas -->
        <div class="oil-canvas-wrap w-full lg:w-[55%]">
          <div class="relative w-full h-[50vh] lg:h-[70vh]">
            <canvas
              ref="oilCanvasRef"
              class="absolute inset-0 w-full h-full"
              style="border-radius: 4px;"
              role="img"
              aria-label="Interactive oil color-mixing simulation representing botanical infusion process. Click to add colors."
            />
          </div>
        </div>
      </div>
    </section>

    <!-- Testimonials Section -->
    <section
      id="testimonials"
      class="testimonials-section"
      style="background-color: var(--alchemy-cream); padding: var(--space-xl) var(--space-md);"
    >
      <div class="testimonial-block max-w-[1000px] mx-auto text-center">
        <span
          class="text-[12px] uppercase tracking-[0.12em] font-body font-normal block mb-8"
          style="color: var(--alchemy-terracotta);"
        >
          From Our Community
        </span>

        <div class="relative min-h-[200px]">
          <TransitionGroup name="testimonial">
            <div
              v-for="(t, i) in testimonials"
              v-show="i === currentTestimonial"
              :key="i"
              class="absolute inset-0 flex flex-col items-center justify-center"
            >
              <p
                class="font-display text-[24px] md:text-[36px] italic leading-[1.4]"
                style="color: var(--alchemy-ink);"
              >
                "{{ t.text }}"
              </p>
              <div class="mt-8">
                <span class="font-body text-sm font-medium" style="color: var(--alchemy-forest);">
                  {{ t.name }}
                </span>
                <span class="font-body text-sm font-light ml-2" style="color: var(--alchemy-forest); opacity: 0.7;">
                  {{ t.location }}
                </span>
              </div>
            </div>
          </TransitionGroup>
        </div>

        <!-- Navigation Dots -->
        <div class="flex items-center justify-center gap-3 mt-12">
          <button
            v-for="(_, i) in testimonials"
            :key="i"
            class="w-2 h-2 rounded-full transition-all duration-300"
            :style="{
              backgroundColor: i === currentTestimonial ? 'var(--alchemy-terracotta)' : 'var(--alchemy-forest)',
              opacity: i === currentTestimonial ? 1 : 0.2
            }"
            :aria-label="`Go to testimonial ${i + 1}`"
            @click="currentTestimonial = i"
          />
        </div>
      </div>
    </section>

    <!-- Newsletter Section -->
    <section
      class="newsletter-section"
      style="background-color: var(--alchemy-ink); padding: var(--space-xl) var(--space-md);"
    >
      <div class="newsletter-content max-w-[600px] mx-auto text-center">
        <h2
          class="font-display text-[32px] md:text-[48px] leading-[1.2] mb-4"
          style="color: var(--alchemy-cream);"
        >
          Join the Inner Circle
        </h2>
        <p
          class="font-body text-base font-light mb-8"
          style="color: var(--alchemy-cream); opacity: 0.8;"
        >
          Be the first to know about new formulations, limited seasonal releases, and stories from our atelier. No noise \u2014 just substance.
        </p>
        <form class="flex flex-col sm:flex-row gap-4 items-center" @submit.prevent>
          <input
            type="email"
            placeholder="your@email.com"
            class="flex-1 w-full bg-transparent font-body text-base font-light py-3 px-0 outline-none transition-colors duration-400"
            style="color: var(--alchemy-cream); border-bottom: 1px solid rgba(245, 240, 232, 0.3);"
            @focus="$event.target.style.borderBottomColor = 'var(--alchemy-terracotta)'"
            @blur="$event.target.style.borderBottomColor = 'rgba(245, 240, 232, 0.3)'"
          >
          <button
            type="submit"
            class="w-full sm:w-auto text-[12px] uppercase tracking-[0.12em] font-body font-normal py-[14px] px-8 transition-colors duration-300"
            style="background-color: var(--alchemy-terracotta); color: var(--alchemy-cream); border: none;"
            @mouseenter="$event.target.style.backgroundColor = '#A85C48'"
            @mouseleave="$event.target.style.backgroundColor = 'var(--alchemy-terracotta)'"
          >
            Subscribe
          </button>
        </form>
      </div>
    </section>

    <!-- Footer -->
    <footer
      class="footer-section"
      style="background-color: var(--alchemy-forest); padding: var(--space-lg) var(--space-md);"
    >
      <div class="max-w-[1400px] mx-auto">
        <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-8 lg:gap-12">
          <!-- Brand Column -->
          <div class="footer-column">
            <span class="font-display text-2xl block mb-2" style="color: var(--alchemy-cream);">
              Seruma
            </span>
            <span class="font-body text-[13px] font-light block" style="color: var(--alchemy-cream); opacity: 0.6;">
              Handcrafted in Provence
            </span>
          </div>

          <!-- Shop Column -->
          <div class="footer-column">
            <h4 class="text-[13px] uppercase tracking-[0.08em] font-body font-medium mb-4" style="color: var(--alchemy-cream);">
              Shop
            </h4>
            <ul class="flex flex-col gap-2">
              <li v-for="item in ['Serums', 'Mists', 'Masks', 'Oils', 'Bundles']" :key="item">
                <a
                  href="#"
                  class="font-body text-sm font-light transition-opacity duration-300 hover:opacity-100"
                  style="color: var(--alchemy-cream); opacity: 0.7;"
                >
                  {{ item }}
                </a>
              </li>
            </ul>
          </div>

          <!-- Company Column -->
          <div class="footer-column">
            <h4 class="text-[13px] uppercase tracking-[0.08em] font-body font-medium mb-4" style="color: var(--alchemy-cream);">
              Company
            </h4>
            <ul class="flex flex-col gap-2">
              <li v-for="item in ['Our Story', 'Ingredients', 'Sustainability', 'Press', 'Careers']" :key="item">
                <a
                  href="#"
                  class="font-body text-sm font-light transition-opacity duration-300 hover:opacity-100"
                  style="color: var(--alchemy-cream); opacity: 0.7;"
                >
                  {{ item }}
                </a>
              </li>
            </ul>
          </div>

          <!-- Connect Column -->
          <div class="footer-column">
            <h4 class="text-[13px] uppercase tracking-[0.08em] font-body font-medium mb-4" style="color: var(--alchemy-cream);">
              Connect
            </h4>
            <ul class="flex flex-col gap-2">
              <li v-for="item in ['Instagram', 'Pinterest', 'Contact', 'FAQ', 'Shipping & Returns']" :key="item">
                <a
                  href="#"
                  class="font-body text-sm font-light transition-opacity duration-300 hover:opacity-100"
                  style="color: var(--alchemy-cream); opacity: 0.7;"
                >
                  {{ item }}
                </a>
              </li>
            </ul>
          </div>
        </div>

        <!-- Bottom Bar -->
        <div
          class="flex flex-col sm:flex-row justify-between items-center gap-4 mt-16 pt-8"
          style="border-top: 1px solid rgba(245, 240, 232, 0.15);"
        >
          <span class="font-body text-xs font-light" style="color: var(--alchemy-cream); opacity: 0.5;">
            2026 Seruma. All rights reserved.
          </span>
          <div class="flex gap-4">
            <a href="#" class="font-body text-xs font-light" style="color: var(--alchemy-cream); opacity: 0.5;">Privacy Policy</a>
            <span style="color: var(--alchemy-cream); opacity: 0.3;">&middot;</span>
            <a href="#" class="font-body text-xs font-light" style="color: var(--alchemy-cream); opacity: 0.5;">Terms of Service</a>
          </div>
        </div>
      </div>
    </footer>

    <!-- Cart Overlay Backdrop -->
    <Transition name="fade">
      <div
        v-show="cart.open"
        class="fixed inset-0 z-[90]"
        style="background: rgba(28, 43, 36, 0.3); backdrop-filter: blur(4px);"
        @click="onCartBackdropClick"
      />
    </Transition>

    <!-- Cart Panel -->
    <aside
      id="cart-panel"
      class="fixed top-0 right-0 bottom-0 z-[100] overflow-y-auto transition-transform duration-500"
      :class="cart.open ? 'translate-x-0' : 'translate-x-full'"
      style="width: 100%; max-width: 420px; background-color: var(--alchemy-cream); box-shadow: -4px 0 30px rgba(0, 0, 0, 0.1); transition-timing-function: cubic-bezier(0.25, 0.1, 0.25, 1);"
      role="dialog"
      aria-modal="true"
      aria-label="Shopping cart"
      tabindex="-1"
      @keydown="onCartKeydown"
    >
      <!-- Cart Header -->
      <div class="flex items-center justify-between p-6">
        <h2 class="text-[13px] uppercase tracking-[0.12em] font-body font-normal" style="color: var(--alchemy-ink);">
          Your Cart
        </h2>
        <button
          class="w-8 h-8 flex items-center justify-center"
          aria-label="Close cart"
          style="color: var(--alchemy-forest);"
          @click="cart.open = false"
        >
          <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24" stroke-width="1.5">
            <path stroke-linecap="round" stroke-linejoin="round" d="M6 18L18 6M6 6l12 12" />
          </svg>
        </button>
      </div>

      <!-- Cart Items -->
      <div v-if="cart.items.length > 0" class="px-6">
        <div
          v-for="item in cart.items"
          :key="item.id"
          class="flex items-start gap-4 py-4"
          style="border-bottom: 1px solid rgba(45, 74, 62, 0.1);"
        >
          <img :src="item.image" :alt="item.name" class="w-[60px] h-[60px] object-cover flex-shrink-0" style="border-radius: 4px;">
          <div class="flex-1 min-w-0">
            <h3 class="font-body text-sm font-medium truncate" style="color: var(--alchemy-ink);">
              {{ item.name }}
            </h3>
            <p class="font-body text-sm font-light mt-1" style="color: var(--alchemy-forest); opacity: 0.7;">
              ${{ item.price }}
            </p>
            <div class="flex items-center gap-2 mt-2">
              <button
                class="w-7 h-7 flex items-center justify-center text-sm transition-colors duration-300"
                style="border: 1px solid var(--alchemy-forest); color: var(--alchemy-forest);"
                aria-label="Decrease quantity"
                @click="updateQuantity(item.id, -1)"
              >
                &minus;
              </button>
              <span class="font-body text-sm font-normal w-6 text-center">{{ item.quantity }}</span>
              <button
                class="w-7 h-7 flex items-center justify-center text-sm transition-colors duration-300"
                style="border: 1px solid var(--alchemy-forest); color: var(--alchemy-forest);"
                aria-label="Increase quantity"
                @click="updateQuantity(item.id, 1)"
              >
                +
              </button>
              <button
                class="ml-auto text-xs underline transition-opacity duration-300 hover:opacity-100"
                style="color: var(--alchemy-terracotta); opacity: 0.7;"
                @click="updateQuantity(item.id, -item.quantity)"
              >
                Remove
              </button>
            </div>
          </div>
        </div>
      </div>

      <!-- Empty State -->
      <div
        v-else
        class="flex flex-col items-center justify-center px-6 py-16"
      >
        <p class="font-body text-base font-light mb-6" style="color: var(--alchemy-forest); opacity: 0.7;">
          Your cart is empty
        </p>
        <button
          class="text-[12px] uppercase tracking-[0.1em] font-body font-normal py-3 px-8 transition-all duration-300"
          style="border: 1px solid var(--alchemy-forest); color: var(--alchemy-forest); background: transparent;"
          @click="cart.open = false; scrollToSection('#collection')"
        >
          Continue Shopping
        </button>
      </div>

      <!-- Cart Footer -->
      <div
        v-if="cart.items.length > 0"
        class="sticky bottom-0 p-6 mt-auto"
        style="background-color: var(--alchemy-cream); border-top: 1px solid rgba(45, 74, 62, 0.1);"
      >
        <div class="flex items-center justify-between mb-2">
          <span class="font-body text-base font-medium" style="color: var(--alchemy-ink);">Subtotal</span>
          <span class="font-body text-base font-medium" style="color: var(--alchemy-ink);">${{ cartTotal }}</span>
        </div>
        <p class="font-body text-[13px] font-light mb-4" style="color: var(--alchemy-forest); opacity: 0.6;">
          Shipping calculated at checkout
        </p>
        <button
          class="w-full text-[14px] uppercase tracking-[0.1em] font-body font-normal py-4 transition-colors duration-300"
          style="background-color: var(--alchemy-terracotta); color: var(--alchemy-cream); border: none;"
          @mouseenter="$event.target.style.backgroundColor = '#A85C48'"
          @mouseleave="$event.target.style.backgroundColor = 'var(--alchemy-terracotta)'"
        >
          Checkout
        </button>
      </div>
    </aside>
  </div>
</template>

<style scoped>
/* Testimonial transition */
.testimonial-enter-active,
.testimonial-leave-active {
  transition: opacity 0.4s ease;
}
.testimonial-enter-from {
  opacity: 0;
}
.testimonial-leave-to {
  opacity: 0;
}

/* Fade transition for cart backdrop */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.4s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

/* Line clamp utility */
.line-clamp-2 {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
</style>