<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Basic FPS Demo</title>
  <style>
    body { margin: 0; overflow: hidden; background: #000; color: white; font-family: Arial, sans-serif; }
    #instructions {
      position: absolute;
      top: 20px;
      width: 100%;
      text-align: center;
      color: white;
      font-size: 16px;
      z-index: 1;
      user-select: none;
      cursor: pointer;
    }
  </style>
</head>
<body>
  <div id="instructions">
    Click to play. WASD to move, mouse to look around.
  </div>

  <!-- Three.js library -->
  <script src="https://cdn.jsdelivr.net/npm/three@0.153.0/build/three.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/three@0.153.0/examples/js/controls/PointerLockControls.js"></script>

  <script>
    const instructions = document.getElementById('instructions');

    let camera, scene, renderer, controls;
    let objects = [];
    let moveForward = false;
    let moveBackward = false;
    let moveLeft = false;
    let moveRight = false;
    let canJump = false;

    let prevTime = performance.now();
    let velocity = new THREE.Vector3();
    let direction = new THREE.Vector3();

    init();
    animate();

    function init() {
      camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 1, 1000);
      camera.position.y = 10;

      scene = new THREE.Scene();
      scene.background = new THREE.Color(0x000000);

      const light = new THREE.HemisphereLight(0xffffff, 0x444444);
      light.position.set(0, 200, 0);
      scene.add(light);

      const light2 = new THREE.DirectionalLight(0xffffff);
      light2.position.set(0, 200, 100);
      light2.castShadow = true;
      scene.add(light2);

      controls = new THREE.PointerLockControls(camera, document.body);

      instructions.addEventListener('click', function () {
        controls.lock();
      }, false);

      controls.addEventListener('lock', function () {
        instructions.style.display = 'none';
      });

      controls.addEventListener('unlock', function () {
        instructions.style.display = '';
      });

      scene.add(controls.getObject());

      const onKeyDown = function (event) {
        switch (event.code) {
          case 'ArrowUp':
          case 'KeyW':
            moveForward = true;
            break;
          case 'ArrowLeft':
          case 'KeyA':
            moveLeft = true;
            break;
          case 'ArrowDown':
          case 'KeyS':
            moveBackward = true;
            break;
          case 'ArrowRight':
          case 'KeyD':
            moveRight = true;
            break;
          case 'Space':
            if (canJump === true) velocity.y += 350;
            canJump = false;
            break;
        }
      };

      const onKeyUp = function (event) {
        switch (event.code) {
          case 'ArrowUp':
          case 'KeyW':
            moveForward = false;
            break;
          case 'ArrowLeft':
          case 'KeyA':
            moveLeft = false;
            break;
          case 'ArrowDown':
          case 'KeyS':
            moveBackward = false;
            break;
          case 'ArrowRight':
          case 'KeyD':
            moveRight = false;
            break;
        }
      };

      document.addEventListener('keydown', onKeyDown);
      document.addEventListener('keyup', onKeyUp);

      // Floor
      const floorGeometry = new THREE.PlaneGeometry(2000, 2000, 100, 100);
      floorGeometry.rotateX(- Math.PI / 2);

      const floorMaterial = new THREE.MeshBasicMaterial({color: 0x808080, wireframe: true});
      const floor = new THREE.Mesh(floorGeometry, floorMaterial);
      scene.add(floor);

      // Some cubes as "walls"
      const boxGeometry = new THREE.BoxGeometry(20, 20, 20);

      for (let i = 0; i < 10; i++) {
        const boxMaterial = new THREE.MeshStandardMaterial({color: 0x888888});
        const box = new THREE.Mesh(boxGeometry, boxMaterial);

        box.position.x = Math.random() * 800 - 400;
        box.position.y = 10;
        box.position.z = Math.random() * 800 - 400;

        scene.add(box);
        objects.push(box);
      }

      renderer = new THREE.WebGLRenderer({antialias: true});
      renderer.setSize(window.innerWidth, window.innerHeight);
      document.body.appendChild(renderer.domElement);

      window.addEventListener('resize', onWindowResize);
    }

    function onWindowResize() {
      camera.aspect = window.innerWidth/window.innerHeight;
      camera.updateProjectionMatrix();

      renderer.setSize(window.innerWidth, window.innerHeight);
    }

    function animate() {
      requestAnimationFrame(animate);

      if (controls.isLocked === true) {
        const time = performance.now();
        const delta = (time - prevTime) / 1000;

        velocity.x -= velocity.x * 10.0 * delta;
        velocity.z -= velocity.z * 10.0 * delta;

        velocity.y -= 9.8 * 100.0 * delta; // gravity

        direction.z = Number(moveForward) - Number(moveBackward);
        direction.x = Number(moveRight) - Number(moveLeft);
        direction.normalize();

        if (moveForward || moveBackward) velocity.z -= direction.z * 400.0 * delta;
        if (moveLeft || moveRight) velocity.x -= direction.x * 400.0 * delta;

        controls.moveRight(- velocity.x * delta);
        controls.moveForward(- velocity.z * delta);

        controls.getObject().position.y += (velocity.y * delta);
        if (controls.getObject().position.y < 10) {
          velocity.y = 0;
          controls.getObject().position.y = 10;
          canJump = true;
        }

        prevTime = time;
      }

      renderer.render(scene, camera);
    }
  </script>
</body>
</html>

