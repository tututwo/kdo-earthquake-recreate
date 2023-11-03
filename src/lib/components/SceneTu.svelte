<script>
  import { T } from "@threlte/core";
  import { Align, OrbitControls } from "@threlte/extras";

  import { getContext } from "svelte";

  import { vertexOnSphere } from "$lib/utility/utility";
  import * as THREE from "three";

  const { data, scales } = getContext("Chart");

  $: ({ depthScale, sizeScale, colorScale } = $scales);
  $: count = $data.length;
  let size = 10;
  const positions = new Float32Array(count * 3);

  $: $data.forEach((earthquake, i) => {
    const earthquakeCoordinates = vertexOnSphere(
      [earthquake.longitude, earthquake.latitude],
      depthScale(earthquake.depth)
    );

    positions[i * 3] = earthquakeCoordinates.x;
    positions[i * 3 + 1] = earthquakeCoordinates.y;
    positions[i * 3 + 2] = earthquakeCoordinates.z;
  });

  const geometry = new THREE.BufferGeometry();
  geometry.setAttribute("position", new THREE.BufferAttribute(positions, 3));
</script>

<T.PerspectiveCamera makeDefault position={[-4, 3, 6]} fov={45}>
  <OrbitControls autoRotate />
</T.PerspectiveCamera>

<T.DirectionalLight position.y={10} position.z={10} />

<Align>
  <T.Points>
    <T is={geometry} />
    <T.PointsMaterial size={5} color={"pink"} sizeAttenuation={true} />
  </T.Points>
</Align>
