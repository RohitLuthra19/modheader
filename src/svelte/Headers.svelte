<script>
  import Dialog, { Title, Content, Actions } from "@smui/dialog";
  import IconButton from "@smui/icon-button";
  import Button, { Label } from "@smui/button";
  import Checkbox from "@smui/checkbox";
  import Menu from "@smui/menu";
  import List, { Item, Separator, Text } from "@smui/list";
  import { mdiPlus, mdiClose, mdiTrashCanOutline, mdiArrowExpand, mdiSort } from "@mdi/js";
  import { createEventDispatcher } from "svelte";
  import lodashUniq from "lodash/uniq";
  import lodashOrderBy from "lodash/orderBy";
  import lodashClone from "lodash/clone";
  import lodashDebounce from "lodash/debounce";
	import { fly } from 'svelte/transition';
  import { selectedProfile, commitChange } from "../js/datasource";
  import { DISABLED_COLOR, PRIMARY_COLOR, MAX_AUTOCOMPLETE_LENGTH } from "../js/constants";
  import AutoComplete from "./Autocomplete.svelte";
  import MdiIcon from "./MdiIcon.svelte";
  import ExpandHeaderDialog from './ExpandHeaderDialog.svelte';

  const dispatch = createEventDispatcher();

  export let headers;
  export let title;
  export let autocompleteListId;
  export let autocompleteNames;
  export let nameLabel = "Name";
  export let valueLabel = "Value";
  let selectedHeaderIndex;
  let selectedHeader;
  let dialog;
  let sortMenu;
  let clazz;
  export { clazz as class };

  let allChecked;
  let allUnchecked;

  function addHeader() {
    dispatch("add");
  }

  function removeHeader(headerIndex) {
    dispatch("remove", headerIndex);
  }

  function expandEditor(headerIndex) {
    selectedHeaderIndex = headerIndex;
    selectedHeader = lodashClone(headers[selectedHeaderIndex]);
    dialog.open();
  }

  function sort(field, order) {
    headers = lodashOrderBy(headers, [field], [order]);
    refreshHeaders();
  }

  function refreshHeaders() {
    dispatch("refresh", headers);
  }

  function toggleAll() {
    if (!allChecked) {
      headers.forEach(h => (h.enabled = true));
    } else {
      headers.forEach(h => (h.enabled = false));
    }
    refreshHeaders();
  }
  
  const refreshHeadersDebounce = lodashDebounce(() => {
    refreshHeaders();
    allChecked = headers.every(h => h.enabled);
    allUnchecked = headers.every(h => !h.enabled);
  }, 500, { leading: true, trailing: true });
  $: headers, refreshHeadersDebounce();
</script>

{#if autocompleteListId && autocompleteNames}
  <datalist id={autocompleteListId}>
    {#each autocompleteNames as item}
      <option value={item}>
    {/each}
  </datalist>
{/if}

<ExpandHeaderDialog
  bind:this={dialog}
  {title}
  {nameLabel}
  {valueLabel}
  {selectedHeader}
  on:save={event => {
    headers[selectedHeaderIndex] = event.detail;
    refreshHeaders();
  }} />

<div class="data-table {clazz}" transition:fly>
  <div class="data-table-row">
    <Checkbox
      class="data-table-cell flex-fixed-icon"
      bind:checked={allChecked}
      indeterminate={!allChecked && !allUnchecked}
      on:click={() => toggleAll()}
      disabled={headers.length === 0} />
    <h3 class="data-table-title data-table-cell flex-grow">{title}</h3>
    <div class="data-table-cell">
      <Button on:click={() => addHeader(headers)} class="small-text-button">
        <MdiIcon size="20" icon={mdiPlus} color={PRIMARY_COLOR} middle />
        Add
      </Button>
    </div>
    <div class="data-table-cell">
      <Button
        class="small-text-button"
        on:click={e => sortMenu.setOpen(true)}
        disabled={headers.length === 0}>
        <MdiIcon
          size="20"
          icon={mdiSort}
          color={headers.length === 0 ? DISABLED_COLOR : PRIMARY_COLOR}
          middle />
        Sort
      </Button>
      
      <Menu bind:this={sortMenu} quickOpen>
        <List class="sort-menu">
          <Item on:SMUI:action={() => sort('name', 'asc')}>
            <Text>{nameLabel} - ascending</Text>
          </Item>
          <Item on:SMUI:action={() => sort('name', 'desc')}>
            <Text>{nameLabel} - descending</Text>
          </Item>
          <Item on:SMUI:action={() => sort('value', 'asc')}>
            <Text>{valueLabel} - ascending</Text>
          </Item>
          <Item on:SMUI:action={() => sort('value', 'desc')}>
            <Text>{valueLabel} - descending</Text>
          </Item>
          {#if !$selectedProfile.hideComment}
            <Item on:SMUI:action={() => sort('comment', 'asc')}>
              <Text>Comment - ascending</Text>
            </Item>
            <Item on:SMUI:action={() => sort('comment', 'desc')}>
              <Text>Comment - descending</Text>
            </Item>
          {/if}
        </List>
      </Menu>
    </div>

    <div class="data-table-cell">
      <Button
        class="small-text-button"
        on:click={() => {
          headers = [];
          refreshHeaders();
        }}
        disabled={headers.length === 0}>
        <MdiIcon
          size="20"
          icon={mdiTrashCanOutline}
          color={headers.length === 0 ? DISABLED_COLOR : PRIMARY_COLOR}
          middle />
        Clear
      </Button>
    </div>
  </div>
  {#each headers as header, headerIndex}
    <div class="data-table-row {header.enabled ? '' : 'data-table-row-unchecked'}">
      <Checkbox
        class="data-table-cell flex-fixed-icon"
        bind:checked={header.enabled}
        on:change={refreshHeaders}
        indeterminate={false} />
      <AutoComplete
        list={autocompleteListId}
        bind:value={header.name}
        placeholder={nameLabel} />
      <AutoComplete
        bind:value={header.value}
        placeholder={valueLabel} />
      {#if !$selectedProfile.hideComment}
        <AutoComplete
          bind:value={header.comment}
          placeholder="Comment" />
      {/if}
      <IconButton
        dense
        aria-label="Expand"
        class="small-icon-button data-table-cell flex-fixed-icon"
        on:click={() => expandEditor(headerIndex)}>
        <MdiIcon size="24" icon={mdiArrowExpand} />
      </IconButton>
      <IconButton
        dense
        aria-label="Delete"
        class="small-icon-button data-table-cell flex-fixed-icon"
        on:click={() => removeHeader(headerIndex)}>
        <MdiIcon size="24" icon={mdiClose} color="red" />
      </IconButton>
    </div>
  {/each}
</div>
