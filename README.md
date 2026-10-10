from flask import Flask, request, jsonify, render_template_string
import os

app = Flask(name)
scores = []

HTML = r"""
<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no,viewport-fit=cover">
<meta name="mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="theme-color" content="#050510">
<title>Neon Strike FPS</title>
<style>
• { box-sizing: border-box; -webkit-tap-highlight-color: transparent; }
html, body { width: 100%; height: 100%; margin: 0; overflow: hidden; touch-action: none; user-select: none; -webkit-user-select: none; background: #050510; color: white; font-family: Arial, sans-serif; }
canvas { display: block; position: fixed; inset: 0; z-index: 1; }
#menu { position: fixed; z-index: 30; top: 50%; left: 50%; width: min(94%, 460px); max-height: 92vh; padding: 26px 24px; transform: translate(-50%, -50%); text-align: center; border: 1px solid #00eaff; border-radius: 18px; background: rgba(5, 8, 25, .97); box-shadow: 0 0 45px rgba(0, 234, 255, .45); overflow-y: auto; }
#menu h1 { margin: 0 0 6px; color: #00eaff; font-size: clamp(34px, 9vw, 58px); letter-spacing: 5px; text-shadow: 0 0 14px #00eaff, 0 0 40px #00eaff; }
#menu .subtitle { color: #ff42d0; font-size: 16px; margin: 0 0 4px; letter-spacing: 2px; }
#menu .tagline { color: #7a8ba8; font-size: 12px; margin: 0 0 18px; letter-spacing: 1px; }
#start { width: 100%; padding: 16px; margin: 14px 0 18px; border: 1px solid #00eaff; border-radius: 10px; color: white; background: linear-gradient(90deg, #0a2a4a, #101d35, #0a2a4a); font-size: 19px; font-weight: bold; letter-spacing: 2px; cursor: pointer; text-shadow: 0 0 8px #00eaff; touch-action: manipulation; }
#start:active { transform: scale(0.98); }
.controls { color: #cbd5e1; line-height: 1.7; font-size: 13px; text-align: left; }
.controls b { color: #00eaff; letter-spacing: 1px; }
.controls .col { display: flex; gap: 12px; }
.controls .col > div { flex: 1; }
#leaderboard { margin-top: 16px; padding-top: 12px; border-top: 1px solid rgba(0, 234, 255, .25); text-align: left; color: #cbd5e1; font-size: 13px; }
#leaderboard b { color: #00eaff; }
#hud { display: none; position: fixed; inset: 0; z-index: 10; pointer-events: none; }
#stats { display: flex; justify-content: space-between; gap: 6px; padding: 14px; flex-wrap: wrap; padding-top: max(14px, env(safe-area-inset-top)); }
#stats span { padding: 7px 9px; border: 1px solid #00eaff; border-radius: 7px; background: rgba(0, 0, 0, .4); font-size: clamp(12px, 3vw, 18px); display: inline-flex; align-items: center; gap: 5px; }
#hp { color: #48ff7b; }
#crosshair { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); color: white; font-size: 28px; font-weight: bold; text-shadow: 0 0 10px #00eaff; }
#message { position: absolute; top: 65%; left: 50%; transform: translateX(-50%); color: #ff416c; font-size: 20px; font-weight: bold; text-align: center; text-shadow: 0 0 8px black; pointer-events: auto; }
#bankDisplay { position: absolute; top: calc(60px + env(safe-area-inset-top)); right: 14px; padding: 8px 14px; border: 1px solid #00eaff; border-radius: 8px; background: rgba(0, 0, 0, .6); font-size: 15px; font-weight: bold; display: none; align-items: center; gap: 10px; flex-wrap: wrap; }
#btvValue { color: #00eaff; }
#bankValue { color: #e9edf5; }
#bloodOverlay { position: fixed; inset: 0; z-index: 15; pointer-events: none; opacity: 0; transition: opacity .35s ease; background: radial-gradient(ellipse at center, transparent 30%, rgba(140,0,0,.55) 75%, rgba(80,0,0,.85) 100%); }
#bloodSplatter { position: fixed; inset: 0; z-index: 16; pointer-events: none; opacity: 0; transition: opacity .5s ease; }
#bloodSplatter.show { opacity: .8; }
.skull-ico { display: inline-block; width: 1.05em; height: 1.05em; vertical-align: -0.18em; background-size: contain; background-repeat: no-repeat; background-position: center; flex: none; }
.btv-ico { display: inline-block; width: 1.05em; height: 1.05em; vertical-align: -0.18em; background-size: contain; background-repeat: no-repeat; background-position: center; flex: none; }
#shopPanel { position: fixed; inset: 0; z-index: 40; display: none; background: rgba(2, 4, 12, .93); overflow-y: auto; }
#shopInner { max-width: 900px; margin: 0 auto; padding: 24px 18px 60px; padding-top: max(24px, env(safe-area-inset-top)); }
#shopHeader { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; flex-wrap: wrap; gap: 10px; }
#shopHeader h2 { margin: 0; color: #00eaff; font-size: clamp(24px, 6vw, 38px); letter-spacing: 3px; }
#shopBalance { color: #ffd166; font-size: clamp(14px, 3.6vw, 20px); font-weight: bold; display: inline-flex; align-items: center; gap: 8px; flex-wrap: wrap; }
#shopBtvValue { color: #00eaff; }
#shopBalanceValue { color: #e9edf5; }
#shopClose { padding: 12px 26px; border: 1px solid #ff416c; border-radius: 8px; background: rgba(150, 20, 50, .75); color: white; font-size: 16px; font-weight: bold; touch-action: manipulation; }
.shopTabs { display: flex; gap: 8px; margin-bottom: 18px; flex-wrap: wrap; }
.shopTab { padding: 9px 18px; border: 1px solid #00eaff; border-radius: 8px; background: rgba(10, 30, 60, .7); color: #cbd5e1; font-size: 14px; touch-action: manipulation; }
.shopTab.active { background: #0e3a5a; color: #00eaff; }
.shopGrid { display: grid; grid-template-columns: repeat(auto-fill, minmax(250px, 1fr)); gap: 14px; }
.shopCard { border: 1px solid rgba(0, 234, 255, .35); border-radius: 12px; padding: 16px; background: rgba(8, 16, 35, .9); display: flex; flex-direction: column; }
.shopCard h3 { margin: 0 0 6px; font-size: 17px; }
.rarity { display: inline-block; padding: 3px 10px; border-radius: 6px; font-size: 11px; font-weight: bold; margin-bottom: 8px; align-self: flex-start; }
.rarity-common { background: #4a5568; }
.rarity-rare { background: #2b6cb0; }
.rarity-epic { background: #6b46c1; }
.rarity-crystal { background: #00a0a8; box-shadow: 0 0 10px rgba(0,255,255,.5); }
.rarity-mythic { background: #9b2c2c; box-shadow: 0 0 12px rgba(255,60,60,.6); }
.rarity-mythicmax { background: linear-gradient(90deg, #ff0080, #ffd166, #00eaff); color: #050510; box-shadow: 0 0 16px rgba(255,180,0,.85); }
.shopCard .desc { color: #a0aec0; font-size: 12px; line-height: 1.5; margin-bottom: 10px; flex: 1; }
.shopCard .stats { display: flex; gap: 10px; margin-bottom: 12px; flex-wrap: wrap; }
.shopCard .stat { background: rgba(0, 234, 255, .08); border-radius: 6px; padding: 4px 8px; font-size: 11px; color: #00eaff; }
.shopCard .price { font-size: 18px; font-weight: bold; color: #ffd166; margin-bottom: 8px; display: flex; align-items: center; gap: 6px; }
.shopCard .price.btv { color: #00eaff; }
.buyBtn { width: 100%; padding: 10px; border: 1px solid #48ff7b; border-radius: 8px; background: rgba(20, 80, 40, .7); color: #48ff7b; font-size: 15px; font-weight: bold; touch-action: manipulation; }
.buyBtn:disabled { border-color: #555; background: rgba(40, 40, 40, .6); color: #777; }
.buyBtn.owned { border-color: #ffd166; background: rgba(80, 60, 10, .7); color: #ffd166; }
#shopMessage { position: fixed; top: 20px; left: 50%; transform: translateX(-50%); z-index: 50; padding: 12px 24px; border-radius: 10px; background: rgba(0, 0, 0, .85); color: #ffd166; font-size: 16px; font-weight: bold; display: none; pointer-events: none; }
#mobile { display: none; position: fixed; inset: 0; z-index: 20; pointer-events: none; }
#joystick { position: absolute; left: 20px; bottom: 22px; bottom: max(22px, env(safe-area-inset-bottom)); width: 130px; height: 130px; border: 2px solid #00eaff; border-radius: 50%; background: rgba(0, 50, 80, .35); pointer-events: auto; touch-action: none; }
#knob { position: absolute; left: 50%; top: 50%; width: 58px; height: 58px; transform: translate(-50%, -50%); border: 2px solid white; border-radius: 50%; background: rgba(0, 234, 255, .45); }
#buttons { position: absolute; right: 15px; bottom: 18px; bottom: max(18px, env(safe-area-inset-bottom)); display: flex; align-items: end; gap: 8px; pointer-events: auto; }
#buttons button { width: 58px; height: 58px; border: 2px solid white; border-radius: 50%; color: white; background: rgba(20, 30, 60, .75); font-size: 25px; touch-action: none; }
#fire { width: 78px !important; height: 78px !important; background: rgba(150, 20, 50, .8) !important; }
#loading { position: fixed; inset: 0; z-index: 60; display: flex; flex-direction: column; gap: 18px; align-items: center; justify-content: center; background: #050510; color: #00eaff; font-size: 22px; letter-spacing: 4px; }
#loading::after { content: ""; width: 46px; height: 46px; border: 3px solid rgba(0, 234, 255, .25); border-top-color: #00eaff; border-radius: 50%; animation: loadSpin .9s linear infinite; }
@keyframes loadSpin { to { transform: rotate(360deg); } }
#perfBadge { position: fixed; top: calc(8px + env(safe-area-inset-top)); left: 50%; transform: translateX(-50%); z-index: 11; padding: 4px 10px; border: 1px solid rgba(0,234,255,.3); border-radius: 6px; background: rgba(0,0,0,.5); color: #00eaff; font-size: 11px; font-family: monospace; display: none; pointer-events: none; }
#bossBar { position: fixed; top: calc(58px + env(safe-area-inset-top)); left: 50%; transform: translateX(-50%); z-index: 12; width: min(92%, 620px); display: none; pointer-events: none; }
#bossBarName { text-align: center; color: #ff416c; font-size: clamp(14px, 3.6vw, 20px); font-weight: bold; letter-spacing: 2px; text-shadow: 0 0 10px #ff0033, 0 0 24px #ff0033; margin-bottom: 5px; }
#bossBarTrack { height: 16px; border: 2px solid #ff416c; border-radius: 9px; background: rgba(0, 0, 0, .65); overflow: hidden; box-shadow: 0 0 18px rgba(255, 40, 80, .55); }
#bossBarFill { height: 100%; width: 100%; background: linear-gradient(90deg, #ff0033, #ff7b00, #ffd166); transition: width .12s linear; }
#gasoBar { position: fixed; bottom: calc(14px + env(safe-area-inset-bottom)); left: 50%; transform: translateX(-50%); z-index: 12; width: min(80%, 420px); display: none; pointer-events: none; }
#gasoLabel { text-align: center; color: #ffd166; font-size: 12px; letter-spacing: 2px; margin-bottom: 4px; font-weight: bold; }
#gasoTrack { height: 12px; border: 2px solid #ffd166; border-radius: 7px; background: rgba(0, 0, 0, .6); overflow: hidden; }
#gasoFill { height: 100%; width: 0%; background: linear-gradient(90deg, #ff416c, #ffd166, #48ff7b); transition: width .15s linear; }
.levelGrid { display: grid; grid-template-columns: repeat(auto-fill, minmax(64px, 1fr)); gap: 8px; }
.levelBtn { padding: 12px 4px; border: 1px solid rgba(0, 234, 255, .45); border-radius: 8px; background: rgba(10, 30, 60, .7); color: #cbd5e1; font-size: 14px; font-weight: bold; touch-action: manipulation; }
.levelBtn.boss { border-color: #ff416c; color: #ff416c; background: rgba(80, 10, 30, .55); }
.levelBtn.current { background: #0e3a5a; color: #00eaff; border-color: #00eaff; box-shadow: 0 0 12px rgba(0,234,255,.5); }
.levelBtn.locked { opacity: .35; border-color: #444; color: #666; background: rgba(20, 20, 20, .6); }
.levelBtn.boss.locked { color: #6b2a3a; }
.levelHint { color: #7a8ba8; font-size: 12px; margin-bottom: 12px; line-height: 1.6; }
.bossCard { border: 1px solid #ff416c; border-radius: 12px; padding: 14px; background: rgba(40, 6, 16, .85); display: flex; flex-direction: column; }
.bossCard h3 { margin: 0 0 6px; font-size: 16px; color: #ff416c; }
.bossCard .bdiff { color: #ffd166; font-size: 12px; font-weight: bold; margin-bottom: 8px; letter-spacing: 1px; }
.bossCard .breward { color: #e9edf5; font-size: 13px; margin-bottom: 10px; display: flex; align-items: center; gap: 5px; flex-wrap: wrap; }
@media (pointer: coarse) { #mobile { display: block; } }
@media (max-width: 600px) {
#stats { padding: 8px; padding-top: max(8px, env(safe-area-inset-top)); }
#stats span { font-size: 11px; padding: 6px; }
#joystick { width: 115px; height: 115px; left: 15px; bottom: max(15px, env(safe-area-inset-bottom)); }
#buttons button { width: 52px; height: 52px; font-size: 22px; }
#fire { width: 70px !important; height: 70px !important; }
.shopGrid { grid-template-columns: 1fr; }
.controls .col { flex-direction: column; gap: 6px; }
.levelGrid { grid-template-columns: repeat(auto-fill, minmax(52px, 1fr)); gap: 6px; }
.levelBtn { font-size: 12px; padding: 9px 2px; }
#bossBar { top: calc(50px + env(safe-area-inset-top)); }
}
</style>
</head>
<body>

<div id="loading"><span>ЗАГРУЗКА...</span></div>
<div id="perfBadge">FPS: --</div>

<div id="menu">
<h1>NEON STRIKE</h1>
<p class="subtitle">3D FPS НА FLASK</p>
<p class="tagline">100 УРОВНЕЙ • 10 БОССОВ • БТВ И ЧЕРЕПКИ</p>
<button id="start">НАЧАТЬ ИГРУ</button>
<div class="controls">
<div class="col">
<div>
<b>КОМПЬЮТЕР</b><br>
WASD — движение<br>
Shift — бег<br>
ЛКМ — стрельба<br>
R — перезарядка<br>
1-9 — смена оружия<br>
Space — прыжок<br>
F3 — счётчик FPS
</div>
<div>
<b>ТЕЛЕФОН</b><br>
Джойстик — движение<br>
Свайп справа — обзор<br>
🔥 — стрелять<br>
↻ — перезарядка<br>
↑ — прыжок
</div>
</div>
</div>
<div id="leaderboard">Загрузка рекордов...</div>
</div>

<div id="hud">
<div id="stats">
<span>HP: <b id="hp">100</b></span>
<span>SCORE: <b id="points">0</b></span>
<span>AMMO: <b id="ammo">30 / 120</b></span>
<span>УРОВЕНЬ: <b id="levelValue">1</b></span>
<span id="weaponName">ПМ-9</span>
</div>
<div id="crosshair">+</div>
<div id="message"></div>
<div id="bankDisplay">
<span><span class="btv-ico"></span> <b id="btvValue">0</b></span>
<span><span class="skull-ico"></span> <b id="bankValue">0</b></span>
</div>
</div>

<div id="bossBar">
<div id="bossBarName">БОСС</div>
<div id="bossBarTrack"><div id="bossBarFill"></div></div>
</div>

<div id="gasoBar">
<div id="gasoLabel">БЕНЗИН</div>
<div id="gasoTrack"><div id="gasoFill"></div></div>
</div>

<div id="bloodOverlay"></div>
<canvas id="bloodSplatter"></canvas>

<div id="shopPanel">
<div id="shopInner">
<div id="shopHeader">
<h2>МАГАЗИН</h2>
<div id="shopBalance">
<span><span class="btv-ico"></span> <b id="shopBtvValue">0</b></span>
<span><span class="skull-ico"></span> <b id="shopBalanceValue">0</b></span>
</div>
<button id="shopClose" type="button">ЗАКРЫТЬ</button>
</div>
<div class="shopTabs">
<button type="button" class="shopTab active" data-tab="weapons">ОРУЖИЕ</button>
<button type="button" class="shopTab" data-tab="bombs">БОМБЫ</button>
<button type="button" class="shopTab" data-tab="items">ПРЕДМЕТЫ</button>
<button type="button" class="shopTab" data-tab="fuel">БЕНЗИН</button>
<button type="button" class="shopTab" data-tab="levels">УРОВНИ</button>
<button type="button" class="shopTab" data-tab="bosses">БОССЫ</button>
</div>
<div class="shopGrid" id="shopGrid"></div>
</div>
</div>

<div id="shopMessage"></div>

<div id="mobile">
<div id="joystick"><div id="knob"></div></div>
<div id="buttons">
<button id="reload">↻</button>
<button id="jump">↑</button>
<button id="fire">🔥</button>
</div>
</div>

<script type="importmap">
{
"imports": {
"three": "https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js",
"three/addons/": "https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/"
}
}
</script>

<script type="module">
import * as THREE from 'three';
import { EffectComposer } from 'three/addons/postprocessing/EffectComposer.js';
import { RenderPass } from 'three/addons/postprocessing/RenderPass.js';
import { UnrealBloomPass } from 'three/addons/postprocessing/UnrealBloomPass.js';
import { OutputPass } from 'three/addons/postprocessing/OutputPass.js';

const isMobile = ("ontouchstart" in window || navigator.maxTouchPoints > 0 || matchMedia("(pointer: coarse)").matches);
const cores = navigator.hardwareConcurrency || 4;
const mem = navigator.deviceMemory || 4;
const LOW_END = isMobile && (cores <= 4 || mem <= 3);
const VERY_LOW = isMobile && (cores <= 2 || mem <= 2);
const FPS_CAP = isMobile ? (LOW_END ? 45 : 60) : 0;
const FRAME_INTERVAL = FPS_CAP ? 1000 / FPS_CAP : 0;

let scene, camera, renderer, composer, bloomPass, clock;
let player, weapon, weaponMagazine, weaponHand, boltMesh, droppedMagazine = null;
let throwAnimTimer = 0;
let isThrowing = false;
const THROW_ANIM_DURATION = 0.38;
let thrownItems = [];
const MAX_THROWN_ITEMS = 8;
const THROWABLE_IDS = ["item_toiletpaper", "item_stub", "item_rag", "item_handkerchief"];
function isThrowable(id) { return THROWABLE_IDS.indexOf(id) >= 0; }

let enemies = [], particles = [], bloodParticles = [], bloodDecals = [], bloodPools = [], shells = [], tracers = [], bullets = [], enemyBullets = [];
let chainsawSparks = [];
let wallBoxes = [], solidMeshes = [], decals = [];
let decalGeometry = null, decalMaterial = null;
let bloodDecalGeometry = null, bloodDecalMaterials = [];
let bloodPoolGeometry = null, bloodPoolMaterial = null;
let shopBuilding = null, shopTrigger = null, shopOpen = false, shopCooldown = false;
let sky, skyMat;
let ambientLight, hemisphereLight, directional, redLight, blueLight;

let started = false, ended = false;
let playerDying = false, playerDeathTimer = 0, playerDeathStartY = 1.7, playerDeathStartPitch = 0, deathMessageShown = false;
const PLAYER_DEATH_DURATION = 1.9;

let score = 0;
let skulls = 0;
let btv = 0;
let totalKills = 0;
let health = 100, ammo = 30, reserveAmmo = 120;
let currentLevel = 1;
let maxLevelReached = 1;
let levelKills = 0;
let enemiesToKill = 8;
let inBossFight = false;
let currentBossIndex = -1;
let activeBoss = null;
let bossDefeatPending = false;

let canShoot = true, reloading = false, reloadTimer = 0, reloadDuration = 2.0, firing = false;
let verticalVelocity = 0, grounded = true;
let yaw = 0, pitch = 0;
let jumpRequest = false, bobTime = 0, recoil = 0;
let screenBloodIntensity = 0;
const PLAYER_RADIUS = 0.45;
const MAP_LIMIT = 33.5;

let chainsawRunning = false;
let sawProgress = 0;
let sawTimer = 0;
let sawTarget = null;
let sawDirection = 1;
let sawBladeSpin = 0;
let gasoline = 0;
const GASOLINE_MAX = 1.5;
const GASOLINE_BURN_PER_SEC = GASOLINE_MAX / 120;
const CHAINSAW_RANGE = 2.8;
const CHAINSAW_STUN_RANGE = 3.6;
const SAW_HIT_INTERVAL = 0.22;
const SAW_HITS_TO_KILL = 6;
const SPARK_MAX = VERY_LOW ? 8 : (LOW_END ? 14 : (isMobile ? 26 : 60));

const MAX_DECALS = VERY_LOW ? 12 : (LOW_END ? 20 : (isMobile ? 32 : 60));
const MAX_BLOOD_DECALS = VERY_LOW ? 15 : (LOW_END ? 25 : (isMobile ? 40 : 80));
const MAX_BLOOD_POOLS = VERY_LOW ? 10 : (LOW_END ? 15 : (isMobile ? 25 : 50));
const MAX_PARTICLES = VERY_LOW ? 15 : (LOW_END ? 30 : (isMobile ? 60 : 200));
const MAX_BLOOD_PARTICLES = VERY_LOW ? 40 : (LOW_END ? 70 : (isMobile ? 130 : 400));
const MAX_TRACERS = VERY_LOW ? 4 : (LOW_END ? 6 : (isMobile ? 12 : 40));
const MAX_SHELLS = VERY_LOW ? 3 : (LOW_END ? 4 : (isMobile ? 8 : 30));
const MAX_BULLETS = VERY_LOW ? 4 : (LOW_END ? 8 : (isMobile ? 16 : 60));
const MAX_ENEMY_BULLETS = VERY_LOW ? 4 : (LOW_END ? 6 : (isMobile ? 12 : 40));
const MAX_ENEMIES = VERY_LOW ? 2 : (LOW_END ? 2 : (isMobile ? 3 : 4));

let dynamicPixelRatio = 1;
let fpsSamples = [];
let fpsSampleTimer = 0;
let adaptiveQualityEnabled = isMobile;
let bloomEnabled = !VERY_LOW;
let perfBadgeVisible = false;

let shellGeoShared = null, shellMatShared = null;
let playerBulletGeoShared = null, playerBulletMatShared = null;
let enemyBulletGeoShared = null, enemyBulletMatShared = null;
let tracerMatShared = null;
let enemyFlashLightShared = null;
let enemyFlashTimer = 0;
const ENEMY_FLASH_DURATION = 0.09;
let lastSplashTime = 0;
let sparkGeoShared = null;
let sparkMatsShared = [];

const keys = {};
const joy = { active: false, x: 0, y: 0, id: null };
const lookTouch = { active: false, id: null, x: 0, y: 0 };

const TEX_BASE = "https://cdn.jsdelivr.net/gh/mrdoob/three.js@dev/examples/textures/";
const loadingManager = new THREE.LoadingManager();
let loadingHidden = false;
function hideLoading() {
if (loadingHidden) return;
loadingHidden = true;
const el = document.getElementById("loading");
if (!el) return;
el.style.transition = "opacity .35s ease";
el.style.opacity = "0";
setTimeout(() => { el.style.display = "none"; }, 400);
}
loadingManager.onLoad = () => hideLoading();
loadingManager.onError = (url) => { try { loadingManager.itemEnd(url); } catch (e) {} };
setTimeout(hideLoading, 8000);

const textureLoader = new THREE.TextureLoader(loadingManager);
textureLoader.setCrossOrigin("anonymous");
const ANISO = VERY_LOW ? 1 : (isMobile ? 2 : 4);

const floorTexture = textureLoader.load(TEX_BASE + "floors/FloorsCheckerboard_S_Diffuse.jpg");
const floorNormal = VERY_LOW ? null : textureLoader.load(TEX_BASE + "floors/FloorsCheckerboard_S_Normal.jpg");
const wallTexture = textureLoader.load(TEX_BASE + "brick_diffuse.jpg");
const wallNormal = VERY_LOW ? null : textureLoader.load(TEX_BASE + "brick_normal.jpg");
const gunCarbonTex = textureLoader.load(TEX_BASE + "carbon/Carbon.png");
const gunCarbonNormal = VERY_LOW ? null : textureLoader.load(TEX_BASE + "carbon/Carbon_Normal.png");

const ITEM_URLS = {
rag: "https://i.ibb.co/vvP3S9ct/rag.jpg",
handkerchief: "https://i.ibb.co/wZbywKYf/scarf.jpg",
stub: "https://i.ibb.co/gMYgBPZ2/pear.jpg",
glove: "https://i.ibb.co/yw09pM9/glove.jpg"
};

function loadItemTexture(url, fallbackColor, opts) {
opts = opts || {};
const mat = new THREE.MeshLambertMaterial({
color: fallbackColor,
side: opts.side || THREE.FrontSide
});
const loader = new THREE.TextureLoader();
loader.setCrossOrigin("anonymous");
loader.load(
url,
(t) => {
t.colorSpace = THREE.SRGBColorSpace;
t.anisotropy = ANISO;
t.wrapS = t.wrapT = THREE.ClampToEdgeWrapping;
mat.map = t;
mat.color.setHex(0xffffff);
mat.needsUpdate = true;
},
undefined,
() => {
mat.color.setHex(fallbackColor);
mat.needsUpdate = true;
}
);
return mat;
}

let MAT_RAG = null, MAT_HANDKERCHIEF = null, MAT_STUB = null, MAT_GLOVE = null;
function getItemMaterials() {
if (MAT_RAG) return;
MAT_RAG = loadItemTexture(ITEM_URLS.rag, 0x8a7a5c, { side: THREE.DoubleSide });
MAT_HANDKERCHIEF = loadItemTexture(ITEM_URLS.handkerchief, 0xcc5522, { side: THREE.DoubleSide });
MAT_STUB = loadItemTexture(ITEM_URLS.stub, 0xd8b46a);
MAT_GLOVE = loadItemTexture(ITEM_URLS.glove, 0x1a1a22);
}

function setupTex(t, rep, srgb) {
if (!t) return t;
t.wrapS = t.wrapT = THREE.RepeatWrapping;
t.repeat.set(rep, rep);
if (srgb) t.colorSpace = THREE.SRGBColorSpace;
t.anisotropy = ANISO;
return t;
}
setupTex(floorTexture, 25, true);
setupTex(floorNormal, 25, false);
setupTex(wallTexture, 3, true);
setupTex(wallNormal, 3, false);
setupTex(gunCarbonTex, 2, true);
setupTex(gunCarbonNormal, 2, false);

const menu = document.getElementById("menu");
const hud = document.getElementById("hud");
const startButton = document.getElementById("start");
const hpText = document.getElementById("hp");
const scoreText = document.getElementById("points");
const ammoText = document.getElementById("ammo");
const levelText = document.getElementById("levelValue");
const weaponNameEl = document.getElementById("weaponName");
const message = document.getElementById("message");
const leaderboard = document.getElementById("leaderboard");
const bankDisplay = document.getElementById("bankDisplay");
const btvValueEl = document.getElementById("btvValue");
const bankValueEl = document.getElementById("bankValue");
const joystick = document.getElementById("joystick");
const knob = document.getElementById("knob");
const fireButton = document.getElementById("fire");
const reloadButton = document.getElementById("reload");
const jumpButton = document.getElementById("jump");
const shopPanel = document.getElementById("shopPanel");
const shopGrid = document.getElementById("shopGrid");
const shopBalance = document.getElementById("shopBalance");
const shopBtvValue = document.getElementById("shopBtvValue");
const shopBalanceValue = document.getElementById("shopBalanceValue");
const shopClose = document.getElementById("shopClose");
const shopMessage = document.getElementById("shopMessage");
const bloodOverlay = document.getElementById("bloodOverlay");
const bloodSplatterCanvas = document.getElementById("bloodSplatter");
const bloodCtx = bloodSplatterCanvas.getContext("2d");
const perfBadge = document.getElementById("perfBadge");
const bossBar = document.getElementById("bossBar");
const bossBarName = document.getElementById("bossBarName");
const bossBarFill = document.getElementById("bossBarFill");
const gasoBar = document.getElementById("gasoBar");
const gasoFill = document.getElementById("gasoFill");
startButton.onclick = startGame;

let audioCtx = null, masterGain = null;

const BOSSES = [
{ index: 0, level: 10, name: "ЭХО ПУСТОТЫ", diff: "1 СЛОЖНОСТЬ — САМАЯ ЛЁГКАЯ", reward: 100, hp: 1600, speed: 0.85, damage: 5, scale: 2.3, aura: 0x8899cc, rate: 2.6 },
{ index: 1, level: 20, name: "ПЕПЕЛЬНЫЙ ШЁПОТ", diff: "2 СЛОЖНОСТЬ — ЧУТЬ СЛОЖНЕЕ", reward: 250, hp: 2800, speed: 0.95, damage: 6, scale: 2.4, aura: 0xcc8844, rate: 2.4 },
{ index: 2, level: 30, name: "ВИХРЬ КОСТЕЙ", diff: "3 СЛОЖНОСТЬ — ЕЩЁ ЧУТЬ СЛОЖНЕЕ", reward: 300, hp: 4200, speed: 1.10, damage: 7, scale: 2.5, aura: 0xdddddd, rate: 2.2 },
{ index: 3, level: 40, name: "БЕЗДНА ОТРАЖЕНИЙ", diff: "4 СЛОЖНОСТЬ — СРЕДНЯЧОК", reward: 400, hp: 6000, speed: 1.20, damage: 8, scale: 2.6, aura: 0x4477ff, rate: 2.0 },
{ index: 4, level: 50, name: "ПЛАМЯ ЗАБВЕНИЯ", diff: "5 СЛОЖНОСТЬ — СРЕДНЯЧОК СЛОЖНЕЕ", reward: 600, hp: 8500, speed: 1.30, damage: 9, scale: 2.7, aura: 0xff5500, rate: 1.9 },
{ index: 5, level: 60, name: "ГРОЗА МИРОВ", diff: "6 СЛОЖНОСТЬ — ТЯЖЕЛО", reward: 1000, hp: 11500, speed: 1.40, damage: 11, scale: 2.8, aura: 0xaa66ff, rate: 1.7 },
{ index: 6, level: 70, name: "КРОВАВЫЙ РАССВЕТ", diff: "7 СЛОЖНОСТЬ — ЕЩЁ СЛОЖНЕЕ", reward: 1500, hp: 15000, speed: 1.55, damage: 13, scale: 2.9, aura: 0xff0033, rate: 1.5 },
{ index: 7, level: 80, name: "КОШМАР БЕСКОНЕЧНОСТИ", diff: "8 СЛОЖНОСТЬ — ОЧЕНЬ ТЯЖЕЛО", reward: 3000, hp: 20000, speed: 1.70, damage: 15, scale: 3.0, aura: 0x00eaff, rate: 1.3 },
{ index: 8, level: 90, name: "КОШМАР БЕЗДНЫ", diff: "9 СЛОЖНОСТЬ — ТЯЖЕЛЕЕ ВСЕГО", reward: 5000, hp: 27000, speed: 1.85, damage: 18, scale: 3.1, aura: 0x220044, rate: 1.1 },
{ index: 9, level: 100, name: "СИНГУЛЯРНОСТЬ РАЗУМА", diff: "10 СЛОЖНОСТЬ — ФИНАЛЬНЫЙ БОСС", reward: 10000, hp: 40000, speed: 2.05, damage: 22, scale: 3.3, aura: 0xffd166, rate: 0.9 }
];

const SHOP_ITEMS = [
{ id: "pistol_basic", name: "ПМ-9", category: "weapons", currency: "btv", price: 0, rarity: "common", desc: "Стандартный пистолет. Выдаётся бесплатно.", stats: { damage: 34, fireRate: "130 мс", mag: 30 }, damage: 34, fireDelay: 130, magSize: 30, reserveSize: 120 },
{ id: "smg_fast", name: "ВЕКТОР-С", category: "weapons", currency: "btv", price: 2500, rarity: "rare", desc: "Скорострельный пистолет-пулемёт.", stats: { damage: 28, fireRate: "90 мс", mag: 40 }, damage: 28, fireDelay: 90, magSize: 40, reserveSize: 160 },
{ id: "shotgun_heavy", name: "ДРОБОВИК М-12", category: "weapons", currency: "btv", price: 5500, rarity: "epic", desc: "Тяжёлый дробовик. Большой урон вблизи.", stats: { damage: 85, fireRate: "600 мс", mag: 8 }, damage: 85, fireDelay: 600, magSize: 8, reserveSize: 48 },
{ id: "rifle_assault", name: "АК-74 НЕОН", category: "weapons", currency: "btv", price: 10000, rarity: "epic", desc: "Штурмовая винтовка. Универсальное оружие.", stats: { damage: 48, fireRate: "120 мс", mag: 30 }, damage: 48, fireDelay: 120, magSize: 30, reserveSize: 150 },
{ id: "sniper_rail", name: "РЕЛЬСОТРОН X", category: "weapons", currency: "btv", price: 20000, rarity: "crystal", desc: "Рельсовая снайперка. Пробивает броню.", stats: { damage: 200, fireRate: "1200 мс", mag: 5 }, damage: 200, fireDelay: 1200, magSize: 5, reserveSize: 25 },
{ id: "chainsaw_mythic", name: "БЕНЗОПИЛА «КОСТОГРЫЗ»", category: "weapons", currency: "skull", price: 5500, rarity: "mythic", desc: "Оружие ближнего боя. Пилить врага 6 раз слева направо и обратно до смерти. Оглушает врага на 3-5 секунд при запуске. Требует БЕНЗИН: 1.5 литра хватает на 2 минуты работы.", stats: { damage: 9999, fireRate: "6 распилов", mag: "6 ударов" } },
{ id: "minigun_chaos", name: "ХАОС-МИНИГАН", category: "weapons", currency: "skull", price: 8000, rarity: "mythicmax", desc: "Миниган хаоса. Шквал свинца.", stats: { damage: 60, fireRate: "60 мс", mag: 200 }, damage: 60, fireDelay: 60, magSize: 200, reserveSize: 600 },
{ id: "bomb_frag", name: "ОСКОЛОЧНАЯ ГРАНАТА", category: "bombs", currency: "skull", price: 400, rarity: "common", desc: "Классическая граната.", stats: { damage: 120, radius: "8 м" } },
{ id: "bomb_flash", name: "СВЕТОВАЯ ГРАНАТА", category: "bombs", currency: "skull", price: 900, rarity: "rare", desc: "Ослепляет врагов.", stats: { duration: "3 с", radius: "12 м" } },
{ id: "bomb_emp", name: "ЭМИ-БОМБА", category: "bombs", currency: "skull", price: 1800, rarity: "epic", desc: "Оглушает всех врагов.", stats: { duration: "5 с", radius: "15 м" } },
{ id: "item_toiletpaper", name: "ТУАЛЕТНАЯ БУМАГА", category: "items", currency: "skull", price: 200, rarity: "common", desc: "Скрученная в рулон. Можно метнуть.", stats: { damage: 5, effect: "Отвлечение" } },
{ id: "item_stub", name: "ОГРЫЗОК", category: "items", currency: "skull", price: 500, rarity: "rare", desc: "Огрызок груши. Можно метнуть.", stats: { damage: 15, effect: "Замедление" } },
{ id: "item_rag", name: "ТРЯПКА", category: "items", currency: "skull", price: 900, rarity: "epic", desc: "Грязная тряпка. Можно метнуть.", stats: { damage: 10, effect: "Ослепление" } },
{ id: "item_handkerchief", name: "ПЛАТОК", category: "items", currency: "skull", price: 1500, rarity: "crystal", desc: "Оранжевый платок. Можно метнуть.", stats: { damage: 20, effect: "Страх" } },
{ id: "fuel_half", name: "КАНЕСТРА БЕНЗИНА 0.5 Л", category: "fuel", currency: "skull", price: 500, rarity: "common", desc: "Пол-литра бензина для бензопилы. Для полного бака нужно 1.5 литра (3 канистры).", stats: { volume: "0.5 Л", fuel: "+0.5 Л" } }
];

let ownedItems = { pistol_basic: true };
let hasChainsaw = false;
let currentWeapon = "pistol_basic";

function getWeaponData(id) {
for (const item of SHOP_ITEMS) if (item.id === id) return item;
return SHOP_ITEMS[0];
}

function makeSkullDataURL() {
const c = document.createElement("canvas");
c.width = 64;
c.height = 64;
const x = c.getContext("2d");
x.clearRect(0, 0, 64, 64);
x.fillStyle = "#e9edf5";
x.beginPath();
x.arc(32, 27, 20, Math.PI, 0);
x.closePath();
x.fill();
x.fillRect(12, 27, 40, 13);
x.beginPath();
x.moveTo(17, 40);
x.lineTo(47, 40);
x.lineTo(43, 54);
x.lineTo(21, 54);
x.closePath();
x.fill();
x.fillStyle = "#05060c";
x.beginPath();
x.ellipse(23, 30, 7, 8.4, 0, 0, Math.PI * 2);
x.fill();
x.beginPath();
x.ellipse(41, 30, 7, 8.4, 0, 0, Math.PI * 2);
x.fill();
x.beginPath();
x.moveTo(32, 36);
x.lineTo(28, 44);
x.lineTo(36, 44);
x.closePath();
x.fill();
x.fillRect(21.5, 45, 3, 8);
x.fillRect(27.5, 45, 3, 8);
x.fillRect(33.5, 45, 3, 8);
x.fillRect(39.5, 45, 3, 8);
x.strokeStyle = "rgba(0,0,0,0.35)";
x.lineWidth = 1;
x.beginPath();
x.moveTo(13, 38);
x.lineTo(51, 38);
x.stroke();
return c.toDataURL("image/png");
}

function makeBtvDataURL() {
const c = document.createElement("canvas");
c.width = 64;
c.height = 64;
const x = c.getContext("2d");
x.clearRect(0, 0, 64, 64);
x.fillStyle = "#00eaff";
x.beginPath();
x.arc(32, 32, 26, 0, Math.PI * 2);
x.fill();
x.fillStyle = "#0a2a4a";
x.beginPath();
x.arc(32, 32, 22, 0, Math.PI * 2);
x.fill();
x.strokeStyle = "#00eaff";
x.lineWidth = 2;
x.beginPath();
x.arc(32, 32, 19, 0, Math.PI * 2);
x.stroke();
x.fillStyle = "#00eaff";
x.font = "bold 32px Arial";
x.textAlign = "center";
x.textBaseline = "middle";
x.fillText("Б", 32, 34);
return c.toDataURL("image/png");
}

function applyCurrencyIcons() {
const skullUrl = makeSkullDataURL();
const btvUrl = makeBtvDataURL();
const skulls = document.querySelectorAll(".skull-ico");
for (let i = 0; i < skulls.length; i++) skulls[i].style.backgroundImage = "url(" + skullUrl + ")";
const btvs = document.querySelectorAll(".btv-ico");
for (let i = 0; i < btvs.length; i++) btvs[i].style.backgroundImage = "url(" + btvUrl + ")";
}

function initAudio() {
if (audioCtx) return;
try {
audioCtx = new (window.AudioContext || window.webkitAudioContext)();
masterGain = audioCtx.createGain();
masterGain.gain.value = 0.42;
masterGain.connect(audioCtx.destination);
} catch (e) { audioCtx = null; }
}
function tone(type, f1, f2, duration, gainValue) {
if (!audioCtx) return;
const now = audioCtx.currentTime;
const osc = audioCtx.createOscillator();
const gain = audioCtx.createGain();
osc.type = type;
osc.frequency.setValueAtTime(f1, now);
if (f2) osc.frequency.exponentialRampToValueAtTime(Math.max(1, f2), now + duration);
gain.gain.setValueAtTime(0, now);
gain.gain.linearRampToValueAtTime(gainValue, now + 0.008);
gain.gain.exponentialRampToValueAtTime(0.0001, now + duration);
osc.connect(gain);
gain.connect(masterGain);
osc.start(now);
osc.stop(now + duration + 0.02);
}
function noise(duration, gainValue, filterFreq) {
if (!audioCtx) return;
const now = audioCtx.currentTime;
const bufferSize = Math.floor(audioCtx.sampleRate * duration);
const buffer = audioCtx.createBuffer(1, bufferSize, audioCtx.sampleRate);
const data = buffer.getChannelData(0);
for (let i = 0; i < bufferSize; i++) data[i] = (Math.random() * 2 - 1);
const src = audioCtx.createBufferSource();
src.buffer = buffer;
const filter = audioCtx.createBiquadFilter();
filter.type = "lowpass";
filter.frequency.value = filterFreq || 3000;
const gain = audioCtx.createGain();
gain.gain.setValueAtTime(0, now);
gain.gain.linearRampToValueAtTime(gainValue, now + 0.005);
gain.gain.exponentialRampToValueAtTime(0.0001, now + duration);
src.connect(filter);
filter.connect(gain);
gain.connect(masterGain);
src.start(now);
src.stop(now + duration + 0.02);
}
function playShootSound() { if (LOW_END) { tone("sine", 220, 80, 0.1, 0.18); return; } tone("sine", 220, 80, 0.13, 0.22); setTimeout(() => tone("triangle", 520, 180, 0.09, 0.14), 6); setTimeout(() => noise(0.06, 0.12, 2400), 0); }
function playThrowSound() { tone("sine", 380, 180, 0.12, 0.16); if (!LOW_END) setTimeout(() => noise(0.05, 0.08, 1500), 0); }
function playEmptyClick() { tone("triangle", 1400, 900, 0.04, 0.08); }
function playReloadStart() { tone("sine", 900, 500, 0.08, 0.1); }
function playReloadEnd() { tone("sine", 500, 900, 0.1, 0.12); setTimeout(() => tone("sine", 700, 1200, 0.08, 0.1), 60); }
function playMagDropSound() { tone("sine", 380, 140, 0.13, 0.11); if (!LOW_END) noise(0.05, 0.05, 800); }
function playMagInsertSound() { tone("sine", 700, 250, 0.1, 0.13); setTimeout(() => tone("triangle", 1100, 400, 0.05, 0.09), 40); }
function playBoltSound() { tone("triangle", 1600, 700, 0.06, 0.1); if (!LOW_END) setTimeout(() => tone("triangle", 1100, 400, 0.07, 0.09), 60); }
function playHitSound() { tone("sine", 850, 500, 0.09, 0.13); if (!LOW_END) setTimeout(() => tone("triangle", 1300, 800, 0.05, 0.09), 15); }
function playWallHitSound() { tone("sine", 600, 200, 0.08, 0.09); if (!LOW_END) noise(0.04, 0.05, 1200); }
function playKillSound() { tone("sine", 780, 1100, 0.16, 0.16); setTimeout(() => tone("sine", 1100, 1500, 0.18, 0.13), 80); setTimeout(() => tone("sine", 1500, 1900, 0.2, 0.1), 160); }
function playDamageSound() { tone("sine", 280, 130, 0.22, 0.16); if (!LOW_END) setTimeout(() => tone("triangle", 180, 90, 0.18, 0.1), 40); }
function playShellDropSound() { if (LOW_END) return; tone("sine", 2000, 700, 0.06, 0.05); }
function playGameOverSound() { tone("sine", 500, 100, 1.1, 0.2); setTimeout(() => tone("sine", 350, 60, 1.2, 0.16), 150); }
function playBuySound() { tone("sine", 900, 1300, 0.12, 0.14); setTimeout(() => tone("sine", 1300, 1700, 0.1, 0.1), 70); }
function playShopOpenSound() { tone("sine", 500, 900, 0.18, 0.12); }
function playEnemyShootSound() { tone("sine", 190, 70, 0.14, 0.13); if (!LOW_END) setTimeout(() => noise(0.05, 0.06, 1500), 0); }
function playBloodSplatSound() { noise(0.08, 0.14, 500); if (!LOW_END) setTimeout(() => tone("sine", 180, 80, 0.15, 0.1), 0); }
function playChainsawStartSound() { tone("sawtooth", 90, 190, 0.28, 0.14); setTimeout(() => tone("sawtooth", 190, 260, 0.2, 0.11), 90); if (!LOW_END) setTimeout(() => noise(0.22, 0.07, 900), 60); }
function playChainsawStopSound() { tone("sawtooth", 230, 70, 0.35, 0.12); }
function playChainsawSawSound() { tone("sawtooth", 320, 140, 0.14, 0.13); if (!LOW_END) setTimeout(() => noise(0.1, 0.1, 1400), 10); }
function playSkullSound() { tone("sine", 1200, 1700, 0.1, 0.12); setTimeout(() => tone("sine", 1700, 2200, 0.1, 0.09), 70); }
function playBtvSound() { tone("triangle", 700, 1200, 0.08, 0.11); setTimeout(() => tone("triangle", 1200, 1600, 0.07, 0.08), 50); }
function playBossSound() { tone("sawtooth", 70, 45, 1.4, 0.2); setTimeout(() => tone("sine", 120, 60, 1.5, 0.15), 200); }
function playBossDefeatSound() { tone("sine", 700, 300, 0.6, 0.18); setTimeout(() => tone("sine", 900, 1400, 0.5, 0.15), 300); setTimeout(() => tone("sine", 1400, 2000, 0.6, 0.12), 600); }

let ENEMY_GEO = null;
let muzzleMaterialShared = null;

function getEnemyGeometries() {
if (ENEMY_GEO) return ENEMY_GEO;
const seg = VERY_LOW ? 4 : (LOW_END ? 5 : (isMobile ? 6 : 8));
const radial = VERY_LOW ? 6 : (LOW_END ? 8 : (isMobile ? 8 : 12));
const radialHi = VERY_LOW ? 8 : (LOW_END ? 10 : (isMobile ? 10 : 16));
ENEMY_GEO = {
pelvis: new THREE.CapsuleGeometry(0.185, 0.12, seg, radial),
waist: new THREE.CapsuleGeometry(0.18, 0.10, seg, radial),
chest: new THREE.CapsuleGeometry(0.225, 0.20, seg, radial),
chestTop: new THREE.CapsuleGeometry(0.235, 0.08, seg, radial),
vest: new THREE.CapsuleGeometry(0.25, 0.20, seg, radial),
collar: new THREE.CapsuleGeometry(0.16, 0.06, seg, radial),
shoulderPad: new THREE.SphereGeometry(0.14, seg, radial),
upperArm: new THREE.CapsuleGeometry(0.075, 0.24, seg, radial),
bicep: new THREE.CapsuleGeometry(0.085, 0.11, seg, radial),
forearm: new THREE.CapsuleGeometry(0.062, 0.24, seg, radial),
hand: new THREE.SphereGeometry(0.075, seg, radial),
neck: new THREE.CylinderGeometry(0.065, 0.085, 0.14, seg),
head: new THREE.SphereGeometry(0.125, radialHi, radialHi),
jaw: new THREE.BoxGeometry(0.11, 0.055, 0.11),
helmet: new THREE.SphereGeometry(0.148, radialHi, radialHi, 0, Math.PI * 2, 0, Math.PI * 0.48),
helmetBrim: new THREE.BoxGeometry(0.28, 0.03, 0.12),
eyeWhite: new THREE.SphereGeometry(0.032, seg, radial),
eyePupil: new THREE.SphereGeometry(0.016, seg, radial),
nose: new THREE.ConeGeometry(0.024, 0.05, 4),
mouth: new THREE.BoxGeometry(0.075, 0.018, 0.012),
eyebrow: new THREE.BoxGeometry(0.045, 0.013, 0.015),
thigh: new THREE.CapsuleGeometry(0.135, 0.32, seg, radial),
calf: new THREE.CapsuleGeometry(0.10, 0.30, seg, radial),
kneePad: new THREE.SphereGeometry(0.105, seg, radial),
foot: new THREE.BoxGeometry(0.17, 0.10, 0.28),
gunBody: new THREE.BoxGeometry(0.09, 0.3, 0.13),
gunBarrel: new THREE.CylinderGeometry(0.028, 0.028, 0.15, seg),
gunGrip: new THREE.BoxGeometry(0.07, 0.12, 0.07),
gunMag: new THREE.BoxGeometry(0.05, 0.1, 0.05),
gunStrip: new THREE.BoxGeometry(0.02, 0.02, 0.2),
muzzle: new THREE.SphereGeometry(0.035, 5, 5)
};
return ENEMY_GEO;
}
function getMuzzleMaterial() {
if (!muzzleMaterialShared) muzzleMaterialShared = new THREE.MeshBasicMaterial({ color: 0xffaa00 });
return muzzleMaterialShared;
}

function loadProgress() {
try {
const s = localStorage.getItem("neon_strike_skulls");
if (s !== null) skulls = Math.max(0, parseInt(s, 10) || 0);
const b = localStorage.getItem("neon_strike_btv");
if (b !== null) btv = Math.max(0, parseInt(b, 10) || 0);
const tk = localStorage.getItem("neon_strike_totalkills");
if (tk !== null) totalKills = Math.max(0, parseInt(tk, 10) || 0);
const lvl = localStorage.getItem("neon_strike_maxlevel");
if (lvl !== null) maxLevelReached = Math.max(1, Math.min(100, parseInt(lvl, 10) || 1));
const gas = localStorage.getItem("neon_strike_gasoline");
if (gas !== null) gasoline = Math.max(0, Math.min(GASOLINE_MAX, parseFloat(gas) || 0));
const own = localStorage.getItem("neon_strike_owned");
if (own) {
const parsed = JSON.parse(own);
for (const k in parsed) if (parsed[k]) ownedItems[k] = true;
}
const saw = localStorage.getItem("neon_strike_chainsaw");
if (saw === "1") { hasChainsaw = true; ownedItems["chainsaw_mythic"] = true; }
const cw = localStorage.getItem("neon_strike_weapon");
if (cw && ownedItems[cw]) currentWeapon = cw;
} catch (e) {}
}
function saveProgress() {
try {
localStorage.setItem("neon_strike_skulls", String(skulls));
localStorage.setItem("neon_strike_btv", String(btv));
localStorage.setItem("neon_strike_totalkills", String(totalKills));
localStorage.setItem("neon_strike_maxlevel", String(maxLevelReached));
localStorage.setItem("neon_strike_gasoline", String(gasoline));
localStorage.setItem("neon_strike_owned", JSON.stringify(ownedItems));
localStorage.setItem("neon_strike_chainsaw", hasChainsaw ? "1" : "0");
localStorage.setItem("neon_strike_weapon", currentWeapon);
} catch (e) {}
}
function addSkulls(amount) {
skulls += amount;
saveProgress();
updateHUD();
if (amount > 0) playSkullSound();
}
function addBtv(amount) {
btv += amount;
saveProgress();
updateHUD();
if (amount > 0) playBtvSound();
}

function giveKillRewardBtv() {
totalKills++;
let amount;
if (totalKills % 5 === 0) {
amount = 1000;
} else {
const r = Math.random();
if (r < 0.6) {
amount = 400 + Math.floor(Math.random() * 101);
} else {
amount = 200 + Math.floor(Math.random() * 801);
}
}
addBtv(amount);
return amount;
}

function startGame() {
if (started) return;
initAudio();
if (audioCtx && audioCtx.state === "suspended") audioCtx.resume();
started = true;
menu.style.display = "none";
hud.style.display = "block";
bankDisplay.style.display = "flex";
bloodSplatterCanvas.width = innerWidth;
bloodSplatterCanvas.height = innerHeight;
createWorld();
startLevel(maxLevelReached);
updateHUD();
if (!isMobile) renderer.domElement.requestPointerLock?.();
clock.start();
animate();
}

function computePixelRatio() {
if (isMobile) {
const dpr = Math.min(devicePixelRatio || 1, 2);
let cap;
if (VERY_LOW) cap = 0.85;
else if (LOW_END) cap = 1.0;
else cap = 1.25;
return Math.max(0.75, Math.min(dpr, cap));
}
return Math.min(devicePixelRatio || 1, 1.5);
}

function createWorld() {
scene = new THREE.Scene();
scene.background = new THREE.Color(0x8fc4f0);
scene.fog = new THREE.Fog(0x8fc4f0, 35, isMobile ? 90 : 130);

camera = new THREE.PerspectiveCamera(75, innerWidth / innerHeight, 0.15, 600);

renderer = new THREE.WebGLRenderer({
antialias: !isMobile && !LOW_END,
powerPreference: "high-performance",
alpha: false, stencil: false, depth: true
});
dynamicPixelRatio = computePixelRatio();
renderer.setPixelRatio(dynamicPixelRatio);
renderer.setSize(innerWidth, innerHeight);
const shadowsEnabled = !LOW_END;
renderer.shadowMap.enabled = shadowsEnabled;
if (shadowsEnabled) renderer.shadowMap.type = isMobile ? THREE.BasicShadowMap : THREE.PCFSoftShadowMap;
if ("outputColorSpace" in renderer) renderer.outputColorSpace = THREE.SRGBColorSpace;
renderer.toneMapping = THREE.ACESFilmicToneMapping;
renderer.toneMappingExposure = 1.15;
document.body.appendChild(renderer.domElement);
clock = new THREE.Clock(false);

createSky();
addLights();
addFloor();
addWalls();
createShopStall();
addShopCollision();
initDecals();
initBloodDecals();
initBloodPools();
initSharedShootingAssets();

player = new THREE.Group();
player.position.set(0, 1.7, 12);
scene.add(player);
player.add(camera);

const viewmodelLight = new THREE.PointLight(0xffffff, 1.7, 4.5, 2);
viewmodelLight.position.set(0.35, 0.25, 0.5);
camera.add(viewmodelLight);
const viewmodelAmbient = new THREE.PointLight(0x88bbff, 0.7, 3.5, 2);
viewmodelAmbient.position.set(-0.35, 0.15, 0.45);
camera.add(viewmodelAmbient);
const viewmodelRim = new THREE.PointLight(0x00eaff, 0.9, 3.0, 2);
viewmodelRim.position.set(0.0, -0.3, 0.6);
camera.add(viewmodelRim);

rebuildWeapon();
for (let i = 0; i < MAX_ENEMIES; i++) createEnemy();
setupPostProcessing();
applyEnvironment();
applyLights();
addEventListener("resize", resize);
hideLoading();
}

function addShopCollision() {
const shopCx = -22;
const shopCz = -22;
const halfW = 6.4 / 2;
const halfD = 4.6 / 2;
const angle = Math.PI / 4;
const cosA = Math.cos(angle);
const sinA = Math.sin(angle);
const step = 1.2;
for (let lx = -halfW; lx <= halfW; lx += step) {
for (let lz = -halfD; lz <= halfD; lz += step) {
const wx = shopCx + lx * cosA - lz * sinA;
const wz = shopCz + lx * sinA + lz * cosA;
const box = new THREE.Box3(
new THREE.Vector3(wx - 0.75, 0, wz - 0.75),
new THREE.Vector3(wx + 0.75, 8, wz + 0.75)
);
wallBoxes.push(box);
}
}
}

function setupPostProcessing() {
if (!bloomEnabled) { composer = null; return; }
try {
composer = new EffectComposer(renderer);
composer.setPixelRatio(dynamicPixelRatio);
composer.setSize(innerWidth, innerHeight);
composer.addPass(new RenderPass(scene, camera));
bloomPass = new UnrealBloomPass(
new THREE.Vector2(innerWidth, innerHeight),
LOW_END ? 0.35 : (isMobile ? 0.5 : 0.8), 0.5, 0.72);
composer.addPass(bloomPass);
composer.addPass(new OutputPass());
} catch (e) { composer = null; bloomPass = null; }
}

function initSharedShootingAssets() {
const seg = VERY_LOW ? 3 : (isMobile ? 4 : 6);
shellGeoShared = new THREE.CylinderGeometry(0.02, 0.02, 0.07, seg);
shellMatShared = new THREE.MeshStandardMaterial({ color: 0xffcc44, metalness: 0.95, roughness: 0.25, emissive: 0x553300, emissiveIntensity: 0.8 });
playerBulletGeoShared = new THREE.SphereGeometry(0.045, VERY_LOW ? 3 : (isMobile ? 4 : 5), VERY_LOW ? 2 : (isMobile ? 3 : 4));
playerBulletMatShared = new THREE.MeshBasicMaterial({ color: 0xfff28a });
enemyBulletGeoShared = new THREE.SphereGeometry(0.07, VERY_LOW ? 3 : (isMobile ? 4 : 6), VERY_LOW ? 2 : (isMobile ? 3 : 4));
enemyBulletMatShared = new THREE.MeshBasicMaterial({ color: 0xff5500 });
tracerMatShared = new THREE.LineBasicMaterial({ color: 0xffdd66, transparent: true, opacity: 0.9 });
sparkGeoShared = new THREE.SphereGeometry(0.026, VERY_LOW ? 3 : 4, VERY_LOW ? 2 : 3);
sparkMatsShared = [
new THREE.MeshBasicMaterial({ color: 0xffd166 }),
new THREE.MeshBasicMaterial({ color: 0xff8c1a }),
new THREE.MeshBasicMaterial({ color: 0xfff3b0 })
 ];
if (!VERY_LOW) {
enemyFlashLightShared = new THREE.PointLight(0xffaa00, 0, 6);
scene.add(enemyFlashLightShared);
}
}

function createSky() {
const vs = [
"varying vec3 vWorldPosition;",
"void main(){",
"vec4 wp=modelMatrixvec4(position,1.0);",
"vWorldPosition=wp.xyz;",
"gl_Position=projectionMatrixmodelViewMatrix*vec4(position,1.0);",
"}"
 ].join("\n");
const fs = [
"uniform vec3 top;",
"uniform vec3 bottom;",
"uniform vec3 horizon;",
"uniform float offset;",
"varying vec3 vWorldPosition;",
"void main(){",
"float h=normalize(vWorldPosition+vec3(0.0,offset,0.0)).y;",
"vec3 col = mix(horizon, top, smoothstep(0.0,0.45,h));",
"col = mix(bottom, col, smoothstep(-0.25,0.15,h));",
"gl_FragColor=vec4(col,1.0);",
"}"
 ].join("\n");
skyMat = new THREE.ShaderMaterial({
uniforms: {
top: { value: new THREE.Color(0x4a9ee8) },
bottom: { value: new THREE.Color(0xcce4ff) },
horizon: { value: new THREE.Color(0xffe8c0) },
offset: { value: 20 }
},
vertexShader: vs, fragmentShader: fs,
side: THREE.BackSide, fog: false, depthWrite: false
});
const skySeg = VERY_LOW ? [16, 12] : (isMobile ? [20, 16] : [32, 20]);
sky = new THREE.Mesh(new THREE.SphereGeometry(150, skySeg[0], skySeg[1]), skyMat);
scene.add(sky);
}

function addLights() {
ambientLight = new THREE.AmbientLight(0xc8d8ec, LOW_END ? 1.2 : 1.0);
scene.add(ambientLight);
if (!VERY_LOW) {
hemisphereLight = new THREE.HemisphereLight(0xe0f0ff, 0x8898a8, 1.2);
scene.add(hemisphereLight);
}
directional = new THREE.DirectionalLight(0xfff8e8, 2.6);
directional.position.set(30, 50, 20);
directional.castShadow = !LOW_END;
if (!LOW_END) {
const size = isMobile ? 512 : 1024;
directional.shadow.mapSize.set(size, size);
directional.shadow.camera.left = -40;
directional.shadow.camera.right = 40;
directional.shadow.camera.top = 40;
directional.shadow.camera.bottom = -40;
directional.shadow.bias = -0.0005;
directional.shadow.camera.near = 1;
directional.shadow.camera.far = 120;
}
scene.add(directional);
scene.add(directional.target);
redLight = new THREE.PointLight(0xff0055, LOW_END ? 4 : 7, LOW_END ? 15 : 28, 2);
redLight.position.set(-15, 6, -12);
scene.add(redLight);
blueLight = new THREE.PointLight(0x00eaff, LOW_END ? 4 : 7, LOW_END ? 15 : 28, 2);
blueLight.position.set(15, 6, 8);
scene.add(blueLight);
}

function addFloor() {
let material;
if (LOW_END) material = new THREE.MeshLambertMaterial({ map: floorTexture });
else material = new THREE.MeshPhysicalMaterial({
map: floorTexture, normalMap: floorNormal || null,
normalScale: new THREE.Vector2(0.9, 0.9), roughness: 0.78, metalness: 0.2,
clearcoat: isMobile ? 0 : 0.25, clearcoatRoughness: 0.6, envMapIntensity: 0.7
});
const floor = new THREE.Mesh(new THREE.PlaneGeometry(140, 140), material);
floor.rotation.x = -Math.PI / 2;
floor.receiveShadow = !LOW_END;
floor.updateMatrixWorld(true);
scene.add(floor);
solidMeshes.push(floor);
}

function addWalls() {
let material;
if (LOW_END) material = new THREE.MeshLambertMaterial({ map: wallTexture });
else material = new THREE.MeshPhysicalMaterial({
map: wallTexture, normalMap: wallNormal || null,
normalScale: new THREE.Vector2(1.3, 1.3), roughness: 0.85, metalness: 0.05, envMapIntensity: 0.5
});
const walls = [[0,4,-35,70,8,1],[0,4,35,70,8,1],[-35,4,0,1,8,70],[35,4,0,1,8,70],[-12,2.5,-10,18,5,1],[15,2.5,-4,1,5,18],[-8,2.5,15,1,5,18],[12,2.5,20,18,5,1]];
wallBoxes = [];
for (const data of walls) {
const x = data[0], y = data[1], z = data[2], w = data[3], h = data[4], d = data[5];
const wall = new THREE.Mesh(new THREE.BoxGeometry(w, h, d), material);
wall.position.set(x, y, z);
wall.castShadow = !LOW_END;
wall.receiveShadow = !LOW_END;
scene.add(wall);
wall.updateMatrixWorld(true);
wallBoxes.push(new THREE.Box3().setFromObject(wall));
solidMeshes.push(wall);
}
}

function makeBloodTexture(size, density, dark) {
const canvas = document.createElement("canvas");
canvas.width = size;
canvas.height = size;
const ctx = canvas.getContext("2d");
const cx = size / 2, cy = size / 2, baseRadius = size * 0.28;
const gradient = ctx.createRadialGradient(cx, cy, 0, cx, cy, baseRadius * 1.6);
gradient.addColorStop(0, dark ? "rgba(90,0,0,1)" : "rgba(150,0,0,1)");
gradient.addColorStop(0.45, dark ? "rgba(60,0,0,0.9)" : "rgba(110,0,0,0.85)");
gradient.addColorStop(0.8, dark ? "rgba(30,0,0,0.5)" : "rgba(70,0,0,0.4)");
gradient.addColorStop(1, "rgba(0,0,0,0)");
ctx.fillStyle = gradient;
ctx.beginPath();
ctx.arc(cx, cy, baseRadius, 0, Math.PI * 2);
ctx.fill();
for (let i = 0; i < density; i++) {
const angle = Math.random() * Math.PI * 2;
const dist = baseRadius * 0.5 + Math.random() * baseRadius * 1.5;
const x = cx + Math.cos(angle) * dist;
const y = cy + Math.sin(angle) * dist;
const r = 1 + Math.random() * (size * 0.045);
const op = 0.4 + Math.random() * 0.55;
ctx.fillStyle = dark ? "rgba(50,0,0," + op + ")" : "rgba(120,0,0," + op + ")";
ctx.beginPath();
ctx.arc(x, y, r, 0, Math.PI * 2);
ctx.fill();
}
const tex = new THREE.CanvasTexture(canvas);
tex.colorSpace = THREE.SRGBColorSpace;
tex.anisotropy = ANISO;
return tex;
}
function initBloodDecals() {
if (bloodDecalGeometry) return;
bloodDecalGeometry = new THREE.PlaneGeometry(0.55, 0.55);
const texSize = VERY_LOW ? 48 : (isMobile ? 64 : 96);
const mats = VERY_LOW ? 1 : (isMobile ? 2 : 3);
for (let v = 0; v < mats; v++) {
const tex = makeBloodTexture(texSize, v === 0 ? 14 : (v === 1 ? 22 : 30), v === mats - 1);
bloodDecalMaterials.push(new THREE.MeshBasicMaterial({
map: tex, transparent: true, depthWrite: false,
polygonOffset: true, polygonOffsetFactor: -5, polygonOffsetUnits: -5, opacity: 0.92, fog: true
}));
}
}
function initBloodPools() {
if (bloodPoolGeometry) return;
bloodPoolGeometry = new THREE.CircleGeometry(0.9, VERY_LOW ? 12 : (isMobile ? 16 : 20));
const texSize = VERY_LOW ? 64 : (isMobile ? 96 : 128);
const tex = makeBloodTexture(texSize, VERY_LOW ? 20 : 40, true);
bloodPoolMaterial = new THREE.MeshBasicMaterial({
map: tex, transparent: true, depthWrite: false,
polygonOffset: true, polygonOffsetFactor: -6, polygonOffsetUnits: -6, opacity: 0.9, fog: true
});
}
function spawnBloodDecal(point, normal) {
if (!bloodDecalGeometry) initBloodDecals();
const mat = bloodDecalMaterials[Math.floor(Math.random() * bloodDecalMaterials.length)];
const decal = new THREE.Mesh(bloodDecalGeometry, mat);
decal.position.copy(point).addScaledVector(normal, 0.015);
const quat = new THREE.Quaternion().setFromUnitVectors(new THREE.Vector3(0, 0, 1), normal);
decal.quaternion.copy(quat);
decal.rotateZ(Math.random() * Math.PI * 2);
decal.scale.setScalar(0.55 + Math.random() * 0.9);
decal.userData.spawnTime = performance.now();
scene.add(decal);
bloodDecals.push(decal);
while (bloodDecals.length > MAX_BLOOD_DECALS) { scene.remove(bloodDecals.shift()); }
}
function spawnBloodPool(x, z) {
if (!bloodPoolGeometry) initBloodPools();
const pool = new THREE.Mesh(bloodPoolGeometry, bloodPoolMaterial);
pool.rotation.x = -Math.PI / 2;
pool.position.set(x, 0.012, z);
const s = 0.7 + Math.random() * 0.7;
pool.scale.setScalar(s * 0.6);
pool.userData.growTime = performance.now();
pool.userData.targetScale = s;
pool.rotateZ(Math.random() * Math.PI * 2);
scene.add(pool);
bloodPools.push(pool);
while (bloodPools.length > MAX_BLOOD_POOLS) { scene.remove(bloodPools.shift()); }
}
function initDecals() {
if (decalGeometry) return;
decalGeometry = new THREE.PlaneGeometry(0.38, 0.38);
const size = VERY_LOW ? 32 : 64;
const canvas = document.createElement("canvas");
canvas.width = size;
canvas.height = size;
const ctx = canvas.getContext("2d");
const gradient = ctx.createRadialGradient(size/2, size/2, 0, size/2, size/2, size/2);
gradient.addColorStop(0, "rgba(2,1,0,1)");
gradient.addColorStop(0.18, "rgba(12,8,4,0.96)");
gradient.addColorStop(0.4, "rgba(35,22,12,0.75)");
gradient.addColorStop(0.7, "rgba(60,45,32,0.35)");
gradient.addColorStop(1, "rgba(0,0,0,0)");
ctx.fillStyle = gradient;
ctx.fillRect(0, 0, size, size);
const tex = new THREE.CanvasTexture(canvas);
tex.colorSpace = THREE.SRGBColorSpace;
decalMaterial = new THREE.MeshBasicMaterial({ map: tex, transparent: true, depthWrite: false, polygonOffset: true, polygonOffsetFactor: -4, polygonOffsetUnits: -4, opacity: 0.95, fog: true });
}
function spawnDecal(point, normal) {
if (!decalGeometry) initDecals();
const decal = new THREE.Mesh(decalGeometry, decalMaterial);
decal.position.copy(point).addScaledVector(normal, 0.012);
const quat = new THREE.Quaternion().setFromUnitVectors(new THREE.Vector3(0, 0, 1), normal);
decal.quaternion.copy(quat);
decal.rotateZ(Math.random() * Math.PI * 2);
decal.scale.setScalar(0.75 + Math.random() * 0.5);
scene.add(decal);
decals.push(decal);
while (decals.length > MAX_DECALS) { scene.remove(decals.shift()); }
}

function createShopStall() {
shopBuilding = new THREE.Group();
const stoneMat = new THREE.MeshLambertMaterial({ color: 0x8a8578 });
const woodMat = new THREE.MeshLambertMaterial({ color: 0x9c6b3a });
const woodDarkMat = new THREE.MeshLambertMaterial({ color: 0x5c3e1f });
const woodLightMat = new THREE.MeshLambertMaterial({ color: 0xb88a52 });
const roofMat = new THREE.MeshLambertMaterial({ color: 0xa03028 });
const roofDarkMat = new THREE.MeshLambertMaterial({ color: 0x701a18 });
const stoneBrickMat = new THREE.MeshLambertMaterial({ color: 0x7a7568 });
const glassMat = new THREE.MeshStandardMaterial({ color: 0xa0d8ff, transparent: true, opacity: 0.5, metalness: 0.2, roughness: 0.1 });
const neonMat = new THREE.MeshBasicMaterial({ color: 0x00eaff });
const pinkNeon = new THREE.MeshBasicMaterial({ color: 0xff42d0 });
const metalMat = new THREE.MeshLambertMaterial({ color: 0x3a4048 });
const awningRed = new THREE.MeshLambertMaterial({ color: 0xc02828, side: THREE.DoubleSide });
const awningWhite = new THREE.MeshLambertMaterial({ color: 0xf5f0e5, side: THREE.DoubleSide });
const itemColors = [0xff416c, 0x48ff7b, 0xffd166, 0x00eaff, 0xff42d0, 0x9a6cff];

const W = 6.4, D = 4.6, H = 3.4;

const foundation = new THREE.Mesh(new THREE.BoxGeometry(W + 0.4, 0.5, D + 0.4), stoneMat);
foundation.position.y = 0.25;
foundation.receiveShadow = !LOW_END;
foundation.castShadow = !LOW_END;
shopBuilding.add(foundation);

const step1 = new THREE.Mesh(new THREE.BoxGeometry(1.8, 0.18, 0.55), stoneMat);
step1.position.set(0, 0.09, D/2 + 0.5);
shopBuilding.add(step1);
const step2 = new THREE.Mesh(new THREE.BoxGeometry(1.8, 0.18, 0.5), stoneMat);
step2.position.set(0, 0.27, D/2 + 0.2);
shopBuilding.add(step2);

const backWall = new THREE.Mesh(new THREE.BoxGeometry(W, H, 0.18), woodMat);
backWall.position.set(0, 0.5 + H/2, -D/2 + 0.09);
backWall.castShadow = !LOW_END;
backWall.receiveShadow = !LOW_END;
shopBuilding.add(backWall);

if (!VERY_LOW) {
for (let i = 0; i < 5; i++) {
const beam = new THREE.Mesh(new THREE.BoxGeometry(W + 0.05, 0.06, 0.22), woodDarkMat);
beam.position.set(0, 0.7 + i * 0.6, -D/2 + 0.09);
shopBuilding.add(beam);
}
}

for (const sx of [-1, 1]) {
const sideWall = new THREE.Mesh(new THREE.BoxGeometry(0.18, H, D), woodMat);
sideWall.position.set(sx * (W/2 - 0.09), 0.5 + H/2, 0);
sideWall.castShadow = !LOW_END;
sideWall.receiveShadow = !LOW_END;
shopBuilding.add(sideWall);
if (!VERY_LOW) {
const cb1 = new THREE.Mesh(new THREE.BoxGeometry(0.22, 0.08, D + 0.05), woodDarkMat);
cb1.position.set(sx * (W/2 - 0.09), 0.5 + H * 0.55, 0);
shopBuilding.add(cb1);
const cb2 = new THREE.Mesh(new THREE.BoxGeometry(0.22, 0.08, D + 0.05), woodDarkMat);
cb2.position.set(sx * (W/2 - 0.09), 0.5 + H * 0.25, 0);
shopBuilding.add(cb2);
}
}

for (const sx of [-1, 1]) {
for (const sz of [-1, 1]) {
const pillar = new THREE.Mesh(new THREE.BoxGeometry(0.24, H + 0.3, 0.24), woodDarkMat);
pillar.position.set(sx * (W/2 - 0.12), 0.5 + (H + 0.3)/2, sz * (D/2 - 0.12));
pillar.castShadow = !LOW_END;
shopBuilding.add(pillar);
}
}

const windowFrame = new THREE.Mesh(new THREE.BoxGeometry(1.6, 1.5, 0.12), woodDarkMat);
windowFrame.position.set(-W/2 + 1.6, 0.5 + 1.4, D/2 - 0.02);
shopBuilding.add(windowFrame);
const windowGlass = new THREE.Mesh(new THREE.BoxGeometry(1.4, 1.3, 0.05), glassMat);
windowGlass.position.set(-W/2 + 1.6, 0.5 + 1.4, D/2 + 0.03);
shopBuilding.add(windowGlass);
for (let i = 0; i < 4; i++) {
const item = new THREE.Mesh(new THREE.BoxGeometry(0.22, 0.28, 0.1), new THREE.MeshBasicMaterial({ color: itemColors[i % itemColors.length] }));
item.position.set(-W/2 + 1.6 + (i - 1.5) * 0.32, 0.5 + 1.4, D/2 - 0.05);
shopBuilding.add(item);
}

const counter = new THREE.Mesh(new THREE.BoxGeometry(W - 1.8, 1.15, 1.0), woodLightMat);
counter.position.set(0.9, 0.5 + 0.575, D/2 - 0.6);
counter.castShadow = !LOW_END;
shopBuilding.add(counter);
const counterTop = new THREE.Mesh(new THREE.BoxGeometry(W - 1.7, 0.1, 1.1), woodDarkMat);
counterTop.position.set(0.9, 0.5 + 1.15 + 0.05, D/2 - 0.6);
counterTop.castShadow = !LOW_END;
shopBuilding.add(counterTop);
const counterNeon = new THREE.Mesh(new THREE.BoxGeometry(W - 1.8, 0.05, 0.06), neonMat);
counterNeon.position.set(0.9, 0.5 + 0.58, D/2 - 0.09);
shopBuilding.add(counterNeon);

for (let i = 0; i < 3; i++) {
const shelfY = 0.5 + 1.3 + i * 0.65;
const shelf = new THREE.Mesh(new THREE.BoxGeometry(W - 0.7, 0.08, 0.55), woodLightMat);
shelf.position.set(0, shelfY, -D/2 + 0.5);
shelf.castShadow = !LOW_END;
shopBuilding.add(shelf);
const itemCount = 4 + (i % 2);
for (let j = 0; j < itemCount; j++) {
const itemColor = itemColors[(i * 3 + j) % itemColors.length];
const shapeType = (i + j) % 3;
let itemMesh;
if (shapeType === 0) itemMesh = new THREE.Mesh(new THREE.BoxGeometry(0.28, 0.34, 0.24), new THREE.MeshLambertMaterial({ color: itemColor }));
else if (shapeType === 1) itemMesh = new THREE.Mesh(new THREE.CylinderGeometry(0.15, 0.15, 0.32, 8), new THREE.MeshLambertMaterial({ color: itemColor }));
else itemMesh = new THREE.Mesh(new THREE.SphereGeometry(0.17, 8, 6), new THREE.MeshLambertMaterial({ color: itemColor }));
itemMesh.position.set(-W/2 + 0.65 + j * (W - 1.3) / (itemCount - 1 || 1), shelfY + 0.21, -D/2 + 0.5);
shopBuilding.add(itemMesh);
}
}

const roofPitch = 0.42;
const roofHalfD = D / 2 + 0.5;
for (const sz of [-1, 1]) {
const slope = new THREE.Mesh(new THREE.BoxGeometry(W + 1.2, 0.15, roofHalfD), roofMat);
slope.position.set(0, 0.5 + H + 0.55, sz * (roofHalfD / 2 - 0.05));
slope.rotation.x = sz * roofPitch;
slope.castShadow = !LOW_END;
slope.receiveShadow = !LOW_END;
shopBuilding.add(slope);
}
const ridge = new THREE.Mesh(new THREE.BoxGeometry(W + 1.3, 0.16, 0.28), roofDarkMat);
ridge.position.set(0, 0.5 + H + 0.55 + Math.tan(roofPitch) * (roofHalfD / 2), 0);
ridge.castShadow = !LOW_END;
shopBuilding.add(ridge);
const ridgeNeon = new THREE.Mesh(new THREE.BoxGeometry(W + 1.4, 0.05, 0.06), pinkNeon);
ridgeNeon.position.set(0, ridge.position.y + 0.12, 0);
shopBuilding.add(ridgeNeon);

const chimney = new THREE.Mesh(new THREE.BoxGeometry(0.55, 1.2, 0.55), stoneBrickMat);
chimney.position.set(W/2 - 1.2, 0.5 + H + 0.9, -D/4);
chimney.castShadow = !LOW_END;
shopBuilding.add(chimney);
const chimneyTop = new THREE.Mesh(new THREE.BoxGeometry(0.68, 0.14, 0.68), stoneMat);
chimneyTop.position.set(W/2 - 1.2, 0.5 + H + 1.55, -D/4);
shopBuilding.add(chimneyTop);

const signBack = new THREE.Mesh(new THREE.BoxGeometry(W - 1.2, 0.9, 0.14), new THREE.MeshLambertMaterial({ color: 0x1a1030 }));
signBack.position.set(0, 0.5 + H + 0.05, D/2 + 0.12);
shopBuilding.add(signBack);
const signBorderTop = new THREE.Mesh(new THREE.BoxGeometry(W - 1.1, 0.05, 0.16), pinkNeon);
signBorderTop.position.set(0, 0.5 + H + 0.5, D/2 + 0.13);
shopBuilding.add(signBorderTop);
const signBorderBottom = new THREE.Mesh(new THREE.BoxGeometry(W - 1.1, 0.05, 0.16), pinkNeon);
signBorderBottom.position.set(0, 0.5 + H - 0.4, D/2 + 0.13);
shopBuilding.add(signBorderBottom);
const letterMat = new THREE.MeshBasicMaterial({ color: 0x00eaff });
for (let i = 0; i < 5; i++) {
const lx = -1.9 + i * 0.95;
const letter = new THREE.Mesh(new THREE.BoxGeometry(0.55, 0.42, 0.1), letterMat);
letter.position.set(lx, 0.5 + H + 0.05, D/2 + 0.2);
shopBuilding.add(letter);
}

const awningGroup = new THREE.Group();
const awningWidth = W + 0.8;
const stripeCount = 10;
const stripeW = awningWidth / stripeCount;
for (let i = 0; i < stripeCount; i++) {
const stripeMat = i % 2 === 0 ? awningRed : awningWhite;
const stripe = new THREE.Mesh(new THREE.BoxGeometry(stripeW, 0.05, 1.6), stripeMat);
stripe.position.set(-awningWidth/2 + stripeW/2 + i * stripeW, 0, 0);
stripe.rotation.x = -0.4;
awningGroup.add(stripe);
}
awningGroup.position.set(0, 0.5 + H - 0.35, D/2 + 0.75);
shopBuilding.add(awningGroup);

const lanternPost = new THREE.Mesh(new THREE.CylinderGeometry(0.06, 0.06, 0.9, 6), metalMat);
lanternPost.position.set(W/2 + 0.6, 0.5 + 0.45, D/2 + 0.3);
shopBuilding.add(lanternPost);
const lanternBody = new THREE.Mesh(new THREE.BoxGeometry(0.25, 0.35, 0.25), new THREE.MeshBasicMaterial({ color: 0xffcc77, transparent: true, opacity: 0.85 }));
lanternBody.position.set(W/2 + 0.6, 0.5 + 1.05, D/2 + 0.3);
shopBuilding.add(lanternBody);
const lanternTop = new THREE.Mesh(new THREE.ConeGeometry(0.22, 0.18, 4), metalMat);
lanternTop.position.set(W/2 + 0.6, 0.5 + 1.32, D/2 + 0.3);
lanternTop.rotation.y = Math.PI / 4;
shopBuilding.add(lanternTop);
if (!LOW_END) {
const lanternLight = new THREE.PointLight(0xffcc77, 6, 8, 2);
lanternLight.position.set(W/2 + 0.6, 0.5 + 1.05, D/2 + 0.3);
shopBuilding.add(lanternLight);
}

if (!VERY_LOW) {
for (let i = 0; i < 2; i++) {
const barrel = new THREE.Mesh(new THREE.CylinderGeometry(0.32, 0.32, 0.75, 10), woodDarkMat);
barrel.position.set(-W/2 - 0.5 + i * 0.75, 0.5 + 0.375, D/2 - 0.3);
barrel.castShadow = !LOW_END;
shopBuilding.add(barrel);
for (const ry of [-0.22, 0.22]) {
const hoop = new THREE.Mesh(new THREE.TorusGeometry(0.33, 0.025, 6, 12), metalMat);
hoop.rotation.x = Math.PI / 2;
hoop.position.set(-W/2 - 0.5 + i * 0.75, 0.5 + 0.375 + ry, D/2 - 0.3);
shopBuilding.add(hoop);
}
}
}

const chest = new THREE.Mesh(new THREE.BoxGeometry(0.9, 0.55, 0.6), woodLightMat);
chest.position.set(-W/2 - 0.3, 0.5 + 0.275, -D/2 - 0.6);
chest.rotation.y = 0.4;
shopBuilding.add(chest);
const chestLid = new THREE.Mesh(new THREE.BoxGeometry(0.95, 0.15, 0.65), woodDarkMat);
chestLid.position.set(-W/2 - 0.3, 0.5 + 0.55, -D/2 - 0.6);
chestLid.rotation.y = 0.4;
shopBuilding.add(chestLid);

shopBuilding.position.set(-22, 0, -22);
shopBuilding.rotation.y = Math.PI / 4;
scene.add(shopBuilding);

shopTrigger = new THREE.Mesh(new THREE.SphereGeometry(6.5, 6, 4), new THREE.MeshBasicMaterial({ visible: false }));
shopTrigger.position.copy(shopBuilding.position);
scene.add(shopTrigger);
}
function checkShopTrigger() {
if (!shopTrigger || shopOpen || ended) return;
const dist = player.position.distanceTo(shopTrigger.position);
if (shopCooldown) { if (dist > 7.5) shopCooldown = false; return; }
if (dist < 6.5) openShop();
}
function collidesAt(x, z, radius) {
radius = radius === undefined ? PLAYER_RADIUS : radius;
for (const box of wallBoxes) {
if (x + radius > box.min.x && x - radius < box.max.x && z + radius > box.min.z && z - radius < box.max.z) return true;
}
return false;
}

function applyEnvironment() {
if (skyMat) {
skyMat.uniforms.top.value.setHex(0x4a9ee8);
skyMat.uniforms.bottom.value.setHex(0xcce4ff);
skyMat.uniforms.horizon.value.setHex(0xffe8c0);
}
if (scene && scene.fog) scene.fog.color.setHex(0x8fc4f0);
if (scene) scene.background.setHex(0x8fc4f0);
}
function applyLights() {
if (ambientLight) ambientLight.intensity = LOW_END ? 1.2 : 1.0;
if (hemisphereLight) hemisphereLight.intensity = 1.2;
if (directional) directional.intensity = 2.6;
if (redLight) redLight.intensity = LOW_END ? 4 : 7;
if (blueLight) blueLight.intensity = LOW_END ? 4 : 7;
}

const handMaterials = { skin: null, glove: null, trim: null };
function initHandMaterials() {
if (handMaterials.skin) return;
getItemMaterials();
handMaterials.skin = MAT_GLOVE.clone();
handMaterials.glove = MAT_GLOVE.clone();
handMaterials.skin.color.setHex(0x3a3a44);
handMaterials.glove.color.setHex(0x1a1a22);
handMaterials.trim = new THREE.MeshStandardMaterial({ color: 0x00eaff, emissive: 0x00eaff, emissiveIntensity: 1.2, roughness: 0.3 });
}
const HAND_SCALE = 1.35;
function getHandGroup(gripX, gripY, gripZ, rotX, rotY, rotZ) {
initHandMaterials();
const hand = new THREE.Group();
const skinMat = handMaterials.skin;
const gloveMat = handMaterials.glove;
const gloveTrim = handMaterials.trim;
const gx = gripX === undefined ? 0.32 : gripX;
const gy = gripY === undefined ? -0.46 : gripY;
const gz = gripZ === undefined ? -0.48 : gripZ;
const palm = new THREE.Mesh(new THREE.BoxGeometry(0.14 * HAND_SCALE, 0.14 * HAND_SCALE, 0.11 * HAND_SCALE), gloveMat);
palm.position.set(gx, gy, gz);
if (rotX || rotY || rotZ) palm.rotation.set(rotX || 0, rotY || 0, rotZ || 0);
hand.add(palm);
for (let i = 0; i < 4; i++) {
const finger = new THREE.Mesh(new THREE.BoxGeometry(0.028 * HAND_SCALE, 0.026 * HAND_SCALE, 0.1 * HAND_SCALE), skinMat);
finger.position.set(gx + (0.052 - i * 0.028) * HAND_SCALE, gy + 0.02 * HAND_SCALE, gz - 0.02 * HAND_SCALE);
hand.add(finger);
const knuckle = new THREE.Mesh(new THREE.BoxGeometry(0.03 * HAND_SCALE, 0.022 * HAND_SCALE, 0.032 * HAND_SCALE), gloveMat);
knuckle.position.set(gx + (0.052 - i * 0.028) * HAND_SCALE, gy - 0.035 * HAND_SCALE, gz - 0.02 * HAND_SCALE);
hand.add(knuckle);
}
const thumbBase = new THREE.Mesh(new THREE.BoxGeometry(0.03 * HAND_SCALE, 0.04 * HAND_SCALE, 0.055 * HAND_SCALE), gloveMat);
thumbBase.position.set(gx - 0.062 * HAND_SCALE, gy + 0.015 * HAND_SCALE, gz + 0.015 * HAND_SCALE);
hand.add(thumbBase);
const thumbTip = new THREE.Mesh(new THREE.BoxGeometry(0.026 * HAND_SCALE, 0.028 * HAND_SCALE, 0.05 * HAND_SCALE), skinMat);
thumbTip.position.set(gx - 0.062 * HAND_SCALE, gy + 0.015 * HAND_SCALE, gz - 0.04 * HAND_SCALE);
hand.add(thumbTip);
const wrist = new THREE.Mesh(new THREE.BoxGeometry(0.145 * HAND_SCALE, 0.055 * HAND_SCALE, 0.145 * HAND_SCALE), gloveMat);
wrist.position.set(gx, gy - 0.09 * HAND_SCALE, gz + 0.02 * HAND_SCALE);
hand.add(wrist);
const trim = new THREE.Mesh(new THREE.BoxGeometry(0.148 * HAND_SCALE, 0.018 * HAND_SCALE, 0.148 * HAND_SCALE), gloveTrim);
trim.position.set(gx, gy - 0.058 * HAND_SCALE, gz + 0.02 * HAND_SCALE);
hand.add(trim);
return hand;
}

const WEAPON_MATERIALS = { metal: null, dark: null, neon: null, wood: null, mag: null, neonPink: null, gold: null, steel: null, orange: null };
function initWeaponMaterials() {
if (WEAPON_MATERIALS.metal) return;
if (LOW_END) {
WEAPON_MATERIALS.metal = new THREE.MeshLambertMaterial({ color: 0x4a5666, map: gunCarbonTex });
} else {
WEAPON_MATERIALS.metal = new THREE.MeshPhysicalMaterial({
color: 0x4a5666, map: gunCarbonTex, normalMap: gunCarbonNormal || null,
normalScale: new THREE.Vector2(0.5, 0.5), metalness: 0.95, roughness: 0.28,
clearcoat: isMobile ? 0 : 0.35, clearcoatRoughness: 0.35
});
}
WEAPON_MATERIALS.dark = new THREE.MeshStandardMaterial({ color: 0x0e1420, roughness: 0.8, metalness: 0.25 });
WEAPON_MATERIALS.neon = new THREE.MeshStandardMaterial({ color: 0x00eaff, emissive: 0x00eaff, emissiveIntensity: 1.6, roughness: 0.3 });
WEAPON_MATERIALS.neonPink = new THREE.MeshStandardMaterial({ color: 0xff42d0, emissive: 0xff42d0, emissiveIntensity: 1.6, roughness: 0.3 });
WEAPON_MATERIALS.wood = new THREE.MeshStandardMaterial({ color: 0x6b4422, roughness: 0.9, metalness: 0.0 });
WEAPON_MATERIALS.mag = new THREE.MeshStandardMaterial({ color: 0x1e2a38, roughness: 0.7, metalness: 0.5 });
WEAPON_MATERIALS.gold = new THREE.MeshStandardMaterial({ color: 0xffd166, emissive: 0xffd166, emissiveIntensity: 1.2, metalness: 0.9, roughness: 0.25 });
WEAPON_MATERIALS.steel = new THREE.MeshStandardMaterial({ color: 0x8a95a5, roughness: 0.3, metalness: 0.9 });
WEAPON_MATERIALS.orange = new THREE.MeshStandardMaterial({ color: 0xff7700, emissive: 0xff5500, emissiveIntensity: 1.4, roughness: 0.4, metalness: 0.5 });
}

function buildWeaponModel(id) {
initWeaponMaterials();
const M = WEAPON_MATERIALS;
const g = new THREE.Group();
let magazine = null, bolt = null;
let handPos = { x: 0.34, y: -0.42, z: -0.72, rx: 0, ry: 0, rz: 0 };

if (id === "pistol_basic") {
const slide = new THREE.Mesh(new THREE.BoxGeometry(0.13, 0.13, 0.46), M.metal); slide.position.set(0.32, -0.22, -0.72);
const frame = new THREE.Mesh(new THREE.BoxGeometry(0.11, 0.1, 0.3), M.dark); frame.position.set(0.32, -0.32, -0.65);
const barrel = new THREE.Mesh(new THREE.CylinderGeometry(0.035, 0.035, 0.2, 8), M.steel); barrel.rotation.x = Math.PI / 2; barrel.position.set(0.32, -0.2, -1.0);
const sightF = new THREE.Mesh(new THREE.BoxGeometry(0.02, 0.03, 0.02), M.dark); sightF.position.set(0.32, -0.13, -0.92);
const sightR = new THREE.Mesh(new THREE.BoxGeometry(0.04, 0.025, 0.02), M.dark); sightR.position.set(0.32, -0.13, -0.53);
const trigger = new THREE.Mesh(new THREE.BoxGeometry(0.03, 0.045, 0.02), M.dark); trigger.position.set(0.32, -0.38, -0.62);
const grip = new THREE.Mesh(new THREE.BoxGeometry(0.11, 0.2, 0.13), M.dark); grip.position.set(0.32, -0.5, -0.56); grip.rotation.x = -0.15;
magazine = new THREE.Mesh(new THREE.BoxGeometry(0.09, 0.17, 0.11), M.mag); magazine.position.set(0.32, -0.5, -0.76); magazine.rotation.x = -0.08;
bolt = new THREE.Mesh(new THREE.BoxGeometry(0.055, 0.035, 0.14), M.steel); bolt.position.set(0.32, -0.16, -0.8);
const neon = new THREE.Mesh(new THREE.BoxGeometry(0.02, 0.02, 0.24), M.neon); neon.position.set(0.32, -0.17, -0.82);
g.add(slide, frame, barrel, sightF, sightR, trigger, grip, magazine, bolt, neon);
handPos = { x: 0.34, y: -0.42, z: -0.72, rx: 0, ry: 0, rz: 0 };
} else if (id === "smg_fast") {
const body = new THREE.Mesh(new THREE.BoxGeometry(0.16, 0.18, 0.55), M.metal); body.position.set(0.32, -0.28, -0.82);
const upper = new THREE.Mesh(new THREE.BoxGeometry(0.14, 0.06, 0.5), M.dark); upper.position.set(0.32, -0.17, -0.82);
const barrel = new THREE.Mesh(new THREE.CylinderGeometry(0.04, 0.04, 0.32, 8), M.steel); barrel.rotation.x = Math.PI / 2; barrel.position.set(0.32, -0.24, -1.25);
const shroud = new THREE.Mesh(new THREE.CylinderGeometry(0.055, 0.055, 0.28, 8), M.dark); shroud.rotation.x = Math.PI / 2; shroud.position.set(0.32, -0.24, -1.15);
const frontGrip = new THREE.Mesh(new THREE.BoxGeometry(0.07, 0.16, 0.09), M.dark); frontGrip.position.set(0.32, -0.4, -1.1);
const grip = new THREE.Mesh(new THREE.BoxGeometry(0.12, 0.22, 0.14), M.dark); grip.position.set(0.32, -0.5, -0.62); grip.rotation.x = -0.2;
magazine = new THREE.Mesh(new THREE.BoxGeometry(0.1, 0.3, 0.12), M.mag); magazine.position.set(0.32, -0.55, -0.82);
const stock = new THREE.Mesh(new THREE.BoxGeometry(0.11, 0.12, 0.32), M.dark); stock.position.set(0.32, -0.28, -0.38);
const neonStrip = new THREE.Mesh(new THREE.BoxGeometry(0.03, 0.03, 0.35), M.neon); neonStrip.position.set(0.32, -0.13, -0.95);
g.add(body, upper, barrel, shroud, frontGrip, grip, magazine, stock, neonStrip);
handPos = { x: 0.34, y: -0.43, z: -0.76, rx: -0.15, ry: 0, rz: 0 };
} else if (id === "shotgun_heavy") {
const body = new THREE.Mesh(new THREE.BoxGeometry(0.2, 0.2, 0.7), M.wood); body.position.set(0.32, -0.28, -0.9);
const receiver = new THREE.Mesh(new THREE.BoxGeometry(0.19, 0.22, 0.28), M.metal); receiver.position.set(0.32, -0.26, -0.65);
const b1 = new THREE.Mesh(new THREE.CylinderGeometry(0.05, 0.05, 0.72, 10), M.steel); b1.rotation.x = Math.PI / 2; b1.position.set(0.27, -0.22, -1.45);
const b2 = new THREE.Mesh(new THREE.CylinderGeometry(0.05, 0.05, 0.72, 10), M.steel); b2.rotation.x = Math.PI / 2; b2.position.set(0.37, -0.22, -1.45);
const pump = new THREE.Mesh(new THREE.BoxGeometry(0.24, 0.14, 0.28), M.wood); pump.position.set(0.32, -0.36, -1.2);
const grip = new THREE.Mesh(new THREE.BoxGeometry(0.13, 0.22, 0.15), M.wood); grip.position.set(0.32, -0.5, -0.6); grip.rotation.x = -0.18;
const neon = new THREE.Mesh(new THREE.BoxGeometry(0.03, 0.03, 0.3), M.neonPink); neon.position.set(0.32, -0.16, -0.85);
g.add(body, receiver, b1, b2, pump, grip, neon);
handPos = { x: 0.34, y: -0.43, z: -0.76, rx: -0.15, ry: 0, rz: 0 };
} else if (id === "rifle_assault") {
const body = new THREE.Mesh(new THREE.BoxGeometry(0.18, 0.19, 0.7), M.metal); body.position.set(0.32, -0.28, -0.88);
const upper = new THREE.Mesh(new THREE.BoxGeometry(0.16, 0.06, 0.6), M.dark); upper.position.set(0.32, -0.17, -0.88);
const barrel = new THREE.Mesh(new THREE.CylinderGeometry(0.035, 0.035, 0.6, 8), M.steel); barrel.rotation.x = Math.PI / 2; barrel.position.set(0.32, -0.24, -1.5);
const handguard = new THREE.Mesh(new THREE.BoxGeometry(0.14, 0.13, 0.35), M.wood); handguard.position.set(0.32, -0.26, -1.25);
const frontSight = new THREE.Mesh(new THREE.BoxGeometry(0.03, 0.07, 0.03), M.dark); frontSight.position.set(0.32, -0.14, -1.45);
const grip = new THREE.Mesh(new THREE.BoxGeometry(0.12, 0.22, 0.14), M.dark); grip.position.set(0.32, -0.5, -0.65); grip.rotation.x = -0.22;
magazine = new THREE.Mesh(new THREE.BoxGeometry(0.1, 0.35, 0.14), M.mag); magazine.position.set(0.32, -0.58, -0.92); magazine.rotation.x = -0.3;
const stock = new THREE.Mesh(new THREE.BoxGeometry(0.12, 0.14, 0.35), M.wood); stock.position.set(0.32, -0.29, -0.4);
const scopeMount = new THREE.Mesh(new THREE.BoxGeometry(0.06, 0.05, 0.25), M.dark); scopeMount.position.set(0.32, -0.14, -0.85);
const neon = new THREE.Mesh(new THREE.BoxGeometry(0.04, 0.03, 0.4), M.neon); neon.position.set(0.32, -0.14, -1.15);
g.add(body, upper, barrel, handguard, frontSight, grip, magazine, stock, scopeMount, neon);
handPos = { x: 0.34, y: -0.43, z: -0.81, rx: -0.2, ry: 0, rz: 0 };
} else if (id === "sniper_rail") {
const body = new THREE.Mesh(new THREE.BoxGeometry(0.15, 0.16, 0.9), M.metal); body.position.set(0.32, -0.28, -1.0);
const barrel = new THREE.Mesh(new THREE.CylinderGeometry(0.028, 0.028, 1.05, 8), M.steel); barrel.rotation.x = Math.PI / 2; barrel.position.set(0.32, -0.24, -1.95);
const muzzleBrake = new THREE.Mesh(new THREE.CylinderGeometry(0.05, 0.045, 0.12, 8), M.dark); muzzleBrake.rotation.x = Math.PI / 2; muzzleBrake.position.set(0.32, -0.24, -2.45);
const scope = new THREE.Mesh(new THREE.CylinderGeometry(0.055, 0.055, 0.42, 10), M.dark); scope.rotation.x = Math.PI / 2; scope.position.set(0.32, -0.11, -1.0);
const scopeLens = new THREE.Mesh(new THREE.CircleGeometry(0.05, 10), M.neon); scopeLens.position.set(0.32, -0.11, -1.22);
const scopeMount = new THREE.Mesh(new THREE.BoxGeometry(0.05, 0.06, 0.3), M.dark); scopeMount.position.set(0.32, -0.16, -1.0);
const grip = new THREE.Mesh(new THREE.BoxGeometry(0.11, 0.22, 0.14), M.dark); grip.position.set(0.32, -0.5, -0.72); grip.rotation.x = -0.22;
const stock = new THREE.Mesh(new THREE.BoxGeometry(0.12, 0.15, 0.42), M.wood); stock.position.set(0.32, -0.3, -0.35);
const boltHandle = new THREE.Mesh(new THREE.CylinderGeometry(0.018, 0.018, 0.1, 6), M.steel); boltHandle.rotation.z = Math.PI / 2; boltHandle.position.set(0.4, -0.28, -0.8);
const neonStrip = new THREE.Mesh(new THREE.BoxGeometry(0.035, 0.035, 0.85), M.neon); neonStrip.position.set(0.32, -0.13, -1.6);
g.add(body, barrel, muzzleBrake, scope, scopeLens, scopeMount, grip, stock, boltHandle, neonStrip);
handPos = { x: 0.34, y: -0.43, z: -0.88, rx: -0.2, ry: 0, rz: 0 };
} else if (id === "chainsaw_mythic") {
const engine = new THREE.Mesh(new THREE.BoxGeometry(0.22, 0.22, 0.28), M.metal);
engine.position.set(0.34, -0.28, -0.68);
const engineTop = new THREE.Mesh(new THREE.BoxGeometry(0.18, 0.08, 0.2), M.orange);
engineTop.position.set(0.34, -0.16, -0.68);
const fuelTank = new THREE.Mesh(new THREE.BoxGeometry(0.16, 0.14, 0.16), M.dark);
fuelTank.position.set(0.34, -0.4, -0.62);
const handleTop = new THREE.Mesh(new THREE.BoxGeometry(0.06, 0.06, 0.34), M.dark);
handleTop.position.set(0.34, -0.14, -0.86);
const handleSide = new THREE.Mesh(new THREE.BoxGeometry(0.06, 0.2, 0.06), M.dark);
handleSide.position.set(0.2, -0.24, -0.72);
const grip = new THREE.Mesh(new THREE.BoxGeometry(0.09, 0.18, 0.11), M.dark);
grip.position.set(0.34, -0.48, -0.6);
grip.rotation.x = 0.18;
const trigger = new THREE.Mesh(new THREE.BoxGeometry(0.035, 0.05, 0.03), M.steel);
trigger.position.set(0.34, -0.42, -0.72);
const barPlate = new THREE.Mesh(new THREE.BoxGeometry(0.09, 0.09, 0.26), M.metal);
barPlate.position.set(0.34, -0.3, -0.95);
const blade = new THREE.Mesh(new THREE.BoxGeometry(0.055, 0.15, 0.78), M.steel);
blade.position.set(0.34, -0.3, -1.4);
const bladeCore = new THREE.Mesh(new THREE.BoxGeometry(0.036, 0.11, 0.76), M.dark);
bladeCore.position.set(0.34, -0.3, -1.4);
const bladeTip = new THREE.Mesh(new THREE.BoxGeometry(0.055, 0.15, 0.1), M.steel);
bladeTip.position.set(0.34, -0.3, -1.82);
bladeTip.rotation.x = -0.25;
g.add(engine, engineTop, fuelTank, handleTop, handleSide, grip, trigger, barPlate, blade, bladeCore, bladeTip);
for (let i = 0; i < 9; i++) {
const toothUp = new THREE.Mesh(new THREE.BoxGeometry(0.018, 0.03, 0.03), M.steel);
toothUp.position.set(0.34, -0.222, -1.06 - i * 0.085);
g.add(toothUp);
const toothDown = new THREE.Mesh(new THREE.BoxGeometry(0.018, 0.03, 0.03), M.steel);
toothDown.position.set(0.34, -0.378, -1.06 - i * 0.085);
g.add(toothDown);
}
const sawGlow = new THREE.Mesh(new THREE.BoxGeometry(0.02, 0.02, 0.7), M.orange);
sawGlow.position.set(0.32, -0.3, -1.4);
g.add(sawGlow);
handPos = { x: 0.34, y: -0.5, z: -0.6, rx: 0.18, ry: 0, rz: 0 };
} else if (id === "minigun_chaos") {
const body = new THREE.Mesh(new THREE.CylinderGeometry(0.14, 0.14, 0.85, 10), M.metal); body.rotation.x = Math.PI / 2; body.position.set(0.32, -0.26, -0.95);
const backPlate = new THREE.Mesh(new THREE.CylinderGeometry(0.15, 0.15, 0.08, 10), M.dark); backPlate.rotation.x = Math.PI / 2; backPlate.position.set(0.32, -0.26, -0.52);
for (let i = 0; i < 6; i++) {
const angle = (i / 6) * Math.PI * 2;
const brl = new THREE.Mesh(new THREE.CylinderGeometry(0.022, 0.022, 0.9, 6), M.steel);
brl.rotation.x = Math.PI / 2;
brl.position.set(0.32 + Math.cos(angle) * 0.075, -0.26 + Math.sin(angle) * 0.075, -1.65);
g.add(brl);
}
const drum = new THREE.Mesh(new THREE.CylinderGeometry(0.22, 0.22, 0.32, 12), M.dark); drum.position.set(0.32, -0.55, -0.65); drum.rotation.z = Math.PI / 2;
const drumRing = new THREE.Mesh(new THREE.TorusGeometry(0.22, 0.02, 6, 12), M.gold); drumRing.rotation.y = Math.PI / 2; drumRing.position.set(0.32, -0.55, -0.65);
const grip = new THREE.Mesh(new THREE.BoxGeometry(0.12, 0.24, 0.15), M.dark); grip.position.set(0.32, -0.62, -0.38); grip.rotation.x = 0.25;
const glowRing = new THREE.Mesh(new THREE.TorusGeometry(0.15, 0.02, 6, 12), M.neonPink); glowRing.rotation.y = Math.PI / 2; glowRing.position.set(0.32, -0.26, -0.52);
const glowRing2 = new THREE.Mesh(new THREE.TorusGeometry(0.15, 0.02, 6, 12), M.neonPink); glowRing2.rotation.y = Math.PI / 2; glowRing2.position.set(0.32, -0.26, -1.38);
g.add(body, backPlate, drum, drumRing, grip, glowRing, glowRing2);
handPos = { x: 0.34, y: -0.55, z: -0.42, rx: 0.2, ry: 0, rz: 0 };
} else {
const body = new THREE.Mesh(new THREE.BoxGeometry(0.13, 0.16, 0.42), M.metal); body.position.set(0.32, -0.28, -0.7);
const grip = new THREE.Mesh(new THREE.BoxGeometry(0.11, 0.2, 0.13), M.dark); grip.position.set(0.32, -0.48, -0.58);
g.add(body, grip);
}
const hand = getHandGroup(handPos.x, handPos.y, handPos.z, handPos.rx, handPos.ry, handPos.rz);
g.add(hand);
return { group: g, hand, magazine, bolt };
}

function buildThrowableModel(id) {
const g = new THREE.Group();
let itemMesh = null;
const handPos = { x: 0.34, y: -0.48, z: -0.92, rx: -0.15, ry: 0, rz: 0 };
const itemPos = { x: 0.34, y: -0.48, z: -1.05 };

getItemMaterials();

if (id === "item_toiletpaper") {
const rollMat = new THREE.MeshStandardMaterial({ color: 0xf5f5f0, roughness: 0.85 });
const innerMat = new THREE.MeshStandardMaterial({ color: 0xa08060, roughness: 0.9 });
itemMesh = new THREE.Mesh(new THREE.CylinderGeometry(0.13, 0.13, 0.20, 14), rollMat);
itemMesh.rotation.z = Math.PI / 2;
itemMesh.position.set(itemPos.x, itemPos.y, itemPos.z);
const inner = new THREE.Mesh(new THREE.CylinderGeometry(0.042, 0.042, 0.21, 10), innerMat);
inner.rotation.z = Math.PI / 2;
inner.position.set(itemPos.x, itemPos.y, itemPos.z);
g.add(itemMesh, inner);
} else if (id === "item_stub") {
itemMesh = new THREE.Mesh(new THREE.SphereGeometry(0.14, 16, 12), MAT_STUB);
itemMesh.scale.set(0.85, 1.35, 0.85);
itemMesh.position.set(itemPos.x, itemPos.y, itemPos.z);
g.add(itemMesh);
} else if (id === "item_rag") {
itemMesh = new THREE.Mesh(new THREE.PlaneGeometry(0.42, 0.42), MAT_RAG);
itemMesh.position.set(itemPos.x, itemPos.y, itemPos.z);
itemMesh.rotation.set(0.15, 0.35, 0.32);
g.add(itemMesh);
} else if (id === "item_handkerchief") {
itemMesh = new THREE.Mesh(new THREE.PlaneGeometry(0.40, 0.40), MAT_HANDKERCHIEF);
itemMesh.position.set(itemPos.x, itemPos.y, itemPos.z);
itemMesh.rotation.set(0.1, -0.2, -0.15);
g.add(itemMesh);
}
const hand = getHandGroup(handPos.x, handPos.y, handPos.z, handPos.rx, handPos.ry, handPos.rz);
g.add(hand);
return { group: g, hand, itemMesh };
}

function rebuildWeapon() {
if (weapon) {
camera.remove(weapon);
weapon.traverse((o) => { if (o.isMesh && o.geometry) o.geometry.dispose(); });
}
weaponMagazine = null;
boltMesh = null;
weaponHand = null;
droppedMagazine = null;
throwAnimTimer = 0;
isThrowing = false;
chainsawRunning = false;
sawProgress = 0;
sawTimer = 0;
sawTarget = null;
if (isThrowable(currentWeapon)) {
const built = buildThrowableModel(currentWeapon);
weapon = built.group;
weaponHand = built.hand;
weapon.userData.kind = "throwable";
} else {
const built = buildWeaponModel(currentWeapon);
weapon = built.group;
weaponHand = built.hand;
weaponMagazine = built.magazine;
boltMesh = built.bolt;
weapon.userData.kind = "gun";
}
if (!weapon.userData.phaseFlags) weapon.userData.phaseFlags = {};
camera.add(weapon);
}
function applyWeaponStats() {
const w = getWeaponData(currentWeapon);
reloadDuration = w.fireDelay > 300 ? 2.8 : (w.fireDelay > 150 ? 2.2 : 1.8);
if (isThrowable(currentWeapon)) { ammo = 1; reserveAmmo = 9999; }
else if (currentWeapon === "chainsaw_mythic") { ammo = 0; reserveAmmo = 0; }
else if (w.magSize) {
ammo = Math.min(Math.max(ammo, 0), w.magSize);
reserveAmmo = Math.max(reserveAmmo, w.reserveSize);
}
rebuildWeapon();
updateHUD();
updateWeaponName();
}
function updateWeaponName() {
if (!weaponNameEl) return;
weaponNameEl.textContent = getWeaponData(currentWeapon).name;
}

function createEnemy() {
const geo = getEnemyGeometries();
const enemy = new THREE.Group();

const skinMat = new THREE.MeshLambertMaterial({ color: 0xc9a07a });
const skinDark = new THREE.MeshLambertMaterial({ color: 0xa8805a });
const shirtMat = new THREE.MeshLambertMaterial({ color: 0x50627a });
const pantsMat = new THREE.MeshLambertMaterial({ color: 0x2e3a4a });
const bootsMat = new THREE.MeshLambertMaterial({ color: 0x1c1c22 });
const vestMat = new THREE.MeshLambertMaterial({ color: 0x2f4636 });
const helmetMat = new THREE.MeshLambertMaterial({ color: 0x2f2f38 });
const eyeWhiteMat = new THREE.MeshStandardMaterial({ color: 0xf2f2f2, roughness: 0.4 });
const eyeRedMat = new THREE.MeshBasicMaterial({ color: 0xff2244 });
const mouthMat = new THREE.MeshLambertMaterial({ color: 0x2a1010 });
const browMat = new THREE.MeshLambertMaterial({ color: 0x1a1008 });
const gunMetal = new THREE.MeshLambertMaterial({ color: 0x2a2a32 });
const gunDark = new THREE.MeshLambertMaterial({ color: 0x101015 });
const gunGlow = new THREE.MeshBasicMaterial({ color: 0xff2266 });

const hips = new THREE.Group();
hips.position.y = 0.99;
enemy.add(hips);

const pelvisMesh = new THREE.Mesh(geo.pelvis, pantsMat);
pelvisMesh.scale.set(1.0, 0.9, 0.85);
pelvisMesh.position.y = -0.06;
pelvisMesh.userData.zone = "body";
hips.add(pelvisMesh);

const torso = new THREE.Group();
torso.position.y = 0.05;
hips.add(torso);

const waistMesh = new THREE.Mesh(geo.waist, shirtMat);
waistMesh.scale.set(1.0, 0.9, 0.75);
waistMesh.position.y = 0.10;
waistMesh.userData.zone = "body";
torso.add(waistMesh);

const chestMesh = new THREE.Mesh(geo.chest, shirtMat);
chestMesh.scale.set(1.10, 1.0, 0.75);
chestMesh.position.y = 0.28;
chestMesh.castShadow = !LOW_END;
chestMesh.userData.zone = "body";
torso.add(chestMesh);

const chestTopMesh = new THREE.Mesh(geo.chestTop, shirtMat);
chestTopMesh.scale.set(1.10, 0.85, 0.75);
chestTopMesh.position.y = 0.40;
chestTopMesh.castShadow = !LOW_END;
chestTopMesh.userData.zone = "body";
torso.add(chestTopMesh);

const vestMesh = new THREE.Mesh(geo.vest, vestMat);
vestMesh.scale.set(1.08, 1.0, 0.80);
vestMesh.position.y = 0.30;
vestMesh.castShadow = !LOW_END;
vestMesh.userData.zone = "armor";
torso.add(vestMesh);

const collarMesh = new THREE.Mesh(geo.collar, shirtMat);
collarMesh.scale.set(1.0, 1.0, 0.75);
collarMesh.position.y = 0.52;
collarMesh.userData.zone = "head";
torso.add(collarMesh);

const neckMesh = new THREE.Mesh(geo.neck, skinMat);
neckMesh.position.y = 0.64;
neckMesh.userData.zone = "head";
torso.add(neckMesh);

const headGroup = new THREE.Group();
headGroup.position.y = 0.75;
headGroup.rotation.x = 0.03;
torso.add(headGroup);

const headMesh = new THREE.Mesh(geo.head, skinMat);
headMesh.scale.set(0.98, 1.10, 1.0);
headMesh.position.y = 0.10;
headMesh.castShadow = !LOW_END;
headMesh.userData.zone = "head";
headGroup.add(headMesh);

const jawMesh = new THREE.Mesh(geo.jaw, skinDark);
jawMesh.scale.set(0.98, 1, 1);
jawMesh.position.set(0, 0.015, -0.045);
jawMesh.userData.zone = "head";
headGroup.add(jawMesh);

const noseMesh = new THREE.Mesh(geo.nose, skinDark);
noseMesh.rotation.x = Math.PI / 2;
noseMesh.position.set(0, 0.10, -0.118);
noseMesh.userData.zone = "head";
headGroup.add(noseMesh);

const eyeL = new THREE.Mesh(geo.eyeWhite, eyeWhiteMat);
eyeL.position.set(-0.046, 0.145, -0.098);
eyeL.scale.set(1, 0.75, 1);
eyeL.userData.zone = "head";
headGroup.add(eyeL);
const eyeR = new THREE.Mesh(geo.eyeWhite, eyeWhiteMat);
eyeR.position.set(0.046, 0.145, -0.098);
eyeR.scale.set(1, 0.75, 1);
eyeR.userData.zone = "head";
headGroup.add(eyeR);

const pupilL = new THREE.Mesh(geo.eyePupil, eyeRedMat);
pupilL.position.set(-0.046, 0.145, -0.122);
pupilL.userData.noFlash = true;
headGroup.add(pupilL);
const pupilR = new THREE.Mesh(geo.eyePupil, eyeRedMat);
pupilR.position.set(0.046, 0.145, -0.122);
pupilR.userData.noFlash = true;
headGroup.add(pupilR);

const browL = new THREE.Mesh(geo.eyebrow, browMat);
browL.position.set(-0.050, 0.180, -0.112);
browL.rotation.z = 0.28;
browL.userData.zone = "head";
headGroup.add(browL);
const browR = new THREE.Mesh(geo.eyebrow, browMat);
browR.position.set(0.050, 0.180, -0.112);
browR.rotation.z = -0.28;
browR.userData.zone = "head";
headGroup.add(browR);

const mouthMesh = new THREE.Mesh(geo.mouth, mouthMat);
mouthMesh.position.set(0, 0.025, -0.115);
mouthMesh.userData.zone = "head";
headGroup.add(mouthMesh);

const helmetMesh = new THREE.Mesh(geo.helmet, helmetMat);
helmetMesh.scale.set(1.02, 1.10, 1.05);
helmetMesh.position.y = 0.145;
helmetMesh.castShadow = !LOW_END;
helmetMesh.userData.zone = "head";
headGroup.add(helmetMesh);

if (!VERY_LOW) {
const brimMesh = new THREE.Mesh(geo.helmetBrim, helmetMat);
brimMesh.position.set(0, 0.115, -0.075);
brimMesh.rotation.x = 0.14;
brimMesh.userData.zone = "head";
headGroup.add(brimMesh);
}

const shoulderL = new THREE.Group();
shoulderL.position.set(-0.32, 0.42, 0);
torso.add(shoulderL);

if (!VERY_LOW) {
const padL = new THREE.Mesh(geo.shoulderPad, vestMat);
padL.scale.set(1.1, 0.65, 1.1);
padL.position.y = 0;
padL.userData.zone = "armor";
shoulderL.add(padL);
}

const upperArmL = new THREE.Mesh(geo.upperArm, shirtMat);
upperArmL.position.y = -0.18;
upperArmL.castShadow = !LOW_END;
upperArmL.userData.zone = "armor";
shoulderL.add(upperArmL);

const bicepL = new THREE.Mesh(geo.bicep, shirtMat);
bicepL.position.y = -0.07;
bicepL.userData.zone = "armor";
shoulderL.add(bicepL);

const elbowL = new THREE.Group();
elbowL.position.y = -0.34;
shoulderL.add(elbowL);

const forearmL = new THREE.Mesh(geo.forearm, skinMat);
forearmL.position.y = -0.15;
forearmL.userData.zone = "armor";
elbowL.add(forearmL);

const handL = new THREE.Mesh(geo.hand, skinMat);
handL.scale.set(0.95, 1.05, 0.78);
handL.position.y = -0.32;
handL.userData.zone = "armor";
elbowL.add(handL);

const shoulderR = new THREE.Group();
shoulderR.position.set(0.32, 0.42, 0);
torso.add(shoulderR);

if (!VERY_LOW) {
const padR = new THREE.Mesh(geo.shoulderPad, vestMat);
padR.scale.set(1.1, 0.65, 1.1);
padR.position.y = 0;
padR.userData.zone = "armor";
shoulderR.add(padR);
}

const upperArmR = new THREE.Mesh(geo.upperArm, shirtMat);
upperArmR.position.y = -0.18;
upperArmR.castShadow = !LOW_END;
upperArmR.userData.zone = "armor";
shoulderR.add(upperArmR);

const bicepR = new THREE.Mesh(geo.bicep, shirtMat);
bicepR.position.y = -0.07;
bicepR.userData.zone = "armor";
shoulderR.add(bicepR);

const elbowR = new THREE.Group();
elbowR.position.y = -0.34;
shoulderR.add(elbowR);

const forearmR = new THREE.Mesh(geo.forearm, skinMat);
forearmR.position.y = -0.15;
forearmR.userData.zone = "armor";
elbowR.add(forearmR);

const handR = new THREE.Mesh(geo.hand, skinMat);
handR.scale.set(0.95, 1.05, 0.78);
handR.position.y = -0.32;
handR.userData.zone = "armor";
elbowR.add(handR);

const gunGroup = new THREE.Group();
gunGroup.position.set(0, -0.36, 0);
gunGroup.rotation.x = -0.15;
elbowR.add(gunGroup);

const gunBody = new THREE.Mesh(geo.gunBody, gunMetal); gunBody.position.set(0, -0.12, 0); gunGroup.add(gunBody);
const gunBarrel = new THREE.Mesh(geo.gunBarrel, gunMetal); gunBarrel.position.set(0, -0.34, 0); gunGroup.add(gunBarrel);
const gunGrip = new THREE.Mesh(geo.gunGrip, gunDark); gunGrip.position.set(0, 0.02, -0.09); gunGrip.rotation.x = -0.2; gunGroup.add(gunGrip);
const gunMag = new THREE.Mesh(geo.gunMag, gunDark); gunMag.position.set(0, 0.04, -0.06); gunGroup.add(gunMag);
const gunStrip = new THREE.Mesh(geo.gunStrip, gunGlow); gunStrip.position.set(0.048, -0.12, 0); gunGroup.add(gunStrip);
const muzzlePoint = new THREE.Mesh(geo.muzzle, getMuzzleMaterial()); muzzlePoint.position.set(0, -0.42, 0); muzzlePoint.visible = false; gunGroup.add(muzzlePoint);

const hipL = new THREE.Group();
hipL.position.set(-0.16, -0.10, 0);
hips.add(hipL);

const thighL = new THREE.Mesh(geo.thigh, pantsMat);
thighL.position.y = -0.22;
thighL.castShadow = !LOW_END;
thighL.userData.zone = "body";
hipL.add(thighL);

const kneeL = new THREE.Group();
kneeL.position.y = -0.48;
hipL.add(kneeL);

if (!VERY_LOW) {
const kneePadL = new THREE.Mesh(geo.kneePad, bootsMat);
kneePadL.scale.set(0.9, 0.7, 0.9);
kneePadL.position.set(0, 0.02, -0.02);
kneePadL.userData.zone = "body";
kneeL.add(kneePadL);
}

const calfL = new THREE.Mesh(geo.calf, pantsMat);
calfL.position.y = -0.18;
calfL.userData.zone = "body";
kneeL.add(calfL);

const footL = new THREE.Mesh(geo.foot, bootsMat);
footL.position.set(0, -0.40, -0.05);
footL.userData.zone = "body";
kneeL.add(footL);

const hipR = new THREE.Group();
hipR.position.set(0.16, -0.10, 0);
hips.add(hipR);

const thighR = new THREE.Mesh(geo.thigh, pantsMat);
thighR.position.y = -0.22;
thighR.castShadow = !LOW_END;
thighR.userData.zone = "body";
hipR.add(thighR);

const kneeR = new THREE.Group();
kneeR.position.y = -0.48;
hipR.add(kneeR);

if (!VERY_LOW) {
const kneePadR = new THREE.Mesh(geo.kneePad, bootsMat);
kneePadR.scale.set(0.9, 0.7, 0.9);
kneePadR.position.set(0, 0.02, -0.02);
kneePadR.userData.zone = "body";
kneeR.add(kneePadR);
}

const calfR = new THREE.Mesh(geo.calf, pantsMat);
calfR.position.y = -0.18;
calfR.userData.zone = "body";
kneeR.add(calfR);

const footR = new THREE.Mesh(geo.foot, bootsMat);
footR.position.set(0, -0.40, -0.05);
footR.userData.zone = "body";
kneeR.add(footR);

const baseShoulderRx = 0.35;
const baseElbowRx = 1.15;
shoulderR.rotation.x = baseShoulderRx;
shoulderR.rotation.z = -0.18;
elbowR.rotation.x = baseElbowRx;
shoulderL.rotation.x = 0.15;
shoulderL.rotation.z = 0.22;
elbowL.rotation.x = 1.35;

enemy.scale.set(1.32, 1.04, 1.24);

enemy.userData = {
health: 100, maxHealth: 100,
speed: 1.0 + Math.random() * 0.35,
cooldown: 1.5 + Math.random() * 1.5,
attackRange: 11, preferredRange: 9,
alive: true, dying: false, deathTimer: 0,
walkPhase: Math.random() * 10,
hitFlash: 0, shooting: 0, walkAmp: 0,
stunTimer: 0,
isBoss: false, bossIndex: -1, bossDefeated: false,
hips, torso, headGroup,
shoulderL, shoulderR, elbowL, elbowR,
hipL, hipR, kneeL, kneeR,
gun: gunGroup, gunMuzzle: muzzlePoint,
baseShoulderRx, baseElbowRx
};

const materials = [];
const seen = new Set();
enemy.traverse((obj) => {
if (obj.isMesh && obj.material && obj.material.emissive !== undefined && !obj.userData.noFlash) {
if (!seen.has(obj.material)) {
seen.add(obj.material);
materials.push({ material: obj.material, original: obj.material.emissive.getHex() });
}
}
});
enemy.userData.materials = materials;

respawnEnemy(enemy);
scene.add(enemy);
enemies.push(enemy);
}
function resetEnemyMaterials(enemy) {
if (!enemy.userData.materials) return;
for (const m of enemy.userData.materials) m.material.emissive.setHex(m.original);
}
function respawnEnemy(enemy) {
const ud = enemy.userData;
let px, pz, tries = 0;
do {
px = -27 + Math.random() * 54;
pz = -28 + Math.random() * 53;
tries++;
} while ((collidesAt(px, pz, 0.8) || Math.hypot(px - player.position.x, pz - player.position.z) < 12) && tries < 80);
enemy.position.set(px, 0, pz);
enemy.rotation.set(0, 0, 0);
enemy.scale.set(1.32, 1.04, 1.24);
enemy.visible = true;
ud.health = ud.maxHealth;
ud.alive = true;
ud.dying = false;
ud.deathTimer = 0;
ud.cooldown = 1.5 + Math.random() * 1.5;
ud.shooting = 0;
ud.hitFlash = 0;
ud.walkAmp = 0;
ud.stunTimer = 0;
ud.isBoss = false;
ud.bossIndex = -1;
ud.bossDefeated = false;
ud.attackRange = 11;
ud.preferredRange = 9;
ud.speed = 1.0 + Math.random() * 0.35 + (currentLevel - 1) * 0.02;
resetEnemyMaterials(enemy);
ud.shoulderR.rotation.set(ud.baseShoulderRx, 0, -0.18);
ud.elbowR.rotation.set(ud.baseElbowRx, 0, 0);
ud.shoulderL.rotation.set(0.15, 0, 0.22);
ud.elbowL.rotation.set(1.35, 0, 0);
ud.hipL.rotation.set(0, 0, 0);
ud.hipR.rotation.set(0, 0, 0);
ud.kneeL.rotation.set(-0.08, 0, 0);
ud.kneeR.rotation.set(-0.08, 0, 0);
ud.torso.rotation.set(0, 0, 0);
ud.torso.position.y = 0.05;
ud.headGroup.rotation.set(0.03, 0, 0);
}

function removeBoss() {
for (const e of enemies) {
if (e.userData.isBoss) {
e.userData.isBoss = false;
e.userData.bossIndex = -1;
e.userData.bossDefeated = false;
e.userData.alive = false;
e.userData.dying = false;
e.visible = false;
}
}
activeBoss = null;
bossBar.style.display = "none";
bossDefeatPending = false;
}

function spawnBoss(index) {
removeBoss();
if (!enemies.length) return;
const data = BOSSES[index];
const boss = enemies[0];
const ud = boss.userData;
boss.visible = true;
boss.position.set(0, 0, -18);
boss.rotation.set(0, 0, 0);
boss.scale.set(data.scale * 1.32, data.scale * 0.55, data.scale * 1.24);
ud.alive = true;
ud.dying = false;
ud.deathTimer = 0;
ud.isBoss = true;
ud.bossIndex = index;
ud.bossDefeated = false;
ud.maxHealth = data.hp;
ud.health = data.hp;
ud.speed = data.speed;
ud.cooldown = 2.2;
ud.attackRange = 42;
ud.preferredRange = 11;
ud.walkAmp = 0;
ud.walkPhase = 0;
ud.hitFlash = 0;
ud.shooting = 0;
ud.stunTimer = 0;
resetEnemyMaterials(boss);
for (const m of ud.materials) m.material.emissive.setHex(data.aura);
activeBoss = boss;
bossBar.style.display = "block";
bossBarName.textContent = data.name + " • " + data.diff;
bossBarFill.style.width = "100%";
playBossSound();
showLevelMessage("БОСС: " + data.name, 2600);
}

function startLevel(level) {
currentLevel = Math.max(1, Math.min(100, level));
if (currentLevel > maxLevelReached) maxLevelReached = currentLevel;
saveProgress();
levelKills = 0;
inBossFight = (currentLevel % 10 === 0);
bossDefeatPending = false;
if (inBossFight) {
currentBossIndex = Math.floor(currentLevel / 10) - 1;
enemiesToKill = 0;
for (const e of enemies) {
if (e.userData.isBoss) continue;
e.userData.alive = false;
e.userData.dying = false;
e.visible = false;
}
spawnBoss(currentBossIndex);
} else {
currentBossIndex = -1;
removeBoss();
enemiesToKill = 5 + Math.floor(currentLevel * 0.6);
for (const e of enemies) respawnEnemy(e);
}
applyWeaponStats();
updateHUD();
showLevelMessage("УРОВЕНЬ " + currentLevel + (inBossFight ? " — БОСС" : ""), 1800);
}

function showLevelMessage(text, ms) {
if (!message) return;
message.innerHTML = text;
setTimeout(() => { if (!ended && !playerDying && message.innerHTML === text) message.innerHTML = ""; }, ms || 1500);
}

function nextLevel() {
if (currentLevel >= 100) {
showLevelMessage("ВСЕ 100 УРОВНЕЙ ПРОЙДЕНЫ!", 5000);
ended = true;
firing = false;
gameOver();
return;
}
const next = currentLevel + 1;
const skullReward = 30 + currentLevel * 10;
addSkulls(skullReward);
startLevel(next);
}

function onBossDefeated(index) {
if (bossDefeatPending) return;
bossDefeatPending = true;
const data = BOSSES[index];
addSkulls(data.reward);
saveProgress();
playBossDefeatSound();
bossBar.style.display = "none";
showLevelMessage("БОСС ПОВЕРЖЕН!<br>" + data.name + "<br>+" + data.reward + " черепков", 3500);
setTimeout(() => {
bossDefeatPending = false;
if (ended || playerDying) return;
if (currentLevel >= 100) {
ended = true;
firing = false;
gameOver();
return;
}
startLevel(currentLevel + 1);
}, 3600);
}

function updateAdaptiveQuality(delta) {
if (!adaptiveQualityEnabled || !renderer) return;
fpsSampleTimer += delta;
if (fpsSampleTimer >= 0.5) {
const fps = 1 / Math.max(delta, 0.0001);
fpsSamples.push(fps);
if (fpsSamples.length > 4) fpsSamples.shift();
const avgFps = fpsSamples.reduce((a, b) => a + b, 0) / fpsSamples.length;
if (perfBadgeVisible) perfBadge.textContent = "FPS: " + Math.round(avgFps) + " | PR: " + dynamicPixelRatio.toFixed(2) + (bloomEnabled ? "" : " | no bloom");
if (avgFps < (LOW_END ? 28 : 35)) {
if (bloomEnabled && composer && !VERY_LOW) {
bloomEnabled = false;
composer = null;
bloomPass = null;
fpsSamples = [];
} else if (dynamicPixelRatio > 0.75) {
dynamicPixelRatio = Math.max(0.75, dynamicPixelRatio - 0.15);
renderer.setPixelRatio(dynamicPixelRatio);
if (composer) composer.setPixelRatio(dynamicPixelRatio);
fpsSamples = [];
}
}
fpsSampleTimer = 0;
}
}

let lastFrameTime = 0;
function animate() {
if (!started) return;
requestAnimationFrame(animate);
if (FRAME_INTERVAL > 0) {
const now = performance.now();
if (now - lastFrameTime < FRAME_INTERVAL) return;
lastFrameTime = now;
}
const delta = Math.min(clock.getDelta(), isMobile ? 0.08 : 0.05);
updateAdaptiveQuality(delta);

if (playerDying) {
updatePlayerDeath(delta);
updateEnemies(delta);
updateParticles(delta);
updateBloodParticles(delta);
updateBloodDecals(delta);
updateBloodPools(delta);
updateShells(delta);
updateTracers(delta);
updateBullets(delta);
updateEnemyBullets(delta);
updateThrownItems(delta);
updateChainsawSparks(delta);
} else if (!ended && !shopOpen) {
updatePlayer(delta);
updateEnemies(delta);
updateParticles(delta);
updateBloodParticles(delta);
updateBloodDecals(delta);
updateBloodPools(delta);
updateShells(delta);
updateTracers(delta);
updateBullets(delta);
updateEnemyBullets(delta);
updateThrownItems(delta);
updateWeapon(delta);
updateReload(delta);
updateChainsaw(delta);
updateChainsawSparks(delta);
updateScreenBlood(delta);
checkShopTrigger();
if (firing && !reloading && currentWeapon !== "chainsaw_mythic") shoot();
}
if (enemyFlashTimer > 0) {
enemyFlashTimer -= delta;
if (enemyFlashLightShared) enemyFlashLightShared.intensity = Math.max(0, (enemyFlashTimer / ENEMY_FLASH_DURATION) * 5);
} else if (enemyFlashLightShared && enemyFlashLightShared.intensity > 0) enemyFlashLightShared.intensity = 0;

updateBossBar();
updateGasolineUI();

if (sky && camera) sky.position.copy(camera.position);

if (composer) composer.render();
else renderer.render(scene, camera);
}

function updateBossBar() {
if (activeBoss && activeBoss.userData.alive && activeBoss.userData.isBoss && !activeBoss.userData.dying) {
bossBar.style.display = "block";
const pct = Math.max(0, Math.min(1, activeBoss.userData.health / activeBoss.userData.maxHealth));
bossBarFill.style.width = (pct * 100).toFixed(1) + "%";
} else if (!activeBoss || !activeBoss.userData.isBoss) {
bossBar.style.display = "none";
}
}

function updateGasolineUI() {
if (currentWeapon === "chainsaw_mythic" && hasChainsaw) {
gasoBar.style.display = "block";
gasoFill.style.width = ((gasoline / GASOLINE_MAX) * 100).toFixed(1) + "%";
} else {
gasoBar.style.display = "none";
}
}

function updateChainsawSparks(delta) {
for (let i = chainsawSparks.length - 1; i >= 0; i--) {
const s = chainsawSparks[i];
s.position.addScaledVector(s.userData.velocity, delta);
s.userData.velocity.y -= 16 * delta;
s.userData.life -= delta;
s.scale.multiplyScalar(0.94);
if (s.userData.life <= 0) {
scene.remove(s);
chainsawSparks.splice(i, 1);
}
}
}

function updateScreenBlood(delta) {
if (screenBloodIntensity > 0) {
screenBloodIntensity -= delta * 0.4;
if (screenBloodIntensity < 0) screenBloodIntensity = 0;
bloodOverlay.style.opacity = Math.min(screenBloodIntensity, 1).toString();
}
}
function splashBloodOnScreen() {
const now = performance.now();
if (now - lastSplashTime < 220) return;
lastSplashTime = now;
bloodCtx.clearRect(0, 0, bloodSplatterCanvas.width, bloodSplatterCanvas.height);
const w = bloodSplatterCanvas.width;
const h = bloodSplatterCanvas.height;
const cx = w / 2;
const cy = h / 2;
const blobCount = LOW_END ? 5 : (isMobile ? 8 : 14);
for (let i = 0; i < blobCount; i++) {
const angle = Math.random() * Math.PI * 2;
const dist = Math.random() * Math.min(w, h) * 0.45;
const x = cx + Math.cos(angle) * dist;
const y = cy + Math.sin(angle) * dist;
const r = 6 + Math.random() * 22;
const g = bloodCtx.createRadialGradient(x, y, 0, x, y, r);
g.addColorStop(0, "rgba(120,0,0,0.95)");
g.addColorStop(0.55, "rgba(80,0,0,0.7)");
g.addColorStop(1, "rgba(40,0,0,0)");
bloodCtx.fillStyle = g;
bloodCtx.beginPath();
bloodCtx.arc(x, y, r, 0, Math.PI * 2);
bloodCtx.fill();
}
bloodSplatterCanvas.classList.add("show");
bloodOverlay.classList.add("show");
screenBloodIntensity = 0.85;
bloodOverlay.style.opacity = "0.85";
setTimeout(() => { bloodSplatterCanvas.classList.remove("show"); }, 550);
setTimeout(() => { bloodCtx.clearRect(0, 0, bloodSplatterCanvas.width, bloodSplatterCanvas.height); }, 1100);
}
function updatePlayerDeath(delta) {
playerDeathTimer += delta;
const t = Math.min(playerDeathTimer / PLAYER_DEATH_DURATION, 1);
const ease = 1 - Math.pow(1 - t, 3);
player.position.y = THREE.MathUtils.lerp(playerDeathStartY, 0.35, ease);
camera.rotation.z = THREE.MathUtils.lerp(0, -Math.PI * 0.42, ease);
camera.rotation.x = THREE.MathUtils.lerp(playerDeathStartPitch, -0.55, ease);
player.rotation.z = THREE.MathUtils.lerp(0, -0.18, ease);
player.rotation.x = THREE.MathUtils.lerp(0, -0.12, ease);
if (t >= 1 && !deathMessageShown) {
deathMessageShown = true;
showDeathMessage();
}
}
function showDeathMessage() {
message.innerHTML = "ИГРА ОКОНЧЕНА<br>УРОВЕНЬ: " + currentLevel + "<br>СЧЁТ: " + score + "<br>БТВ: " + btv.toLocaleString() + "<br>ЧЕРЕПКОВ: " + skulls.toLocaleString() + "<br><br><button onclick='location.reload()'>ЗАНОВО</button>";
}
function updatePlayer(delta) {
const forward = new THREE.Vector3();
const right = new THREE.Vector3();
camera.getWorldDirection(forward);
forward.y = 0;
forward.normalize();
right.crossVectors(forward, new THREE.Vector3(0, 1, 0)).normalize();
let fp = 0, rp = 0;
if (keys["KeyW"]) fp++;
if (keys["KeyS"]) fp--;
if (keys["KeyD"]) rp++;
if (keys["KeyA"]) rp--;
fp += -joy.y;
rp += joy.x;
const direction = new THREE.Vector3();
direction.addScaledVector(forward, fp);
direction.addScaledVector(right, rp);
if (direction.length() > 1) direction.normalize();
const sprint = keys["ShiftLeft"] || keys["ShiftRight"];
const speed = sprint ? 10 : 6;
const stepX = direction.x * speed * delta;
const stepZ = direction.z * speed * delta;
let nextX = THREE.MathUtils.clamp(player.position.x + stepX, -MAP_LIMIT, MAP_LIMIT);
let nextZ = THREE.MathUtils.clamp(player.position.z + stepZ, -MAP_LIMIT, MAP_LIMIT);
if (!collidesAt(nextX, player.position.z)) player.position.x = nextX;
if (!collidesAt(player.position.x, nextZ)) player.position.z = nextZ;
if (jumpRequest && grounded) { jumpRequest = false; grounded = false; verticalVelocity = 8; }
verticalVelocity -= 22 * delta;
player.position.y += verticalVelocity * delta;
if (player.position.y <= 1.7) { player.position.y = 1.7; verticalVelocity = 0; grounded = true; }
player.rotation.y = yaw;
camera.rotation.x = pitch;
}
function enemyShoot(enemy) {
const ud = enemy.userData;
if (!ud.elbowR) return;
ud.shooting = 0.3;
playEnemyShootSound();
if (ud.gunMuzzle) {
ud.gunMuzzle.visible = true;
setTimeout(() => { if (ud.gunMuzzle) ud.gunMuzzle.visible = false; }, 70);
}
const muzzleWorld = new THREE.Vector3();
if (ud.gunMuzzle) ud.gunMuzzle.getWorldPosition(muzzleWorld);
else { muzzleWorld.copy(enemy.position); muzzleWorld.y += 1.3; }
const targetPos = player.position.clone();
targetPos.y -= 0.2;
const shots = ud.isBoss ? 3 : 1;
for (let s = 0; s < shots; s++) {
const direction = new THREE.Vector3().subVectors(targetPos, muzzleWorld).normalize();
const spread = ud.isBoss ? 0.13 : 0.1;
direction.x += (Math.random() - 0.5) * spread;
direction.y += (Math.random() - 0.5) * spread;
direction.z += (Math.random() - 0.5) * spread;
direction.normalize();
if (enemyBullets.length >= MAX_ENEMY_BULLETS) { scene.remove(enemyBullets.shift()); }
const bullet = new THREE.Mesh(enemyBulletGeoShared, enemyBulletMatShared);
bullet.position.copy(muzzleWorld);
bullet.userData.velocity = direction.clone().multiplyScalar(65);
bullet.userData.life = 2;
bullet.userData.damage = ud.isBoss ? (BOSSES[ud.bossIndex] ? BOSSES[ud.bossIndex].damage : 8) : 4;
scene.add(bullet);
enemyBullets.push(bullet);
}
if (enemyFlashLightShared) {
enemyFlashLightShared.position.copy(muzzleWorld);
enemyFlashLightShared.intensity = ud.isBoss ? 9 : 5;
enemyFlashTimer = ENEMY_FLASH_DURATION;
}
const tracerEnd = muzzleWorld.clone().addScaledVector(new THREE.Vector3().subVectors(targetPos, muzzleWorld).normalize(), 4);
if (tracers.length >= MAX_TRACERS) {
const old = tracers.shift();
scene.remove(old);
if (old.geometry) old.geometry.dispose();
}
const tracerGeo = new THREE.BufferGeometry().setFromPoints([muzzleWorld.clone(), tracerEnd]);
const tracer = new THREE.Line(tracerGeo, tracerMatShared);
tracer.userData.life = 0.08;
tracer.userData.maxLife = 0.08;
scene.add(tracer);
tracers.push(tracer);
}
function updateEnemies(delta) {
for (const enemy of enemies) {
const ud = enemy.userData;
if (ud.hitFlash > 0) {
ud.hitFlash -= delta;
if (ud.hitFlash <= 0) {
if (ud.isBoss && ud.bossIndex >= 0) {
const aura = BOSSES[ud.bossIndex].aura;
for (const m of ud.materials) m.material.emissive.setHex(aura);
} else {
resetEnemyMaterials(enemy);
}
}
}
if (ud.dying) {
ud.deathTimer += delta;
const t = Math.min(ud.deathTimer / 0.7, 1);
const ease = 1 - Math.pow(1 - t, 3);
enemy.rotation.x = -Math.PI * 0.5 * ease;
enemy.position.y = ease * 0.13;
if (ud.deathTimer >= 2.2) {
if (ud.bossDefeated) {
enemy.visible = false;
ud.alive = false;
ud.dying = false;
ud.isBoss = false;
ud.bossIndex = -1;
enemy.position.y = 0;
enemy.rotation.x = 0;
} else {
enemy.rotation.x = 0;
enemy.position.y = 0;
respawnEnemy(enemy);
}
}
continue;
}
if (!ud.alive) continue;

if (ud.stunTimer > 0) {
ud.stunTimer -= delta;
const shiver = Math.sin(performance.now() * 0.045) * 0.045;
ud.torso.rotation.z = shiver;
ud.headGroup.rotation.z = -shiver * 1.4;
ud.walkAmp = 0;
ud.hipL.rotation.x = 0;
ud.hipR.rotation.x = 0;
ud.kneeL.rotation.x = -0.08;
ud.kneeR.rotation.x = -0.08;
continue;
}

let sepX = 0, sepZ = 0;
for (const other of enemies) {
if (other === enemy) continue;
const oud = other.userData;
if (!oud.alive || oud.dying) continue;
const ox = enemy.position.x - other.position.x;
const oz = enemy.position.z - other.position.z;
const od = Math.hypot(ox, oz) || 0.01;
if (od < 5.5) {
const force = (5.5 - od) / 5.5;
sepX += (ox / od) * force * 1.4;
sepZ += (oz / od) * force * 1.4;
}
}
const dx = player.position.x - enemy.position.x;
const dz = player.position.z - enemy.position.z;
const distance = Math.hypot(dx, dz) || 0.001;
let moving = false;
const attackRange = ud.attackRange;
const preferredRange = ud.preferredRange;
if (distance > preferredRange + 1) {
const nx = dx / distance;
const nz = dz / distance;
const amount = ud.speed * delta;
const stepX = nx * amount + sepX * delta;
const stepZ = nz * amount + sepZ * delta;
const nextX = enemy.position.x + stepX;
if (!collidesAt(nextX, enemy.position.z, 0.6)) { enemy.position.x = nextX; moving = true; }
const nextZ = enemy.position.z + stepZ;
if (!collidesAt(enemy.position.x, nextZ, 0.6)) { enemy.position.z = nextZ; moving = true; }
} else if (distance < preferredRange - 2) {
const nx = -dx / distance;
const nz = -dz / distance;
const amount = ud.speed * 0.5 * delta;
const nextX = enemy.position.x + nx * amount + sepX * delta;
if (!collidesAt(nextX, enemy.position.z, 0.6)) { enemy.position.x = nextX; moving = true; }
const nextZ = enemy.position.z + nz * amount + sepZ * delta;
if (!collidesAt(enemy.position.x, nextZ, 0.6)) { enemy.position.z = nextZ; moving = true; }
} else {
const strafe = Math.sin(ud.walkPhase * 0.4) * 0.5;
const nx = -dz / distance;
const nz = dx / distance;
const amount = ud.speed * 0.3 * delta * strafe;
const nextX = enemy.position.x + nx * amount + sepX * delta;
if (!collidesAt(nextX, enemy.position.z, 0.6)) { enemy.position.x = nextX; moving = true; }
const nextZ = enemy.position.z + nz * amount + sepZ * delta;
if (!collidesAt(enemy.position.x, nextZ, 0.6)) { enemy.position.z = nextZ; moving = true; }
}
const angle = Math.atan2(-dx, -dz);
enemy.rotation.y = angle;
const targetAmp = moving ? 1 : 0;
ud.walkAmp += (targetAmp - ud.walkAmp) * Math.min(1, delta * 8);
if (moving) ud.walkPhase += delta * 8;
else ud.walkPhase += delta * 1.5;
const phase = ud.walkPhase;
const walkAmp = ud.walkAmp;
const legSwing = Math.sin(phase);
ud.hipL.rotation.x = legSwing * 0.5 * walkAmp;
ud.hipR.rotation.x = -legSwing * 0.5 * walkAmp;
const kneeAmtL = Math.max(0, Math.sin(phase + Math.PI * 0.7)) * 0.9;
const kneeAmtR = Math.max(0, Math.sin(phase + Math.PI * 0.7 + Math.PI)) * 0.9;
ud.kneeL.rotation.x = -(0.08 + kneeAmtL * walkAmp);
ud.kneeR.rotation.x = -(0.08 + kneeAmtR * walkAmp);
const bob = Math.abs(Math.sin(phase)) * 0.03 * walkAmp;
const breathe = Math.sin(phase * 0.6) * 0.008 * (1 - walkAmp * 0.5);
ud.torso.position.y = 0.05 + bob + breathe;
ud.torso.rotation.z = Math.sin(phase) * 0.045 * walkAmp;
ud.torso.rotation.y = Math.sin(phase * 0.5) * 0.06 * walkAmp;
ud.headGroup.rotation.y = Math.sin(phase * 0.5) * 0.05 * walkAmp;
ud.headGroup.rotation.z = -Math.sin(phase) * 0.02 * walkAmp;
ud.headGroup.rotation.x = 0.03 + Math.sin(phase * 0.4) * 0.015 * walkAmp;

if (moving) {
ud.shoulderL.rotation.x = 0.15 - legSwing * 0.45 * walkAmp;
ud.shoulderL.rotation.z = 0.22;
ud.elbowL.rotation.x = 1.35 - Math.max(0, -legSwing) * 0.35 * walkAmp;
} else {
ud.shoulderL.rotation.x = 0.15;
ud.shoulderL.rotation.z = 0.22;
ud.elbowL.rotation.x = 1.35;
}
const shooting = ud.shooting > 0;
if (shooting) ud.shooting -= delta;
const recoilOffset = shooting ? Math.max(0, ud.shooting * 3) : 0;
ud.shoulderR.rotation.x = ud.baseShoulderRx - recoilOffset * 0.15 + Math.sin(phase) * 0.04 * walkAmp;
ud.elbowR.rotation.x = ud.baseElbowRx - recoilOffset * 0.25;
ud.shoulderR.rotation.z = -0.18;
ud.cooldown -= delta;
if (!playerDying && distance < attackRange && ud.cooldown <= 0 && ud.shooting <= 0) {
ud.cooldown = ud.isBoss ? BOSSES[ud.bossIndex].rate + Math.random() * 0.6 : 2.4 + Math.random() * 1.6;
enemyShoot(enemy);
}
}
}
function getMuzzleWorldPosition() {
if (isThrowable(currentWeapon)) {
const local = new THREE.Vector3(0.34, -0.48, -1.25);
const world = local.clone();
weapon.localToWorld(world);
return world;
}
if (currentWeapon === "chainsaw_mythic") {
const local = new THREE.Vector3(0.34, -0.3, -1.8);
const world = local.clone();
weapon.localToWorld(world);
return world;
}
const local = new THREE.Vector3(0.32, -0.22, -1.52);
const world = local.clone();
weapon.localToWorld(world);
return world;
}
function spawnShell() {
if (isThrowable(currentWeapon) || currentWeapon === "chainsaw_mythic") return;
if (shells.length >= MAX_SHELLS) { scene.remove(shells.shift()); }
const shell = new THREE.Mesh(shellGeoShared, shellMatShared);
const ejectLocal = new THREE.Vector3(0.5, -0.22, -0.65);
const ejectWorld = ejectLocal.clone();
weapon.localToWorld(ejectWorld);
shell.position.copy(ejectWorld);
const camDir = new THREE.Vector3();
camera.getWorldDirection(camDir);
const up = new THREE.Vector3(0, 1, 0);
const rightVec = new THREE.Vector3().crossVectors(camDir, up).normalize();
const velocity = new THREE.Vector3();
velocity.addScaledVector(rightVec, 2.4 + Math.random() * 1.6);
velocity.addScaledVector(up, 2.0 + Math.random() * 1.2);
velocity.addScaledVector(camDir, -0.6 - Math.random() * 0.6);
shell.userData.velocity = velocity;
shell.userData.spin = new THREE.Vector3((Math.random() - 0.5) * 30, (Math.random() - 0.5) * 30, (Math.random() - 0.5) * 30);
shell.userData.life = LOW_END ? 1.5 : (isMobile ? 2 : 7);
shell.userData.landed = false;
shell.userData.groundTimer = LOW_END ? 0.8 : (isMobile ? 1.2 : 5);
scene.add(shell);
shells.push(shell);
}
function spawnDroppedMagazine() {
if (!weaponMagazine || droppedMagazine) return;
const geo = new THREE.BoxGeometry(0.13, 0.3, 0.18);
const mat = new THREE.MeshStandardMaterial({ color: 0x1e2a38, roughness: 0.7, metalness: 0.5 });
const mag = new THREE.Mesh(geo, mat);
const worldPos = new THREE.Vector3();
weaponMagazine.getWorldPosition(worldPos);
mag.position.copy(worldPos);
const camDir = new THREE.Vector3();
camera.getWorldDirection(camDir);
const right = new THREE.Vector3().crossVectors(camDir, new THREE.Vector3(0, 1, 0)).normalize();
mag.userData.vel = new THREE.Vector3().addScaledVector(right, 1.2).addScaledVector(camDir, -0.4);
mag.userData.vel.y = -1.5;
mag.userData.spin = new THREE.Vector3((Math.random() - 0.5) * 12, (Math.random() - 0.5) * 12, (Math.random() - 0.5) * 12);
mag.userData.life = 3;
scene.add(mag);
droppedMagazine = mag;
}
function spawnTracer(start, end) {
if (tracers.length >= MAX_TRACERS) {
const old = tracers.shift();
scene.remove(old);
if (old.geometry) old.geometry.dispose();
}
const geometry = new THREE.BufferGeometry().setFromPoints([start.clone(), end.clone()]);
const line = new THREE.Line(geometry, tracerMatShared);
line.userData.life = 0.07;
line.userData.maxLife = 0.07;
scene.add(line);
tracers.push(line);
}
function spawnBullet(start, end) {
const direction = new THREE.Vector3().subVectors(end, start);
const distance = direction.length();
if (distance < 0.01) return;
direction.normalize();
const speed = 180;
if (bullets.length >= MAX_BULLETS) { scene.remove(bullets.shift()); }
const bullet = new THREE.Mesh(playerBulletGeoShared, playerBulletMatShared);
bullet.position.copy(start);
bullet.userData.velocity = direction.clone().multiplyScalar(speed);
bullet.userData.life = distance / speed;
scene.add(bullet);
bullets.push(bullet);
}

function spawnChainsawSpark(position, strength) {
if (!sparkGeoShared) return;
if (chainsawSparks.length >= SPARK_MAX) { scene.remove(chainsawSparks.shift()); }
const mat = sparkMatsShared[Math.floor(Math.random() * sparkMatsShared.length)];
const spark = new THREE.Mesh(sparkGeoShared, mat);
spark.position.copy(position);
spark.position.x += (Math.random() - 0.5) * 0.08;
spark.position.y += (Math.random() - 0.5) * 0.08;
spark.position.z += (Math.random() - 0.5) * 0.08;
const dir = new THREE.Vector3((Math.random() - 0.5) * 2, Math.random() * 1.4 + 0.4, (Math.random() - 0.5) * 2).normalize();
spark.userData.velocity = dir.multiplyScalar((3 + Math.random() * 4) * (strength || 1));
spark.userData.life = 0.35 + Math.random() * 0.35;
spark.scale.setScalar(0.6 + Math.random() * 0.8);
scene.add(spark);
chainsawSparks.push(spark);
}

function updateChainsaw(delta) {
if (currentWeapon !== "chainsaw_mythic") {
if (chainsawRunning) {
chainsawRunning = false;
playChainsawStopSound();
}
sawProgress = 0;
sawTimer = 0;
sawTarget = null;
return;
}
if (!hasChainsaw) return;
const wantRun = firing && !reloading;
if (wantRun && gasoline > 0) {
if (!chainsawRunning) {
chainsawRunning = true;
playChainsawStartSound();
}
gasoline = Math.max(0, gasoline - delta * GASOLINE_BURN_PER_SEC);
sawBladeSpin += delta * 42;
if (weapon) {
const glow = weapon.children.find((c) => c.isMesh && c.material === WEAPON_MATERIALS.orange);
if (glow) glow.rotation.z = Math.sin(sawBladeSpin) * 0.06;
}
const sparkPos = getMuzzleWorldPosition();
const sparkCount = LOW_END ? 1 : (isMobile ? 2 : 4);
for (let i = 0; i < sparkCount; i++) spawnChainsawSpark(sparkPos, 1);
if (gasoline <= 0) {
chainsawRunning = false;
playChainsawStopSound();
showLevelMessage("БЕНЗИН ЗАКОНЧИЛСЯ", 1400);
return;
}
for (const enemy of enemies) {
const ud = enemy.userData;
if (!ud.alive || ud.dying) continue;
const d = Math.hypot(enemy.position.x - player.position.x, enemy.position.z - player.position.z);
if (d < CHAINSAW_STUN_RANGE && ud.stunTimer <= 0) {
ud.stunTimer = 3 + Math.random() * 2;
}
}
let nearest = null;
let nearestDist = Infinity;
for (const enemy of enemies) {
const ud = enemy.userData;
if (!ud.alive || ud.dying) continue;
const d = Math.hypot(enemy.position.x - player.position.x, enemy.position.z - player.position.z);
if (d < CHAINSAW_RANGE && d < nearestDist) {
nearestDist = d;
nearest = enemy;
}
}
if (nearest) {
if (sawTarget !== nearest) {
sawTarget = nearest;
sawProgress = 0;
sawTimer = 0;
sawDirection = 1;
}
sawTimer += delta;
if (sawTimer >= SAW_HIT_INTERVAL) {
sawTimer -= SAW_HIT_INTERVAL;
sawProgress++;
sawDirection = -sawDirection;
playChainsawSawSound();
const hitPoint = new THREE.Vector3(nearest.position.x, nearest.position.y + 1.1, nearest.position.z);
createBloodSplatter(hitPoint, LOW_END ? 4 : (isMobile ? 6 : 12));
createBloodSpray(hitPoint, new THREE.Vector3(sawDirection, 0.4, 0).normalize(), LOW_END ? 4 : (isMobile ? 6 : 14));
spawnBloodDecal(hitPoint.clone(), new THREE.Vector3(sawDirection, 0.2, 0.2).normalize());
const sparkCount2 = LOW_END ? 3 : (isMobile ? 6 : 14);
for (let i = 0; i < sparkCount2; i++) spawnChainsawSpark(hitPoint, 1.6);
if (weapon) {
weapon.rotation.z = sawDirection * 0.24;
weapon.position.x = sawDirection * 0.05;
}
nearest.userData.hitFlash = 0.1;
for (const m of nearest.userData.materials) m.material.emissive.setHex(0xff2200);
if (sawProgress >= SAW_HITS_TO_KILL) {
killEnemy(nearest, "body");
sawTarget = null;
sawProgress = 0;
sawTimer = 0;
}
}
} else {
sawTarget = null;
sawProgress = 0;
sawTimer = 0;
if (weapon) {
weapon.rotation.z *= 0.85;
weapon.position.x *= 0.85;
}
}
} else {
if (chainsawRunning) {
chainsawRunning = false;
playChainsawStopSound();
}
sawTarget = null;
sawProgress = 0;
sawTimer = 0;
if (weapon) {
weapon.rotation.z *= 0.85;
weapon.position.x *= 0.85;
}
}
}

function shoot() {
if (!canShoot || reloading || ended || shopOpen) return;
if (currentWeapon === "chainsaw_mythic") return;
if (isThrowable(currentWeapon)) {
if (isThrowing) return;
canShoot = false;
throwItem(currentWeapon);
setTimeout(() => { canShoot = true; }, 500);
return;
}
const w = getWeaponData(currentWeapon);
if (ammo <= 0) { playEmptyClick(); reload(); return; }
canShoot = false;
ammo--;
recoil = 0.12;
playShootSound();
updateHUD();
const muzzlePos = getMuzzleWorldPosition();
spawnShell();
const raycaster = new THREE.Raycaster();
raycaster.setFromCamera(new THREE.Vector2(0, 0), camera);
const targets = [];
for (const enemy of enemies) {
if (!enemy.userData.alive || enemy.userData.dying) continue;
enemy.traverse((object) => { if (object.isMesh) targets.push(object); });
}
const enemyHits = raycaster.intersectObjects(targets, false);
const solidHits = raycaster.intersectObjects(solidMeshes, false);
const enemyDist = enemyHits.length > 0 ? enemyHits[0].distance : Infinity;
const solidDist = solidHits.length > 0 ? solidHits[0].distance : Infinity;
let hitPoint = null;
if (enemyHits.length > 0 && enemyDist <= solidDist) {
const hit = enemyHits[0];
hitPoint = hit.point.clone();
const object = hit.object;
let enemy = object;
while (enemy && !enemy.userData.hips && !enemy.userData.health) enemy = enemy.parent;
if (!enemy) enemy = object.parent;
if (enemy && enemy.userData && enemy.userData.alive) {
const zone = object.userData.zone || "body";
let damage = w.damage;
if (zone === "head") damage *= 2.5;
else if (zone === "armor") damage *= 0.5;
enemy.userData.health -= damage;
enemy.userData.hitFlash = 0.12;
if (enemy.userData.materials) {
for (const m of enemy.userData.materials) m.material.emissive.setHex(0xff3333);
}
const color = zone === "armor" ? 0x88ffcc : 0xff0055;
if (!LOW_END) createParticles(hit.point, color);
if (zone === "head") {
const cnt = LOW_END ? 6 : (isMobile ? 10 : 30);
createBloodSplatter(hit.point, cnt);
createBloodSpray(hit.point, hit.face ? hit.face.normal : new THREE.Vector3(0, 1, 0), LOW_END ? 6 : (isMobile ? 8 : 22));
spawnBloodDecal(hit.point, (hit.face ? hit.face.normal.clone().applyMatrix3(new THREE.Matrix3().getNormalMatrix(hit.object.matrixWorld)).normalize() : new THREE.Vector3(0, 1, 0)));
spawnBloodPool(hit.point.x, hit.point.z);
playBloodSplatSound();
} else if (zone === "armor") {
createBloodSplatter(hit.point, LOW_END ? 1 : (isMobile ? 1 : 5));
} else {
createBloodSplatter(hit.point, LOW_END ? 3 : (isMobile ? 4 : 14));
createBloodSpray(hit.point, hit.face ? hit.face.normal : new THREE.Vector3(0, 1, 0), LOW_END ? 3 : (isMobile ? 4 : 10));
spawnBloodDecal(hit.point, (hit.face ? hit.face.normal.clone().applyMatrix3(new THREE.Matrix3().getNormalMatrix(hit.object.matrixWorld)).normalize() : new THREE.Vector3(0, 1, 0)));
}
playHitSound();
if (enemy.userData.health <= 0) killEnemy(enemy, zone);
}
} else if (solidHits.length > 0 && solidDist < Infinity) {
const hit = solidHits[0];
hitPoint = hit.point.clone();
if (hit.face) {
const normalMatrix = new THREE.Matrix3().getNormalMatrix(hit.object.matrixWorld);
const worldNormal = hit.face.normal.clone().applyMatrix3(normalMatrix).normalize();
if (!LOW_END) spawnDecal(hit.point, worldNormal);
createDustPuff(hit.point, worldNormal);
}
playWallHitSound();
} else {
const far = new THREE.Vector3(0, 0, -1).applyQuaternion(camera.quaternion).multiplyScalar(200).add(muzzlePos);
hitPoint = far;
}
spawnTracer(muzzlePos, hitPoint);
if (!LOW_END) spawnBullet(muzzlePos, hitPoint);
setTimeout(() => { canShoot = true; }, w.fireDelay);
}
function throwItem(itemId) {
isThrowing = true;
throwAnimTimer = 0;
playThrowSound();
getItemMaterials();
let geo, mat;
if (itemId === "item_toiletpaper") {
geo = new THREE.CylinderGeometry(0.12, 0.12, 0.18, 12);
mat = new THREE.MeshStandardMaterial({ color: 0xf5f5f0, roughness: 0.85 });
} else if (itemId === "item_stub") {
geo = new THREE.SphereGeometry(0.14, 14, 10);
mat = MAT_STUB;
} else if (itemId === "item_rag") {
geo = new THREE.PlaneGeometry(0.42, 0.42);
mat = MAT_RAG;
} else if (itemId === "item_handkerchief") {
geo = new THREE.PlaneGeometry(0.40, 0.40);
mat = MAT_HANDKERCHIEF;
} else {
geo = new THREE.BoxGeometry(0.15, 0.15, 0.15);
mat = new THREE.MeshStandardMaterial({ color: 0xcccccc });
}
if (thrownItems.length >= MAX_THROWN_ITEMS) {
const old = thrownItems.shift();
scene.remove(old);
if (old.geometry) old.geometry.dispose();
}
const item = new THREE.Mesh(geo, mat);
if (itemId === "item_stub") item.scale.set(0.85, 1.35, 0.85);
const startPos = new THREE.Vector3(0.34, -0.42, -0.95);
weapon.localToWorld(startPos);
item.position.copy(startPos);
const dir = new THREE.Vector3();
camera.getWorldDirection(dir);
const velocity = dir.clone().multiplyScalar(22);
velocity.y += 3.0;
velocity.x += (Math.random() - 0.5) * 0.6;
velocity.z += (Math.random() - 0.5) * 0.6;
item.userData.velocity = velocity;
item.userData.spin = new THREE.Vector3((Math.random() - 0.5) * 14, (Math.random() - 0.5) * 14, (Math.random() - 0.5) * 14);
item.userData.life = 4;
item.userData.damage = (getWeaponData(itemId).stats.damage) || 10;
item.userData.itemId = itemId;
scene.add(item);
thrownItems.push(item);
}
function updateThrownItems(delta) {
for (let i = thrownItems.length - 1; i >= 0; i--) {
const item = thrownItems[i];
item.position.addScaledVector(item.userData.velocity, delta);
item.userData.velocity.y -= 22 * delta;
item.rotation.x += item.userData.spin.x * delta;
item.rotation.y += item.userData.spin.y * delta;
item.rotation.z += item.userData.spin.z * delta;
item.userData.life -= delta;
let hitEnemy = false;
for (const enemy of enemies) {
if (!enemy.userData.alive || enemy.userData.dying) continue;
const dx = item.position.x - enemy.position.x;
const dy = item.position.y - (enemy.position.y + 1.2);
const dz = item.position.z - enemy.position.z;
if (dx * dx + dy * dy + dz * dz < 0.9) {
enemy.userData.health -= item.userData.damage;
enemy.userData.hitFlash = 0.15;
if (enemy.userData.materials) {
for (const m of enemy.userData.materials) m.material.emissive.setHex(0xffaa00);
}
playHitSound();
createBloodSplatter(item.position.clone(), 5);
if (enemy.userData.health <= 0) killEnemy(enemy, "body");
hitEnemy = true;
break;
}
}
if (hitEnemy) { scene.remove(item); thrownItems.splice(i, 1); continue; }
if (item.position.y <= 0.05) {
item.position.y = 0.05;
item.userData.velocity.set(0, 0, 0);
if (item.userData.life > 1.5) item.userData.life = 1.5;
}
if (item.userData.life <= 0) { scene.remove(item); thrownItems.splice(i, 1); }
}
}
function createDustPuff(position, normal) {
if (LOW_END) return;
const geometry = new THREE.SphereGeometry(0.04, 3, 3);
const material = new THREE.MeshBasicMaterial({ color: 0xaaaaaa, transparent: true, opacity: 0.8 });
const count = isMobile ? 3 : 6;
for (let i = 0; i < count; i++) {
const particle = new THREE.Mesh(geometry, material.clone());
particle.position.copy(position).addScaledVector(normal, 0.05);
const scatter = new THREE.Vector3((Math.random() - 0.5) * 1.2, (Math.random() - 0.5) * 1.2, (Math.random() - 0.5) * 1.2).add(normal.clone().multiplyScalar(0.8));
particle.userData.velocity = scatter.multiplyScalar(2.5);
particle.userData.life = 0.3;
scene.add(particle);
particles.push(particle);
}
while (particles.length > MAX_PARTICLES) { scene.remove(particles.shift()); }
}
function killEnemy(enemy, zone) {
if (!enemy.userData.alive) return;
enemy.userData.alive = false;
enemy.userData.dying = true;
enemy.userData.deathTimer = 0;
enemy.userData.walkPhase = 0;
enemy.userData.hitFlash = 0;
enemy.userData.stunTimer = 0;
const isBoss = enemy.userData.isBoss;
if (isBoss) {
enemy.userData.bossDefeated = true;
if (activeBoss === enemy) activeBoss = null;
} else {
resetEnemyMaterials(enemy);
}
const points = isBoss ? 2000 + currentBossIndex * 1500 : (zone === "head" ? 250 : (zone === "armor" ? 50 : 100));
score += points;
updateHUD();
playKillSound();
playBloodSplatSound();
const center = enemy.position.clone().add(new THREE.Vector3(0, 1.2, 0));
if (!LOW_END) { createParticles(center, 0xff0055); createBloodEffect(center); }
createBloodBurst(center, LOW_END ? 12 : (isMobile ? 25 : 90));
createBloodSpray(center, new THREE.Vector3(0, 1, 0), LOW_END ? 6 : (isMobile ? 14 : 45));
spawnBloodPool(enemy.position.x, enemy.position.z);
if (!LOW_END) {
const bloodAngles = isMobile ? 4 : 8;
for (let i = 0; i < bloodAngles; i++) {
const ang = Math.random() * Math.PI * 2;
const r = 1.5 + Math.random() * 2.5;
const bx = enemy.position.x + Math.cos(ang) * r;
const bz = enemy.position.z + Math.sin(ang) * r;
if (!collidesAt(bx, bz, 0.3)) spawnBloodPool(bx, bz);
}
createBloodTrail(enemy.position);
}
if (isBoss) {
onBossDefeated(enemy.userData.bossIndex);
return;
}
const btvGain = giveKillRewardBtv();
showLevelMessage("+" + btvGain + " БТВ", 900);
levelKills++;
if (!inBossFight && levelKills >= enemiesToKill) nextLevel();
}
function createBloodTrail(position) {
const count = isMobile ? 6 : 14;
for (let i = 0; i < count; i++) {
const angle = Math.random() * Math.PI * 2;
const r = 0.3 + Math.random() * 1.2;
const x = position.x + Math.cos(angle) * r;
const z = position.z + Math.sin(angle) * r;
const geo = new THREE.CircleGeometry(0.08 + Math.random() * 0.15, 8);
const mat = new THREE.MeshBasicMaterial({ color: new THREE.Color(0.3 + Math.random() * 0.3, 0, 0), transparent: true, opacity: 0.85, depthWrite: false, polygonOffset: true, polygonOffsetFactor: -7, polygonOffsetUnits: -7, fog: true });
const drop = new THREE.Mesh(geo, mat);
drop.rotation.x = -Math.PI / 2;
drop.position.set(x, 0.014, z);
scene.add(drop);
bloodDecals.push(drop);
while (bloodDecals.length > MAX_BLOOD_DECALS) { scene.remove(bloodDecals.shift()); }
}
}
function createBloodBurst(position, count) {
const geo = new THREE.SphereGeometry(0.06 + Math.random() * 0.05, 3, 3);
const mats = [
new THREE.MeshBasicMaterial({ color: 0x9a0000 }),
new THREE.MeshBasicMaterial({ color: 0x6a0000 }),
new THREE.MeshBasicMaterial({ color: 0xc00000 })
 ];
for (let i = 0; i < count; i++) {
const particle = new THREE.Mesh(geo, mats[Math.floor(Math.random() * mats.length)]);
particle.position.copy(position);
particle.position.x += (Math.random() - 0.5) * 0.4;
particle.position.y += (Math.random() - 0.5) * 0.4;
particle.position.z += (Math.random() - 0.5) * 0.4;
particle.userData.velocity = new THREE.Vector3((Math.random() - 0.5) * 14, Math.random() * 10 + 3, (Math.random() - 0.5) * 14);
particle.userData.life = 1.4 + Math.random() * 0.8;
particle.userData.isBlood = true;
particle.scale.setScalar(0.7 + Math.random() * 0.9);
scene.add(particle);
bloodParticles.push(particle);
}
while (bloodParticles.length > MAX_BLOOD_PARTICLES) { scene.remove(bloodParticles.shift()); }
}
function createBloodSpray(position, normal, count) {
const geo = new THREE.SphereGeometry(0.04, 3, 3);
const mats = [ new THREE.MeshBasicMaterial({ color: 0xb00000 }), new THREE.MeshBasicMaterial({ color: 0x700000 }) ];
const n = normal.clone().normalize();
for (let i = 0; i < count; i++) {
const particle = new THREE.Mesh(geo, mats[i % 2]);
particle.position.copy(position);
const spread = 0.4;
const vx = n.x * (4 + Math.random() * 6) + (Math.random() - 0.5) * spread * 10;
const vy = n.y * (3 + Math.random() * 5) + Math.random() * 4 + 2;
const vz = n.z * (4 + Math.random() * 6) + (Math.random() - 0.5) * spread * 10;
particle.userData.velocity = new THREE.Vector3(vx, vy, vz);
particle.userData.life = 0.8 + Math.random() * 0.7;
particle.userData.isBlood = true;
particle.scale.setScalar(0.6 + Math.random() * 0.7);
scene.add(particle);
bloodParticles.push(particle);
}
while (bloodParticles.length > MAX_BLOOD_PARTICLES) { scene.remove(bloodParticles.shift()); }
}
function createParticles(position, color) {
if (LOW_END) return;
const geometry = new THREE.SphereGeometry(0.07, 4, 3);
const material = new THREE.MeshBasicMaterial({ color });
const cnt = isMobile ? 5 : 10;
for (let i = 0; i < cnt; i++) {
const particle = new THREE.Mesh(geometry, material);
particle.position.copy(position);
particle.userData.velocity = new THREE.Vector3(-1 + Math.random() * 2, -1 + Math.random() * 2, -1 + Math.random() * 2).multiplyScalar(5);
particle.userData.life = 0.5;
scene.add(particle);
particles.push(particle);
}
while (particles.length > MAX_PARTICLES) { scene.remove(particles.shift()); }
}
function createBloodSplatter(position, count) {
const geometry = new THREE.SphereGeometry(0.045, 3, 3);
const material = new THREE.MeshBasicMaterial({ color: 0xaa0000 });
for (let i = 0; i < count; i++) {
const particle = new THREE.Mesh(geometry, material);
particle.position.copy(position);
particle.userData.velocity = new THREE.Vector3((Math.random() - 0.5) * 6, Math.random() * 4 + 1, (Math.random() - 0.5) * 6);
particle.userData.life = 0.6;
particle.userData.isBlood = true;
scene.add(particle);
bloodParticles.push(particle);
}
while (bloodParticles.length > MAX_BLOOD_PARTICLES) { scene.remove(bloodParticles.shift()); }
}
function createBloodEffect(position) {
const geometry = new THREE.SphereGeometry(0.07, 3, 3);
const material = new THREE.MeshBasicMaterial({ color: 0x8a0303 });
const count = isMobile ? 10 : 24;
for (let i = 0; i < count; i++) {
const particle = new THREE.Mesh(geometry, material);
particle.position.copy(position);
particle.userData.velocity = new THREE.Vector3((Math.random() - 0.5) * 10, Math.random() * 7 + 2, (Math.random() - 0.5) * 10);
particle.userData.life = 1 + Math.random() * 0.4;
particle.userData.isBlood = true;
scene.add(particle);
bloodParticles.push(particle);
}
while (bloodParticles.length > MAX_BLOOD_PARTICLES) { scene.remove(bloodParticles.shift()); }
}
function updateParticles(delta) {
for (let i = particles.length - 1; i >= 0; i--) {
const particle = particles[i];
particle.position.addScaledVector(particle.userData.velocity, delta);
particle.userData.velocity.y -= 10 * delta;
particle.userData.life -= delta;
particle.scale.multiplyScalar(0.94);
if (particle.material.transparent && particle.material.opacity !== undefined) {
particle.material.opacity = Math.max(0, particle.userData.life / 0.3) * 0.8;
}
if (particle.userData.life <= 0) { scene.remove(particle); particles.splice(i, 1); }
}
}
function updateBloodParticles(delta) {
for (let i = bloodParticles.length - 1; i >= 0; i--) {
const particle = bloodParticles[i];
particle.position.addScaledVector(particle.userData.velocity, delta);
particle.userData.velocity.y -= 18 * delta;
particle.userData.life -= delta;
particle.scale.multiplyScalar(0.97);
if (particle.position.y < 0.02) {
spawnBloodDecal(new THREE.Vector3(particle.position.x, 0.01, particle.position.z), new THREE.Vector3(0, 1, 0));
scene.remove(particle);
bloodParticles.splice(i, 1);
continue;
}
if (particle.userData.life <= 0) { scene.remove(particle); bloodParticles.splice(i, 1); }
}
}
function updateBloodDecals(delta) {
for (const d of bloodDecals) {
if (d.userData && d.userData.spawnTime) {
const age = (performance.now() - d.userData.spawnTime) / 1000;
if (age < 0.3) d.material.opacity = Math.min(0.92, age / 0.3 * 0.92);
}
}
}
function updateBloodPools(delta) {
for (const p of bloodPools) {
if (p.userData && p.userData.growTime) {
const age = (performance.now() - p.userData.growTime) / 1000;
if (age < 1.5) {
const t = Math.min(age / 1.5, 1);
p.scale.setScalar(p.userData.targetScale * (0.3 + t * 0.7));
}
}
}
}
function updateShells(delta) {
for (let i = shells.length - 1; i >= 0; i--) {
const shell = shells[i];
if (!shell.userData.landed) {
shell.position.addScaledVector(shell.userData.velocity, delta);
shell.userData.velocity.y -= 14 * delta;
shell.rotation.x += shell.userData.spin.x * delta;
shell.rotation.y += shell.userData.spin.y * delta;
shell.rotation.z += shell.userData.spin.z * delta;
if (shell.position.y <= 0.03) {
shell.position.y = 0.03;
if (Math.abs(shell.userData.velocity.y) < 1.2) {
shell.userData.landed = true;
shell.rotation.x = Math.PI / 2;
shell.userData.velocity.set(0, 0, 0);
if (Math.random() < 0.6) playShellDropSound();
} else {
shell.userData.velocity.y *= -0.3;
shell.userData.velocity.x *= 0.6;
shell.userData.velocity.z *= 0.6;
shell.userData.spin.multiplyScalar(0.5);
}
}
} else {
shell.userData.groundTimer -= delta;
if (shell.userData.groundTimer < 1) shell.scale.multiplyScalar(0.97);
if (shell.userData.groundTimer <= 0) { scene.remove(shell); shells.splice(i, 1); }
}
}
}
function updateTracers(delta) {
for (let i = tracers.length - 1; i >= 0; i--) {
const line = tracers[i];
line.userData.life -= delta;
if (line.userData.life <= 0) {
scene.remove(line);
if (line.geometry) line.geometry.dispose();
tracers.splice(i, 1);
}
}
}
function updateBullets(delta) {
for (let i = bullets.length - 1; i >= 0; i--) {
const bullet = bullets[i];
bullet.position.addScaledVector(bullet.userData.velocity, delta);
bullet.userData.life -= delta;
if (bullet.userData.life <= 0) { scene.remove(bullet); bullets.splice(i, 1); }
}
}
function updateEnemyBullets(delta) {
for (let i = enemyBullets.length - 1; i >= 0; i--) {
const bullet = enemyBullets[i];
bullet.position.addScaledVector(bullet.userData.velocity, delta);
bullet.userData.life -= delta;
if (!playerDying) {
const dx = bullet.position.x - player.position.x;
const dy = bullet.position.y - player.position.y;
const dz = bullet.position.z - player.position.z;
const dist = Math.sqrt(dx * dx + dy * dy + dz * dz);
if (dist < 0.75) {
damagePlayer(bullet.userData.damage);
createBloodSplatter(bullet.position, 8);
splashBloodOnScreen();
scene.remove(bullet);
enemyBullets.splice(i, 1);
continue;
}
}
if (collidesAt(bullet.position.x, bullet.position.z, 0.05) || bullet.position.y < 0.05) {
createDustPuff(bullet.position, new THREE.Vector3(0, 1, 0));
scene.remove(bullet);
enemyBullets.splice(i, 1);
continue;
}
if (bullet.userData.life <= 0) { scene.remove(bullet); enemyBullets.splice(i, 1); }
}
}
function updateWeapon(delta) {
if (!weapon) return;
if (isThrowing) {
throwAnimTimer += delta;
const t = throwAnimTimer / THROW_ANIM_DURATION;
if (t >= 1) {
isThrowing = false;
throwAnimTimer = 0;
weapon.position.set(0, 0, 0);
weapon.rotation.set(0, 0, 0);
} else {
let px = 0, py = 0, pz = 0, rx = 0, rz = 0;
if (t < 0.35) {
const p = t / 0.35;
pz = p * 0.15;
py = -p * 0.15;
rx = -p * 0.8;
rz = -p * 0.25;
} else {
const p = (t - 0.35) / 0.65;
pz = 0.15 - p * 0.9;
py = -0.15 + p * 0.35;
rx = -0.8 + p * 1.6;
rz = -0.25 + p * 0.5;
}
weapon.position.set(px, py, pz);
weapon.rotation.set(rx, 0, rz);
return;
}
}
if (currentWeapon === "chainsaw_mythic") {
bobTime += delta * (grounded ? 8 : 2);
const moving = keys["KeyW"] || keys["KeyS"] || keys["KeyA"] || keys["KeyD"] || Math.abs(joy.x) > 0.1 || Math.abs(joy.y) > 0.1;
const bobAmount = (moving && grounded) ? 1 : 0;
const bobX = Math.sin(bobTime) * 0.012 * bobAmount;
const bobY = Math.abs(Math.cos(bobTime)) * 0.014 * bobAmount;
const vibr = chainsawRunning ? (Math.sin(performance.now() * 0.09) * 0.012) : 0;
weapon.position.y = bobY + vibr;
weapon.position.z = recoil;
weapon.rotation.x = vibr * 2;
return;
}
bobTime += delta * (grounded ? 8 : 2);
const moving = keys["KeyW"] || keys["KeyS"] || keys["KeyA"] || keys["KeyD"] || Math.abs(joy.x) > 0.1 || Math.abs(joy.y) > 0.1;
const bobAmount = (moving && grounded) ? 1 : 0;
const bobX = Math.sin(bobTime) * 0.012 * bobAmount;
const bobY = Math.abs(Math.cos(bobTime)) * 0.014 * bobAmount;
recoil *= Math.pow(0.0001, delta);
if (!weapon.userData.phaseFlags) weapon.userData.phaseFlags = {};
if (isThrowable(currentWeapon)) {
weapon.position.set(bobX, bobY, recoil);
weapon.rotation.x = 0;
weapon.rotation.z = 0;
return;
}
if (reloading) {
const t = 1 - reloadTimer / reloadDuration;
let px = bobX, py = bobY, pz = recoil, rx = 0, rz = 0;
if (t < 0.20) { const p = t / 0.20; py = -p * 0.35; rx = p * 1.1; rz = p * 0.55; }
else if (t < 0.35) { py = -0.35; rx = 1.1; rz = 0.55; }
else if (t < 0.60) { const p = (t - 0.35) / 0.25; py = -0.35 + Math.sin(p * Math.PI) * 0.08; rx = 1.1 - p * 0.5; rz = 0.55 - p * 0.3; }
else if (t < 0.80) { const p = (t - 0.60) / 0.20; py = -0.35 + p * 0.15; rx = 0.6 - p * 0.4; rz = 0.25 - p * 0.15; }
else if (t < 0.95) { const p = (t - 0.80) / 0.15; py = -0.20 + Math.sin(p * Math.PI) * 0.04; rx = 0.2; rz = 0.1; }
else { const p = (t - 0.95) / 0.05; py = -0.20 + p * 0.20; rx = 0.2 * (1 - p); rz = 0.1 * (1 - p); }
weapon.position.set(px + bobX, py + bobY, pz);
weapon.rotation.x = rx;
weapon.rotation.z = rz;
if (weaponMagazine) {
if (t < 0.20) { weaponMagazine.visible = true; }
else if (t < 0.60) weaponMagazine.visible = false;
else if (t < 0.72) {
const p = (t - 0.60) / 0.12;
weaponMagazine.visible = true;
weaponMagazine.position.y = -0.5 - (1 - p) * 0.5;
} else {
weaponMagazine.visible = true;
weaponMagazine.position.set(0.32, -0.5, -0.72);
}
}
if (weaponHand) {
if (t < 0.35) weaponHand.position.y = -(t / 0.35) * 0.5;
else if (t < 0.60) weaponHand.position.y = -0.5;
else weaponHand.position.y = -0.5 * (1 - (t - 0.60) / 0.40);
}
if (t >= 0.20 && !weapon.userData.phaseFlags.mag) { weapon.userData.phaseFlags.mag = true; spawnDroppedMagazine(); playMagDropSound(); }
if (t >= 0.60 && !weapon.userData.phaseFlags.insert) { weapon.userData.phaseFlags.insert = true; playMagInsertSound(); }
if (t >= 0.80 && !weapon.userData.phaseFlags.bolt) { weapon.userData.phaseFlags.bolt = true; playBoltSound(); }
if (t < 0.05) weapon.userData.phaseFlags = {};
} else {
weapon.position.set(bobX, bobY, recoil);
weapon.rotation.x = 0;
weapon.rotation.z = 0;
if (weaponHand) weaponHand.position.y = 0;
}
if (droppedMagazine) {
droppedMagazine.userData.vel.y -= 20 * delta;
droppedMagazine.position.addScaledVector(droppedMagazine.userData.vel, delta);
droppedMagazine.rotation.x += droppedMagazine.userData.spin.x * delta;
droppedMagazine.rotation.y += droppedMagazine.userData.spin.y * delta;
droppedMagazine.rotation.z += droppedMagazine.userData.spin.z * delta;
droppedMagazine.userData.life -= delta;
if (droppedMagazine.position.y < 0.05) {
droppedMagazine.position.y = 0.05;
droppedMagazine.userData.vel.set(0, 0, 0);
}
if (droppedMagazine.userData.life <= 0) {
scene.remove(droppedMagazine);
if (droppedMagazine.geometry) droppedMagazine.geometry.dispose();
if (droppedMagazine.material) droppedMagazine.material.dispose();
droppedMagazine = null;
}
}
}
function updateReload(delta) {
if (!reloading) return;
if (currentWeapon === "chainsaw_mythic") {
reloading = false;
reloadTimer = 0;
return;
}
reloadTimer += delta;
if (reloadTimer >= reloadDuration) {
reloading = false;
reloadTimer = 0;
const w = getWeaponData(currentWeapon);
const magSize = w.magSize || 30;
const needed = magSize - ammo;
const amount = Math.min(needed, reserveAmmo);
ammo += amount;
reserveAmmo -= amount;
message.innerHTML = "";
playReloadEnd();
updateHUD();
}
}
function reload() {
if (ended || reloading || shopOpen) return;
if (isThrowable(currentWeapon)) return;
if (currentWeapon === "chainsaw_mythic") {
if (gasoline <= 0) showLevelMessage("НУЖЕН БЕНЗИН! Купи канистру в магазине", 1800);
else showLevelMessage("БЕНЗИН: " + gasoline.toFixed(2) + " Л", 1200);
return;
}
const w = getWeaponData(currentWeapon);
const magSize = w.magSize || 30;
if (ammo >= magSize) return;
if (reserveAmmo <= 0) return;
reloading = true;
reloadTimer = 0;
droppedMagazine = null;
message.innerHTML = "ПЕРЕЗАРЯДКА...";
playReloadStart();
}
function damagePlayer(amount) {
if (ended || playerDying) return;
health = Math.max(0, health - amount);
playDamageSound();
updateHUD();
if (health <= 0) gameOver();
}
function gameOver() {
if (ended) return;
ended = true;
firing = false;
chainsawRunning = false;
playerDying = true;
playerDeathTimer = 0;
playerDeathStartY = player.position.y;
playerDeathStartPitch = pitch;
deathMessageShown = false;
saveProgress();
updateHUD();
playGameOverSound();
saveScore();
}
async function saveScore() {
try {
const response = await fetch("/api/score", {
method: "POST",
headers: { "Content-Type": "application/json" },
body: JSON.stringify({ name: "Player", score: score })
});
if (response.ok) loadLeaderboard();
} catch (e) {}
}
async function loadLeaderboard() {
try {
const response = await fetch("/api/scores");
const data = await response.json();
const board = data.map((item, index) => "<div>" + (index + 1) + ". " + escapeHTML(item.name) + " — <b>" + Number(item.score).toLocaleString() + "</b></div>").join("");
leaderboard.innerHTML = "<b>ТОП-10</b><br>" + board;
} catch (e) { leaderboard.textContent = "Рекорды недоступны"; }
}
function escapeHTML(value) {
return String(value)
.replaceAll("&", "&")
.replaceAll("<", "<")
.replaceAll(">", ">")
.replaceAll('"', """)
.replaceAll("'", "'");
}
function updateHUD() {
hpText.textContent = health;
scoreText.textContent = score;
if (isThrowable(currentWeapon)) ammoText.textContent = "∞";
else if (currentWeapon === "chainsaw_mythic") ammoText.textContent = gasoline.toFixed(2) + " Л";
else ammoText.textContent = ammo + " / " + reserveAmmo;
levelText.textContent = currentLevel;
btvValueEl.textContent = btv.toLocaleString();
bankValueEl.textContent = skulls.toLocaleString();
if (shopBtvValue) shopBtvValue.textContent = btv.toLocaleString();
if (shopBalanceValue) shopBalanceValue.textContent = skulls.toLocaleString();
if (health <= 25) hpText.style.color = "#ff3158";
else if (health <= 50) hpText.style.color = "#ffd166";
else hpText.style.color = "#48ff7b";
updateWeaponName();
}
function openShop() {
if (shopOpen || ended) return;
shopOpen = true;
shopPanel.style.display = "block";
playShopOpenSound();
renderShop();
if (!isMobile) document.exitPointerLock?.();
}
function closeShop() {
if (!shopOpen) return;
shopOpen = false;
shopCooldown = true;
shopPanel.style.display = "none";
if (!isMobile && started && !ended) renderer.domElement.requestPointerLock?.();
}

function getRarityName(r) {
const names = {
common: "ОБЫЧНЫЙ",
rare: "РЕДКИЙ",
epic: "ЭПИК",
crystal: "КРИСТАЛЬНЫЙ",
mythic: "МИФИЧЕСКИЙ",
mythicmax: "МИФИЧЕСКИЙ САМЫЙ КРУТОЙ"
};
return names[r] || r;
}

function renderShop() {
const activeTab = document.querySelector(".shopTab.active");
const tab = activeTab ? activeTab.dataset.tab : "weapons";
if (tab === "levels") { renderLevelsTab(); return; }
if (tab === "bosses") { renderBossesTab(); return; }
const items = SHOP_ITEMS.filter(i => i.category === tab);
shopGrid.innerHTML = items.map(item => {
const owned = ownedItems[item.id];
const isBtv = item.currency === "btv";
const balance = isBtv ? btv : skulls;
const canAfford = balance >= item.price;
const isCurrent = currentWeapon === item.id;
const iconClass = isBtv ? "btv-ico" : "skull-ico";
const priceClass = isBtv ? "price btv" : "price";
let btnClass = "buyBtn";
let btnText = "КУПИТЬ";
let disabled = "";
if (item.category === "fuel") {
const full = gasoline >= GASOLINE_MAX - 0.001;
if (full) { btnClass += " owned"; btnText = "БАК ПОЛОН"; disabled = "disabled"; }
else if (!canAfford) { btnClass += " owned"; btnText = "НЕ ХВАТАЕТ"; disabled = "disabled"; }
return <div class="shopCard"&gt; &lt;h3&gt;${item.name}</h3>
<span class="rarity rarity-${item.rarity}"&gt;${getRarityName(item.rarity)}</span>
<div class="desc">${item.desc}&lt;/div&gt; &lt;div class="stats"&gt; &lt;span class="stat"&gt;ОБЪЁМ: ${item.stats.volume}</span>
<span class="stat">В БАКЕ: ${gasoline.toFixed(2)} / ${GASOLINE_MAX.toFixed(2)} Л</span>
</div>
<div class="${priceClass}"&gt;&lt;span class="${iconClass}"></span>${item.price.toLocaleString()}&lt;/div&gt; &lt;button type="button" class="${btnClass}" data-id="${item.id}" ${disabled}>${btnText}&lt;/button&gt; &lt;/div>;
}
if (isCurrent) { btnClass += " owned"; btnText = "АКТИВНО"; disabled = "disabled"; }
else if (owned) { btnClass += " owned"; btnText = "ВЫБРАТЬ"; }
else if (!canAfford) { btnClass += " owned"; btnText = "НЕ ХВАТАЕТ"; disabled = "disabled"; }
return <div class="shopCard"&gt; &lt;h3&gt;${item.name}</h3>
<span class="rarity rarity-${item.rarity}"&gt;${getRarityName(item.rarity)}</span>
<div class="desc">${item.desc}&lt;/div&gt; &lt;div class="stats"&gt; ${item.stats.damage ? <span class="stat"&gt;УРОН:${item.stats.damage}</span>: ""} ${item.stats.fireRate ? <span class="stat"&gt;ТЕМП:${item.stats.fireRate}</span>: ""} ${item.stats.mag ? <span class="stat"&gt;МАГАЗИН:${item.stats.mag}</span>: ""} ${item.stats.radius ? <span class="stat"&gt;РАДИУС:${item.stats.radius}</span>: ""} ${item.stats.duration ? <span class="stat"&gt;ДЛИТ:${item.stats.duration}</span>: ""} ${item.stats.effect ? <span class="stat"&gt;ЭФФЕКТ:${item.stats.effect}</span>: ""} &lt;/div&gt; &lt;div class="${priceClass}"><span class="${iconClass}"&gt;&lt;/span&gt;${item.price.toLocaleString()}</div>
<button type="button" class="${btnClass}" data-id="${item.id}" ${disabled}&gt;${btnText}</button>
</div>`;
}).join("");
applyCurrencyIcons();
shopGrid.querySelectorAll(".buyBtn").forEach(btn => {
if (btn.disabled) return;
btn.addEventListener("click", (e) => {
e.preventDefault();
e.stopPropagation();
const id = btn.dataset.id;
const item = getWeaponData(id);
const isBtv = item.currency === "btv";
if (item.category === "fuel") {
const need = 0.5;
if (gasoline >= GASOLINE_MAX - 0.001) return;
if (skulls < item.price) return;
skulls -= item.price;
gasoline = Math.min(GASOLINE_MAX, gasoline + need);
saveProgress();
playBuySound();
showShopMsg("ЗАЛИТО: +0.5 Л (сейчас " + gasoline.toFixed(2) + " Л)");
updateHUD();
renderShop();
return;
}
if (ownedItems[id]) {
currentWeapon = id;
reloading = false;
reloadTimer = 0;
droppedMagazine = null;
applyWeaponStats();
saveProgress();
playBuySound();
showShopMsg("ВЫБРАНО: " + item.name);
renderShop();
return;
}
const balance = isBtv ? btv : skulls;
if (balance >= item.price) {
if (isBtv) btv -= item.price;
else skulls -= item.price;
ownedItems[id] = true;
if (id === "chainsaw_mythic") hasChainsaw = true;
currentWeapon = id;
reloading = false;
reloadTimer = 0;
droppedMagazine = null;
applyWeaponStats();
saveProgress();
playBuySound();
showShopMsg("КУПЛЕНО: " + item.name);
updateHUD();
renderShop();
}
});
});
}

function renderLevelsTab() {
let html = '<div class="levelHint">Всего 100 уровней. Каждый 10-й уровень — БОСС. Нажми на доступный уровень, чтобы начать его прямо сейчас. Пройденные боссы можно перепроходить для получения черепков.</div>';
html += '<div class="levelGrid">';
for (let i = 1; i <= 100; i++) {
const isBoss = i % 10 === 0;
const locked = i > maxLevelReached;
const isCurrent = i === currentLevel;
let cls = "levelBtn";
if (isBoss) cls += " boss";
if (locked) cls += " locked";
if (isCurrent) cls += " current";
html += '<button type="button" class="' + cls + '" data-level="' + i + '"' + (locked ? ' disabled' : '') + '>' + i + (isBoss ? "!" : "") + '</button>';
}
html += '</div>';
shopGrid.innerHTML = html;
shopGrid.querySelectorAll(".levelBtn").forEach(btn => {
if (btn.disabled) return;
btn.addEventListener("click", (e) => {
e.preventDefault();
e.stopPropagation();
const lvl = parseInt(btn.dataset.level, 10);
if (!lvl) return;
startLevel(lvl);
closeShop();
});
});
}

function renderBossesTab() {
let html = '<div class="levelHint">Боссы открываются при достижении соответствующего уровня. Пройденного босса можно вызвать снова и снова — награда в черепках будет выдаваться каждый раз.</div>';
html += '<div class="shopGrid">';
for (let i = 0; i < BOSSES.length; i++) {
const b = BOSSES[i];
const unlocked = maxLevelReached >= b.level;
const btnText = unlocked ? "СРАЗИТЬСЯ" : "НУЖЕН УРОВЕНЬ " + b.level;
html += <div class="bossCard"&gt; &lt;h3&gt;${b.name}</h3>
<div class="bdiff">{b.level} • НАГРАДА: <span class="skull-ico"></span> {unlocked ? "" : "owned"}" data-boss="{unlocked ? "" : "disabled"}>${btnText}&lt;/button&gt; &lt;/div>;
}
html += '</div>';
shopGrid.innerHTML = html;
applyCurrencyIcons();
shopGrid.querySelectorAll(".buyBtn").forEach(btn => {
if (btn.disabled) return;
btn.addEventListener("click", (e) => {
e.preventDefault();
e.stopPropagation();
const idx = parseInt(btn.dataset.boss, 10);
if (isNaN(idx)) return;
startLevel(BOSSES[idx].level);
closeShop();
});
});
}

function showShopMsg(text) {
shopMessage.textContent = text;
shopMessage.style.display = "block";
setTimeout(() => { shopMessage.style.display = "none"; }, 1800);
}
document.querySelectorAll(".shopTab").forEach(tab => {
tab.addEventListener("click", () => {
document.querySelectorAll(".shopTab").forEach(t => t.classList.remove("active"));
tab.classList.add("active");
renderShop();
});
});
shopClose.addEventListener("click", (e) => { e.preventDefault(); e.stopPropagation(); closeShop(); });
shopClose.addEventListener("pointerdown", (e) => { e.stopPropagation(); });
window.closeShop = closeShop;

function resize() {
if (!camera || !renderer) return;
camera.aspect = innerWidth / innerHeight;
camera.updateProjectionMatrix();
renderer.setSize(innerWidth, innerHeight);
renderer.setPixelRatio(dynamicPixelRatio);
if (composer) {
composer.setSize(innerWidth, innerHeight);
composer.setPixelRatio(dynamicPixelRatio);
}
if (bloomPass) bloomPass.setSize(innerWidth, innerHeight);
bloodSplatterCanvas.width = innerWidth;
bloodSplatterCanvas.height = innerHeight;
}

addEventListener("keydown", (event) => {
if (playerDying) return;
keys[event.code] = true;
if (event.code === "KeyR") reload();
if (event.code === "Space") jumpRequest = true;
if (event.code === "Escape" && shopOpen) closeShop();
if (event.code === "F3") {
perfBadgeVisible = !perfBadgeVisible;
perfBadge.style.display = perfBadgeVisible ? "block" : "none";
}
const numMatch = event.code.match(/^Digit([1-9])`$/);
if (numMatch) {
const idx = parseInt(numMatch[1], 10) - 1;
const ownedList = SHOP_ITEMS.filter(i => i.category === "weapons" && ownedItems[i.id]);
if (idx >= 0 && idx < ownedList.length) {
const newId = ownedList[idx].id;
if (newId !== currentWeapon) {
currentWeapon = newId;
reloading = false;
reloadTimer = 0;
droppedMagazine = null;
applyWeaponStats();
saveProgress();
playBuySound();
}
}
}
});
addEventListener("keyup", (event) => { keys[event.code] = false; });
addEventListener("mousedown", (event) => {
if (isMobile) return;
if (!started || ended || shopOpen) return;
if (event.button === 0) firing = true;
});
addEventListener("mouseup", (event) => { if (event.button === 0) firing = false; });
addEventListener("mousemove", (event) => {
if (isMobile) return;
if (!started || ended || shopOpen || playerDying) return;
yaw -= event.movementX * 0.0025;
pitch -= event.movementY * 0.0025;
pitch = THREE.MathUtils.clamp(pitch, -1.45, 1.45);
});

joystick.addEventListener("pointerdown", (event) => {
event.preventDefault();
if (playerDying) return;
joy.active = true;
joy.id = event.pointerId;
try { joystick.setPointerCapture(event.pointerId); } catch (e) {}
updateJoystick(event);
});
joystick.addEventListener("pointermove", (event) => {
if (!joy.active) return;
if (event.pointerId !== joy.id) return;
event.preventDefault();
updateJoystick(event);
});
joystick.addEventListener("pointerup", resetJoystick);
joystick.addEventListener("pointercancel", resetJoystick);
function updateJoystick(event) {
const rect = joystick.getBoundingClientRect();
const centerX = rect.left + rect.width / 2;
const centerY = rect.top + rect.height / 2;
let dx = event.clientX - centerX;
let dy = event.clientY - centerY;
const max = rect.width * 0.36;
const distance = Math.hypot(dx, dy);
if (distance > max) { dx = dx / distance * max; dy = dy / distance * max; }
joy.x = dx / max;
joy.y = dy / max;
knob.style.transform = "translate(calc(-50% + " + dx + "px), calc(-50% + " + dy + "px))";
}
function resetJoystick() {
joy.active = false;
joy.id = null;
joy.x = 0;
joy.y = 0;
knob.style.transform = "translate(-50%, -50%)";
}

document.addEventListener("pointerdown", (event) => {
if (!isMobile) return;
if (!started || ended || shopOpen || playerDying) return;
if (event.target.closest("#joystick") || event.target.closest("#buttons") || event.target.closest("#menu") || event.target.closest("#shopPanel")) return;
if (event.clientX < innerWidth * 0.42) return;
lookTouch.active = true;
lookTouch.id = event.pointerId;
lookTouch.x = event.clientX;
lookTouch.y = event.clientY;
}, { passive: true });
document.addEventListener("pointermove", (event) => {
if (!lookTouch.active) return;
if (event.pointerId !== lookTouch.id) return;
const dx = event.clientX - lookTouch.x;
const dy = event.clientY - lookTouch.y;
lookTouch.x = event.clientX;
lookTouch.y = event.clientY;
const sens = isMobile ? 0.006 : 0.008;
yaw -= dx * sens;
pitch -= dy * sens;
pitch = THREE.MathUtils.clamp(pitch, -1.45, 1.45);
}, { passive: true });
document.addEventListener("pointerup", (event) => {
if (event.pointerId === lookTouch.id) { lookTouch.active = false; lookTouch.id = null; }
}, { passive: true });

fireButton.addEventListener("pointerdown", (event) => { event.preventDefault(); if (!ended && !shopOpen && !playerDying) firing = true; }, { passive: false });
fireButton.addEventListener("pointerup", () => { firing = false; });
fireButton.addEventListener("pointercancel", () => { firing = false; });
fireButton.addEventListener("pointerleave", () => { firing = false; });
reloadButton.addEventListener("pointerdown", (event) => { event.preventDefault(); if (!playerDying) reload(); }, { passive: false });
jumpButton.addEventListener("pointerdown", (event) => { event.preventDefault(); if (!playerDying) jumpRequest = true; }, { passive: false });

document.addEventListener("visibilitychange", () => {
if (document.hidden && audioCtx && audioCtx.state === "running") audioCtx.suspend();
else if (!document.hidden && audioCtx && audioCtx.state === "suspended" && started) audioCtx.resume();
});
document.addEventListener("gesturestart", (e) => e.preventDefault());
document.addEventListener("gesturechange", (e) => e.preventDefault());
document.addEventListener("dblclick", (e) => e.preventDefault(), { passive: false });
document.addEventListener("contextmenu", (e) => { if (isMobile) e.preventDefault(); });

loadProgress();
applyCurrencyIcons();
loadLeaderboard();
</script>
</body>
</html>
"""

@app.route("/")
def home():
return render_template_string(HTML)

@app.post("/api/score")
def save_score():
data = request.get_json(silent=True) or {}
name = str(data.get("name", "Player")).strip()[:20] or "Player"
try:
value = int(data.get("score", 0))
except (TypeError, ValueError):
value = 0
value = max(0, min(value, 10000000))
scores.append({"name": name, "score": value})
scores.sort(key=lambda item: item["score"], reverse=True)
del scores[10:]
return jsonify({"success": True, "scores": scores[:10]})

@app.get("/api/scores")
def get_scores():
return jsonify(scores[:10])

if name == "main":
app.run(host="0.0.0.0", port=5000, debug=True)# Neon-strike