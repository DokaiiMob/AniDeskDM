<script>
    import { localStorageWritable } from "@babichjacob/svelte-localstorage";
    import MetaInfo from "../components/gui/MetaInfo.svelte";
    import { onMount } from "svelte";

    let username = "";
    let password = "";
    let errorMessage = "";
    let loading = false;

    const user = localStorageWritable("user_token", null);

    async function login() {
        errorMessage = "";

        if (!username.trim() || !password.trim()) {
            errorMessage = "Введите логин и пароль";
            return;
        }

        loading = true;

        try {
            const res = await anixApi.auth.signIn({
                login: username,
                password,
            });

            switch (res.code) {
                case 0:
                    user.set(
                        JSON.stringify({
                            id: res.profile.id,
                            token: res.profileToken.token,
                        }),
                    );
                    analytics.trackEvent("new_login");
                    location.reload();
                    break;
                case 2:
                    errorMessage = "Неверный логин или пароль";
                    break;
                default:
                    errorMessage = "Произошла ошибка авторизации";
            }
        } catch (err) {
            console.error("[Auth Error]", err);
            errorMessage = "Сервер не отвечает. Повторите позже.";
        } finally {
            loading = false;
        }
    }
</script>

<MetaInfo subTitle="AniDesk — Вход" />

<div class="login-wrapper">
    <div class="login-box">
        <h2 class="login-title">Вход в AniDesk</h2>

        {#if errorMessage}
            <div class="error-message">{errorMessage}</div>
        {/if}

        <input
            type="text"
            placeholder="Логин"
            bind:value={username}
            class="login-input"
        />

        <input
            type="password"
            placeholder="Пароль"
            bind:value={password}
            class="login-input"
        />

        <button
            class="login-button {loading ? 'loading' : ''}"
            disabled={loading || !username || !password}
            on:click={login}
        >
            {#if loading}
                <span class="dots"
                    ><span>.</span><span>.</span><span>.</span></span
                >
            {:else}
                Войти
            {/if}
        </button>
    </div>
</div>

<style>
    .login-wrapper {
        height: 100vh;
        display: flex;
        align-items: center;
        justify-content: center;
        background: var(--background-color, #0e0e0e);
    }

    .login-box {
        background: var(--alt-background-color, #1c1c1c);
        padding: 40px;
        border-radius: 16px;
        box-shadow: 0 8px 24px rgba(0, 0, 0, 0.4);
        width: 360px;
        display: flex;
        flex-direction: column;
        align-items: center;
    }

    .login-logo {
        width: 80px;
        margin-bottom: 16px;
        filter: drop-shadow(0 0 4px rgba(255, 255, 255, 0.1));
    }

    .login-title {
        color: var(--main-text-color, #ffffff);
        font-size: 22px;
        font-weight: 600;
        margin-bottom: 20px;
    }

    .login-input {
        width: 100%;
        padding: 12px;
        margin-bottom: 14px;
        background: #2a2a2a;
        color: #fff;
        border: 1px solid #444;
        border-radius: 8px;
        font-size: 14px;
        transition: border 0.2s;
    }

    .login-input:focus {
        border-color: #4a90e2;
        outline: none;
    }

    .login-button {
        width: 100%;
        padding: 12px;
        background-color: #4a90e2;
        color: #fff;
        border: none;
        border-radius: 8px;
        font-weight: bold;
        cursor: pointer;
        transition: background-color 0.2s;
    }

    .login-button:hover:not(:disabled) {
        background-color: #3b7cc4;
    }

    .login-button:disabled {
        opacity: 0.6;
        cursor: not-allowed;
    }

    .login-button .dots {
        display: flex;
        justify-content: center;
        gap: 3px;
        font-size: 18px;
    }

    .login-button .dots span {
        animation: blink 1s infinite;
    }

    .login-button .dots span:nth-child(2) {
        animation-delay: 0.2s;
    }

    .login-button .dots span:nth-child(3) {
        animation-delay: 0.4s;
    }

    @keyframes blink {
        0%,
        80%,
        100% {
            opacity: 0;
        }
        40% {
            opacity: 1;
        }
    }

    .error-message {
        color: #ff4d4f;
        margin-bottom: 14px;
        text-align: center;
        font-size: 13px;
    }
</style>
