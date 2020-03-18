<script>
  import { createEventDispatcher } from "svelte";
  import { getNotificationsContext } from "svelte-notifications";
  import { myAlarms, myId } from "../stores";
  phoneNumber;
  import serverApi from "../utils/ServerApi";
  import AddAlarmByList from "./AddAlarmByList.svelte";
  import AddAlarmBySearch from "./AddAlarmBySearch.svelte";
  import Icon from "fa-svelte";
  import { faSearch } from "@fortawesome/free-solid-svg-icons/faSearch";
  import { faList } from "@fortawesome/free-solid-svg-icons/faList";

  export let active;

  let phoneNumber = "";
  let phoneErrorMessage = "";
  $: disablePhoneNumberButton =
    !/^010\d{8}$/.test(phoneNumber) && !showCodeInput ? true : null;

  let showCodeInput = false;

  let authCode = "";
  $: disableAuthCodeButton = !/\d{4}/.test(authCode) ? true : null;
  let authErrorMessage = "";

  const checkPhoneNumber = async () => {
    if (disablePhoneNumberButton) return;

    if (!/010\d{8}/.test(phoneNumber)) {
      phoneErrorMessage = "올바른 전화번호를 입력하세요.";
      notifyError(phoneErrorMessage);
      return;
    }

    phoneErrorMessage = "";
    console.log(phoneNumber);

    try {
      const { message, error, lectures } = await serverApi.myAlarms(
        phoneNumber
      );
      console.log(message);

      if (error) {
        throw new Error(message);
      }

      myAlarms.set(lectures);
      myId.set(phoneNumber);
      phoneNumber;
      closeModal();
      return notifySuccess(message);
    } catch (e) {
      console.log(e);

      notifyError(e.toString());
    }

    try {
      const { message, error } = await serverApi.getAuthCode(phoneNumber);
      if (error) {
        throw new Error(message);
      }

      notifySuccess(message);
      showCodeInput = true;
    } catch (e) {
      console.log(e);

      return notifyError(e.toString());
    }
  };

  const checkAuthCode = async () => {
    if (!showCodeInput) return;
    try {
      const { message, error } = await serverApi.checkAuthCode(
        phoneNumber,
        authCode
      );
      if (error) {
        throw new Error(message);
      }

      notifySuccess(message);
      myId.set(phoneNumber);
      closeModal();
    } catch (e) {
      console.log(e);
      notifyError(e.toString());
    }
  };

  const dispatch = createEventDispatcher();
  const closeModal = () => dispatch("close");

  const { addNotification } = getNotificationsContext();

  const notifySuccess = text => {
    addNotification({
      text,
      position: "bottom-right",
      theme: "is-success",
      removeAfter: 4000
    });
  };

  const notifyError = text => {
    addNotification({
      text,
      position: "bottom-right",
      theme: "is-danger",
      removeAfter: 3000
    });
  };
</script>

<style>
  .modal-margin {
    padding: 0 16px;
  }

  .ml-1 {
    margin-left: 0.5rem;
  }
</style>

<div class="modal" class:is-active={active}>
  <div class="modal-background" />

  <div class="modal-card modal-margin animated fadeInUp">
    <header class="modal-card-head">
      <p class="modal-card-title">전화번호 입력</p>
    </header>

    <section class="modal-card-body">

      <div class="field has-addons">
        <div class="control">
          <input
            bind:value={phoneNumber}
            class:is-danger={phoneErrorMessage}
            disabled={showCodeInput || null}
            class="input is-flex"
            type="text"
            placeholder="전화번호를 입력하세요." />
        </div>
        <div class="control">
          <span
            class="button is-primary"
            disabled={disablePhoneNumberButton}
            on:click={checkPhoneNumber}>
            확인
          </span>
        </div>
      </div>

      <div class="subtitle is-6">01000000000 형식으로 입력하세요.</div>

      {#if showCodeInput}
        <hr />
        <div>인증번호 4자리를 입력해주세요.</div>

        <div class="field has-addons">
          <div class="control">
            <input
              bind:value={authCode}
              class:is-danger={authErrorMessage}
              class="input is-flex"
              type="text"
              placeholder="인증번호 입력" />
          </div>
          <div class="control">
            <span
              class="button is-primary"
              disabled={disableAuthCodeButton}
              on:click={checkAuthCode}>
              확인
            </span>
          </div>
        </div>
      {/if}
    </section>

    <footer class="modal-card-foot" />
  </div>
</div>
