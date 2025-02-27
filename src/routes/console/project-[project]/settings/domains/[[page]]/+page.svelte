<script lang="ts">
    import { page } from '$app/stores';
    import { Empty, Heading } from '$lib/components';
    import { Pill } from '$lib/elements';
    import { Button } from '$lib/elements/forms';
    import {
        Table,
        TableBody,
        TableCell,
        TableCellHead,
        TableCellText,
        TableHeader,
        TableRow
    } from '$lib/elements/table';
    import { Container } from '$lib/layout';
    import { addNotification } from '$lib/stores/notifications';
    import { sdkForConsole } from '$lib/stores/sdk';
    import { wizard } from '$lib/stores/wizard';
    import { invalidate } from '$app/navigation';
    import { Dependencies } from '$lib/constants';
    import type { Models } from '@aw-labs/appwrite-console';
    import type { PageData } from './$types';
    import Create from '../create.svelte';
    import Delete from '../delete.svelte';
    import { trackEvent } from '$lib/actions/analytics';

    export let data: PageData;

    const projectId = $page.params.project;
    const target = window?.location.hostname ?? '';
    let showDelete = false;
    let selectedDomain: Models.Domain;
    let isVerifying = {};

    const openWizard = () => {
        wizard.start(Create);
    };

    const refreshDomain = async (domain: Models.Domain) => {
        const domainId = domain.$id;
        try {
            isVerifying[domainId] = true;

            if (domain.verification) {
                invalidate(Dependencies.DOMAINS);
                return;
            }
            await sdkForConsole.projects.updateDomainVerification(projectId, domainId);
            invalidate(Dependencies.DOMAINS);
            trackEvent('submit_domain_update_verification');
        } catch (error) {
            addNotification({
                message: error.message,
                type: 'error'
            });
        } finally {
            isVerifying[domainId] = false;
        }
    };
</script>

<Container>
    <div class="u-flex u-gap-12 common-section u-main-space-between">
        <Heading tag="h2" size="5">Custom Domains</Heading>

        <Button on:click={openWizard}>
            <span class="icon-plus" aria-hidden="true" /> <span class="text">Create domain</span>
        </Button>
    </div>
    {#if data.domains.total}
        <Table>
            <TableHeader>
                <TableCellHead>Domain Name</TableCellHead>
                <TableCellHead width={100} />
                <TableCellHead width={60}>Type</TableCellHead>
                <TableCellHead>Name</TableCellHead>
                <TableCellHead>Value</TableCellHead>
                <TableCellHead />
            </TableHeader>
            <TableBody>
                {#each data.domains.domains as domain}
                    <TableRow>
                        <TableCellText title="Domain">
                            {domain.domain}
                        </TableCellText>
                        <TableCellText title="Status">
                            <Pill warning={!domain.verification} success={domain.verification}>
                                {domain.verification ? 'verified' : 'unverified'}
                            </Pill>
                        </TableCellText>
                        <TableCellText title="Type">CNAME</TableCellText>
                        <TableCellText title="Name">
                            {domain.domain}
                        </TableCellText>
                        <TableCellText title="Name">
                            {target}
                        </TableCellText>
                        <TableCell title="Actions">
                            <div class="u-flex u-gap-8 u-cross-center u-main-end">
                                {#if isVerifying[domain.$id]}
                                    <!-- TODO: remove inline styles -->
                                    <div
                                        class="loader"
                                        style="color: hsl(var(--color-neutral-50)); inline-size: 1.25rem; block-size: 1.25rem" />
                                {:else if !domain.certificateId}
                                    <!-- TODO: remove inline styles -->
                                    <button
                                        class="button is-text is-only-icon u-padding-inline-0"
                                        style="--p-button-size: var(--button-size, 2.0rem);"
                                        aria-label="Verify item"
                                        on:click={() => refreshDomain(domain)}>
                                        <span class="icon-refresh" aria-hidden="true" />
                                    </button>
                                {/if}
                                <!-- TODO: remove inline styles -->
                                <button
                                    class="button tooltip is-text is-only-icon u-padding-inline-0"
                                    style="--p-button-size: var(--button-size, 2.0rem);"
                                    aria-label="Delete item"
                                    on:click={async () => {
                                        showDelete = true;
                                        selectedDomain = domain;
                                    }}>
                                    <span class="icon-trash" aria-hidden="true" />
                                    <span class="tooltip-popup is-bottom" role="tooltip">
                                        Delete
                                    </span>
                                </button>
                            </div>
                        </TableCell>
                    </TableRow>
                {/each}
            </TableBody>
        </Table>
    {:else}
        <Empty
            single
            href="https://appwrite.io/docs/custom-domains"
            target="domain"
            on:click={openWizard} />
    {/if}
</Container>

<Delete bind:showDelete bind:selectedDomain />
