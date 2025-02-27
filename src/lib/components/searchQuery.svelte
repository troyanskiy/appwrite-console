<script lang="ts">
    import { goto } from '$app/navigation';
    import { page } from '$app/stores';
    import { onMount } from 'svelte';
    import { onDestroy } from 'svelte';

    export let search = '';
    export let placeholder = '';
    export let debounce = 250;
    export let required = false;
    export let disabled = false;
    export let autofocus = false;
    export let isWithEndButton = true;

    let element: HTMLInputElement;
    let timer: ReturnType<typeof setTimeout>;

    onMount(() => {
        if (element && autofocus) {
            element.focus();
        }
    });

    onDestroy(() => {
        search = '';
        if (timer) {
            clearTimeout(timer);
        }
    });

    $: valueChange(search);

    const valueChange = (value: string) => {
        clearTimeout(timer);
        timer = setTimeout(async () => {
            const url = new URL($page.url);
            if (url.search === value) {
                return;
            }
            /**
             * Reset to first page if search changes.
             */
            if ($page.data.page > 1) {
                url.pathname = url.pathname.replace(new RegExp(`/${$page.data.page}*$`), '/');
            }

            url.search = value;

            goto(url, { keepfocus: true });
        }, debounce);
    };
</script>

<div class="u-flex u-gap-12 common-section u-main-space-between">
    <div class="u-flex-basis-50-percent">
        <div class="input-text-wrapper" class:is-with-end-button={isWithEndButton}>
            <input
                {placeholder}
                {disabled}
                {required}
                type="search"
                class="input-text"
                bind:value={search} />
            <span class="icon-search" aria-hidden="true" />
            {#if isWithEndButton && search}
                <button class="x-button" aria-label="Clear search" on:click={() => (search = '')}>
                    <span class="icon-x" aria-hidden="true" />
                </button>
            {/if}
        </div>
    </div>
    <slot />
</div>
