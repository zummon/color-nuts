<script>
  import { onMount } from "svelte";

  let chooseknots = $state([]);
  let screws = $state([]);
	let chosenpipe = $state()

  function shuffle({ colors, sizes, extras }) {
    let array = []
    let result = []
    sizes.forEach((size, index) => {
      array.push(... new Array(size).fill(colors[index]))
    })

    // https://stackoverflow.com/questions/2450954/how-to-randomize-shuffle-a-javascript-array
    let currentIndex = array.length;
    // While there remain elements to shuffle...
    while (currentIndex != 0) {
      // Pick a remaining element...
      let randomIndex = Math.floor(Math.random() * currentIndex);
      currentIndex--;
      // And swap it with the current element.
      [array[currentIndex], array[randomIndex]] = [array[randomIndex], array[currentIndex]];
    } //

    sizes.forEach((size, index) => {
      result[index] = { knots: [], size: size };
      for (let _ of new Array(size)) {
        result[index].knots.push(array.pop())
      }
    });
    extras.forEach((extra) => {
      result.push({ knots: [], size: extra })
    })
    
    screws = result 
  }
  onMount(() => {
    let params = new URLSearchParams(location.search)
    let colors = params.get('color')?.split(',') || [];
    let sizes = params.get('size')?.split(',') || [];
    let extras = params.get('extra')?.split(',') || [];
    if (!colors[0]){
      colors = ["#FF69B4","#FFD700","#00FFFF","#008000","#800080","#FFA500"]
    } 
    if (sizes[0]) {
      sizes = sizes.map((size) => isNaN(size) ? 4 : Number(size))
    } else {
      sizes = [4,4,4,4,4,4]
    }
    if (extras[0]) {
      extras = extras.map((extra) => isNaN(extra) ? 4 : Number(extra))
    } else {
      extras = [4,2]
    }
    shuffle({ colors, sizes, extras });
  });
</script>

<div class="">
  <div class="flex flex-row gap-2">
    {#each chooseknots as knot}
      <span
        class="h-16 w-8 rounded-full"
        style="background-color: {knot || 'transparent'};"
      ></span>
    {:else}
      <span class="h-16 w-8 rounded-full"></span>
    {/each}
  </div>
</div>

<div class="flex flex-wrap justify-center gap-4 px-4">
  {#each screws as screw, index (index)}
		<button
			class="flex flex-col items-center justify-end relative cursor-pointer"
			onclick={() => {
				if (chooseknots[0]) {
					let part = chooseknots[0]
					if ((!screw.knots[0] || part == screw.knots[0]) && screw.knots.length < screw.size) {
						screws[index].knots.unshift(part)
						chooseknots.pop()
					}
				} else {
					let part = screw.knots[0]
					if (part) {
						chooseknots.push(part)
						screws[index].knots.shift()
					}
				}
			}}
		>
			<span class="absolute w-4 bg-zinc-400 z-0 rounded-t-md" style="height: {(screw.size * 2.5) + 1}rem"></span>
			{#each new Array(screw.size) as _, idx (`${index}-${idx}`)}
				<span
					class="{screw.knots[idx] ? 'h-8 mt-2 rounded-full shadow' : 'h-0'} w-16 z-10"
					style="background-color: {screw.knots[idx] || 'transparent'};"
				></span>
			{/each}
			<span class="h-4 w-16 bg-zinc-400 rounded-md z-0"></span>
		</button>
  {/each}
</div>
