<script>
    import { onMount } from "svelte";

    let username = $state("sveltejs");
    /** @type {Array<{name: string, description: string | null, language: string | null, stargazers_count: number, html_url: string}>} */
    let repositories = $state([]);
    let isLoading = $state(true);
    let errorMessage = $state("");
    let darkMode = $state(false);

    async function loadRepositories() {
        const requestedUsername = username.trim();
        if (!requestedUsername) {
            errorMessage = "Enter a GitHub username to search.";
            return;
        }

        isLoading = true;
        errorMessage = "";
        try {
            const response = await fetch(
                `https://api.github.com/users/${encodeURIComponent(requestedUsername)}/repos?sort=updated&direction=desc&per_page=5`,
            );
            if (!response.ok) {
                throw new Error(
                    response.status === 404
                        ? `We couldn't find a GitHub user named "${requestedUsername}".`
                        : "GitHub is temporarily unavailable. Please try again.",
                );
            }
            repositories = await response.json();
        } catch (error) {
            repositories = [];
            errorMessage =
                error instanceof Error
                    ? error.message
                    : "An unexpected error occurred.";
        } finally {
            isLoading = false;
        }
    }

    onMount(() => {
        darkMode = localStorage.getItem("repository-index-theme") === "dark";
        loadRepositories();
    });

    function toggleTheme() {
        darkMode = !darkMode;
        localStorage.setItem(
            "repository-index-theme",
            darkMode ? "dark" : "light",
        );
    }
</script>

<svelte:head>
    <title>Repository Index | GitHub</title>
    <meta
        name="description"
        content="Explore the five most recently updated repositories from any GitHub user."
    />
</svelte:head>

<main class:dark-mode={darkMode}>
    <nav class="topbar" aria-label="Primary navigation">
        <a class="brand" href="/" aria-label="Repository Index home"
            ><span class="brand-mark">RI</span><span>Repository Index</span></a
        >
        <div class="nav-actions">
            <span class="api-status"><span></span> GitHub API</span>
            <button
                class="theme-toggle"
                type="button"
                onclick={toggleTheme}
                aria-label={darkMode
                    ? "Switch to light mode"
                    : "Switch to dark mode"}
                aria-pressed={darkMode}
            >
                <span aria-hidden="true">{darkMode ? "☼" : "☾"}</span>
                {darkMode ? "Light" : "Dark"}
            </button>
        </div>
    </nav>

    <section class="intro">
        <p class="eyebrow">A small window into open source</p>
        <h1>Five places where<br /><em>ideas are built.</em></h1>
        <p class="lede">
            Browse the latest work from any GitHub profile, one repository at a
            time.
        </p>
        <form
            class="search"
            onsubmit={(event) => {
                event.preventDefault();
                loadRepositories();
            }}
        >
            <label for="username">GitHub username</label>
            <div class="search-row">
                <div class="input-wrap">
                    <span aria-hidden="true">@</span><input
                        id="username"
                        bind:value={username}
                        placeholder="username"
                        autocomplete="off"
                    />
                </div>
                <button type="submit" disabled={isLoading}
                    >{isLoading ? "Loading" : "Explore"}
                    <span aria-hidden="true">↗</span></button
                >
            </div>
        </form>
    </section>

    <section class="results" aria-live="polite">
        <div class="section-heading">
            <div>
                <p class="eyebrow">Recently updated</p>
                <h2>
                    {repositories.length
                        ? `Work by ${username}`
                        : "Repository shelf"}
                </h2>
            </div>
            <span class="count">{repositories.length}/05</span>
        </div>
        {#if isLoading}
            <div class="loading-state">
                <span class="spinner"></span> Pulling repositories from GitHub...
            </div>
        {:else if errorMessage}
            <div class="message error-message">
                <strong>Something went wrong.</strong><span>{errorMessage}</span
                ><button type="button" onclick={loadRepositories}
                    >Try again</button
                >
            </div>
        {:else if repositories.length === 0}
            <div class="message">
                This profile does not have any public repositories yet.
            </div>
        {:else}
            <div class="repo-grid">
                {#each repositories as repo, index}
                    <article class="repo-card">
                        <div class="card-topline">
                            <span>0{index + 1}</span><span class="repo-icon"
                                >↗</span
                            >
                        </div>
                        <h3>{repo.name}</h3>
                        <p>
                            {repo.description ||
                                "No description provided for this repository."}
                        </p>
                        <div class="card-footer">
                            <span class="language"
                                ><i
                                    style={`background: ${repo.language ? "#e05d44" : "#a6a6a6"}`}
                                ></i>{repo.language || "Code"}</span
                            >
                            <span>★ {repo.stargazers_count}</span>
                            <a
                                href={repo.html_url}
                                target="_blank"
                                rel="noreferrer">View repo</a
                            >
                        </div>
                    </article>
                {/each}
            </div>
        {/if}
    </section>

    <footer>
        <span>Public repositories only</span><span>Powered by GitHub</span>
    </footer>
</main>

<style>
    @import url("https://fonts.googleapis.com/css2?family=DM+Mono:wght@400;500&family=Space+Grotesk:wght@400;500;600;700&display=swap");
    :global(*) {
        box-sizing: border-box;
    }
    :global(body) {
        margin: 0;
        color: #202b27;
        background: #f1efe8;
        font-family: "Space Grotesk", sans-serif;
    }
    :global(button),
    :global(input) {
        font: inherit;
    }
    main {
        position: relative;
        min-height: 100vh;
        overflow: hidden;
        background-image: linear-gradient(
                90deg,
                rgba(217, 215, 206, 0.22) 1px,
                transparent 1px
            ),
            linear-gradient(rgba(217, 215, 206, 0.22) 1px, transparent 1px);
        background-size: 72px 72px;
    }
    main::before {
        position: absolute;
        inset: 0;
        z-index: -1;
        content: "";
        background: radial-gradient(
                circle at 82% 7%,
                rgba(224, 93, 68, 0.14),
                transparent 24rem
            ),
            linear-gradient(180deg, rgba(241, 239, 232, 0.2), #f1efe8 70%);
        pointer-events: none;
    }
    .topbar,
    .intro,
    .results,
    footer {
        max-width: 1180px;
        margin: 0 auto;
    }
    .topbar {
        position: relative;
        z-index: 1;
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 28px 36px;
        border-bottom: 1px solid #d9d7ce;
    }
    .brand {
        display: flex;
        gap: 11px;
        align-items: center;
        color: inherit;
        text-decoration: none;
        font-weight: 700;
        letter-spacing: -0.03em;
    }
    .brand-mark {
        display: grid;
        place-items: center;
        width: 31px;
        height: 31px;
        color: #f1efe8;
        background: #e05d44;
        font:
            500 10px "DM Mono",
            monospace;
        box-shadow: 4px 4px 0 #202b27;
    }
    .api-status,
    .count,
    .eyebrow,
    label,
    .card-topline,
    .card-footer,
    footer {
        font:
            500 11px "DM Mono",
            monospace;
        letter-spacing: 0.04em;
        text-transform: uppercase;
    }
    .api-status {
        display: flex;
        align-items: center;
        gap: 7px;
        color: #68716b;
    }
    .nav-actions {
        display: flex;
        align-items: center;
        gap: 22px;
    }
    .theme-toggle {
        display: flex;
        align-items: center;
        gap: 7px;
        padding: 7px 10px;
        color: #202b27;
        border: 1px solid #c6c7bc;
        background: rgba(248, 247, 242, 0.7);
        font:
            500 10px "DM Mono",
            monospace;
        letter-spacing: 0.04em;
        text-transform: uppercase;
        transition:
            color 0.2s ease,
            border-color 0.2s ease,
            background 0.2s ease;
    }
    .theme-toggle span {
        color: #e05d44;
        font-size: 15px;
        line-height: 1;
    }
    .theme-toggle:hover {
        color: #e05d44;
        border-color: #e05d44;
    }
    .api-status span {
        width: 7px;
        height: 7px;
        border-radius: 50%;
        background: #3d9b63;
    }
    .intro {
        padding: 88px 36px 82px;
    }
    .eyebrow {
        margin: 0 0 20px;
        color: #e05d44;
    }
    h1 {
        max-width: 760px;
        margin: 0;
        font-size: clamp(3.4rem, 7vw, 6.9rem);
        font-weight: 600;
        line-height: 0.93;
        letter-spacing: -0.08em;
    }
    h1 em {
        color: #6e7871;
        font-style: normal;
    }
    .lede {
        max-width: 380px;
        margin: 27px 0 48px;
        color: #68716b;
        font-size: 17px;
        line-height: 1.45;
    }
    .search {
        max-width: 590px;
    }
    label {
        display: block;
        margin-bottom: 10px;
        color: #68716b;
    }
    .search-row {
        display: flex;
        gap: 10px;
    }
    .input-wrap {
        display: flex;
        align-items: center;
        flex: 1;
        min-width: 0;
        padding: 0 17px;
        border: 1px solid #c6c7bc;
        background: #f8f7f2;
        transition:
            border-color 0.2s ease,
            box-shadow 0.2s ease;
    }
    .input-wrap:focus-within {
        border-color: #202b27;
        box-shadow: 4px 4px 0 #e05d44;
    }
    .input-wrap span {
        color: #e05d44;
        font:
            500 17px "DM Mono",
            monospace;
    }
    input {
        width: 100%;
        min-width: 0;
        padding: 17px 8px;
        color: #202b27;
        border: 0;
        outline: 0;
        background: transparent;
    }
    input::placeholder {
        color: #a1a69e;
    }
    button {
        border: 0;
        cursor: pointer;
    }
    .search button {
        display: flex;
        align-items: center;
        gap: 22px;
        padding: 0 19px;
        color: #f8f7f2;
        background: #202b27;
        font-size: 14px;
        font-weight: 600;
        transition:
            background 0.2s ease,
            transform 0.2s ease;
    }
    .search button:hover:not(:disabled) {
        background: #e05d44;
        transform: translate(2px, -2px);
    }
    button:focus-visible,
    a:focus-visible,
    input:focus-visible {
        outline: 2px solid #e05d44;
        outline-offset: 4px;
    }
    button:disabled {
        cursor: wait;
        opacity: 0.65;
    }
    .results {
        padding: 0 36px 80px;
    }
    .section-heading {
        display: flex;
        justify-content: space-between;
        align-items: end;
        padding-bottom: 17px;
        border-bottom: 1px solid #c6c7bc;
    }
    .section-heading .eyebrow {
        margin-bottom: 9px;
    }
    h2 {
        margin: 0;
        font-size: 27px;
        letter-spacing: -0.05em;
    }
    .count {
        color: #68716b;
    }

    .dark-mode {
        color: #edf3f6;
        background-color: #0b1d32;
        background-image: linear-gradient(
                90deg,
                rgba(100, 143, 178, 0.11) 1px,
                transparent 1px
            ),
            linear-gradient(rgba(100, 143, 178, 0.11) 1px, transparent 1px);
    }
    .dark-mode::before {
        background: radial-gradient(
                circle at 82% 7%,
                rgba(66, 145, 190, 0.25),
                transparent 24rem
            ),
            linear-gradient(180deg, rgba(11, 29, 50, 0.15), #0b1d32 70%);
    }
    .dark-mode .topbar,
    .dark-mode footer {
        border-color: #29425d;
    }
    .dark-mode .api-status,
    .dark-mode .count,
    .dark-mode .lede,
    .dark-mode label,
    .dark-mode .repo-card p,
    .dark-mode .card-footer,
    .dark-mode footer {
        color: #9eb2c2;
    }
    .dark-mode h1 em {
        color: #8fa6b8;
    }
    .dark-mode .theme-toggle {
        color: #edf3f6;
        border-color: #41617c;
        background: rgba(17, 42, 68, 0.8);
    }
    .dark-mode .section-heading {
        border-color: #41617c;
    }
    .dark-mode .input-wrap,
    .dark-mode .repo-card,
    .dark-mode .loading-state,
    .dark-mode .message {
        border-color: #365571;
        background: #112a44;
        box-shadow: 4px 4px 0 rgba(2, 10, 20, 0.24);
    }
    .dark-mode .input-wrap:focus-within {
        border-color: #9ed8ed;
        box-shadow: 4px 4px 0 #e05d44;
    }
    .dark-mode input,
    .dark-mode h2,
    .dark-mode h3,
    .dark-mode .card-footer a {
        color: #edf3f6;
    }
    .dark-mode input::placeholder {
        color: #6f8ba3;
    }
    .dark-mode .card-footer a {
        border-color: #29425d;
    }
    .dark-mode .search button {
        color: #0b1d32;
        background: #9ed8ed;
    }
    .dark-mode .search button:hover:not(:disabled) {
        color: #fff;
        background: #e05d44;
    }
    .dark-mode .repo-card:hover {
        border-color: #e05d44;
    }
    .repo-grid {
        display: grid;
        grid-template-columns: repeat(5, 1fr);
        gap: 10px;
        margin-top: 10px;
    }
    .repo-card {
        display: flex;
        flex-direction: column;
        min-height: 285px;
        padding: 20px 17px 16px;
        border: 1px solid #d0d0c6;
        background: #f8f7f2;
        transition:
            transform 0.2s ease,
            border-color 0.2s ease;
        box-shadow: 4px 4px 0 rgba(32, 43, 39, 0.08);
    }
    .repo-card:hover {
        border-color: #e05d44;
        transform: translateY(-5px);
    }
    .card-topline {
        display: flex;
        justify-content: space-between;
        color: #a1a69e;
    }
    .repo-icon {
        color: #e05d44;
        font-size: 17px;
    }
    h3 {
        margin: 47px 0 13px;
        overflow-wrap: anywhere;
        font-size: 21px;
        line-height: 1.05;
        letter-spacing: -0.06em;
    }
    .repo-card p {
        margin: 0;
        color: #68716b;
        font-size: 13px;
        line-height: 1.4;
    }
    .card-footer {
        display: flex;
        flex-wrap: wrap;
        gap: 10px;
        align-items: center;
        margin-top: auto;
        padding-top: 23px;
        color: #68716b;
        font-size: 9px;
    }
    .language {
        display: flex;
        gap: 5px;
        align-items: center;
    }
    .language i {
        width: 7px;
        height: 7px;
        border-radius: 50%;
    }
    .card-footer a {
        width: 100%;
        padding-top: 10px;
        color: #202b27;
        border-top: 1px solid #d9d7ce;
        text-decoration: none;
    }
    .card-footer a:hover {
        color: #e05d44;
        text-decoration: underline;
        text-underline-offset: 4px;
    }
    .loading-state,
    .message {
        margin-top: 10px;
        padding: 52px 20px;
        color: #68716b;
        border: 1px solid #d0d0c6;
        background: #f8f7f2;
        text-align: center;
    }
    .spinner {
        display: inline-block;
        width: 13px;
        height: 13px;
        margin-right: 8px;
        border: 2px solid #c6c7bc;
        border-top-color: #e05d44;
        border-radius: 50%;
        animation: spin 0.8s linear infinite;
        vertical-align: -2px;
    }
    .error-message {
        display: flex;
        flex-direction: column;
        gap: 7px;
    }
    .error-message strong {
        color: #202b27;
    }
    .error-message button {
        align-self: center;
        margin-top: 12px;
        padding: 9px 13px;
        color: #f8f7f2;
        background: #e05d44;
        font-size: 12px;
    }
    footer {
        display: flex;
        justify-content: space-between;
        padding: 20px 36px 28px;
        color: #92988f;
        border-top: 1px solid #d9d7ce;
    }
    @keyframes spin {
        to {
            transform: rotate(360deg);
        }
    }
    @media (max-width: 900px) {
        .repo-grid {
            grid-template-columns: repeat(2, 1fr);
        }
    }
    @media (max-width: 560px) {
        .topbar,
        .intro,
        .results,
        footer {
            padding-left: 20px;
            padding-right: 20px;
        }
        .topbar {
            padding-top: 20px;
            padding-bottom: 20px;
        }
        .api-status {
            display: none;
        }
        .intro {
            padding-top: 63px;
            padding-bottom: 60px;
        }
        h1 {
            font-size: clamp(3rem, 16vw, 4.2rem);
            letter-spacing: -0.07em;
        }
        .search-row {
            flex-direction: column;
        }
        .search button {
            justify-content: space-between;
            padding: 15px 17px;
        }
        .repo-grid {
            grid-template-columns: 1fr;
        }
        .repo-card {
            min-height: 230px;
        }
        footer {
            gap: 20px;
            flex-direction: column;
        }
    }
</style>
