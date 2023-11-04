<script>
  import { Canvas } from "@threlte/core";

  import { setContext } from "svelte";
  import { writable } from "svelte/store";

  import * as d3 from "d3";
  import Scene from "./Scene.svelte";

  // @ts-ignore
  import earthquake from "$lib/earthquakes.csv";
  let colorSchemes = [
    "BuGn",
    "BuPu",
    "GnBu",
    "OrRd",
    "PuBuGn",
    "PuBu",
    "PuRd",
    "RdPu",
    "YlGnBu",
    "YlGn",
    "YlOrBr",
    "YlOrRd",
    "Cividis",
    "Viridis",
    "Inferno",
    "Magma",
    "Plasma",
    "Warm",
    "Cool",
    "CubehelixDefault",
    "Turbo",
  ];
  let radius = 10;
  // TODO Save scales and data in stores
  const scales = writable({});
  const data = writable({});
  $: data.set(earthquake);
  $: scales.set({
    depthScale: d3
      .scaleLinear()
      .domain([0,d3.max($data, (d) => d.depth)])
      .range([radius, radius * .35]),
    sizeScale: d3
      .scaleSqrt()
      .domain(d3.extent($data, (d) => d.magnitude))
      .range([1, radius]),
    colorScale: d3
      .scaleSequential(d3[`interpolate${colorSchemes[17]}`])
      .domain(d3.extent($data, (d) => d.magnitude)),
    timeScale: d3
      .scaleLinear()
      .domain(d3.extent($data, (d) => d.date))
      .range([0, 1]),
  });
  $: chartContext = { scales, data };
  $: setContext("Chart", chartContext);
</script>

<Canvas>
  <Scene />
</Canvas>
