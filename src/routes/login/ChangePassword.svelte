<script lang="ts">
    import TextField from '@components/widgets/TextField.svelte';
    import { locales } from '@db/Database';
    import { updatePassword, type User } from 'firebase/auth';
    import isValidPassword from './IsValidPassword';
    import { FirebaseError } from 'firebase/app';
    import getLoginErrorDescription from './getAuthErrorDescription';
    import Button from '@components/widgets/Button.svelte';
    import Feedback from '@components/app/Feedback.svelte';

    export let user: User;

    let password1 = '';
    let password2 = '';
    let submitted = false;
    let feedback: string | undefined = undefined;

    async function update() {
        if (isValidPassword(password2) && password1 === password2) {
            try {
                submitted = true;
                await updatePassword(user, password2);
            } catch (error) {
                if (error instanceof FirebaseError) {
                    feedback = getLoginErrorDescription($locales, error);
                }
            } finally {
                submitted = false;
                feedback = $locales.get(
                    (l) => l.ui.page.login.feedback.updatedPassword
                );
                password1 = '';
                password2 = '';
            }
        }
    }
</script>

<p>{$locales.get((l) => l.ui.page.login.prompt.changePassword)}</p>

<form on:submit={update}>
    <TextField
        kind={'password'}
        description={$locales.get(
            (l) => l.ui.page.login.field.password.description
        )}
        placeholder={$locales.get(
            (l) => l.ui.page.login.field.password.placeholder
        )}
        bind:text={password1}
        editable={!submitted}
        validator={(pass) => isValidPassword(pass)}
    />
    <TextField
        kind={'password'}
        description={$locales.get(
            (l) => l.ui.page.login.field.password.description
        )}
        placeholder={$locales.get(
            (l) => l.ui.page.login.field.password.placeholder
        )}
        bind:text={password2}
        editable={!submitted}
        validator={(pass) => pass === password1 && isValidPassword(pass)}
    />
    <Button
        submit
        background
        tip={$locales.get((l) => l.ui.page.login.button.updatePassword)}
        active={!submitted && isValidPassword(password2)}
        action={() => undefined}>&gt;</Button
    >
    {#if password2.length > 0 && !isValidPassword(password2)}
        <Feedback
            >{$locales.get(
                (l) => l.ui.page.login.prompt.passwordrule
            )}</Feedback
        >
    {:else if feedback}
        <Feedback>{feedback}</Feedback>
    {/if}
</form>
