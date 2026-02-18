<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Velocity Ultra - Mobile Racing</title>
    <style>
        body { margin: 0; overflow: hidden; background: #000; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; }
        #ui-layer { position: absolute; top: 0; left: 0; width: 100%; height: 100%; pointer-events: none; color: white; }
        
        /* Mobile Controls */
        .touch-btn { 
            position: absolute; bottom: 20px; width: 80px; height: 80px; 
            background: rgba(255,255,255,0.2); border-radius: 50%; 
            pointer-events: auto; display: flex; align-items: center; justify-content: center;
            user-select: none; -webkit-tap-highlight-color: transparent;
        }
        #brake { left: 20px; bottom: 120px; background: rgba(255, 50, 50, 0.4); }
        #accel { right: 20px; bottom: 120px; background: rgba(50, 255, 50, 0.4); }
        #steer-left { left: 20px; }
        #steer-right { left: 110px; }

        /* HUD */
        #hud { position: absolute; top: 20px; left: 20px; font-size: 24px; font-weight: bold; text-shadow: 2px 2px #000; }
        #mission-info { position: absolute; top: 20px; right: 20px; text-align: right; }
        
        /* Menu */
        #menu { 
            position: absolute; top: 0; left: 0; width: 100%; height: 100%; 
            background: rgba(0,0,0,0.85); pointer-events: auto; 
            display: flex; flex-direction: column; align-items: center; justify-content: center;
        }
        button { 
            padding: 15px 40px; margin: 10px; font-size: 20px; cursor: pointer;
            background: #e74c3c; color: white; border: none; border-radius: 5px;
        }
    </style>
</head>
<body>

    <div id="ui-layer">
        <div id="hud">0 KM/H</div>
        <div id="mission-info">Mission 1: Reach the End<br>Time: <span id="timer">60</span>s</div>
        
        <div id="brake" class="touch-btn">STOP</div>
        <div id="steer-left" class="touch-btn">←</div>
        <div id="steer-right" class="touch-btn">→</div>
        <div id="accel" class="touch-btn">GO</div>

        <div id="menu">
            <h1>VELOCITY ULTRA</h1>
            <p>High Performance Racing</p>
            <button onclick="startGame()">START MISSION</button>
            <button onclick="openGarage()">GARAGE</button>
        </div>
    </div>

    <script type="importmap">
        { "imports": { "three": "https://unpkg.com/three@0.160.0/build/three.module.js" } }
    </script>

    <script type="module">
        import * as THREE from 'three';
const Missions = [
  { id: 1, type: "TIME_TRIAL", target: 1000, timeLimit: 60, reward: 500 },
  { id: 2, type: "EVADE", policeCount: 2, timeLimit: 120, reward: 1200 }
];
        // --- GAME STATE ---
        const GameState = {
            speed: 0,
            maxSpeed: 0.8,
            acceleration: 0.005,
            friction: 0.002,
            steering: 0,
            isMoving: false,
            currentMission: 1,
            currency: 0
        };

        // --- SCENE SETUP ---
        const scene = new THREE.Scene();
        scene.fog = new THREE.FogExp2(0x1a1a1a, 0.02);
        
        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        const renderer = new THREE.WebGLRenderer({ antialias: true, powerPreference: "high-performance" });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
        renderer.shadowMap.enabled = true;
        document.body.appendChild(renderer.domElement);

        // --- LIGHTING (Dynamic Day/Night) ---
        const sun = new THREE.DirectionalLight(0xffffff, 1.0);
        sun.position.set(50, 100, 50);
        sun.castShadow = true;
        scene.add(sun);
        scene.add(new THREE.AmbientLight(0x404040, 0.5));

        // --- CAR CREATION ---
        function createCar() {
function checkMission(){
  if(currentMission >= Missions.length) return;

  if(car.position.z < -Missions[currentMission].target){
    alert("Mission " + Missions[currentMission].id + " Complete!");
    currentMission++;
  }
}
            const group = new THREE.Group();
            
            // Body (Chassis) - Using a sleek metallic material
            const bodyGeom = new THREE.BoxGeometry(1.2, 0.5, 2.5);
            const bodyMat = new THREE.MeshStandardMaterial({ 
                color: 0xee0000, metalness: 0.9, roughness: 0.1 
            });
            const body = new THREE.Mesh(bodyGeom, bodyMat);
            body.position.y = 0.4;
            body.castShadow = true;
            group.add(body);

            // Cabin
            const cabinGeom = new THREE.BoxGeometry(1, 0.4, 1.2);
            const cabin = new THREE.Mesh(cabinGeom, bodyMat);
            cabin.position.set(0, 0.7, -0.2);
            group.add(cabin);

            // Wheels
            const wheelGeom = new THREE.CylinderGeometry(0.3, 0.3, 0.2, 16);
            const wheelMat = new THREE.MeshStandardMaterial({ color: 0x222222 });
            const wheelPos = [[0.7, 0.3, 0.8], [-0.7, 0.3, 0.8], [0.7, 0.3, -0.8], [-0.7, 0.3, -0.8]];
            
            wheelPos.forEach(pos => {
                const wheel = new THREE.Mesh(wheelGeom, wheelMat);
                wheel.rotation.z = Math.PI / 2;
                wheel.position.set(pos[0], pos[1], pos[2]);
                group.add(wheel);
            });

            return group;
        }

        const car = createCar();
        scene.add(car);

        // --- ENVIRONMENT (Road Generation) ---
        const roadGroup = new THREE.Group();
        const roadMat = new THREE.MeshStandardMaterial({ color: 0x222222 });
        const roadGeom = new THREE.PlaneGeometry(10, 1000);
        const road = new THREE.Mesh(roadGeom, roadMat);
        road.rotation.x = -Math.PI / 2;
        road.receiveShadow = true;
        roadGroup.add(road);
        scene.add(roadGroup);

        // Add Simple Buildings for "City"
        for(let i = 0; i < 50; i++) {
            const bGeom = new THREE.BoxGeometry(5, 10 + Math.random() * 20, 5);
            const bMat = new THREE.MeshStandardMaterial({ color: 0x444444 });
            const b = new THREE.Mesh(bGeom, bMat);
            b.position.set(i % 2 === 0 ? 10 : -10, 5, -i * 20);
            scene.add(b);
        }

        // --- INPUT HANDLING ---
        const keys = { accel: false, brake: false, left: false, right: false };
        
        const setupTouch = (id, key) => {
            const btn = document.getElementById(id);
            btn.addEventListener('touchstart', () => keys[key] = true);
            btn.addEventListener('touchend', () => keys[key] = false);
        };

        setupTouch('accel', 'accel');
        setupTouch('brake', 'brake');
        setupTouch('steer-left', 'left');
        setupTouch('steer-right', 'right');

        // --- PHYSICS LOOP ---
        function updatePhysics() {
        checkMission();
            // Speed Logic
            if (keys.accel) GameState.speed += GameState.acceleration;
            else if (keys.brake) GameState.speed -= GameState.acceleration * 2;
            else GameState.speed *= (1 - GameState.friction);

            GameState.speed = Math.max(0, Math.min(GameState.speed, GameState.maxSpeed));

            // Steering Logic
            if (GameState.speed > 0.01) {
                const steerAmount = 0.04 * (GameState.speed / GameState.maxSpeed);
                if (keys.left) car.rotation.y += steerAmount;
                if (keys.right) car.rotation.y -= steerAmount;
            }

            // Movement
            car.position.x -= Math.sin(car.rotation.y) * GameState.speed;
            car.position.z -= Math.cos(car.rotation.y) * GameState.speed;

            // Camera Follow
            const camOffset = new THREE.Vector3(0, 2, 5).applyQuaternion(car.quaternion);
            camera.position.lerp(car.position.clone().add(camOffset), 0.1);
            camera.lookAt(car.position);

            // Update UI
            document.getElementById('hud').innerText = `${Math.floor(GameState.speed * 300)} KM/H`;
        }

        // --- ANIMATION LOOP ---
        function animate() {
            requestAnimationFrame(animate);
            updatePhysics();
            renderer.render(scene, camera);
        }

        window.startGame = () => {
            document.getElementById('menu').style.display = 'none';
            animate();
        };

        window.openGarage = () => alert("Garage: Upgrade Speed using mission rewards!");

    </script>
</body>
</html>
