<script>
    import { _ } from 'svelte-i18n';
    import { getContext } from "svelte";
    import { API } from "../../stores/api";
    import { shuffleArray, setIndexes, getPlaylistIDFromUrl } from "../../logic/helper";

    export let contextKey;

    const { dataDisplay, isEditMode } = getContext(contextKey);

    let playlistID = getPlaylistIDFromUrl();

    function handleStart() {
        $isEditMode = true;
    }

    function handleRandom() {
        $dataDisplay = shuffleArray($dataDisplay);
    }

    async function handleSave() {
        // filter out any 'removed' items
        let actual = $dataDisplay.filter(el => el.isDeleted !== true);

        let ids = actual.map((obj) => obj.id);
        let newOrders = Array.from(actual.keys(), n => n + 1);

        let result = await $API.playlistEdit({ filter: playlistID, items: ids.join(","), tracks: newOrders.join(",")} );

        if (result.success) {
            $dataDisplay = setIndexes(actual);

            $isEditMode = false;
        }
    }

    function handleCancel() {
        $isEditMode = false;
    }
</script>

{#if $isEditMode}
    <button class="button button--primary" type="button" title="{$_('text.saveOrder')}" on:click={handleSave}>{$_('text.saveOrder')}</button>
    <button class="button button--regular" type="button" title="{$_('text.random')}" on:click={handleRandom}>{$_('text.random')}</button>
    <button class="button button--tertiary" type="button" title="{$_('text.cancel')}" on:click={handleCancel}>{$_('text.cancel')}</button>
{:else}
    <button class="button button--regular" type="button" title="{$_('text.reorder')}" on:click={handleStart}>{$_('text.reorder')}</button>
{/if}


