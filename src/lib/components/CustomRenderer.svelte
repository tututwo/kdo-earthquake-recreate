<script>
  // @ts-nocheck
  import * as THREE from "three";
  import { useThrelte, useRender } from "@threlte/core";
  import {
    EffectComposer,
    GlitchEffect,
    GlitchMode,
    BloomEffect,
    EffectPass,
    RenderPass,
    OutlineEffect,
    BlendFunction,
  } from "postprocessing";

  export let selectedMesh;

  const { scene, renderer, camera, size } = useThrelte();

  const composer = new EffectComposer(renderer);

  const setupEffectComposer = (camera, selectedMesh) => {
    composer.removeAllPasses();
    composer.addPass(new RenderPass(scene, camera));

    const bloomEffect = new BloomEffect({
      threshold: 0,
      radius: 0,
      intensity: 0.51,
      blendFunction: BlendFunction.ADD,
    });
    bloomEffect.luminancePass.enabled = false;

    composer.addPass(new EffectPass(camera, bloomEffect));
    const glitchEffect = new GlitchEffect({
      perturbationMap: null, // You can use a texture for more control over the effect
      // chromaticAberrationOffset: new THREE.Vector2(0.005, 0.005), // slight color shift
      delay: new THREE.Vector2(10.5, 15.5), // random delay interval for glitches
      duration: new THREE.Vector2(0.6, 1.0), // random duration interval for glitches
      strength: new THREE.Vector2(0.3, 1.0), // random strength interval for glitches
      mode: GlitchMode.SPORADIC, // for sporadic glitches
    });
    composer.addPass(new EffectPass(camera, glitchEffect));
  };

 setupEffectComposer($camera, selectedMesh);
  $: composer.setSize($size.width, $size.height);

  useRender((_, delta) => {
    renderer.toneMapping = THREE.ReinhardToneMapping;
    composer.render(delta);
  });
</script>
