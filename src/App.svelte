<script lang="ts">
  import { forEach, map, reject, sample } from "lodash";
  import Door from "./Door.svelte";
  import { slide } from "svelte/transition";
  import { ProgressRadial } from "@skeletonlabs/skeleton"

  let door = random(3)
  let flow = -1
  let user = -1
  let doors = [
    false,
    false,
    false
  ]  

  let prizes = Promise.all([
    door === 0 ? prize("sports,car") : prize("farm,goat"),
    door === 1 ? prize("sports,car") : prize("farm,goat"),
    door === 2 ? prize("sports,car") : prize("farm,goat"),
  ])

  function prize(what: string) {
    return fetch(`https://loremflickr.com/144/288/${what}`).then(data => data.blob()).then(blob => URL.createObjectURL(blob))
  }

  function random(n: number) {
    return Math.floor(Math.random() * n)
  }

  function onReady() {
    if(flow === -1 && user !== -1) {
      const i = sample(reject(map(doors, (_, i) => i), i => door === i || user === i))
      if(i !== undefined) {
        doors[i] = true
        doors  =  doors
      }      
      flow += 1
    }
  }

  function onSwap () {
    if(flow === 0) {
      const i = reject(map(doors, (_, i) => i), i => doors[i] || user === i)[0]
      user  = i
      flow += 1

      forEach(doors, (_, i) => doors[i] = true)
      doors = doors
    }
  }

  function onKeep () {
    if(flow === 0) {
      flow += 1

      forEach(doors, (_, i) => doors[i] = true)
      doors = doors
    }
  }

  async function onPlayAgain() {
    flow = -1
    user = -1
    doors = [
      false,
      false,
      false
    ]
    
    await new Promise(res => {
      setTimeout(res, 350)
    })

    door = random(3)
    prizes = Promise.all([
      door === 0 ? prize("sports,car") : prize("farm,goat"),
      door === 1 ? prize("sports,car") : prize("farm,goat"),
      door === 2 ? prize("sports,car") : prize("farm,goat"),
    ])
  }

  function onChoose(i: number) {
    if(flow === -1) {
      user = i
    }
  }

</script>

<div class="w-full h-dvh flex flex-col justify-center items-center space-y-4">
  <div class="w-fit h-[720px] space-y-4">
    <!-- Header -->
    <span class="text-3xl flex justify-center">The Monty Hall Problem</span>
    <article class="prose dark:invert">
      Named after the eponymous host of the American game show <em>"Let's Make a Deal"</em>, the Monty Hall Problem offers a glimpse into the often counter-intuitive nature of probability.
    </article>
    {#await prizes}
      <div class="flex flex-col justify-center items-center space-y-4" in:slide={{ delay: 200, duration: 200 }} out:slide={{ delay: 0, duration: 200 }}>
        <span>Shuffling Goats...</span>
        <ProgressRadial width="w-24"/>
      </div>
    {:then  prizes}
      <div class="flex flex-col space-y-4" in:slide={{ delay: 200, duration: 200 }} out:slide={{ delay: 0, duration: 200 }}>
        <!-- Doors -->
        <div class="w-full justify-center flex flex-row space-x-4">
          {#each doors as open, i}
            <Door {open} pick={user === i} on:click={() => onChoose(i)}>
              <!-- svelte-ignore a11y-missing-attribute -->
              <img src={prizes[i]}/>
            </Door>
          {/each}
        </div>

        <!-- Flows -->
        {#if flow === -1}
          <div in:slide={{delay: 200, duration: 200}} out:slide={{delay:0, duration: 200}} class="space-y-2">
            <span class="text-3xl flex justify-center">Pick a door, any door...</span>
            <article class="prose dark:invert">
              The deal in question presents the player with a choice of three doors behind one of which is a brand-new car. Click on one of the doors above to make your choice.
            </article>
            <button on:click={onReady} disabled={user === -1} type="button" class="w-full btn variant-filled">Ready</button>
          </div>
        {/if}

        {#if flow === 0}
          <div in:slide={{delay: 200, duration: 200}} out:slide={{delay:0, duration: 200}} class="space-y-2">
            <span class="text-3xl flex justify-center">Pick the other door?</span>
            <article class="prose dark:invert">
              After the player makes their initial choice the host reveals a gag gift, typically a goat, behind one of the other doors and then the player is offered a final chance to switch to the remaining door...
            </article>
            <div class="flex flex-row space-x-2">
              <button on:click={onKeep} type="button" class="btn variant-filled grow">Keep</button>
              <button on:click={onSwap} type="button" class="btn variant-filled grow">Swap</button>
            </div>
          </div>
        {/if}

        {#if flow === 1 && user === door }
          <div in:slide={{delay: 200, duration: 200}} out:slide={{delay:0, duration: 200}} class="space-y-2">
            <span class="text-3xl flex justify-center">Congratulations!</span>
            <article class="prose dark:invert">
              After the player makes their final choice, the host reveals what is behind the other two doors. In this case, you succeeded in picking the correct door, but did your final choice ultimately matter? Try playing a few more times to get a sense of how your final choice affects the outcome...
            </article>
            <button on:click={onPlayAgain} type="button" class="w-full btn variant-filled">Play Again</button>
          </div>
        {/if}

        {#if flow === 1 && user !== door }
          <div in:slide={{delay: 200, duration: 200}} out:slide={{delay:0, duration: 200}} class="space-y-2">
            <span class="text-3xl flex justify-center">Better luck next time...</span>
            <article class="prose dark:invert">
              After the player makes their final choice, the host reveals what is behind the other two doors. You didn't pick the correct door this time, but did your final choice ultimately matter? Try playing a few more times to get a sense of how your final choice affects the outcome...
            </article>
            <button on:click={onPlayAgain} type="button" class="w-full btn variant-filled">Play Again</button>
          </div>
        {/if}
      </div>
    {/await}
  <div class="flex text-xs justify-center"><em>Images provided by <u><a href="https://loremflickr.com/">loremflickr</a></u></em></div>
  </div>
</div>

