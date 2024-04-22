<script>
  import "../styles.scss";
  import CurrConverter from "./libs/currConverter.svelte";
  import { browser } from "$app/environment";

  let apiKeyInput;
  let key;
  let currentAction = "apiKey";
  let apiContainer;
  let currContainer;
  let errorContainer;
  let errorMsg;

  //
  function setApiKey() {
    var apiKey = apiKeyInput.value;
    if (isValidKey(apiKey)) {
      errorMsg = null;
      key = apiKey;
      changeContainerAnime();
    }
  }

  function isValidKey(value) {
    if (value.length > 15) {
      return true;
    } else {
      showError();
    }
  }

  function changeContainerAnime() {
    apiContainer.style.opacity = 0;

    setTimeout(() => {
      currentAction = "currency";
      setTimeout(() => {
        currContainer.style.opacity = 1;
      }, 100);
    }, 500);
  }

  function showError() {
    errorMsg = "min-length is 15";
    setTimeout(() => {
      errorContainer.style.opacity = 1;
    }, 1);
  }

  function hideError() {
    errorContainer.style.opacity = 0;
    setTimeout(() => {
      errorMsg = "";
    }, 700);
  }

  function setActive(ele) {
    if (!ele.classList.contains("input--active")) {
      ele.classList.add("input--active");
    }
  }

  function setInactive(ele) {
    ele.classList.remove("input--active");
  }

  function listen() {
    if (browser) {
      setTimeout(() => {
        document.onclick = (e) => {
          if (apiKeyInput.value == "" && e.target != apiKeyInput) {
            setInactive(apiKeyInput);
          }
        };
      }, 1);
    }
  }

  listen();
</script>

<main class="main">
  {#if currentAction == "apiKey"}
    <div class="apikey-container" bind:this={apiContainer}>
      <input
        class="apikey-container__input input nostyle-input"
        type="text"
        placeholder="api key"
        bind:this={apiKeyInput}
        on:input={(e) => hideError()}
        on:click={() => setActive(apiKeyInput)}
      />
      {#if errorMsg}
        <div bind:this={errorContainer} class="apikey-container__error">
          <p>{errorMsg}</p>
        </div>
      {/if}

      {#if currentAction == "apiKey"}
        <button on:click={() => setApiKey()} class="btn nostyle-btn"
          >confirm</button
        >
      {/if}
    </div>
  {:else if currentAction == "currency"}
    <div class="currency-container" bind:this={currContainer}>
      <CurrConverter {key} />
    </div>
  {/if}
</main>

<style lang="scss">
  @import "../styles/variables.scss";

  .main {
    display: flex;
    flex-direction: column;
    justify-content: center;
    width: 400px;
    height: 200px;
    background: repeating-linear-gradient(
      -45deg,
      $gray-saturate 0px,
      $gray-saturate 13px,
      transparent 13px,
      transparent 26px
    );
    border-radius: 50px;
  }

  .apikey-container {
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    align-items: center;
    height: 110px;
    transition: 0.6s;

    &__input {
      width: 250px;
      border-radius: 20px;
    }

    &__error {
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      width: 180px;
      height: 25px;
      padding-top: 2px;
      text-align: center;
      background-color: $gray-saturate;
      border-radius: 20px;
      text-transform: uppercase;
      color: $red;
      transition: 0.7s;
      opacity: 0;
    }
  }

  .currency-container {
    opacity: 0;
    transition: 0.6s;
  }

  .btn {
    text-transform: uppercase;
    transition: 0.3s;
    border-radius: 20px;
    padding: 5px 10px;
    color: $white-transparent;
    // border: 2px solid transparent;

    &:hover {
      background-color: $green;
      // border: 2px solid $gray-saturate;
    }
  }
</style>
