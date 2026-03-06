<script>
  import { page } from "$app/state";
</script>

<header>
  <div class="corner">
    <a href="https://svelte.dev/docs/kit"> </a>
  </div>

  <nav>
    <div class="nav-left">
      <ul>
        <li aria-current={page.url.pathname === "/" ? "page" : undefined}>
          <a href="/">Home</a>
        </li>
        <li aria-current={page.url.pathname === "/about" ? "page" : undefined}>
          <a href="/about">About</a>
        </li>
      </ul>
    </div>
    <div class="nav-right">
      <ul>
        <li
          aria-current={page.url.pathname.startsWith("/sverdle")
            ? "page"
            : undefined}
        ></li>
      </ul>
    </div>
  </nav>
</header>

<style>
  * {
    box-sizing: border-box;
  }

  header {
    width: 100%;
    max-width: 100vw;
    display: flex;
    align-items: center;
    background: white;
    padding: 0 1rem;
    overflow: hidden;
  }

  .corner {
    min-width: 3em;
    height: 3em;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  nav {
    flex: 1;
    display: flex;
    justify-content: space-between;
    align-items: center;
    --background: rgba(255, 255, 255, 0.7);
  }

  .nav-left {
    flex: 1;
    display: flex;
    justify-content: center;
  }

  .nav-right {
    flex-shrink: 0;
  }

  .nav-left ul,
  .nav-right ul {
    display: flex;
    list-style: none;
    margin: 0;
    padding: 0;
    height: 3em;
    background: var(--background);
    align-items: center;
  }

  li {
    position: relative;
    height: 100%;
    white-space: nowrap;
  }

  li[aria-current="page"]::before {
    --size: 6px;
    content: "";
    width: 0;
    height: 0;
    position: absolute;
    top: 0;
    left: calc(50% - var(--size));
    border: var(--size) solid transparent;
    border-top: var(--size) solid #ff3e00;
  }

  .nav-left a {
    display: flex;
    height: 100%;
    align-items: center;
    padding: 0 0.6rem;
    color: #333;
    font-weight: 700;
    font-size: 0.8rem;
    text-transform: uppercase;
    letter-spacing: 0.1em;
    text-decoration: none;
    transition: color 0.2s linear;
  }

  .nav-left a:hover {
    color: #ff3e00;
  }

  .nav-right a {
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 0.3rem 0.6rem; /* ปรับ padding เอง */
    margin: 0.2rem 0; /* เพิ่ม margin บนล่างเล็กน้อย */
    border: 1px solid #ff3e00;
    border-radius: 6px;
    background-color: rgba(255, 62, 0, 0.05);
    color: #ff3e00;
    font-weight: 600;
    font-size: 0.75rem;
    text-transform: uppercase;
    text-decoration: none;
    transition: all 0.2s ease;
    box-sizing: border-box;
    margin-top: 0.8rem;
  }

  .nav-right a:hover {
    background-color: #ff3e00;
    color: white;
  }

  @media (max-width: 600px) {
    .nav-left a,
    .nav-right a {
      font-size: 0.7rem;
      padding: 0.3rem 0.6rem;
    }
  }
</style>
