<script>
  import { currencieTypes } from "./currencies";
  import { browser } from "$app/environment";

  let items = Object.entries(currencieTypes);
  let currenciesShorts = items.map((val) => val[0]);

  let firstCurrencies = items;
  let firstInput;
  let firstAmountInput;

  let secondCurrencies = items;
  let secondInput;
  let secondAmountInput;

  let lastValues = { first: "", second: "", amount: "" };
  let amount;
  let showDdon;
  let isFirst;
  let isPending = false;
  export let key;

  // ввод кода валюты
  // можно вводить и код и полное название валюты
  function currencyNameOrCodeInput(inputELe, isFirst) {
    let input = inputELe.value.toLowerCase();
    inputELe.value = inputELe.value.toUpperCase();
    if (isFirst) {
      firstCurrencies = [];
      firstCurrencies = getItems(input);
    } else {
      secondCurrencies = [];
      secondCurrencies = getItems(input);
    }
    setActive(inputELe);
  }

  // возвращает валюту
  // где в коде или полном названии валюты
  // содержится входные данные
  function getItems(input) {
    let result = [];
    let len = items.length;
    for (let q = 0; q < len; q++) {
      let keyValue = items[q];
      let currAbbreviature = keyValue[0].toLowerCase();
      let currFullName = keyValue[1].toLowerCase();
      if (
        currFullName.indexOf(input) != -1 ||
        currAbbreviature.indexOf(input) != -1
      ) {
        result.push(keyValue);
      }
    }
    return result;
  }

  // вставляет в поле ввода код валюты
  // прячет шторку с валютами
  // обновляет наполнение шторки
  // активирует поле ввода
  function setCurrencyCode(input, currencyType, isFirst) {
    input.value = currencyType;
    showDdon = "";
    refreshItems(isFirst);
    setActive(input);
  }

  // обновляет первую или вторую
  // шторки валюты значением при инициализации
  function refreshItems(isFirst) {
    if (isFirst) {
      firstCurrencies = Object.entries(currencieTypes);
    } else {
      secondCurrencies = Object.entries(currencieTypes);
    }
  }

  // отправляет запрос на конвертацию валюты
  // уменьшает частые запросы
  // после каждого клика или ввода символа
  function convert() {
    if (!isPending) {
      isPending = true;
      setTimeout(() => {
        let values = getValues();
        console.log(values);
        if (values != null) {
          lastValues = values;
          let api = `https://v6.exchangerate-api.com/v6/${key}/pair/${values.first}/${values.second}/${values.amount}`;

          fetch(api)
            .then((response) => response.json())
            .then((data) => {
              console.log(data);
              setResult(data.conversion_result);
              isPending = false;
            })
            .catch((error) => {
              console.log(error);
              isPending = false;
            });
        } else {
          isPending = false;
        }
      }, 1000);
    }
  }

  //вставляет в поле ввода результат конвертации
  function setResult(conversionResult) {
    console.log(conversionResult);
    if (conversionResult != undefined && conversionResult != null) {
      if (isFirst) {
        secondAmountInput.value = conversionResult;
        setActive(secondAmountInput);
      } else {
        firstAmountInput.value = conversionResult;
        setActive(firstAmountInput);
      }
    }
  }

  // сохраняет сумму валюты
  // определает в какое поле было введено последныз раз значение
  // что бы определить порядок кодов валюты в запросе к АПИ
  function setCurrencyAmount(isFirstInput, eve) {
    isFirst = isFirstInput;
    amount = eve.target.value;
    setActive(eve.target);
  }

  // активирует представление поля ввода
  function setActive(ele) {
    if (!ele.classList.contains("input--active")) {
      ele.classList.add("input--active");
    }
  }

  // деактивирует представление поля ввода
  function setInactive(ele) {
    ele.classList.remove("input--active");
  }

  // проверяет произошли ли изменения в полях ввода
  // сверяя с предыдущими значениями
  function changesDetected(newValues) {
    return (
      newValues.first != lastValues.first ||
      newValues.second != lastValues.second ||
      newValues.amount != lastValues.amount
    );
  }

  // возвращает параметры для запроса
  // первый код второй код и сумма валюты
  // если все поля корректно введены
  // и ввод не равен предыдущему вводу
  function getValues() {
    if (isInputsValid()) {
      var newValues = null;
      if (isFirst) {
        newValues = {
          first: firstInput.value,
          second: secondInput.value,
          amount: amount,
        };
      } else {
        newValues = {
          first: secondInput.value,
          second: firstInput.value,
          amount: amount,
        };
      }
      if (changesDetected(newValues)) {
        return newValues;
      }
    }
    return null;
  }

  // проверяет корректность полей ввода
  function isInputsValid() {
    let regex = new RegExp(/^\d+(\.\d+)?$/);
    let firstCurr = firstInput.value;
    let secondCurr = secondInput.value;
    if (
      regex.test(amount) &&
      currenciesShorts.indexOf(firstCurr) != -1 &&
      currenciesShorts.indexOf(secondCurr) != -1
    ) {
      return true;
    } else {
      return false;
    }
  }

  // прячет шторку если пользователь кликает по дому
  // деактивирует представление полей ввода если они пустые
  function listen() {
    if (browser) {
      setTimeout(() => {
        var inputs = [
          firstInput,
          secondInput,
          firstAmountInput,
          secondAmountInput,
        ];
        document.onclick = (e) => {
          if (e.target != firstInput && e.target != secondInput) {
            showDdon = "";
          }
          inputs.forEach((input) => {
            if (input.value == "") {
              setInactive(input);
            }
          });
        };
      }, 1);
    }
  }

  listen();
</script>

<!-- svelte-ignore a11y-no-static-element-interactions -->
<div
  class="input-container"
  on:click={() => convert()}
  on:keyup={() => convert()}
>
  <div class="currency-input">
    <input
      class="currency-input__input currency-input__input--left input nostyle-input"
      placeholder="curr"
      type="text"
      bind:this={firstInput}
      on:input={() => currencyNameOrCodeInput(firstInput, true)}
      on:click={() => {
        showDdon = "first";
      }}
    />
    <input
      class="currency-input__input currency-input__input--right input nostyle-input"
      placeholder="amount"
      type="text"
      bind:this={firstAmountInput}
      on:input={(eve) => setCurrencyAmount(true, eve)}
    />
    {#if showDdon == "first"}
      <ul class="currency-input__ddown nostyle-list">
        {#each Object.entries(firstCurrencies) as [idx, currency]}
          <li class="currency-input__ddown-item">
            <p on:click={() => setCurrencyCode(firstInput, currency[0], true)}>
              {currency[0]}
            </p>
          </li>
        {/each}
      </ul>
    {/if}
  </div>

  <div class="currency-input">
    <input
      class="currency-input__input currency-input__input--left input nostyle-input"
      type="text"
      placeholder="curr"
      bind:this={secondInput}
      on:input={() => currencyNameOrCodeInput(secondInput, false)}
      on:click={() => {
        showDdon = "second";
      }}
    />
    <input
      class="currency-input__input currency-input__input--right input nostyle-input"
      type="text"
      placeholder="amount"
      bind:this={secondAmountInput}
      on:input={(eve) => setCurrencyAmount(false, eve)}
    />
    {#if showDdon == "second"}
      <ul class="currency-input__ddown nostyle-list">
        {#each Object.entries(secondCurrencies) as [idx, currency]}
          <li
            class="currency-input__ddown-item"
            on:click={() => setCurrencyCode(secondInput, currency[0], false)}
          >
            <p>{currency[0]}</p>
          </li>
        {/each}
      </ul>
    {/if}
  </div>
</div>

<style lang="scss">
  @import "../../styles/variables.scss";

  .input-container {
    display: flex;
    flex-direction: column;
    justify-content: space-around;
    height: 100px;

    .currency-input {
      position: relative;
      display: flex;
      flex-direction: row;
      justify-content: center;

      &__input {
        padding: 5px 0;

        &--left {
          text-transform: uppercase;
          border-top-left-radius: 20px;
          border-bottom-left-radius: 20px;
          width: 80px;
        }

        &--right {
          border-left: 5px solid $green;
          border-top-right-radius: 20px;
          border-bottom-right-radius: 20px;
          width: 180px;
        }
      }

      &__ddown {
        position: absolute;
        width: 80px;
        left: 68px;
        top: 35px;
        background-color: $gray-saturate;
        border-radius: 20px;
        font-size: 12px;
        color: $white-transparent;
        text-align: center;
        max-height: 70px;
        overflow-y: auto;
        z-index: 10;
        cursor: pointer;
      }

      &__ddown-item {
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        max-height: 25px;
        transition: 0.15s;

        &:hover {
          color: $white-transparent;
          background-color: $red;
        }
      }
    }
  }
</style>
