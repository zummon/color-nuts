<script>
  import { onMount } from "svelte";
  let chooseknots = $state([]);
  let screws = $state([
    ["#ef4444", "#ef4444", "#ef4444", "#ef4444"],
    ["#f59e0b", "#f59e0b", "#f59e0b", "#f59e0b"],
    ["#22c55e", "#22c55e", "#22c55e", "#22c55e"],
    ["#14b8a6", "#14b8a6", "#14b8a6", "#14b8a6"],
    ["#3b82f6", "#3b82f6", "#3b82f6", "#3b82f6"],
    ["#8b5cf6", "#8b5cf6", "#8b5cf6", "#8b5cf6"],
    ["", "", "", ""],
    ["", "", "", ""],
  ]);
  function shuffle() {
    let array = screws.slice(0, -2).flat();

    // https://stackoverflow.com/questions/2450954/how-to-randomize-shuffle-a-javascript-array
    let currentIndex = array.length;

    // While there remain elements to shuffle...
    while (currentIndex != 0) {
      // Pick a remaining element...
      let randomIndex = Math.floor(Math.random() * currentIndex);
      currentIndex--;

      // And swap it with the current element.
      [array[currentIndex], array[randomIndex]] = [
        array[randomIndex],
        array[currentIndex],
      ];
    }
    //

    screws.slice(0, -2).forEach((knots, index) => {
      knots.forEach((knot, idx) => {
        screws[index][idx] = array.pop();
      });
    });
  }
  onMount(() => {
    shuffle();
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
  {#each screws as knots, index (index)}
    <button
      class="flex flex-col gap-2"
      onclick={() => {
        let ready = [];
        let position;
        if (chooseknots[0]) {
          let count = 0;
          for (let knot of knots.slice()) {
            if (knot) {
              break;
            } else {
              count += 1;
            }
          }
          if (chooseknots.length <= count) {
            screws[index].splice(
              screws[index].length - chooseknots.length,
              chooseknots.length,
              ...chooseknots.slice()
            );
            chooseknots = [];
          }
        } else {
          let idx = 0;
          for (let knot of knots.slice()) {
            if (knot) {
              if (position === undefined) {
                position = idx;
              }
              if (!ready[0] || ready[0] == knot) {
                ready.push(knot);
              } else {
                break;
              }
            } else {
              break;
            }
            idx += 1;
          }
          chooseknots = ready;
          screws[index].splice(
            position,
            ready.length,
            ...ready.slice().fill("")
          );
        }
      }}
    >
      {#each knots as knot, idx (`${index}-${idx}`)}
        <span
          class="h-8 w-16 rounded-full"
          style="background-color: {knot || '#9ca3af'};"
        ></span>
      {/each}
    </button>
  {/each}
</div>
