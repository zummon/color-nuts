<script>
  import { onMount } from "svelte";
  import { crossfade, fly, fade } from "svelte/transition";
  import { cubicOut } from "svelte/easing";

  const [send, receive] = crossfade({
    duration: 300,
    easing: cubicOut,
    fallback(node, params) {
      return fade(node, { duration: 300 });
    },
  });

  let chooseknots = $state([]);
  let screws = $state([]);
  let chosenpipe = $state();
  let gameConfig = $state({ colors: [], sizes: [], extras: [] });
  let initialScrews = $state([]);

  function restartLevel() {
    chooseknots = [];
    chosenpipe = null;
    screws = JSON.parse(JSON.stringify(initialScrews));
  }

  let isWin = $derived.by(() => {
    if (screws.length === 0) return false;
    if (chooseknots.length > 0) return false;

    let allSorted = true;
    for (let screw of screws) {
      if (screw.knots.length === 0) continue;
      if (screw.knots.length !== screw.size) {
        allSorted = false;
        break;
      }
      const color = screw.knots[0].color;
      for (let i = 1; i < screw.knots.length; i++) {
        if (screw.knots[i].color !== color) {
          allSorted = false;
          break;
        }
      }
      if (!allSorted) break;
    }
    return allSorted;
  });

  function shuffle({ colors, sizes, extras }) {
    let array = [];
    let result = [];
    let idCounter = 0;
    sizes.forEach((size, index) => {
      for (let i = 0; i < size; i++) {
        array.push({ id: idCounter++, color: colors[index] });
      }
    });

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
    } //

    sizes.forEach((size, index) => {
      result[index] = { knots: [], size: size };
      for (let _ of new Array(size)) {
        result[index].knots.push(array.pop());
      }
    });
    extras.forEach((extra) => {
      result.push({ knots: [], size: extra });
    });

    chooseknots = [];
    chosenpipe = null;
    screws = result;
    initialScrews = JSON.parse(JSON.stringify(result));
  }
  onMount(() => {
    let params = new URLSearchParams(location.search);
    let colors = params.get("color")?.split(",") || [];
    let sizes = params.get("size")?.split(",") || [];
    let extras = params.get("extra")?.split(",") || [];
    if (!colors[0]) {
      colors = [
        "#FF69B4",
        "#FFD700",
        "#00FFFF",
        "#008000",
        "#800080",
        "#FFA500",
      ];
    }
    if (sizes[0]) {
      sizes = sizes.map((size) => (isNaN(size) ? 4 : Number(size)));
    } else {
      sizes = [4, 4, 4, 4, 4, 4];
    }
    if (extras[0]) {
      extras = extras.map((extra) => (isNaN(extra) ? 4 : Number(extra)));
    } else {
      extras = [4, 2];
    }
    gameConfig = { colors, sizes, extras };
    shuffle(gameConfig);
  });
</script>

<div class="flex flex-col items-center pt-12 gap-12">
  <div class="flex flex-wrap justify-center gap-6 px-4 max-w-4xl mt-16">
    {#each screws as screw, index (index)}
      <button
        class="flex flex-col items-center justify-end relative cursor-pointer"
        style="height: {screw.size * 2.5 + 2}rem; width: 5rem;"
        onclick={() => {
          if (chooseknots.length > 0) {
            let color = chooseknots[0].color;
            let isValidColor =
              !screw.knots[0] || screw.knots[0].color === color;
            let available = screw.size - screw.knots.length;

            if (isValidColor && available > 0 && index !== chosenpipe) {
              let toPlace = Math.min(chooseknots.length, available);
              let placedKnots = chooseknots.splice(
                chooseknots.length - toPlace,
                toPlace,
              );
              screws[index].knots = [...placedKnots, ...screws[index].knots];
            }

            if (chooseknots.length > 0) {
              screws[chosenpipe].knots = [
                ...chooseknots,
                ...screws[chosenpipe].knots,
              ];
            }

            chooseknots = [];
            chosenpipe = null;
          } else {
            if (screw.knots.length > 0) {
              let color = screw.knots[0].color;
              let count = 0;
              for (let i = 0; i < screw.knots.length; i++) {
                if (screw.knots[i].color === color) {
                  count++;
                } else {
                  break;
                }
              }
              chooseknots = screw.knots.splice(0, count);
              chosenpipe = index;
            }
          }
        }}
      >
        <div
          class="absolute bottom-full mb-2 left-1/2 -translate-x-1/2 z-50 flex flex-col items-center w-full"
        >
          {#each chosenpipe === index ? chooseknots : [] as knot (knot.id)}
            <span
              in:receive={{ key: knot.id }}
              out:send={{ key: knot.id }}
              class="h-8 mt-2 rounded-full relative w-16 z-10 border-2 border-black/10 block shadow-md -translate-y-1"
              style="background-color: {knot.color};"
            >
            </span>
          {/each}
        </div>

        <span
          class="absolute w-4 z-0 rounded-t-full bottom-4 bg-slate-200"
          style="height: {screw.size * 2.5 + 1}rem;"
        ></span>
        <div class="flex flex-col items-center justify-end w-full z-10 mb-1">
          {#each screw.knots as knot (knot.id)}
            <span
              in:receive={{ key: knot.id }}
              out:send={{ key: knot.id }}
              class="h-8 mt-2 rounded-full relative w-16 z-10 border-2 border-black/10 block"
              style="background-color: {knot.color};"
            >
            </span>
          {/each}
        </div>
        <span
          class="h-4 w-20 rounded-md z-0 relative mt-1 bg-slate-300 border-b-4 border-slate-400 block"
        ></span>
      </button>
    {/each}
  </div>

  {#if !isWin}
    <div class="mt-8 flex gap-4">
      <button
        class="px-6 py-3 bg-stone-800/80 hover:bg-stone-700 text-stone-200 font-semibold rounded-2xl shadow-lg shadow-black/20 border border-stone-600/50 transition-all flex items-center gap-2 cursor-pointer backdrop-blur-md"
        onclick={restartLevel}
      >
        <svg
          xmlns="http://www.w3.org/2000/svg"
          width="20"
          height="20"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2"
          stroke-linecap="round"
          stroke-linejoin="round"
          ><path d="M3 12a9 9 0 1 0 9-9 9.75 9.75 0 0 0-6.74 2.74L3 8" /><path
            d="M3 3v5h5"
          /></svg
        >
        Restart
      </button>
      <button
        class="px-6 py-3 bg-linear-to-r from-orange-600 to-amber-600 hover:from-orange-500 hover:to-amber-500 text-white font-bold rounded-2xl shadow-lg shadow-orange-900/40 border border-orange-500/30 transition-all flex items-center gap-2 cursor-pointer"
        onclick={() => shuffle(gameConfig)}
      >
        <svg
          xmlns="http://www.w3.org/2000/svg"
          width="20"
          height="20"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2"
          stroke-linecap="round"
          stroke-linejoin="round"
          ><rect width="18" height="18" x="3" y="3" rx="2" ry="2" /><path
            d="M16 8h.01"
          /><path d="M8 8h.01" /><path d="M8 16h.01" /><path
            d="M16 16h.01"
          /><path d="M12 12h.01" /></svg
        >
        New Game
      </button>
    </div>
  {/if}

  {#if isWin}
    <div
      class="fixed inset-0 z-50 flex items-center justify-center bg-black/60 backdrop-blur-md"
    >
      <div
        class="bg-linear-to-b from-stone-800 to-stone-900 p-8 rounded-3xl shadow-2xl flex flex-col items-center gap-6 max-w-sm w-full mx-4 border border-stone-700/50 relative overflow-hidden"
        in:fly={{ y: 50, duration: 400, easing: cubicOut }}
      >
        <div
          class="absolute -top-24 -left-24 w-48 h-48 bg-orange-500/20 blur-3xl rounded-full pointer-events-none"
        ></div>
        <div
          class="absolute -bottom-24 -right-24 w-48 h-48 bg-amber-500/20 blur-3xl rounded-full pointer-events-none"
        ></div>

        <div
          class="text-7xl drop-shadow-xl z-10 animate-bounce"
          style="animation-duration: 2s;"
        >
          🎉
        </div>
        <h2
          class="text-3xl font-black text-transparent bg-clip-text bg-linear-to-r from-orange-400 to-amber-300 text-center z-10"
        >
          Completed!
        </h2>
        <p class="text-stone-400 text-center font-medium z-10">
          You sorted all the colors perfectly.
        </p>
        <button
          class="w-full mt-2 px-6 py-4 bg-linear-to-r from-orange-600 to-amber-600 hover:from-orange-500 hover:to-amber-500 text-white font-bold rounded-2xl transition-all hover:scale-[1.02] active:scale-95 text-lg shadow-lg shadow-orange-900/30 border border-orange-500/30 z-10 cursor-pointer"
          onclick={() => shuffle(gameConfig)}
        >
          Play Again
        </button>
      </div>
    </div>
  {/if}
</div>
