<script>
  import { goto } from "$app/navigation";
  let username = "";
  let password = "";
  let remember = false;

  function login() {
    alert(
      `Username: ${username}, Password: ${password}, Remember: ${remember}`
    );
    goto("/");
  }
</script>

<div class="container">
  <h1>my-to do list</h1>
  <h2>🔐เข้าสู่ระบบ</h2>

  <div class="input-group">
    <span class="emoji">👤</span>
    <input type="text" placeholder="Username" bind:value={username} />
  </div>

  <div class="input-group">
    <span class="emoji">🔒</span>
    <input type="password" placeholder="Password" bind:value={password} />
  </div>

  <div class="checkbox">
    <input type="checkbox" bind:checked={remember} />
    <label>จดจำรหัสผ่าน</label>
  </div>

  <button class="login-btn" on:click={login}>LOGIN</button>

  <div class="or">หรือ</div>

  <div class="social-login">
    <button class="facebook-btn">เข้าสู่ระบบด้วย Facebook</button>
    <button class="google-btn">เข้าสู่ระบบด้วย Google</button>
  </div>
</div>

<style>
  .container {
    max-width: 420px;
    margin: auto;
    padding: 2rem;
    font-family: sans-serif;
    text-align: center;
  }

  h2 {
    font-size: 2rem;
    margin-bottom: 2rem;
  }

  .input-group {
    display: flex;
    align-items: center;
    background: #f9f9f9;
    padding: 1rem;
    margin-bottom: 1.2rem;
    border-radius: 1rem;
    border: 1px solid #ddd;
    font-size: 1.1rem;
  }

  .emoji {
    margin-right: 0.75rem;
    font-size: 1.4rem;
  }

  .input-group input {
    flex: 1;
    border: none;
    background: transparent;
    outline: none;
    font-size: 1.1rem;
  }

  .login-btn {
    width: 100%;
    padding: 1rem;
    background-color: #4caf50;
    color: white;
    border: none;
    border-radius: 1rem;
    font-size: 1.1rem;
    cursor: pointer;
    margin-top: 0.5rem;
  }

  .login-btn:hover {
    background-color: #45a049;
  }

  .checkbox {
    margin-bottom: 1rem;
    text-align: left;
    display: flex;
    align-items: center;
    font-size: 1rem;
  }

  .checkbox input {
    margin-right: 0.5rem;
  }

  .or {
    margin: 1.5rem 0 1rem;
    font-weight: bold;
    color: #666;
    font-size: 1rem;
  }

  .social-login {
    display: flex;
    flex-direction: column;
    gap: 0.75rem;
  }

  .facebook-btn,
  .google-btn {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 0.5rem;
    color: white;
    padding: 1rem;
    border: none;
    border-radius: 1rem;
    cursor: pointer;
    font-size: 1rem;
  }

  .facebook-btn {
    background-color: #3b5998;
  }

  .google-btn {
    background-color: #db4437;
  }
</style>
