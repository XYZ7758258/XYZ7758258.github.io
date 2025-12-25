<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>可交互圣诞树</title>
    <script src="https://cdn.jsdelivr.net/npm/three@0.132.2/build/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.132.2/examples/js/controls/OrbitControls.js"></script>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            background: #000;
        }
        .info {
            position: absolute;
            top: 10px;
            left: 50%;
            transform: translateX(-50%);
            color: #fff;
            font-family: Arial, sans-serif;
        }
        .controls {
            position: absolute;
            bottom: 10px;
            left: 50%;
            transform: translateX(-50%);
            display: flex;
            gap: 10px;
        }
        button {
            background: #333;
            color: #fff;
            border: none;
            padding: 8px 15px;
            border-radius: 4px;
            cursor: pointer;
        }
        button:hover {
            background: #555;
        }
    </style>
</head>
<body>
    <div class="info">Merry Christmas</div>
    <div class="controls">
        <button id="addPhoto">添加照片</button>
        <button id="reset">重置</button>
    </div>
    <input type="file" id="photoInput" accept="image/*" style="display: none;">
    <script>
        // 初始化场景、相机、渲染器
        const scene = new THREE.Scene();
        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        camera.position.z = 5;
        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        document.body.appendChild(renderer.domElement);

        // 轨道控制器（支持触摸交互）
        const orbitControls = new THREE.OrbitControls(camera, renderer.domElement);
        orbitControls.enableDamping = true;
        orbitControls.enablePan = true;
        orbitControls.enableZoom = true;

        // 粒子数量和几何结构
        const particleCount = 8000;
        const particles = new THREE.BufferGeometry();
        const positions = new Float32Array(particleCount * 3);
        const colors = new Float32Array(particleCount * 3);
        const sizes = new Float32Array(particleCount);

        // 初始化粒子位置（分散状态）
        for (let i = 0; i < particleCount; i++) {
            positions[i * 3] = (Math.random() - 0.5) * 5;
            positions[i * 3 + 1] = (Math.random() - 0.5) * 5;
            positions[i * 3 + 2] = (Math.random() - 0.5) * 5;

            // 金色系颜色
            const gold = Math.random() * 0.5 + 0.5;
            colors[i * 3] = gold;
            colors[i * 3 + 1] = gold * 0.8 + 0.2;
            colors[i * 3 + 2] = Math.random() * 0.3;

            sizes[i] = Math.random() * 0.1 + 0.05;
        }

        particles.setAttribute('position', new THREE.BufferAttribute(positions, 3));
        particles.setAttribute('color', new THREE.BufferAttribute(colors, 3));
        particles.setAttribute('size', new THREE.BufferAttribute(sizes, 1));

        // 粒子材质（支持大小变化）
        const material = new THREE.PointsMaterial({
            vertexColors: true,
            transparent: true,
            opacity: 0.9,
            sizeAttenuation: true
        });

        const particleSystem = new THREE.Points(particles, material);
        scene.add(particleSystem);

        // 顶部装饰
        const topGeometry = new THREE.SphereGeometry(0.2);
        const topMaterial = new THREE.MeshBasicMaterial({ color: 0xffd700 });
        const top = new THREE.Mesh(topGeometry, topMaterial);
        top.position.y = 2.5;
        scene.add(top);

        // 照片对象数组
        const photos = [];

        // 添加照片函数
        function addPhoto(file) {
            const reader = new FileReader();
            reader.onload = function(event) {
                const texture = new THREE.TextureLoader().load(event.target.result);
                const geometry = new THREE.PlaneGeometry(0.8, 0.5);
                const material = new THREE.MeshBasicMaterial({ map: texture, transparent: true });
                const photo = new THREE.Mesh(geometry, material);
                photo.position.set(
                    (Math.random() - 0.5) * 3,
                    (Math.random() - 0.5) * 2,
                    (Math.random() - 0.5) * 3
                );
                scene.add(photo);
                photos.push(photo);
            };
            reader.readAsDataURL(file);
        }

        // 重置函数
        function reset() {
            photos.forEach(photo => scene.remove(photo));
            photos.length = 0;

            // 重置粒子位置为分散状态
            for (let i = 0; i < particleCount; i++) {
                positions[i * 3] = (Math.random() - 0.5) * 5;
                positions[i * 3 + 1] = (Math.random() - 0.5) * 5;
                positions[i * 3 + 2] = (Math.random() - 0.5) * 5;
            }
   
