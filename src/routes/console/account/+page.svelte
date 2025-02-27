<script lang="ts">
    import { Button, Form, FormList, InputText, InputPassword } from '$lib/elements/forms';
    import { CardGrid, Box, Heading, AvatarInitials } from '$lib/components';
    import { Container } from '$lib/layout';
    import { onMount } from 'svelte';
    import { user } from '$lib/stores/user';
    import { sdkForConsole } from '$lib/stores/sdk';
    import { addNotification } from '$lib/stores/notifications';
    import { base } from '$app/paths';
    import Delete from './delete.svelte';
    import { invalidate } from '$app/navigation';
    import { Dependencies } from '$lib/constants';
    import { trackEvent } from '$lib/actions/analytics';

    let name: string = null,
        email: string = null,
        emailPassword: string = null,
        newPassword: string = null,
        oldPassword: string = null;
    let showDelete = false;

    onMount(async () => {
        name ??= $user.name;
        email ??= $user.email;
    });

    async function updateName() {
        try {
            await sdkForConsole.account.updateName(name);
            invalidate(Dependencies.ACCOUNT);
            addNotification({
                message: 'Name has been updated',
                type: 'success'
            });
            trackEvent('submit_account_update_name');
        } catch (error) {
            addNotification({
                message: error.message,
                type: 'error'
            });
        }
    }
    async function updateEmail() {
        try {
            await sdkForConsole.account.updateEmail(email, emailPassword);
            invalidate(Dependencies.ACCOUNT);
            addNotification({
                message: 'Email has been updated',
                type: 'success'
            });
            trackEvent('submit_account_update_email');
        } catch (error) {
            addNotification({
                message: error.message,
                type: 'error'
            });
        }
    }

    async function updatePassword() {
        try {
            await sdkForConsole.account.updatePassword(newPassword, oldPassword);
            newPassword = oldPassword = null;
            addNotification({
                message: 'Password has been updated',
                type: 'success'
            });
            trackEvent('submit_account_update_password');
        } catch (error) {
            addNotification({
                message: error.message,
                type: 'error'
            });
        }
    }
</script>

<Container>
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
                        autofocus={true}
                        bind:value={name} />
                </ul>
            </svelte:fragment>

            <svelte:fragment slot="actions">
                <Button disabled={name === $user.name || !name} submit>Update</Button>
            </svelte:fragment>
        </CardGrid>
    </Form>
    <Form on:submit={updateEmail}>
        <CardGrid>
            <Heading tag="h6" size="7">Update Email</Heading>

            <svelte:fragment slot="aside">
                <FormList>
                    <InputText
                        id="email"
                        label="Email"
                        placeholder="Enter email"
                        autocomplete={false}
                        bind:value={email} />
                    {#if email !== $user.email && email}
                        <InputPassword
                            id="emailPassword"
                            label="Password"
                            placeholder="Enter password"
                            autocomplete={false}
                            showPasswordButton={true}
                            bind:value={emailPassword} />
                    {/if}
                </FormList>
            </svelte:fragment>

            <svelte:fragment slot="actions">
                <Button disabled={email === $user.email || !email || !emailPassword} submit
                    >Update</Button>
            </svelte:fragment>
        </CardGrid>
    </Form>
    <Form on:submit={updatePassword}>
        <CardGrid>
            <Heading tag="h6" size="7">Update Password</Heading>
            <p class="text">
                Forgot your password? <a class="link" href={`${base}/recover`}
                    >Recover your password</a>
            </p>

            <svelte:fragment slot="aside">
                <FormList>
                    <InputPassword
                        id="oldPassword"
                        label="Old password"
                        placeholder="Enter password"
                        autocomplete={false}
                        showPasswordButton={true}
                        bind:value={oldPassword} />
                    <InputPassword
                        id="newPassword"
                        label="New password"
                        placeholder="Enter password"
                        autocomplete={false}
                        showPasswordButton={true}
                        bind:value={newPassword} />
                </FormList>
            </svelte:fragment>

            <svelte:fragment slot="actions">
                <Button disabled={!newPassword || !oldPassword} submit>Update</Button>
            </svelte:fragment>
        </CardGrid>
    </Form>
    <CardGrid danger>
        <div>
            <Heading tag="h6" size="7">Delete Account</Heading>
        </div>
        <p>
            Your account will be permanently deleted and access will be lost to any of your teams
            and data. This action is irreversible.
        </p>
        <svelte:fragment slot="aside">
            <Box>
                <svelte:fragment slot="image">
                    <AvatarInitials size={48} name={$user.name} />
                </svelte:fragment>
                <svelte:fragment slot="title">
                    <h6 class="u-bold u-trim-1">{$user.name}</h6>
                </svelte:fragment>
            </Box>
        </svelte:fragment>

        <svelte:fragment slot="actions">
            <Button secondary on:click={() => (showDelete = true)}>Delete</Button>
        </svelte:fragment>
    </CardGrid>
</Container>

<Delete bind:showDelete />
