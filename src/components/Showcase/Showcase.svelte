<script>
  import ShowcaseLine from './ShowcaseLine.svelte'
  import ShowcaseLineEssay from './ShowcaseLineEssay.svelte'
  import queryString from 'query-string'
  import { onMount } from 'svelte'

  let urlState = { filter: '', show: [] }
  let defaultURLState = { filter: '', show: ['Essays', 'Talks', 'Podcasts'] }

  const setURLState = (newState) => {
    const finalState = { ...urlState, ...newState } // merge with existing urlstate
    urlState = finalState
    Object.keys(finalState).forEach(function (k) {
      if (
        // don't save some state values if it meets the conditions below
        !finalState[k] || // falsy
        finalState[k] === '' || // string
        (Array.isArray(finalState[k]) && !finalState[k].length) || // array
        finalState[k] === defaultURLState[k] // same as default state, unnecessary
      ) {
        delete finalState[k] // drop query params with new values = falsy
      }
    })
    if (typeof window !== 'undefined')
      history.pushState(
        {},
        '',
        document.location.origin +
          document.location.pathname +
          '?' +
          queryString.stringify(finalState)
      )
  }

  export let data

  let essays = true
  let talks = true
  let podcasts = true
  let tutorials = false
  let notes = false

  let filterStr = ''

  onMount(() => {
    if (location.search.length < 1) return // early terminate if no search
    let givenstate = queryString.parse(location.search)
    if (!Array.isArray(givenstate.show)) givenstate.show = [givenstate.show]
    if (!givenstate.show.includes('Essays')) essays = false
    if (!givenstate.show.includes('Talks')) talks = false
    if (!givenstate.show.includes('Podcasts')) podcasts = false
    if (!givenstate.show.includes('Tutorials')) tutorials = false
    if (!givenstate.show.includes('Notes')) notes = false
    if (givenstate.filter) filterStr = givenstate.filter
    urlState = { ...defaultURLState, ...givenstate }
  })
  function saveURLState() {
    setURLState({
      filter: filterStr,
      show: [
        essays && 'Essays',
        talks && 'Talks',
        podcasts && 'Podcasts',
        tutorials && 'Tutorials',
        notes && 'Notes'
      ].filter(Boolean)
    })
  }
  // $: console.log({urlState, essays, talks, podcasts})

  $: showAll = filterStr.length > 2
  $: filteredData = data
    .map((x) => {
      if (x.date) x.effectiveDate = new Date(x.date)
      if (x.instances) x.effectiveDate = new Date(x.instances[0].date)
      return x
    })
    .sort((a, z) => z.effectiveDate - a.effectiveDate)
    .filter((_, i) => (showAll ? true : i < 30))
    .filter((x) => {
      if (filterStr && notIncludes(filterStr, x)) {
        return false
      } else {
        if (essays && x.type === 'Essays') return true
        if (talks && x.type === 'Talks') return true
        if (podcasts && x.type === 'Podcasts') return true
        if (tutorials && x.type === 'Tutorials') return true
        if (notes && x.type === 'Notes') return true
      }
    })
  // $: console.log({ filteredData })

  function notIncludes(_filterStr, item) {
    let res = true
    _filterStr = _filterStr.toLowerCase().replace('/', '')
    // if (JSON.stringify(item).toLowerCase().includes(_filterStr)) return true
    if (item.title && item.title.toLowerCase().includes(_filterStr)) res = false
    if (item.slug && item.slug.toLowerCase().includes(_filterStr)) res = false
    if (
      item.categories &&
      item.categories.join().toLowerCase().includes(_filterStr)
    )
      res = false
    if (item.description && item.description.toLowerCase().includes(_filterStr))
      res = false
    return res
  }

  let inputEl
  function focusSearch(e) {
    if (e.key === '/' && inputEl) inputEl.select()
  }
</script>

<svelte:window on:keyup={focusSearch} />

<div class="relative mb-8 flex flex-col md:flex-row">
  <div class="flex-1 mb-4 md:mb-0">
    <h1
      class="text-5xl leading-9 tracking-tight font-extrabold text-gray-200
      sm:text-4xl sm:leading-10">
      Idea Showcase
    </h1>
    <p class="mt-3 text-xl leading-7 text-gray-400 sm:mt-4">
      For Free: Great Ideas. Lightly Used.
      <span class="mt-4 italic block">This site is still under construction,
        pardon our appearance...</span>
    </p>
  </div>

  <div class="flex-1 flex flex-col gap-4 md:gap-0 rounded-md mb-8">
    <!-- search -->
    <label for="search_candidate" class="sr-only">Search</label>
    <div class="relative flex-grow focus-within:z-10">
      <!-- <div class="absolute w-full inset-y-0 left-0 px-3 flex items-center"> -->
      <div class="w-full inset-y-0 left-0 px-3 flex items-center">
        <!-- Heroicon name: search -->
        <label for="search_candidate">
          <svg
            class="h-5 w-5 text-gray-400"
            viewBox="0 0 20 20"
            fill="currentColor">
            <path
              fill-rule="evenodd"
              d="M8 4a4 4 0 100 8 4 4 0 000-8zM2 8a6 6 0 1110.89 3.476l4.817 4.817a1 1 0 01-1.414 1.414l-4.816-4.816A6 6 0 012 8z"
              clip-rule="evenodd" />
          </svg>
        </label>
        <!-- <input
          id="search_candidate"
          type="text"
          class="form-input block w-full rounded-md pl-2 transition ease-in-out
          duration-150 sm:hidden focus:bg-yellow-200 bg-gray-800"
          placeholder="Filter (/ to focus)"
          bind:value={filterStr} /> -->
        <input
          id="search_candidate"
          type="text"
          class="form-input w-full rounded-md pl-2 transition ease-in-out text-white
          duration-150 sm:block sm:text-sm sm:leading-5 py-2 ml-4 bg-gray-800
          focus:bg-yellow-200 focus:text-gray-800"
          placeholder="Filter ideas (press / to focus)"
          bind:this={inputEl}
          on:input={saveURLState}
          bind:value={filterStr} />
      </div>
    </div>
    <!-- categories -->
    <span
    class="relative z-0 inline-flex flex-col sm:flex-row shadow-sm rounded-md">
    <div class="inline-flex items-center mr-2 text-gray-400">Show:</div>
    <button
      type="button"
      on:click={() => saveURLState(essays = !essays)}
      class:bg-gray-300={essays}
      class:text-gray-700={essays}
      class="-ml-px sm:ml-0 relative inline-flex items-center px-4 py-2
        sm:rounded-l-md border border-gray-300 text-sm leading-5
        font-medium text-gray-100 focus:z-10
        focus:outline-none focus:border-blue-300 focus:shadow-outline-blue
         transition ease-in-out
        duration-150">
      Essays
    </button>
    <button
      type="button"
      on:click={() => saveURLState(talks = !talks)}
      class:bg-gray-300={talks}
      class:text-gray-700={talks}
      class="-ml-px relative inline-flex items-center px-4 py-2 border
        border-gray-300 text-sm leading-5 font-medium text-gray-100
       focus:z-10 focus:outline-none
        focus:border-blue-300 focus:shadow-outline-blue 
       transition ease-in-out duration-150">
      Talks
    </button>
    <button
      type="button"
      on:click={() => saveURLState(podcasts = !podcasts)}
      class:bg-gray-300={podcasts}
      class:text-gray-700={podcasts}
      class="-ml-px relative inline-flex items-center px-4 py-2 border
        border-gray-300 text-sm leading-5 font-medium text-gray-100
       focus:z-10 focus:outline-none
        focus:border-blue-300 focus:shadow-outline-blue 
       transition ease-in-out duration-150">
      Podcasts
    </button>
    <button
      type="button"
      on:click={() => alert('coming soon')}
      class:bg-gray-300={tutorials}
      class:text-gray-700={tutorials}
      class="-ml-px relative items-center px-4 py-2 border
        hidden md:inline-flex
        border-gray-300 text-sm leading-5 font-medium text-gray-100
       focus:z-10 focus:outline-none
        focus:border-blue-300 focus:shadow-outline-blue 
       transition ease-in-out duration-150">
      <strike>Tutorials</strike>
    </button>
    <button
      type="button"
      on:click={() => alert('coming soon')}
      class:bg-gray-300={notes}
      class:text-gray-700={notes}
      class="-ml-px relative items-center px-4 py-2
        hidden md:inline-flex
        sm:rounded-r-md border border-gray-300 text-sm leading-5
        font-medium text-gray-100 focus:z-10
        focus:outline-none focus:border-blue-300 focus:shadow-outline-blue
         transition ease-in-out
        duration-150">
      <strike>Notes</strike>
    </button>
  </span>
  </div>
</div>
<!-- <div class="pb-5 border-b border-gray-200 space-y-3 sm:flex sm:flex-col sm:items-center sm:justify-between sm:space-x-4 sm:space-y-0"> -->
<div>
  <!-- <div class=" bg-gray-200 shadow overflow-hidden sm:rounded-md"> -->
  {#if filteredData.length || !showAll}
    <!-- <ul class="grid grid-cols-1 gap-6 sm:grid-cols-2 lg:grid-cols-3"> -->
    <ul class="flex flex-col max-w-4xl mx-auto">
      {#each filteredData as item}
        {#if item.type === 'Essays'}
          <ShowcaseLineEssay {item} />
        {:else}
          <ShowcaseLine {item} uno={filteredData.length === 1} />
        {/if}
      {/each}
    </ul>
    {#if !showAll}
      <span
        class="flex justify-center my-8 rounded-md shadow-sm animate-bounce">
        <button
          on:click={() => (showAll = true)}
          type="button"
          class="inline-flex items-center border border-transparent leading-6
            font-medium rounded-md text-white bg-indigo-600 hover:bg-indigo-500
            focus:outline-none focus:border-indigo-700
            focus:shadow-outline-indigo active:bg-indigo-700 transition
            ease-in-out duration-150 text-4xl p-8">
          Show All
        </button>
      </span>
    {/if}
  {:else}
    <div class="p-8 text-red-500">
      Nothing found! The filter was too restrictive. {#if !urlState.show}Please
        pick either Essays, Talks, or Podcasts to show.{/if}
      {#if urlState.filter}
        Please clear the filter bar and you'll see more stuff.
      {/if}
    </div>
  {/if}
  <!-- </div> -->
</div>
<!-- </div> -->
