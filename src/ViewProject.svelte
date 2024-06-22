<script>
    export let webAuth;
    import { onMount } from 'svelte';
    import { writable } from 'svelte/store';
    import { Link } from "svelte-routing";
    import { navigate } from 'svelte-routing';
    import { loadStripe } from '@stripe/stripe-js';

    let project = writable(null);


    const stripePromise = loadStripe('pk_live_51PMCFt06wu9e4njhVLypDI8aZi9CB653NXS3729hAjXxuzC84b3Y3v7Swd23gGSspSXjqq2Uwrv9FjAPiyV2fDwO00FJq6gYZs');

    const getProjectIdFromPath = () => {
        const pathParts = location.pathname.split('/');
        return pathParts[pathParts.length - 1];
    };

    const project_id = getProjectIdFromPath();

    const saveCurrentPath = () => {
        const currentPath = window.location.pathname;
        sessionStorage.setItem('initialPath', currentPath);
    };

    onMount(() => {
      const fetchProject = async () => {
          if (project_id) {
              const accessToken = localStorage.getItem('accessToken');
              const response = await fetch(`https://verboat.com/api/project/${project_id}`, {
                  method: 'GET',
                  headers: {
                      'Content-Type': 'application/json',
                      'Authorization': `Bearer ${accessToken}`
                  }
              });

              if (response.ok) {
                  const data = await response.json();
                  project.set(data.project);
              } else if (response.status === 401) {
                  saveCurrentPath(); // Assuming you have this function defined
                  webAuth.authorize();
              } else if (response.status === 403) {
                  console.error('Access denied. You do not have permission to view this project.');
                  window.alert('Access denied. You do not have permission to view this project.');
                  navigate('/');  // Redirect to home or any other page
              } else {
                  console.error('Failed to fetch project data.');
              }
          } else {
              console.error('Project ID is undefined.');
          }
      };

      // Adding a delay before executing the fetch request
      setTimeout(fetchProject, 3000); // Delay of 3 seconds
  });


  function printPDF() {
        window.print();
    }
    

    async function handleBuyYacht() {
      const accessToken = localStorage.getItem('accessToken');
        const stripe = await stripePromise;
        const response = await fetch(`https://verboat.com/api/buy_yacht/${project_id}`, {
            method: 'GET',
            headers: {
                'Authorization': `Bearer ${accessToken}`,
                'Content-Type': 'application/json'
            }
        });
        const data = await response.json();
        
        const { error } = await stripe.redirectToCheckout({
            sessionId: data.sessionId
        });
        if (error) {
            console.error("Error redirecting to checkout:", error);
        }
    }
</script>

<style>
  main {
  font-family: 'Inter', sans-serif;
}

.main-container {
  background-color: #f0f0f0; /* Background color for the margins */
  padding: 20px;
}
.container {
  width: 100%;
  max-width: 1200px;
  margin: 0 auto;
  background-color: #fff; /* White background for the main content */
  padding: 4.5rem;
}
.header {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 1rem;
  margin-bottom: 20px;
}
.header img {
  max-height: 100px;
}
.project-info {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 40px;
  flex-direction: column-reverse;
}
.info-text {
  display: flex;
  flex-direction: column;
  gap: 24px;
  color: rgba(0, 0, 0, 1);
height: auto;
align-self: stretch;
text-align: left;
line-height: normal;
}

.info-text__model {
font-size: 3rem;
  font-style: normal;
  font-weight: 900;
  font-stretch: normal;
  letter-spacing: -0.02em;
  color: rgba(0, 0, 0, 1);
  height: auto;
  align-self: stretch;
  text-align: left;
  line-height: normal;
}

.info-text__year {
color: rgba(0, 0, 0, 1);
  height: auto;
  font-size: 1.5rem;
  align-self: stretch;
  font-style: bold;
  text-align: left;
  font-weight: 800;
  line-height: normal;
  font-stretch: normal;
  text-decoration: none;
}

.info-text__boat_registration {
color: rgba(0, 0, 0, 1);
  height: auto;
  font-size: 1.5rem;
  align-self: stretch;
  font-style: italic;
  text-align: left;
  font-weight: 300;
  line-height: normal;
  font-stretch: normal;
  text-decoration: none;
}

.info-text__engine {
color: rgba(0, 0, 0, 1);
height: auto;
font-size: 1.5rem;
align-self: stretch;
text-align: left;
font-weight: 400;
line-height: normal;
font-stretch: normal;
text-decoration: none;
}

.info-text__price {
color: rgba(0, 0, 0, 1);
height: auto;
font-size: 1.5rem;
align-self: stretch;
font-style: Bold;
text-align: left;
font-weight: 900;
line-height: normal;
font-stretch: normal;
text-decoration: none;
}

.info-text__city {
color: rgba(0, 0, 0, 1);
height: auto;
font-size: 1.5rem;
align-self: stretch;
text-align: left;
font-weight: 400;
line-height: normal;
font-stretch: normal;
text-decoration: none;
}

.info-text__owner_contact {
color: rgba(0, 0, 0, 1);
height: auto;
font-size: 1.5rem;
align-self: stretch;
font-style: italic;
text-align: left;
font-weight: 300;
line-height: normal;
font-stretch: normal;
text-decoration: none;
}
.info-text__description {
font-size: 1.1rem;
  font-style: normal;
  font-weight: 400px;
  font-stretch: normal;
  letter-spacing: 0em;
  color: rgba(130, 130, 130, 1);
  height: auto;
  align-self: stretch;
  text-align: left;
  line-height: 150%;
}

.info-text__created_at {
color: rgba(130, 130, 130, 1);
height: auto;
align-self: stretch;
text-align: left;
font-size: 1.1rem;
  font-style: normal;
  font-weight: 400px;
  font-stretch: normal;
  letter-spacing: 0em;
line-height: 150%;
}
.info-text__code {
color: rgba(130, 130, 130, 1);
height: auto;
align-self: stretch;
text-align: left;
line-height: 150%;
font-size: 1.1rem;
  font-style: normal;
  font-weight: 400px;
  font-stretch: normal;
  letter-spacing: 0em;
}
.project-image {
  width: 100%;
  height: 550px;
  object-fit: cover;
}
.project-image-glav {
  width: 100%;
  height: 34.5rem;
  object-fit: cover;
  border-radius: 20px;
}
.section {
  margin: 40px 0;
  page-break-inside: avoid; /* Prevents section from breaking */
}
.section-title {
  font-size: 2em;
  text-align: center;
  margin-bottom: 20px;
}
.subsection {
  width: 100%;
  text-align: left;
  margin-bottom: 20px;
  page-break-inside: avoid; /* Prevents subsection from breaking */
}
.subsection-title {
  font-size: 1.3rem;
  font-weight: bold;
  margin-bottom: 10px;
}
.elements-container {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 2.5rem;
}
.elements-container li {
font-family: 'Arial', sans-serif;
  color: rgba(120, 120, 120, 1);
  list-style-type: none;
  font-weight: 600;
}
.element {
  box-sizing: border-box;
  margin-bottom: 20px;
  display: flex;
  flex-direction: column;
  page-break-inside: avoid; /* Prevents element from breaking */
}
.element-title {
  font-size: 1.3em;
  margin-bottom: 10px;
  font-weight: 500;
}
.element-content {
  border-radius: 5px;
  flex-direction: column;
  justify-content: left;
  align-items: left;
  flex-grow: 1;
  border-radius: 10px;
}
.element img {
  width: 100%;
  max-height: 350px; /* Fixed height for the images */
  object-fit: cover; /* Maintains aspect ratio while covering the entire area */
  border-radius: 20px;
}
.element-text {
  margin-top: 10px;
  font-size: 1em;
  flex-grow: 1; /* Allows the text to grow and take up available space */
}
.final-section {
  text-align: center;
  margin-top: 40px;
  padding: 0 100px; /* Added padding for text */
  page-break-inside: avoid; /* Prevents final section from breaking */
}
.final-section h2 {
  font-size: 2em;
  margin-bottom: 20px;
}
  .final-section li {
    list-style-type: none; /* Убирает маркеры у элементов списка */
  }

  .final-section ul {
    list-style-type: none;
  }
.inspection-text {
  font-size: 1em;
  color: #666;
  margin-bottom: 20px;
}
.page-break {
  page-break-before: always; /* Forces page break before this element */
}
.section-title {
text-align: left;
}
ul {
padding-left: 0;
}

.info {
  display: flex;
  flex-direction: row;
  align-items: top;
  justify-content: right;
  gap: 8rem;
}
@media (max-width: 768px), (max-height: 768px) {
.info-text {
padding: 1rem;
}
.info-text__model {
font-size: 2rem;
}

.info-text__year {
font-size: 1.1rem;
}


.info-text__boat_registration {
font-size: 1.1rem;
}


.info-text__engine {
font-size: 1.1rem;
}


.info-text__price {
font-size: 1.1rem;
}


.info-text__city {
font-size: 1.1rem;
}

.info-text__owner_contact {
font-size: 1.1rem;
}
.info-text__description {
font-size: 0.9rem;
}

.info-text__created_at {
font-size: 0.9rem;
}
.info-text__code {
font-size: 0.9rem;
}

.header {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0;
  margin-bottom: 20px;
  flex-direction: column;
}
.header img {
  max-width: 50px;
  max-height: 50px;
}
.container {
  width: 100%;
  margin: 0 auto;
  padding: 0;
}
.elements-container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  padding: 1rem;
}
.section h1,
.section h2 {
  padding: 1rem;
}
.final-section {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  padding: 1rem;
}
.info {
  flex-direction: column;
  gap: 0;
  margin-bottom: 1rem;
}

}
@media screen and (max-height: 768px) {
.element img {
  max-height: inherit
}
.element-content {
  padding: 3rem;
  max-height: 350px;
}
}
.pdf {
display: flex;
justify-content: space-between;
}
</style>

<main>
  <div class="main-container">
    <!--<button on:click={handleBuyYacht}>Buy Project</button>-->
    <div class="pdf">
      <Link to='/'>Home</Link>
      <button on:click={printPDF}>Download PDF</button>
    </div>
    <div class="container" id="pdfContent">
      <header class="header">
          <h1>VerBoat.Com</h1>
          <img src="/static/VerboatLogo02.png" alt="Logo">
          <h1>Boat Inspection</h1>
      </header>
      {#if $project}
      <div class="project-info">
        <img src={ $project.main_image } alt="Main Image" class="project-image-glav">
        <div class="info">
        <div class="info-text">
          <span class="info-text__model"><span>{ $project.boat_make } { $project.boat_model }</span></span>
          <span class="info-text__year"><span>{ $project.length } ft / { $project.year }</span></span>
          <span class="info-text__boat_registration"><span>{ $project.boat_registration }</span></span>
          <span class="info-text__engine"><span>{ $project.engine }</span></span>
          <span class="info-text__price"><span>{ $project.price }$</span></span>
          <span class="info-text__city"><span>{ $project.city }</span></span>
          <span class="info-text__owner_contact "><span>Owner Contact: { $project.owner_contact }</span></span>
          <span class="info-text__description"><span>Fully functional yacht without significant defects, the yacht is 
            missing several parts.</span></span>
          <span class="info-text__created_at "><span>{ $project.created_at }</span></span>
          <span class="info-text__code "><span>{ $project.project_code }</span></span>
        </div>
        <div class="info-qr">
          <img src={ $project.qr_code } alt="QR-CODE">
        </div>
      </div>
      </div>
      <div class="project-details">
        {#each Object.keys($project.sections) as sectionKey}
          {#if Object.keys($project.sections[sectionKey]).some(subsectionKey => Object.keys($project.sections[sectionKey][subsectionKey]).some(elementKey => $project.sections[sectionKey][subsectionKey][elementKey].steps?.length > 0 || $project.sections[sectionKey][subsectionKey][elementKey].images?.length > 0))}
            <div class="section">
              <h1 class="section-title">{sectionKey.replace(/_/g, ' ')}</h1>
              {#each Object.keys($project.sections[sectionKey]) as subsectionKey}
                {#if Object.keys($project.sections[sectionKey][subsectionKey]).some(elementKey => $project.sections[sectionKey][subsectionKey][elementKey].steps?.length > 0 || $project.sections[sectionKey][subsectionKey][elementKey].images?.length > 0)}
                  <div class="subsection">
                    <h2 class="subsection-title">{subsectionKey.replace(/_/g, ' ')}</h2>
                    <div class="elements-container">
                      {#each Object.keys($project.sections[sectionKey][subsectionKey]) as elementKey}
                        {#if $project.sections[sectionKey][subsectionKey][elementKey].steps?.length > 0 || $project.sections[sectionKey][subsectionKey][elementKey].images?.length > 0}
                          <div class="element">
                           
                            <div class="element-content">
                              {#if $project.sections[sectionKey][subsectionKey][elementKey].images?.length > 0}
                                {#each $project.sections[sectionKey][subsectionKey][elementKey].images as image}
                                  <img src={image} alt="Element Image" class="project-image" />
                                {/each}
                              {:else}
                                <img src="/static/Noimage.PNG" alt="No Image" class="project-image">
                              {/if}
                              <h2 class="element-title">{elementKey.replace(/_/g, ' ')}</h2>
                              <div class="element-text">
                                {#if $project.sections[sectionKey][subsectionKey][elementKey].text}
                                  <p>{ $project.sections[sectionKey][subsectionKey][elementKey].text }</p>
                                {/if}
                              </div>
                              {#if $project.sections[sectionKey][subsectionKey][elementKey].steps?.length > 0}
                                <ul>
                                  {#each $project.sections[sectionKey][subsectionKey][elementKey].steps as step}
                                    <li>{step}</li>
                                  {/each}
                                </ul>
                              {/if}
                            </div>
                          </div>
                        {/if}
                      {/each}
                    </div>
                  </div>
                {/if}
              {/each}
            </div>
          {/if}
        {/each}
      </div>
      <div class="final-section">
        <h2>Disclaimer:</h2>
        <p class="inspection-text">This summary has been compiled by AI based on the provided data and represents a relatively objective opinion. However, neither we as the website, the AI, nor the authors of the AI bear any responsibility for your decision regarding the purchase of this boat and any consequences thereof.</p>
        {@html $project.final_note }
        {#if $project.final_kartinka}
        <div class="image-container" style="display: flex; justify-content:center align-items: center;">
          <img src={ $project.final_kartinka } alt="Final Image" class="project-image-glav" />
        </div>
        {/if}
        <div class="header">
            <h1>VerBoat.com 
                <img src="/static/VerboatLogo02.png" alt="Logo">
            Boat Inspection</h1>
        </div>
      </div>
      {/if}
    </div>
  </div>
</main>


