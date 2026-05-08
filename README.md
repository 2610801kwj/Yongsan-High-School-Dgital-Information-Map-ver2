<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>학교 디지털 정보 지도 (OBJ+MTL)</title>
    <style>
        body { margin: 0; overflow: hidden; background-color: #f0f0f0; font-family: 'Malgun Gothic', sans-serif; }
        #ui-layer {
            position: absolute; top: 20px; width: 100%;
            text-align: center; z-index: 10; pointer-events: none;
        }
        .title-box {
            display: inline-block; background: rgba(255, 255, 255, 0.9);
            padding: 15px 25px; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.1);
        }
        #popup {
            position: fixed; top: 50%; left: 50%; transform: translate(-50%, -50%);
            background: white; padding: 30px; border-radius: 15px;
            display: none; z-index: 100; box-shadow: 0 0 30px rgba(0,0,0,0.3);
            text-align: center; min-width: 280px;
        }
        #popup button { 
            margin-top: 20px; padding: 10px 25px; border: none;
            background: #4A90E2; color: white; border-radius: 5px; cursor: pointer;
        }
    </style>
</head>
<body>

    <div id="ui-layer">
        <div class="title-box">
            <h1 style="margin:0; font-size: 22px;">🏫 학교 디지털 정보 지도</h1>
            <p style="margin:5px 0 0; font-size: 14px;">객체를 클릭하여 정보를 확인하세요</p>
        </div>
    </div>

    <div id="popup">
        <h2 id="obj-name">시설 정보</h2>
        <p id="obj-desc">해당 구역의 상세 디지털 정보입니다.</p>
        <button onclick="closePopup()">닫기</button>
    </div>

    <script type="importmap">
        {
            "imports": {
                "three": "https://unpkg.com/three@0.160.0/build/three.module.js",
                "three/addons/": "https://unpkg.com/three@0.160.0/examples/jsm/"
            }
        }
    </script>

    <script type="module">
        import * as THREE from 'three';
        import { OBJLoader } from 'three/addons/loaders/OBJLoader.js';
        import { MTLLoader } from 'three/addons/loaders/MTLLoader.js';
        import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0xf0f0f0);

        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        camera.position.set(50, 50, 50);

        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        document.body.appendChild(renderer.domElement);

        // 조명 (색감이 잘 보이도록 밝게 설정)
        const ambientLight = new THREE.AmbientLight(0xffffff, 0.8);
        scene.add(ambientLight);
        const dirLight = new THREE.DirectionalLight(0xffffff, 1);
        dirLight.position.set(10, 20, 10);
        scene.add(dirLight);

        const controls = new OrbitControls(camera, renderer.domElement);

        // MTL 파일 먼저 로드 후 OBJ 로드
        const mtlLoader = new MTLLoader();
        mtlLoader.load('./Fantastic Jofo-Lahdi.mtl', (materials) => {
            materials.preload();
            
            const objLoader = new OBJLoader();
            objLoader.setMaterials(materials); // 로드된 재질 적용
            objLoader.load('./Fantastic Jofo-Lahdi.obj', (object) => {
                
                // 모델 중앙 정렬
                const box = new THREE.Box3().setFromObject(object);
                const center = box.getCenter(new THREE.Vector3());
                object.position.sub(center);
                
                scene.add(object);
            });
        });

        const raycaster = new THREE.Raycaster();
        const mouse = new THREE.Vector2();

        window.addEventListener('click', (event) => {
            mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
            mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;

            raycaster.setFromCamera(mouse, camera);
            const intersects = raycaster.intersectObjects(scene.children, true);

            if (intersects.length > 0) {
                // 클릭한 객체 이름 표시 (이름이 없으면 기본값)
                const clickedObj = intersects[0].object;
                const name = clickedObj.name || "디지털 정보 구역";
                
                document.getElementById('obj-name').innerText = name;
                document.getElementById('popup').style.display = 'block';
            }
        });

        window.closePopup = () => {
            document.getElementById('popup').style.display = 'none';
        };

        function animate() {
            requestAnimationFrame(animate);
            controls.update();
            renderer.render(scene, camera);
        }
        animate();

        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        });
    </script>
</body>
</html>
