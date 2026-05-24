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
      colors: ["#ff6b6b", "#4dadff", "#51cf66", "#fcc419"],
      sizes: [4, 4, 4, 4],
      extras: [4, 4],
    },
    normal: {
      colors: [
        "#ff6b6b",
        "#4dadff",
        "#51cf66",
        "#fcc419",
        "#b197fc",
        "#ff922b",
      ],
      sizes: [4, 4, 4, 4, 4, 4],
      extras: [4, 4],
    },
    hard: {
      colors: [
        "#ff6b6b",
        "#4dadff",
        "#51cf66",
        "#fcc419",
        "#b197fc",
        "#ff922b",
        "#22b8cf",
        "#f06595",
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

    let currentIndex = array.length;
    while (currentIndex != 0) {
      let randomIndex = Math.floor(Math.random() * currentIndex);
      currentIndex--;
      [array[currentIndex], array[randomIndex]] = [
        array[randomIndex],
        array[currentIndex],
      ];
    }

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

<div class="game-header">
  <h1 class="game-title">Color Nuts</h1>

  <div class="controls-wrapper">
    <div class="select-container">
      <select
        value={currentDifficulty}
        onchange={(e) => setDifficulty(e.target.value)}
        class="cute-select"
      >
        {#each Object.keys(difficulties) as level}
          <option value={level}>
            {level}
          </option>
        {/each}
      </select>
      <div class="select-icon">
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

    <button onclick={() => (showHowToPlay = true)} class="cute-btn">
      <span>Guide</span>
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
        <circle cx="12" cy="12" r="10"></circle>
        <path d="M9.09 9a3 3 0 0 1 5.83 1c0 2-3 3-3 3"></path>
        <line x1="12" y1="17" x2="12.01" y2="17"></line>
      </svg>
    </button>
  </div>
</div>

<div class="game-main">
  <div class="screws-grid">
    {#each screws as screw, index (index)}
      <button
        class="screw-button"
        style="height: {screw.size * 2.3 + 3.2}rem; width: 6rem;"
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
        <div class="chosen-container">
          {#each chosenpipe === index ? chooseknots : [] as knot (knot.id)}
            <span
              in:receive={{ key: knot.id }}
              out:send={{ key: knot.id }}
              class="nut-element nut-floating"
              style="background-color: {knot.color};"
            >
              <div class="nut-highlight"></div>
            </span>
          {/each}
        </div>

        <span class="screw-shaft" style="height: {screw.size * 2.3 + 1.2}rem;">
          <div class="shaft-shine"></div>
        </span>

        <div class="nuts-stack">
          {#each screw.knots as knot (knot.id)}
            <span
              in:receive={{ key: knot.id }}
              out:send={{ key: knot.id }}
              class="nut-element"
              style="background-color: {knot.color};"
            >
              <div class="nut-highlight"></div>
            </span>
          {/each}
        </div>

        <span class="screw-base">
          <div class="base-shine"></div>
        </span>
      </button>
    {/each}
  </div>

  {#if !isWin}
    <div class="actions-footer">
      <button class="cute-btn" onclick={restartLevel}>
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
          ><path d="M3 12a9 9 0 1 0 9-9 9.75 9.75 0 0 0-6.74 2.74L3 8" /><path
            d="M3 3v5h5"
          /></svg
        >
        Restart
      </button>
      <button
        class="cute-btn cute-btn-primary"
        onclick={() => shuffle(gameConfig)}
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
    <div class="popup-overlay" transparent>
      <div
        class="popup-card"
        in:fly={{ y: 40, duration: 400, easing: cubicOut }}
      >
        <div class="celebrate-emoji">🎉</div>
        <h2 class="popup-title">Completed!</h2>
        <p class="popup-desc">You sorted all the cute colors perfectly!</p>
        <button
          class="cute-btn cute-btn-primary"
          style="width: 100%; margin-top: 0.5rem;"
          onclick={() => shuffle(gameConfig)}
        >
          Play Again
        </button>
      </div>
    </div>
  {/if}
</div>

<div class="game-footer">
  <p>
    Made by Ai, <a
      class="footer-link"
      target="_blank"
      href="https://github.com/zummon">Teerapat Anantarattanachai</a
    >
  </p>
  <p>Something breaks, needs upgrade. Let me know.</p>
</div>

<dialog
  bind:this={howToPlayDialog}
  onclose={() => (showHowToPlay = false)}
  onclick={(e) => e.target === howToPlayDialog && (showHowToPlay = false)}
>
  <div
    class="popup-card popup-card-large"
    in:fly={{ y: 40, duration: 400, easing: cubicOut }}
  >
    <div class="dialog-header">
      <h2 class="popup-title">How to Play</h2>
      <button onclick={() => (showHowToPlay = false)} class="close-dialog-btn">
        <svg
          xmlns="http://www.w3.org/2000/svg"
          width="20"
          height="20"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2.5"
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

    <div class="video-link-wrapper">
      <a
        class="cute-btn"
        style="color: #ff7043;"
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

    <div class="guide-steps">
      <div class="step-item">
        <div class="step-number">1</div>
        <p class="step-text">You have bolts filled with colored nuts.</p>
      </div>
      <div class="step-item">
        <div class="step-number">2</div>
        <p class="step-text">
          Your goal is to sort them so each bolt contains only <span
            class="highlight-text">one color</span
          >.
        </p>
      </div>
      <div class="step-item">
        <div class="step-number">3</div>
        <p class="step-text">
          You can move the top nuts to another bolt if there is available space
          and the colors match.
        </p>
      </div>
    </div>

    <button
      class="cute-btn cute-btn-primary"
      style="margin-top: 0.5rem;"
      onclick={() => (showHowToPlay = false)}
    >
      Got it!
    </button>
  </div>
</dialog>
