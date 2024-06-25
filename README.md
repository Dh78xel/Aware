<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Survival Horror Game</title>
    <style>
        body { 
            margin: 0; 
            overflow: hidden; 
            font-family: Arial, sans-serif;
        }

        #healthBar {
            position: absolute;
            top: 10px;
            left: 10px;
            width: 200px;
            height: 20px;
            background-color: red;
        }
    </style>
</head>
<body>
    <div id="healthBar"></div>
    <audio id="shootSound" src="https://www.myinstants.com/media/sounds/laser.mp3"></audio>
    <audio id="hitSound" src="https://www.myinstants.com/media/sounds/zombie-1.mp3"></audio>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/loaders/GLTFLoader.js"></script>
    <script>
        // إعداد المشهد، الكاميرا، وعناصر التحكم
        const scene = new THREE.Scene();
        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        const renderer = new THREE.WebGLRenderer();
        renderer.setSize(window.innerWidth, window.innerHeight);
        document.body.appendChild(renderer.domElement);

        // إعداد إضاءة المشهد
        const light = new THREE.PointLight(0xffffff, 1, 100);
        light.position.set(10, 10, 10);
        scene.add(light);

        const ambientLight = new THREE.AmbientLight(0x404040);
        scene.add(ambientLight);

        const directionalLight = new THREE.DirectionalLight(0xffffff, 0.5);
        directionalLight.position.set(1, 1, 1).normalize();
        scene.add(directionalLight);

        // تحميل النموذج الثلاثي الأبعاد للاعب
        const loader = new THREE.GLTFLoader();
        let player;
        loader.load('https://threejsfundamentals.org/threejs/resources/models/cartoon_lowpoly/scene.gltf', function (gltf) {
            player = gltf.scene;
            player.scale.set(0.5, 0.5, 0.5);
            player.position.set(0, 0, 0);
            scene.add(player);
            camera.position.set(0, 2, 5);
        });

        // إعداد الأعداء
        const enemies = [];
        loader.load('https://threejsfundamentals.org/threejs/resources/models/robot/robot.gltf', function (gltf) {
            for (let i = 0; i < 5; i++) {
                let zombie = gltf.scene.clone();
                zombie.position.set(Math.random() * 20 - 10, 0, Math.random() * 20 - 10);
                zombie.scale.set(0.5, 0.5, 0.5);
                enemies.push(zombie);
                scene.add(zombie);
            }
        });

        // إعداد عناصر اللعبة
        const items = [];
        loader.load('https://threejsfundamentals.org/threejs/resources/models/box/box.gltf', function (gltf) {
            for (let i = 0; i < 3; i++) {
                let item = gltf.scene.clone();
                item.position.set(Math.random() * 20 - 10, 0, Math.random() * 20 - 10);
                item.scale.set(0.3, 0.3, 0.3);
                items.push(item);
                scene.add(item);
            }
        });

        // إعداد صحة اللاعب
        let playerHealth = 100;
        const healthBar = document.getElementById('healthBar');
        healthBar.style.width = `${playerHealth}%`;

        let bullets = [];
        const shootSound = document.getElementById('shootSound');
        const hitSound = document.getElementById('hitSound');

        // إنشاء طلقة جديدة
        function shoot() {
            shootSound.play();
            const bulletGeometry = new THREE.SphereGeometry(0.05, 32, 32);
            const bulletMaterial = new THREE.MeshBasicMaterial({ color: 0xff0000 });
            const bullet = new THREE.Mesh(bulletGeometry, bulletMaterial);
            bullet.position.copy(player.position);
            bullets.push(bullet);
            scene.add(bullet);
        }

        // تحديث الطلقات
        function updateBullets() {
            bullets.forEach((bullet, index) => {
                bullet.position.z -= 0.2;

                // تحقق من التصادم مع الأعداء
                enemies.forEach((zombie, zombieIndex) => {
                    if (bullet.position.distanceTo(zombie.position) < 0.5) {
                        scene.remove(zombie);
                        enemies.splice(zombieIndex, 1);
                        scene.remove(bullet);
                        bullets.splice(index, 1);
                        hitSound.play();
                    }
                });
            });
        }

        // تحديث مواقع الأعداء مع الذكاء الاصطناعي المحسن
        function updateEnemies() {
            enemies.forEach(zombie => {
                let direction = new THREE.Vector3();
                direction.subVectors(player.position, zombie.position).normalize();
                zombie.position.add(direction.multiplyScalar(0.02));
            });
        }

        // تحديث عناصر اللعبة
        function updateItems() {
            items.forEach((item, index) => {
                if (player.position.distanceTo(item.position) < 1) {
                    scene.remove(item);
                    items.splice(index, 1);
                    playerHealth = Math.min(100, playerHealth + 20);
                }
            });
        }

        // التحديثات والحركة
        function animate() {
            requestAnimationFrame(animate);
            updateEnemies();
            updateBullets();
            updateItems();
            healthBar.style.width = `${playerHealth}%`;
            renderer.render(scene, camera);
        }
        animate();

        // التحكم في اللاعب
        window.addEventListener('keydown', function (event) {
            switch (event.key) {
                case 'w':
                    player.position.z -= 0.1;
                    break;
                case 's':
                    player.position.z += 0.1;
                    break;
                case 'a':
                    player.position.x -= 0.1;
                    break;
                case 'd':
                    player.position.x += 0.1;
                    break;
                case 'Shift':
                    player.position.z -= 0.2;
                    break;
            }
        });

        // إطلاق النار عند النقر
        window.addEventListener('click', shoot);
    </script>
</body>
</html>
