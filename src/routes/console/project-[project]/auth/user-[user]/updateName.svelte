<script lang="ts">
    import { invalidate } from '$app/navigation';
    import { trackEvent } from '$lib/actions/analytics';
    import { CardGrid, Heading } from '$lib/components';
    import { Dependencies } from '$lib/constants';
    import { Button, Form, InputText } from '$lib/elements/forms';
    import { addNotification } from '$lib/stores/notifications';
    import { sdkForProject } from '$lib/stores/sdk';
    import { onMount } from 'svelte';
    import { user } from './store';

    let userName: string = null;
    onMount(async () => {
        userName ??= $user.name;
    });

    async function updateName() {
        try {
            await sdkForProject.users.updateName($user.$id, userName);
            invalidate(Dependencies.USER);
            addNotification({
                message: 'Name has been updated',
                type: 'success'
            });
            trackEvent('submit_user_update_name');
        } catch (error) {
            addNotification({
                message: error.message,
                type: 'error'
            });
        }
    }
</script>

<Form on:submit={updateName}>
    <CardGrid>
        <Heading tag="h6" size="7">Update Name</Heading>

        <svelte:fragment slot="aside">
            <ul>
                <InputText
                    id="name"
                    label="Name"
                    placeholder="Enter name"
                    autocomplete={false}
                    bind:value={userName} />
            </ul>
        </svelte:fragment>

        <svelte:fragment slot="actions">
            <Button disabled={userName === $user.name} submit>Update</Button>
        </svelte:fragment>
    </CardGrid>
</Form>
