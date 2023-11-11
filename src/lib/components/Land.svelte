<script>
  // @ts-nocheck

  import { T } from "@threlte/core";
  import { MeshLineMaterial } from "@threlte/extras";
  import * as THREE from "three";
  import { vertexOnSphere } from "$lib/utility/utility";
  import land from "$lib/topoland.json";

  export let radius = 10;
  let width = 50;
  let dashOffset = 0;
  // let color = new THREE.Color();
  const orange = new THREE.Color("#fe3d00");
  const purple = new THREE.Color("#9800fe");

  let tempPosition = [];
  const geometry = new THREE.BufferGeometry();
  land.coordinates.forEach((landSection) => {
    //* for each land section
    for (
      let firstPoint = new THREE.Vector3(0, 0, 0),
        secondPoint = vertexOnSphere(landSection[0], radius),
        i = 1;
      i < landSection.length;
      ++i
    ) {
      firstPoint = secondPoint;
      secondPoint = vertexOnSphere(landSection[i], radius);
      tempPosition.push(firstPoint, secondPoint);
    }
  });
  let positions = [];
  tempPosition.forEach((v) => {
    positions.push(v.x, v.y, v.z);
  });

  const positionAttribute = new THREE.Float32BufferAttribute(positions, 3);
  geometry.setAttribute("position", positionAttribute);

  let material = new THREE.LineBasicMaterial({
    color: 0xffffff,
    opacity: 0.2,
  });
  // let mesh = new THREE.LineSegments(geometry, material);
</script>

<!-- <T is={mesh} /> -->
<T is={new THREE.LineSegments()} >
  <T is={geometry} />
  <T is={material} />
</T>
