<script>
  import { onMount } from 'svelte'

  let map, nodes = [], ways = []

  $: bounds = map?.querySelector('osm > bounds')
  $: viewport = {
    top: parseFloat(bounds?.getAttribute('maxlat')),
    bottom: parseFloat(bounds?.getAttribute('minlat')),
    left: parseFloat(bounds?.getAttribute('minlon')),
    right: parseFloat(bounds?.getAttribute('maxlon'))
  }
  $: width = Math.abs(viewport.right - viewport.left)
  $: height = Math.abs(viewport.bottom - viewport.top)

  onMount(async () => {
    map = await parseMap()
    nodes = Object.fromEntries([...map.querySelectorAll('node[visible="true"]')]
      .map(node => [node.id, node]))
    ways = [...map.querySelectorAll('way[visible="true"]')]
  })

  async function parseMap() {
    const xml = await fetch('/map.osm').then(r => r.text())
    const parser = new DOMParser()

    return parser.parseFromString(xml, "text/xml")
  }
</script>

<h1>OpenStreetMap Demo</h1>
<p>Example of rendering OSM format with Svelte &amp; SVG. <a href="https://www.openstreetmap.org/export">Export a .osm file</a> and save it as <code>static/map.osm</code></p>

<pre>{JSON.stringify({ viewport, height, width }, null, 2)}</pre>

{#if map}
  <svg viewBox="{viewport.left} {viewport.top} {width} {height}" preserveAspectRatio="xMidYMid meet">
    {#each ways.filter(p => p.querySelector('tag[k="highway"]')) as way}
      {@const points = [...way.querySelectorAll('nd')]}
      {@const wayNodes = points.map(point => nodes[point.getAttribute('ref')])}
      {@const expression = wayNodes.map(node => `${node.getAttribute('lon')},${height + viewport.bottom + (viewport.top-node.getAttribute('lat'))}`).join(' ')}

      <polyline points={expression} class="road"/>
    {/each}
  </svg>
{/if}

<style>
  svg {
    border: solid 2px #ccc;
    background: #ccc8;
    border-radius: 5px;
  }
  .road {
    stroke-width: 0.00004;
    stroke: #4446;
    fill: none;
    shape-rendering: geometricPrecision;
  }
</style>
