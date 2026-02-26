<script>
    export let data = {};
</script>

<article class="reading-item">
    <div class="book-cover">
        <img src={data.image} alt="">
    </div>

    <div class="book-details">
        <h3>{data.title}</h3>
        <h4>{data.author}</h4>
        <p><strong>My Thoughts:</strong> {data.thoughts}</p>
    </div>
    
</article>

<style>
    img {
        max-width: 100%;
        max-height: 20rem;
    }

    /* Making the title feel like a book header */
    h3 {
        font-family: serif;
        font-weight: 800;
        color: #020202; 
        margin-bottom: 0rem;
    }

    /* Softening the authro sub-header to a dark grey */
    h4 {
        font-family: sans-serif; 
        color: #414141; 
        font-weight: 500;
        margin-top: 0rem;
    }

    .reading-item {
        display: flex;
        gap: 2rem;
        align-items: flex-start;
        padding: 1rem 0;
    }

    .book-cover {
        flex-shrink: 0;
        width: 120px;
    }

    .book-cover img {
        width: 100%; /* Fills the 120px container */
        display: block;
        aspect-ratio: 2 / 3; /* Everything same proportions */
        object-fit: cover;
    }

    .book-details {
        display: flex;
        flex-direction: column; /* Stacks Title, Author, and Thoughts */
    }
</style>