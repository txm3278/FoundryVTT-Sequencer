<script>

  import { ApplicationShell } from "@typhonjs-fvtt/runtime/svelte/component/core";
  import { localize } from '@typhonjs-fvtt/runtime/svelte/helper';
  import { getContext } from "svelte";

  import DatabaseEntry from "./DatabaseEntry.svelte";
  import VirtualScroll from "svelte-virtual-scroll-list"

  import SequencerDatabase from "../../modules/sequencer-database.js";
  import { databaseStore } from "./DatabaseStore.js";
  import * as lib from "../../lib/lib.js";
  import CONSTANTS from "../../constants.js";

  const { application } = getContext("#external");

  export let elementRoot;

  let entries = [];

  let filteredEntries = [];

  const selectedPackStore = databaseStore.selectedPackStore;
  const packStore = databaseStore.packStore;
  const metadata = databaseStore.metadata;
  const allRanges = databaseStore.allRanges;
  const subLists = databaseStore.subLists;
  const listView = databaseStore.listView;
  const visibleTreeStore = databaseStore.visibleTreeStore;
  const search = databaseStore.search;
  const cleanSearchStore = databaseStore.cleanSearchStore;
  const searchRegex = databaseStore.searchRegex;
  const entriesStore = SequencerDatabase.entriesStore;

  listView.set(game.settings.get(CONSTANTS.MODULE_NAME, "db-list-view"));

  $: game.settings.set(CONSTANTS.MODULE_NAME, "db-list-view", $listView);

  $: {
    $entriesStore;
    const specificRanges = $allRanges
      ? Sequencer.Database.publicFlattenedEntries
      : Sequencer.Database.publicFlattenedSimpleEntries;
    entries = specificRanges.map(
      (entry) => {
        return {
          pack: entry.split(".")[0],
          entry: entry
        };
      }
    );
  }

  $: {
    const searchParts = $cleanSearchStore.split('|');
    filteredEntries = entries.filter((part) => {
      const matchParts = lib.make_array_unique(part.entry.match($searchRegex) || []);
      return (
        ($selectedPackStore === "all" || $selectedPackStore === part.pack) &&
        ($search === "" || matchParts.length >= searchParts.length)
      );
    });
  }

</script>

<svelte:options accessors={true}/>

<ApplicationShell bind:elementRoot>

	<div class="sequencer-database-content">

		<div class="sequencer-database-header">
			<select bind:value={$selectedPackStore} name="pack-select">
				<option value="all">{localize("SEQUENCER.Database.AllPacks")}</option>
				{#each $packStore as pack, index}
					<option>{ pack }</option>
				{/each}
			</select>
			<input bind:value={$search} class="ml-2" placeholder='{localize("SEQUENCER.Database.Search")}' type="text">
			<input bind:checked={$allRanges} class="ml-2" id="database-all-ranges" type="checkbox">
			<label class="all-ranges-label" for="database-all-ranges">{localize("SEQUENCER.Database.ShowAllRanges")}</label>
			<input bind:checked={$subLists} class="ml-2" id="include-sub-lists" type="checkbox">
			<label class="all-ranges-label" for="include-sub-lists">{localize("SEQUENCER.Database.ShowSubLists")}</label>
			<input bind:checked={$listView} class="ml-2" id="treeview" type="checkbox">
			<label class="all-ranges-label" for="treeview">{localize("SEQUENCER.Database.ListView")}</label>
		</div>

		<div class="sequencer-database-entries-container">

			{#if $listView}
        <div class="sequencer-database-entries">
          <VirtualScroll
            data={filteredEntries}
            key="entry"
            let:data
          >
            <DatabaseEntry entry={data.entry}/>
          </VirtualScroll>
        </div>
			{:else}
        <div class="sequencer-database-entries-tree">
          <VirtualScroll
            data={$visibleTreeStore}
            key="fullPath"
            let:data
          >
            <svelte:component this={data.class} {data}/>
          </VirtualScroll>
        </div>
			{/if}

			<div class="sequencer-database-player-container">
				<video autoplay bind:this={databaseStore.elements.player} class="database-player" height="335" loop
							 on:mouseenter={() => { databaseStore.elements.player.controls = true; }}
							 on:mouseleave={() => { databaseStore.elements.player.controls = false; }}
							 preload
							 width="335"
				>
				</video>
				<img bind:this={databaseStore.elements.image} class="database-image hidden">
				<audio bind:this={databaseStore.elements.audio} class="database-audio hidden"
							 on:mouseenter={() => { databaseStore.elements.audio.controls = true; }}
							 on:mouseleave={() => { databaseStore.elements.audio.controls = false; }}
				>
					<source type="audio/ogg">
				</audio>
				<div class="sequencer-database-metadata-container">
					{#if $metadata}
						Type: {$metadata.type} | Duration: {$metadata.duration}
					{:else}
						No file loaded...
					{/if}
				</div>
			</div>

		</div>

	</div>

</ApplicationShell>

<style lang="scss">

  .sequencer-database {
    &-content {
      display: flex;
      flex-direction: column;
    }

    &-header {
      display: flex;
      align-items: center;
      flex-direction: row;

      > select {
				flex: 0 1 auto;
        border-radius: 5px 0 0 5px;
        border-right: 0;
        height: 28px;
      }

      > input[type="text"] {
        border-radius: 0 5px 5px 0;
        padding-left: 0.5rem;
        height: 28px;
      }

			> label {
				flex: 1 0 auto;
        padding-right: 0.25rem;
			}
    }

    &-entries-container {
      padding-top: 0.5rem;
      max-height: calc(420px - 5rem);
      display: flex;
    }

    &-entries {
      height: 339px;
      flex: 1;
      margin-right: 0.5rem;

			& :global(.virtual-scroll-item) {
        padding: 2px;
        margin-right: 5px;
        border-radius: 4px;
      }

      & :global(.virtual-scroll-item):nth-child(odd) {
        background-color: rgba(0, 0, 0, 0.1);
      }
    }

    &-entries-tree {
      height: 339px;
      flex: 1;
      margin-right: 0.5rem;
    }

    &-player-container {
      display: flex;
      flex-direction: column;
      align-items: center;
      width: 335px;
      height: 335px;

      & > * {
        width: 320px;
        height: 320px;
        border-radius: 5px;
      }
    }

    &-metadata-container {
      text-align: center;
      margin-top: 5px;
      flex: 0;
    }
  }

  .hidden {
    display: none;
  }

  .database-player-container {
    padding: 5px;
    width: 355px;
    height: 355px;
  }

  .database-player {
    background-color: #999999;
    padding: 10px;
    max-width: 320px;
    height: 320px;
    max-height: 320px;
		min-height: 320px;
  }

  .database-image {
    max-width: 320px;
    height: 320px;
    max-height: 320px;
    min-height: 320px;
  }

  .database-audio {
    max-width: 320px;
    height: 320px;
    max-height: 320px;
    min-height: 320px;
    position: absolute;
    bottom: 0;
  }

</style>
