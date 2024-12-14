<script>
  import { onMount } from "svelte";

  let chooseknots = $state([]);
  let screws = $state([]);

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
    let colors = params.getAll('color')
    let sizes = params.getAll('size')
    let extras = params.getAll('extra')
    if (!colors[0]){
      colors = ['#ef4444','#f59e0b','#84cc16','#14b8a6','#3b82f6','#8b5cf6']
    } 
    if (!sizes[0]) {
      sizes = [4,4,4,4,4,4]
    }
    if (!extras[0]) {
      extras = [4,4]
    }
    shuffle({ colors, sizes, extras });
  });
</script>

<div class="">
  <div class="flex flex-row gap-2">
    {#each chooseknots as knot}
      <span
        class="h-16 w-8 rounded-full"
        style="background-color: {knot || '#9ca3af'};"
      ></span>
    {:else}
      <span class="h-16 w-8 rounded-full"></span>
    {/each}
  </div>
</div>

<div class="flex flex-wrap gap-4 px-4">
  {#each screws as { knots, size }, index (index)}
    <button
      class="flex flex-col gap-2"
      onclick={() => {
        if (chooseknots[0]) {
          let part = chooseknots.pop()
          screws[index].knots.push(part)
        } else {
          let part = screws[index].knots.pop()
          chooseknots.push(part)
        }
      }}
    >
      {#each new Array(size) as _, idx (`${index}-${idx}`)}
        <span
          class="h-8 w-16 rounded-full"
          style="background-color: {knots[idx] || '#9ca3af'};"
        ></span>
      {/each}
    </button>
  {/each}
</div>
