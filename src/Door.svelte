<script lang="ts">
  export let open = false
  export let pick = false

  const TILT = -15;
  let tiltX = 0;
  let tiltY = 0;
  let self: HTMLDivElement

  function tilt(me: MouseEvent) {
    if(!self) return

    const 
      b = self.getBoundingClientRect(),
      dx = 2 * (me.pageX - b.x - b.width  / 2) / b.width,
      dy = 2 * (me.pageY - b.y - b.height / 2) / b.width;

    tiltX = -dx * TILT
    tiltY =  dy * TILT
  }

  function onMouseOver(me: MouseEvent) {
    tilt(me)
  }

  function onMouseMove(me: MouseEvent) {
    tilt(me)
  }

  function onMouseOut(me: MouseEvent) {
    tiltX = 0
    tiltY = 0
  }
</script>

<!-- svelte-ignore a11y-no-static-element-interactions -->
<!-- svelte-ignore a11y-mouse-events-have-key-events -->
<!-- svelte-ignore a11y-click-events-have-key-events -->
<div 
  class="door" class:open class:pick style="--tilt-x:{tiltX}deg; --tilt-y:{tiltY}deg;"
  on:mouseover={onMouseOver}
  on:mousemove={onMouseMove}
  on:mouseout ={onMouseOut }
  on:click
  
  bind:this={self}
>
  <img class="back" src="./red-door.jpg" alt="A Red Door"/>
  <div class="face bg-blue-500">
    <slot/>
  </div>
</div>


<style>
  .door {
    --tilt-x: 0deg;
    --tilt-y: 0deg;
    width : 1.5in;
    height: 3.0in;
    
    user-select: none;

    transform-style: preserve-3d;
    transform:
      rotateX(0deg)
      rotateY(0deg);
    transition: transform 100ms linear;
  }

  .door * {
    position: absolute;

    width : 1.5in;
    height: 3in;

    border-radius: 3mm;
    overflow: hidden;
  }

  .door.pick * {
    box-shadow: 0px 0px 10px white;
  }

  .door:hover {
    transform-style: preserve-3d;
    transform:
      perspective(1000px)
      scale(1.1)
      rotateX(var(--tilt-y))
      rotateY(var(--tilt-x));
    transition: transform 100ms linear
  }

  .door .face {
    backface-visibility: hidden;
    transform-style: preserve-3d;
    transform: rotateY(-180deg);
    transition: transform 300ms ease-in
  }

  .door .back {
    backface-visibility: hidden;
    transform-style: preserve-3d;
    transform: rotateY(0deg);
    transition: transform 300ms ease-in
  }

  .door.open .face {
    backface-visibility: hidden;
    transform-style: preserve-3d;
    transform: rotateY(0deg);
    transition: transform 300ms ease-in
  }

  .door.open .back {
    backface-visibility: hidden;
    transform-style: preserve-3d;
    transform: rotateY(180deg);
    transition: transform 300ms ease-in
  }
</style>