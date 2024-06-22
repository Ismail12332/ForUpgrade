<script>
    export let webAuth;

    import { onMount } from 'svelte';
    import { writable } from 'svelte/store';
    import { Link, navigate } from "svelte-routing";


    let boughtProjects = writable([]);

    async function fetchBoughtProjects() {
        const accessToken = localStorage.getItem('accessToken');
        const response = await fetch(`https://verboat.com/api/bought_projects`, {
            method: 'GET',
            headers: {
                'Authorization': `Bearer ${accessToken}`,
                'Content-Type': 'application/json'
            }
        });

        if (response.ok) {
            const data = await response.json();
            boughtProjects.set(data.projects);
        } else if (response.status === 401 || response.status === 403) {
            // Повторная аутентификация
            saveCurrentPath();
            webAuth.authorize();
        } else {
            console.error("Failed to fetch bought projects");
        }
    }

    const saveCurrentPath = () => {
        const currentPath = window.location.pathname;
        sessionStorage.setItem('initialPath', currentPath);
    };

    onMount(() => {
        fetchBoughtProjects();
    });

    function handleClick(project_code) {
        navigate(`/yachtpreview/${project_code}`);
    }
</script>

<style>
    main {
        font-family: "Roboto", sans-serif;
    }
    .container {
        max-width: 1200px;
        margin: 0 auto;
        padding: 20px;
    }
    .project-list {
        display: flex;
        flex-wrap: wrap;
        gap: 20px;
        list-style: none;
        padding: 0;
    }
    .project-item {
        flex: 1 1 calc(33.333% - 20px);
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
        transition: transform 0.2s;
        cursor: pointer;
    }
    .project-item:hover {
        transform: scale(1.05);
    }
    .project-item img {
        width: 100%;
        height: 200px;
        object-fit: cover;
    }
    .project-item h2 {
        font-size: 1.25em;
        margin: 10px;
    }
    .project-item p {
        margin: 10px;
    }
    .forback {
        display: inline-block;
        margin-bottom: 20px;
        color: #007BFF;
        text-decoration: none;
    }
    .forback:hover {
        text-decoration: underline;
    }
</style>

<main class="container">
    <Link to="/" class="forback">Back</Link>
    <div>
        <h1>My Boats</h1>
        {#if $boughtProjects.length > 0}
            <ul class="project-list">
                {#each $boughtProjects as project}
                    <li class="project-item" on:click={() => handleClick(project.project_code)}>
                        <img src={project.gen_info_image} alt={project.vessel_name} />
                        <h2>{project.vessel_name}</h2>
                        <p>{project.description}</p>
                        <p>{project.project_code}</p>
                    </li>
                {/each}
            </ul>
        {/if}
        {#if $boughtProjects.length === 0}
            <p>No projects bought yet.</p>
        {/if}
    </div>
</main>
