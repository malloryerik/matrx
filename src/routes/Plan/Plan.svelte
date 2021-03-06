<script>
  // Import packages
  import {fly} from 'svelte/transition'
  import {arrowCircleLeft, arrowCircleRight, spinner} from 'svelte-awesome/icons'
  import Icon from 'svelte-awesome'
  import {ViewstateStore} from '@matrx/svelte-viewstate-store'

  // Import local code
  import {addDragster} from '../../stores'
  import {dropPan, dragOver} from './plan-helpers'
  import FormulationGrid from './FormulationGrid'
  import DoingKanban from './DoingKanban'

  const slides = [
    {label: 'Todo'},
    {label: 'Doing'},
    {label: 'Done'},
  ]

  const NUMBER_OF_SLIDES = slides.length
  const startOn = new ViewstateStore({
    identifier: 'startOn', 
    type: 'Int', 
    defaultValue: 0
  })
  const slidesToDisplay = new ViewstateStore({
    identifier: 'slidesToDisplay', 
    type: 'Int', 
    defaultValue: 1, 
    updateLocalStorageOnURLChange: true
  })
  let endOn
  $: {
    $startOn = Math.min($startOn, NUMBER_OF_SLIDES - $slidesToDisplay)
    endOn = $startOn + $slidesToDisplay - 1
  }

  let panTimer = null
  let inX = 1000
  let outX = -1000
  const duration = 125

  function panLeft() {
    startOn.update(value => Math.max(0, value - 1))
    panTimer = null
    inX = -1000
    outX = 1000
  }

  function panRight() {
    startOn.update(value => Math.min(NUMBER_OF_SLIDES - 1, value + 1))
    panTimer = null
    inX = 1000
    outX = -1000
  }

  function startPanTimer(event) {
    event.target.classList.add('has-background-grey-lighter')
    if (event.target.id === "pan-right") {
      panTimer = setTimeout(panRight, 1000)
    } else if (event.target.id === "pan-left") {
      panTimer = setTimeout(panLeft, 1000)
    }
  }

  function clearPanTimer(event) {
    event.target.classList.remove('has-background-grey-lighter')
    clearTimeout(panTimer)
    panTimer = null
  }

  function dropLeft(event) {
    clearPanTimer(event)
    dropPan(event, slides[$startOn - 1].label)
  }

  function dropRight(event) {
    clearPanTimer(event)
    dropPan(event, slides[$startOn + 1].label)
  }

</script>

<h1>Plan</h1>

<div class="section">
  <div class="columns has-background-primary">
    {#if $startOn > 0}
      <div id="pan-left" use:addDragster in:fly={{x: inX, duration}} out:fly={{x: outX, duration}} class="column drop-zone is-narrow has-text-centered" on:click={panLeft} on:drop={dropLeft} on:dragster-enter={startPanTimer} on:dragster-leave={clearPanTimer} on:dragover={dragOver}>
        {#if panTimer}
          <Icon data={spinner} pulse scale="1.75" style="fill: white; padding: 5px"/>
        {:else}
          <Icon data={arrowCircleLeft} scale="1.75" style="fill: white; padding: 5px"/>
        {/if}
        <div class="rotate-left has-text-centered has-text-white">{slides[$startOn - 1].label}&nbsp;&nbsp;&nbsp;</div>
      </div>
    {/if}

    {#if $startOn <= 0 &&  endOn >= 0}
      <div in:fly={{x: inX, duration}} out:fly={{x: outX, duration}} class="column has-text-centered has-background-info">
        <FormulationGrid slideLabel={slides[0].label} />
      </div>
    {/if}

    {#if $startOn <= 1 && endOn >= 1}
      <div in:fly={{x: inX, duration}} out:fly={{x: outX, duration}} class="column has-text-centered has-background-primary">
        <DoingKanban />
      </div>
    {/if}

    {#if $startOn <= 2 &&  endOn >= 2}
      <div in:fly={{x: inX, duration}} out:fly={{x: outX, duration}} class="column has-text-centered has-background-info">
        <FormulationGrid slideLabel={slides[2].label} />
      </div>
    {/if}

    {#if endOn < NUMBER_OF_SLIDES - 1}
      <div id="pan-right" use:addDragster in:fly={{x: inX, duration}} out:fly={{x: outX, duration}} on:click={panRight} on:drop={dropRight} on:dragster-enter={startPanTimer} on:dragster-leave={clearPanTimer} on:dragover={dragOver} class="column drop-zone is-narrow has-text-centered">
        {#if panTimer}
          <Icon data={spinner} pulse scale="1.75" style="fill: white; padding: 5px"/>
        {:else}
          <Icon data={arrowCircleRight} scale="1.75" style="fill: white; padding: 5px"/>
        {/if}
        <div class="rotate-right has-text-centered has-text-white">&nbsp;&nbsp;&nbsp;{slides[$startOn + 1].label}</div>
      </div>
    {/if}
  </div>
</div>

<style>
  .rotate-left {
    transform: rotate(-90deg);
  }
  .rotate-right {
    transform: rotate(90deg);
  }
</style>