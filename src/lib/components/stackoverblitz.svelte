<script>
  import * as THREE from "three";
  import { T, useFrame } from "@threlte/core";
  import { OrbitControls, Grid } from "@threlte/extras";
  import CustomRenderer from "./CustomRenderer.svelte";
  const count = 5000;

  const positions = new Float32Array(count * 3);

  for (let i = 0; i < count * 3; i++) {
    positions[i] = (Math.random() - 0.5) * 10;
  }
  const geometry = new THREE.BufferGeometry();
  $: geometry.setAttribute("position", new THREE.BufferAttribute(positions, 3));

  let pointsMesh;
</script>

<T.Points bind:ref={pointsMesh}>
  <T is={geometry} />
  <!-- <T.PointsMaterial size={0.1} color={"pink"} /> -->
  <T.ShaderMaterial
    uniforms={{
      progress: { value: 0 },
      speed: { value: speed },
      shakeStrength: { value: shakeStrength },
    }}
    fragmentShader={`

varying vec3 vColor;
void main() {
  gl_FragColor = vec4(vColor, 1.0);
}`}

    vertexShader={`
    varying vec3 vColor;
void main() {	
  vec4 localPosition = vec4(position, 1.0);
 gl_PointSize = 0.2;
  gl_Position = projectionMatrix * modelViewMatrix * localPosition;
}`}
    opacity={0.8}
    vertexColors={true}
    blending={THREE.AdditiveBlending}
    depthWrite={false}
  />
</T.Points>

<CustomRenderer selectedMesh={pointsMesh} />

<T.PerspectiveCamera makeDefault position={[0, 6, -10]} fov={15} zoom={0.2}>
  <OrbitControls />
</T.PerspectiveCamera>
