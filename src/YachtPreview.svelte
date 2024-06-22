<script>
    import { onMount } from 'svelte';
    import { writable } from 'svelte/store';
    import { navigate, Link } from 'svelte-routing';
    import { loadStripe } from '@stripe/stripe-js';
    import Auth0 from 'auth0-js';

    let project = writable(null);
    let vitrine = writable([]);

    let userName = '';
    let userAvatar = '';

    const webAuth = new Auth0.WebAuth({
        domain: 'dev-whbba5qnfveb88fc.us.auth0.com',
        clientID: 'lmZzOfWN5OU25eodYKHpgPaiN67UQ5m3',
        redirectUri: 'https://verboat.com/',
        responseType: 'token id_token',
        scope: 'openid profile email',
        audience: 'http://Survzilla'
    });

    const stripePromise = loadStripe('pk_live_51PMCFt06wu9e4njhVLypDI8aZi9CB653NXS3729hAjXxuzC84b3Y3v7Swd23gGSspSXjqq2Uwrv9FjAPiyV2fDwO00FJq6gYZs');

    const getProjectIdFromPath = () => {
        const pathParts = location.pathname.split('/');
        return pathParts[pathParts.length - 1];
    };

    const project_code = getProjectIdFromPath();

    onMount(async () => {
        saveCurrentPath;
        userName = localStorage.getItem('user_name') || 'User';
        userAvatar = localStorage.getItem('user_avatar');

        const projectResponse = await fetch(`https://verboat.com/api/preview/${project_code}`, {
            method: 'GET',
            headers: {
                'Content-Type': 'application/json'
            }
        });

        if (projectResponse.ok) {
            const projectData = await projectResponse.json();
            project.set(projectData.project);
        } else if (projectResponse.status === 403) {
            console.error('Access denied. You do not have permission to view this project.');
            navigate('/');  // Redirect to home or any other page
        } else {
            console.error('Failed to fetch project data.');
        }

        const vitrineResponse = await fetch(`https://verboat.com/api/vitrine`, {
            method: 'GET',
            headers: {
                'Content-Type': 'application/json'
            }
        });

        if (vitrineResponse.ok) {
            const vitrineData = await vitrineResponse.json();
            vitrine.set(vitrineData.projects);
        } else {
            console.error('Failed to fetch vitrine data.');
        }
    });

    const handleAuthentication = () => {
        webAuth.parseHash((err, authResult) => {
            if (authResult && authResult.accessToken && authResult.idToken) {
                window.location.hash = authResult.state || '';
                localStorage.setItem('accessToken', authResult.accessToken);
                localStorage.setItem('idToken', authResult.idToken);
                console.log('Authentication successful:', authResult);
                const userId = authResult.idTokenPayload.sub; // assuming 'sub' is the user_id
                localStorage.setItem('user_id', userId);
                localStorage.setItem('user_name', authResult.idTokenPayload.name);
                localStorage.setItem('user_avatar', authResult.idTokenPayload.picture);
                navigate(authResult.state || '/');
            } else if (err) {
                console.error(err);
                navigate('/');
            }
        });
    };

    if (window.location.hash.includes('access_token')) {
        handleAuthentication();
    }

    const saveCurrentPath = (path) => {
        sessionStorage.setItem('initialPath', path);
    };

    const viewProject = async (project_id) => {
        const requestedPath = `/viewproject/${project_id}`;
        const accessToken = localStorage.getItem('accessToken');
        if (!accessToken) {
            saveCurrentPath(requestedPath);
            webAuth.authorize({
                state: window.location.pathname + window.location.search + window.location.hash
            });
            return;
        }

        const response = await fetch(`https://verboat.com/api/check_access/${project_id}`, {
            method: 'GET',
            headers: {
                "Content-Type": "application/json",
                "Authorization": `Bearer ${accessToken}`
            }
        });

        if (response.ok) {
            const data = await response.json();
            if (data.access) {
                navigate(requestedPath);
            } else {
                const stripe = await stripePromise;
                const { error } = await stripe.redirectToCheckout({
                    sessionId: data.sessionId
                });
                if (error) {
                    console.error('Stripe error:', error);
                }
            }
        } else {
            console.error("Failed to check project access.");
            saveCurrentPath(requestedPath);
            webAuth.authorize({
            });
        }
    };

    const fetchnewproject = async (project_code) => {
        const projectResponse = await fetch(`https://verboat.com/api/preview/${project_code}`, {
            method: 'GET',
            headers: {
                'Content-Type': 'application/json'
            }
        });

        if (projectResponse.ok) {
            const projectData = await projectResponse.json();
            project.set(projectData.project);
        } else if (projectResponse.status === 403) {
            console.error('Access denied. You do not have permission to view this project.');
            navigate('/');  // Redirect to home or any other page
        } else {
            console.error('Failed to fetch project data.');
        }

        const vitrineResponse = await fetch(`https://verboat.com/api/vitrine`, {
            method: 'GET',
            headers: {
                'Content-Type': 'application/json'
            }
        });

        if (vitrineResponse.ok) {
            const vitrineData = await vitrineResponse.json();
            vitrine.set(vitrineData.projects);
        } else {
            console.error('Failed to fetch vitrine data.');
        }
        window.scrollTo(0, 0)
    }

    const login = () => {
        saveCurrentPath();
        webAuth.authorize({
        });
    }
</script>

<style>
    /* @import '.\public\static\css\style.css'; */
    @font-face {
	font-family: 'Inter';
	src: url('/static/fonts/Inter-Bold.ttf');
	font-weight: bold;
	font-display: swap;
}
@font-face {
	font-family: 'Inter';
	src: url('/static/fonts/Inter-Regular.ttf');
	font-weight: normal;
	font-display: swap;
}
* {
	font-family: 'Inter', sans-serif;
	margin: 0;
	padding: 0;
	box-sizing: border-box;
}
body {
	display: flex;
	flex-direction: column;
	min-height: 100dvh;
	scroll-behavior: smooth;
}
img {
	max-width: 100%;
	height: auto;
}
.container {
	max-width: 1440px;
	margin: 2.5rem auto;
	padding: 0 3.5vw;
}
.header__nav {
	max-width: 92%;
	font-size: 20px;
	display: flex;
	justify-content: space-between;
}
.header__logo {
	display: flex;
	gap: 1rem;
	align-items: center;
	text-decoration: none;
	color: black;
}
.header__link {
	text-decoration: none;
	color: black;
}
.header__links {
	display: flex;
	align-items: center;
	gap: 2.1rem;
}

.header__burger {
	background-color: white;
	border: none;
	display: none;
}
.button {
	padding: 14px;
	background-color: black;
	color: white;
	border-radius: 8px;
	font-size: 14px;
	letter-spacing: 0.03rem;
	cursor: pointer;
	transition: background-color 0.2s ease-out, transform 0.2s ease-out;
}
.button:hover {
	background-color: rgb(50, 50, 50);

}
.button:active {
	background-color: rgb(65, 65, 65);
	transform: scale(1.01);
} 
.button:focus {
	outline: 2px solid gray;
} 
.product__container {
	display: grid;
	gap: 8rem;
	grid-template-columns: repeat(auto-fit, minmax(22rem, 1fr));
}
.product__aside {
	background-color: #f7f7f7;
	border-radius: 8px;
	display: flex;
	align-items: center;
	justify-content: center;
}
.product__info {
	width: 80%;
	display: flex;
	flex-direction: column;
	gap: 1.2rem;

}
.product__title {
	margin: 0;
}
.product__title {
	font-weight: normal;
}
.product__subtitle {
	font-size: 1.2rem;
	color: #7f7f7f;
}
.product__price {
	font-size: 1.2rem;
	font-weight: normal;
	letter-spacing: 0.05rem;
}
.product__description {
	color: #7f7f7f;
	line-height: 1.5;
	text-align: justify;
	letter-spacing: 0.03rem;
}

.product__signature {
	color: #7f7f7f;
	font-size: 14px;
}

.category__title {
	font-weight: normal;
}
.category__card-list {
	display: grid;
	gap: 3vw;
	grid-template-columns: repeat(auto-fill, minmax(16rem, 1fr));
	grid-auto-rows: minmax(100px, auto);
	list-style: none;
}
.category__card-item {
	display: flex;
	flex-direction: column;
	gap: 0.7rem;
}
.category__card-img {
	background-color: #f7f7f7;
	border-radius: 8px;
}
.category__container {
	display: flex;
	flex-direction: column;
	gap: 2rem;
}
.category__card-title {
	font-weight: normal;
	letter-spacing: 0.05rem;
	font-size: 1.3rem;

}
.category__card-subtitle {
	color: #7f7f7f;
	font-size: 1.2rem;
	letter-spacing: 0.03rem;
}
.category__card-year {
	color: #7f7f7f;
	font-size: 1.2rem;
	letter-spacing: 0.03rem;
}
.category__card-price {
	font-weight: normal;
	font-size: 1.2rem;
	letter-spacing: 0.05rem;
}

img {
	padding: 0;
}
li img {
	width: 80%;
	height: 80%;
	vertical-align:middle;
}
.product__info-button {
	display: flex;
	flex-direction: column;
	gap: 0.5rem;
}
.product__info-button button {
	width: 100%;
}

@media (max-width:768px) {
.header__links {
	display: none;
}
.header__burger {
	display: block;
}
}
@media (max-width:480px) {
	.product__container {
		display: flex;
		flex-direction: column;
		gap: 3rem;
	}
	h1 {
		font-size: 1.5rem;
	}
	p {
		font-size: 0.9rem;
	}

	.product__price,
	.product__subtitle {
		font-size: 1.2rem;
	}
	.category__card-list {
		gap: 3rem;
	}
}
.product__img {
        border-radius: 6px;
    }
    .product__title {
        font-weight: bold;
        font-size: 2rem;
    }

    .user-avatar {
        width: 32px;
        height: 32px;
        border-radius: 50%;
        background: #fff;
    }
</style>

<header class="header">
    <div class="container header__container">
        <nav class="header__nav">
            <a href="/" class="header__logo">
                <img class="header__img" src="/static/VerboatLogo02.png" width="70" height="70" alt="">
                <span class="header__title">Ver Boat</span>
            </a>
            <div style=""></div>
            <di class="header__burger">
                <button class="header__burger" id="header-burger">
                    <svg width="64px" height="64px" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg"><g id="SVGRepo_bgCarrier" stroke-width="0"></g><g id="SVGRepo_tracerCarrier" stroke-linecap="round" stroke-linejoin="round"></g><g id="SVGRepo_iconCarrier"> <path fill-rule="evenodd" clip-rule="evenodd" d="M3.46447 20.5355C4.92893 22 7.28595 22 12 22C16.714 22 19.0711 22 20.5355 20.5355C22 19.0711 22 16.714 22 12C22 7.28595 22 4.92893 20.5355 3.46447C19.0711 2 16.714 2 12 2C7.28595 2 4.92893 2 3.46447 3.46447C2 4.92893 2 7.28595 2 12C2 16.714 2 19.0711 3.46447 20.5355ZM18.75 16C18.75 16.4142 18.4142 16.75 18 16.75H6C5.58579 16.75 5.25 16.4142 5.25 16C5.25 15.5858 5.58579 15.25 6 15.25H18C18.4142 15.25 18.75 15.5858 18.75 16ZM18 12.75C18.4142 12.75 18.75 12.4142 18.75 12C18.75 11.5858 18.4142 11.25 18 11.25H6C5.58579 11.25 5.25 11.5858 5.25 12C5.25 12.4142 5.58579 12.75 6 12.75H18ZM18.75 8C18.75 8.41421 18.4142 8.75 18 8.75H6C5.58579 8.75 5.25 8.41421 5.25 8C5.25 7.58579 5.58579 7.25 6 7.25H18C18.4142 7.25 18.75 7.58579 18.75 8Z" fill="#000"></path> </g></svg>
                </button>
            </di>
            <div class="header__links">
                <a class="header__link" href="/">Home</a>
                <a class="header__link" href="/main">Add Survey</a>
                <a class="header__link" href="/bought-projects">My Boats</a>
                <a class="header__link" href="/">Resources</a>
                {#if userAvatar}
                    <span>{userName}</span>
                    <img src={userAvatar} alt="User Avatar" class="user-avatar"/>
                {:else}
                    <div on:click={login} class="button header__button">Login/Sign Up</div>
                {/if}
            </div>
        </nav>
    </div>
</header>

<main class="main">
    <section class="product">
        <div class="container product__container">
            {#if $project}
            <figure class="product__aside">
                <img class="product__img" src={$project.gen_info_image} alt="Product image">
            </figure>
            <article class="product__info">
                <h1 class="product__title">{$project.vessel_name}</h1>
                <span class="product__subtitle">{$project.length} / {$project.year}</span>
                <strong class="product__price">${$project.price}</strong>
                <p class="product__description">{$project.description}</p>
                <div style="flex:content"></div>
                <div class="product__info-button">
                    <p class="product__description">{$project.project_code}</p>
                    <button class="button product__button" type="button"  on:click={viewProject($project.project_id)}>Open Survey</button>
                    <small class="product__signature" type="button">Survey access free is 10$</small>
                </div>
            </article>
            {/if}
        </div>
    </section>
</main>
<footer>
</footer>

