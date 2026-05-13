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

  const difficulties = {
    easy: {
      colors: ["#ff4d4d", "#4d79ff", "#4dff4d", "#ffff4d"],
      sizes: [4, 4, 4, 4],
      extras: [4, 4],
    },
    normal: {
      colors: [
        "#ff4d4d",
        "#4d79ff",
        "#4dff4d",
        "#ffff4d",
        "#b366ff",
        "#ffa64d",
      ],
      sizes: [4, 4, 4, 4, 4, 4],
      extras: [4, 4],
    },
    hard: {
      colors: [
        "#ff4d4d",
        "#4d79ff",
        "#4dff4d",
        "#ffff4d",
        "#b366ff",
        "#ffa64d",
        "#4dffff",
        "#ff4dff",
      ],
      sizes: [4, 4, 4, 4, 4, 4, 4, 4],
      extras: [4, 2],
    },
  };

  let currentDifficulty = $state("normal");
  let chooseknots = $state([]);
  let screws = $state([]);
  let chosenpipe = $state();
  let gameConfig = $state({ colors: [], sizes: [], extras: [] });
  let initialScrews = $state([]);
  let showHowToPlay = $state(false);
  let howToPlayDialog;

  $effect(() => {
    if (showHowToPlay) {
      howToPlayDialog?.showModal();
    } else {
      howToPlayDialog?.close();
    }
  });

  function restartLevel() {
    chooseknots = [];
    chosenpipe = null;
    screws = JSON.parse(JSON.stringify(initialScrews));
  }

  function setDifficulty(level) {
    currentDifficulty = level;
    gameConfig = difficulties[level];
    shuffle(gameConfig);
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
    let diff = params.get("difficulty");
    if (difficulties[diff]) {
      currentDifficulty = diff;
    }

    let colors = params.get("color")?.split(",") || [];
    let sizes = params.get("size")?.split(",") || [];
    let extras = params.get("extra")?.split(",") || [];

    if (colors[0] || sizes[0] || extras[0]) {
      // Custom config from URL
      if (!colors[0]) {
        colors = difficulties[currentDifficulty].colors;
      }
      if (sizes[0]) {
        sizes = sizes.map((size) => (isNaN(size) ? 4 : Number(size)));
      } else {
        sizes = difficulties[currentDifficulty].sizes;
      }
      if (extras[0]) {
        extras = extras.map((extra) => (isNaN(extra) ? 4 : Number(extra)));
      } else {
        extras = difficulties[currentDifficulty].extras;
      }
      gameConfig = { colors, sizes, extras };
    } else {
      gameConfig = difficulties[currentDifficulty];
    }

    shuffle(gameConfig);
  });
</script>

<!-- Header / Instructions -->
<div
  class="w-full max-w-5xl px-6 pt-8 pb-4 flex flex-col md:flex-row items-center justify-between gap-6 z-30"
>
  <h1
    class="text-5xl font-black text-transparent bg-clip-text bg-linear-to-r from-orange-400 to-amber-300 drop-shadow-sm tracking-tight shrink-0"
  >
    Color Nuts
  </h1>

  <div class="flex items-center gap-3">
    <!-- Difficulty Select -->
    <div class="relative">
      <select
        value={currentDifficulty}
        onchange={(e) => setDifficulty(e.target.value)}
        class="appearance-none bg-stone-900/60 backdrop-blur-md border border-stone-700/50 rounded-xl px-5 py-2 pr-10 font-bold text-stone-300 cursor-pointer focus:outline-none focus:border-amber-500/50 transition-all shadow-lg capitalize text-sm tracking-wide hover:border-stone-600"
      >
        {#each Object.keys(difficulties) as level}
          <option value={level} class="bg-stone-900 text-stone-200">
            {level}
          </option>
        {/each}
      </select>
      <div
        class="absolute right-3 top-1/2 -translate-y-1/2 pointer-events-none text-stone-500"
      >
        <svg
          xmlns="http://www.w3.org/2000/svg"
          width="16"
          height="16"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="3"
          stroke-linecap="round"
          stroke-linejoin="round"
          ><path d="m6 9 12 0" /><path d="m6 9 6 6 6-6" /></svg
        >
      </div>
    </div>

    <!-- Guide Button -->
    <button
      onclick={() => (showHowToPlay = true)}
      class="group tracking-wide bg-stone-900/60 backdrop-blur-md border border-stone-700/50 rounded-xl px-4 py-2 shadow-lg transition-all hover:border-stone-600 flex items-center gap-2 cursor-pointer whitespace-nowrap"
    >
      <span
        class="font-bold text-stone-300 group-hover:text-amber-400 transition-colors text-sm"
      >
        Guide
      </span>
      <svg
        xmlns="http://www.w3.org/2000/svg"
        width="18"
        height="18"
        viewBox="0 0 24 24"
        fill="none"
        stroke="currentColor"
        stroke-width="2.5"
        stroke-linecap="round"
        stroke-linejoin="round"
        class="text-stone-400 group-hover:text-amber-400 transition-transform group-hover:scale-110"
      >
        <circle cx="12" cy="12" r="10"></circle>
        <path d="M9.09 9a3 3 0 0 1 5.83 1c0 2-3 3-3 3"></path>
        <line x1="12" y1="17" x2="12.01" y2="17"></line>
      </svg>
    </button>
  </div>
</div>

<!-- Game Container -->
<div class="grow flex flex-col justify-center items-center w-full z-10">
  <div class="flex flex-col items-center pt-8 gap-8">
    <div class="flex flex-wrap justify-center gap-6 px-4 max-w-4xl">
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
                class="h-8 mt-2 rounded-2xl relative w-16 z-10 border border-white/10 block shadow-2xl -translate-y-2 overflow-hidden"
                style="background-color: {knot.color};"
              >
                <!-- Glossy highlight -->
                <div
                  class="absolute inset-0 bg-linear-to-b from-white/20 to-transparent opacity-50"
                ></div>
              </span>
            {/each}
          </div>

          <!-- Polished Shaft -->
          <span
            class="absolute w-5 z-0 rounded-t-full bottom-4 bg-linear-to-r from-stone-500 via-stone-200 to-stone-500 border-x border-white/20 shadow-2xl"
            style="height: {screw.size * 2.5 + 1}rem;"
          >
            <div
              class="absolute inset-y-0 left-1 w-1.5 bg-white/40 blur-xs"
            ></div>
          </span>

          <div class="flex flex-col items-center justify-end w-full z-10 mb-1">
            {#each screw.knots as knot (knot.id)}
              <span
                in:receive={{ key: knot.id }}
                out:send={{ key: knot.id }}
                class="h-8 mt-2 rounded-2xl relative w-16 z-10 border border-white/10 block shadow-lg overflow-hidden"
                style="background-color: {knot.color};"
              >
                <!-- Glossy highlight -->
                <div
                  class="absolute inset-0 bg-linear-to-b from-white/20 to-transparent opacity-50"
                ></div>
              </span>
            {/each}
          </div>

          <!-- Polished Base -->
          <span
            class="h-6 w-24 rounded-xl z-0 relative mt-1 bg-linear-to-b from-stone-300 to-stone-500 border border-white/20 shadow-2xl block overflow-hidden"
          >
            <div
              class="absolute inset-0 bg-linear-to-tr from-white/10 to-transparent"
            ></div>
            <div class="absolute inset-x-0 top-0 h-px bg-white/40"></div>
          </span>
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
</div>

<!-- Footer -->
<div class="text-center text-stone-300 px-4 py-6 mt-8 w-full z-20">
  <p class="font-medium text-sm">
    Made by Antigravity, <a
      class="text-stone-300 hover:text-amber-400 transition-colors hover:underline underline-offset-4"
      target="_blank"
      href="https://github.com/zummon">Teerapat Anantarattanachai</a
    >
  </p>
  <p class="text-xs mt-2 text-stone-300">
    Something breaks, needs upgrade. Let me know.
  </p>
</div>

<dialog
  bind:this={howToPlayDialog}
  onclose={() => (showHowToPlay = false)}
  onclick={(e) => e.target === howToPlayDialog && (showHowToPlay = false)}
  class="backdrop:bg-black/60 backdrop:backdrop-blur-md bg-transparent border-none outline-none p-0 overflow-visible fixed inset-0 m-auto"
>
  <div
    class="bg-linear-to-b from-stone-800 to-stone-900 p-8 rounded-3xl shadow-2xl flex flex-col gap-6 max-w-md w-full border border-stone-700/50 relative overflow-hidden"
    in:fly={{ y: 50, duration: 400, easing: cubicOut }}
  >
    <div
      class="absolute -top-24 -left-24 w-48 h-48 bg-orange-500/10 blur-3xl rounded-full pointer-events-none"
    ></div>

    <div class="flex items-center justify-between z-10">
      <h2
        class="text-3xl font-black text-transparent bg-clip-text bg-linear-to-r from-orange-400 to-amber-300"
      >
        How to Play
      </h2>
      <button
        onclick={() => (showHowToPlay = false)}
        class="p-2 hover:bg-white/5 rounded-full transition-colors cursor-pointer text-stone-400 hover:text-white"
      >
        <svg
          xmlns="http://www.w3.org/2000/svg"
          width="24"
          height="24"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2"
          stroke-linecap="round"
          stroke-linejoin="round"
          ><line x1="18" y1="6" x2="6" y2="18" /><line
            x1="6"
            y1="6"
            x2="18"
            y2="18"
          /></svg
        >
      </button>
    </div>

    <div class="flex justify-center gap-6 mb-4">
      <a
        class="px-5 py-2.5 rounded-xl bg-stone-800/80 hover:bg-stone-700 text-amber-500 hover:text-amber-400 border border-stone-700 hover:border-stone-600 transition-all shadow-lg flex items-center gap-2 font-semibold text-sm backdrop-blur-md"
        href="https://youtu.be/8puTMO_Wjwg?si=7VmaAQ1FKyHbk57-&t=12"
        target="_blank"
      >
        <svg
          xmlns="http://www.w3.org/2000/svg"
          width="18"
          height="18"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2.5"
          stroke-linecap="round"
          stroke-linejoin="round"
        >
          <polygon points="5 3 19 12 5 21 5 3"></polygon>
        </svg>
        Watch Video
      </a>
    </div>

    <div
      class="text-stone-300 text-base leading-relaxed space-y-4 font-medium z-10"
    >
      <div class="flex gap-4">
        <div
          class="shrink-0 w-8 h-8 rounded-full bg-orange-500/20 flex items-center justify-center text-orange-400 font-bold border border-orange-500/30"
        >
          1
        </div>
        <p>You have bolts filled with colored nuts.</p>
      </div>
      <div class="flex gap-4">
        <div
          class="shrink-0 w-8 h-8 rounded-full bg-orange-500/20 flex items-center justify-center text-orange-400 font-bold border border-orange-500/30"
        >
          2
        </div>
        <p>
          Your goal is to sort them so each bolt contains only <span
            class="text-amber-400">one color</span
          >.
        </p>
      </div>
      <div class="flex gap-4">
        <div
          class="shrink-0 w-8 h-8 rounded-full bg-orange-500/20 flex items-center justify-center text-orange-400 font-bold border border-orange-500/30"
        >
          3
        </div>
        <p>
          You can move the top nuts to another bolt if there is available space
          and the colors match.
        </p>
      </div>
    </div>

    <button
      class="w-full mt-4 px-6 py-4 bg-linear-to-r from-orange-600 to-amber-600 hover:from-orange-500 hover:to-amber-500 text-white font-bold rounded-2xl transition-all hover:scale-[1.02] active:scale-95 shadow-lg shadow-orange-900/30 border border-orange-500/30 z-10 cursor-pointer"
      onclick={() => (showHowToPlay = false)}
    >
      Got it!
    </button>
  </div>
</dialog>
