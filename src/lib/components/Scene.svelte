<script>
  // @ts-nocheck
  import * as THREE from "three";

  import { T, useFrame } from "@threlte/core";
  import { Align, OrbitControls } from "@threlte/extras";

  import { getContext, onMount } from "svelte";

  import { vertexOnSphere } from "$lib/utility/utility";
  import vertexShader from "$lib/shader/vertexShader.glsl";
  import fragmentShader from "$lib/shader/fragmentShader.glsl";

  import CustomRenderer from "./CustomRenderer.svelte";

  const { data, scales } = getContext("Chart");
  const geometry = new THREE.BufferGeometry();

  $: ({ depthScale, sizeScale, colorScale, timeScale } = $scales);
  $: count = $data.length;

  $: positions = new Float32Array(count * 3);
  $: atDepth = new Float32Array(count * 3);
  $: colors = new Float32Array(count * 3);
  $: sizes = new Float32Array(count);
  $: time = new Float32Array(count);
  let initialColor = new THREE.Color(0xffffff);
  let scaleFactor = 3;
  let earthMesh;


  $: $data.forEach((earthquake, i) => {
    const earthquakeCoordinates = vertexOnSphere(
      [earthquake.longitude, earthquake.latitude],
      depthScale(earthquake.depth)
    );
    const earthquakeSurfaceCoordinates = vertexOnSphere(
      [earthquake.longitude, earthquake.latitude],
      10
    );

    positions[i * 3] = earthquakeSurfaceCoordinates.x;
    positions[i * 3 + 1] = earthquakeSurfaceCoordinates.y;
    positions[i * 3 + 2] = earthquakeSurfaceCoordinates.z;

    atDepth[i * 3] = earthquakeCoordinates.x;
    atDepth[i * 3 + 1] = earthquakeCoordinates.y;
    atDepth[i * 3 + 2] = earthquakeCoordinates.z;

    let color = initialColor.setStyle(colorScale(earthquake.magnitude));
    colors[i * 3] = color.r;
    colors[i * 3 + 1] = color.g;
    colors[i * 3 + 2] = color.b;

    sizes[i] = sizeScale(earthquake.magnitude) * scaleFactor;
    time[i] = timeScale(earthquake.date);
  });

  $: geometry.setAttribute("position", new THREE.BufferAttribute(positions, 3));
  $: geometry.setAttribute("atDepth", new THREE.BufferAttribute(atDepth, 3));
  $: geometry.setAttribute("color", new THREE.BufferAttribute(colors, 3));
  $: geometry.setAttribute("time", new THREE.BufferAttribute(time, 1));
  $: geometry.setAttribute("size", new THREE.BufferAttribute(sizes, 1));
  onMount(() => {
    if (earthMesh) {
      // Position the mesh within the group to set the pivot offset
      earthMesh.position.x += 1; // Adjust this value to move the pivot to the right
      // Call this after updating positions to ensure the scene is updated
      earthMesh.updateMatrixWorld();
    }
  });


  const speed = 0.1; // Speed of the progress increment
  const shakeStrength = 0.2; // Strength of the shake effect[0,2]
  useFrame(({ camera }, delta) => {
    // Update the progress based on the frame's delta time

    const progress = earthMesh.material.uniforms.progress.value;
    // Update the shader uniform if the particle system is initialized
    if (earthMesh.material.uniforms.progress) {
      // console.log(progress)
      earthMesh.material.uniforms.progress.value = progress + speed;
    }
  });

</script>

<T.PerspectiveCamera makeDefault position={[10, 10, 40]} fov={45}>
  <OrbitControls
    enableZoom={true}
    enableDamping
    autoRotate
    target={[0, 0, 5]}
  />
</T.PerspectiveCamera>

<!-- <T.DirectionalLight position.y={10} position.z={10} /> -->

<!-- <Align> -->
<T.Points bind:ref={earthMesh}>
  <T is={geometry} />
  <T.ShaderMaterial
    uniforms={{
      progress: { value: 0 },
      speed: { value: speed },
      shakeStrength: { value: shakeStrength },
    }}
    {vertexShader}
    {fragmentShader}
    vertexColors={true}
    blending={THREE.AdditiveBlending}
    depthWrite={false}
  />
  <!-- <T.ShaderMaterial
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
        vColor = color;
       gl_PointSize =10.0;
        gl_Position = projectionMatrix * modelViewMatrix * localPosition;
      }`}
    vertexColors={true}
    blending={THREE.AdditiveBlending}
    depthWrite={false}
  /> -->
  <!-- <T.PointsMaterial size={0.5} color={"pink"} sizeAttenuation={true} /> -->
</T.Points>
<!-- </Align> -->
{#if earthMesh}
  <CustomRenderer selectedMesh={earthMesh} />
{/if}
