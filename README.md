<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>학교 디지털 정보 지도 (OBJ+MTL)</title>
    <style>
        body { margin: 0; overflow: hidden; background-color: #f0f0f0; font-family: 'Malgun Gothic', sans-serif; }
        
        /* 1. 상단 배너를 화면 중앙으로 정렬 */
        #ui-layer {
            position: absolute; 
            top: 20px; 
            left: 50%;
            transform: translateX(-50%); /* 화면 가로 기준 완벽한 중앙 정렬 */
            z-index: 10; 
            pointer-events: none;
        }
        .title-box {
            display: inline-block; 
            background: rgba(255, 255, 255, 0.95);
            padding: 15px 25px; 
            border-radius: 12px; 
            box-shadow: 0 4px 15px rgba(0,0,0,0.15);
            text-align: center;
        }
        
        /* 2. 정보 팝업창 스타일 (중앙 배치 유지 및 스타일 고도화) */
        #popup {
            position: fixed; 
            top: 50%; 
            left: 50%; 
            transform: translate(-50%, -50%);
            background: white; 
            padding: 30px; 
            border-radius: 15px;
            display: none; 
            z-index: 100; 
            box-shadow: 0 0 30px rgba(0,0,0,0.3);
            text-align: center; 
            min-width: 280px;
        }
        #popup button { 
            margin-top: 20px; 
            padding: 10px 25px; 
            border: none;
            background: #4A90E2; 
            color: white; 
            border-radius: 5px; 
            cursor: pointer;
            font-weight: bold;
        }
        #popup button:hover {
            background: #357ABD;
        }
    </style>
</head>
<body>

    <div id="ui-layer">
        <div class="title-box">
            <h1 style="margin:0; font-size: 22px; color: #333;">🏫 학교 디지털 정보 지도</h1>
            <p style="margin:5px 0 0; font-size: 14px; color: #666;">객체를 클릭하여 정보를 확인하세요</p>
        </div>
    </div>

    <div id="popup">
        <h2 id="obj-name">시설 정보</h2>
        <p id="obj-desc">해당 구역의 상세 디지털 정보입니다.</p>
        <button id="close-btn">닫기</button>
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

        // [중요] 3D 모델 내부의 메쉬 이름별 한글 이름/설정 매핑 테이블
        // 콘솔창(F12)에 찍히는 clickedObj.name을 확인한 후 매핑해 주시면 됩니다.
        const buildingData = {
            "Cube001": { name: "본관 대강당", desc: "교무실, 교장실 및 행정 부서가 위치한 메인 건물입니다." },
            "Cube002": { name: "정보과학관", desc: "컴퓨터실, AI 실습실 및 디지털 도서관이 있는 곳입니다." },
            "Gym": { name: "체육관", desc: "학생들의 실내 체육 활동 및 입학식/졸업식이 진행되는 곳입니다." },
            "Playground": { name: "운동장", desc: "인조잔디가 깔린 친환경 대운동장 구역입니다." }
        };

        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0xf0f0f0);

        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        camera.position.set(50, 50, 50);

        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        document.body.appendChild(renderer.domElement);

        const ambientLight = new THREE.AmbientLight(0xffffff, 0.8);
        scene.add(ambientLight);
        const dirLight = new THREE.DirectionalLight(0xffffff, 1);
        dirLight.position.set(10, 20, 10);
        scene.add(dirLight);

        const controls = new OrbitControls(camera, renderer.domElement);

        const mtlLoader = new MTLLoader();
        mtlLoader.load('./Fantastic Jofo-Lahdi.mtl', (materials) => {
            materials.preload();
            
            const objLoader = new OBJLoader();
            objLoader.setMaterials(materials);
            objLoader.load('./Fantastic Jofo-Lahdi.obj', (object) => {
                
                const box = new THREE.Box3().setFromObject(object);
                const center = box.getCenter(new THREE.Vector3());
                object.position.sub(center);
                
                scene.add(object);
            });
        });

        const raycaster = new THREE.Raycaster();
        const mouse = new THREE.Vector2();

        // 클릭 이벤트 핸들러
        window.addEventListener('click', (event) => {
            // UI 레이어나 팝업 버블 내부 클릭 시 3D 레이캐스팅 무시 방지 처리
            if (event.target.closest('#ui-layer') || event.target.closest('#popup')) return;

            mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
            mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;

            raycaster.setFromCamera(mouse, camera);
            const intersects = raycaster.intersectObjects(scene.children, true);

            // 팝업 초기화: 새 객체를 클릭할 때 기존 팝업 내용을 초기화하거나 먼저 닫아 중복 발생 방지
            const popup = document.getElementById('popup');
            popup.style.display = 'none';

            if (intersects.length > 0) {
                const clickedObj = intersects[0].object;
                
                // 개발자 도구(F12) 콘솔창에서 내가 클릭한 3D 객체의 원래 고유 이름을 확인할 수 있습니다.
                console.log("클릭한 객체의 고유 ID/Name:", clickedObj.name);

                let displayName = "디지털 정보 구역";
                let displayDesc = "해당 구역의 상세 디지털 정보입니다.";

                // 매핑 테이블에 존재하는 이름 파일인지 체크 후 변경
                if (buildingData[clickedObj.name]) {
                    displayName = buildingData[clickedObj.name].name;
                    displayDesc = buildingData[clickedObj.name].desc;
                } else if (clickedObj.parent && buildingData[clickedObj.parent.name]) {
                    // 그룹(Parent)으로 묶여있는 경우 부모 이름도 체크
                    displayName = buildingData[clickedObj.parent.name].name;
                    displayDesc = buildingData[clickedObj.parent.name].desc;
                } else if (clickedObj.name) {
                    // 데이터 등록은 안 되었으나 3D 파일 내에 고유 이름 명명이 되어있을 때
                    displayName = clickedObj.name;
                }
                
                // 단일 팝업 요소에 데이터 갱신 후 화면에 노출
                document.getElementById('obj-name').innerText = displayName;
                document.getElementById('obj-desc').innerText = displayDesc;
                popup.style.display = 'block';
            }
        });

        // 팝업 닫기 이벤트 바인딩 (스코프 에러 방지를 위해 표준 리스너 형식 사용)
        document.getElementById('close-btn').addEventListener('click', () => {
            document.getElementById('popup').style.display = 'none';
        });

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
