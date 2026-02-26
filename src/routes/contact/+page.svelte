<svelte:head>
  <title>Contact me</title>
</svelte:head>
    <h1>Contact Me</h1>
    <form action="mailto:mrsun@mit.edu" method="GET">
        <label>Email: <input name="email" type="email"></label>
        <label>Subject: <input name="subject"></label>
        <label>Body: <textarea name="body"></textarea></label>
        <button>Submit</button>
    </form>

    <nav>
        <a href="..">Home</a>
        <a href="../projects">Projects</a>
        <a href="." class="current">Contact</a>
        <a href="../resume">CV</a>
        <a href="https://github.com/msun2022" target="_blank">Github</a>
    </nav>

