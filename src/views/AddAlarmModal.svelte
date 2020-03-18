<script>
  import { createEventDispatcher } from "svelte";
  import { getNotificationsContext } from "svelte-notifications";
  import { myAlarms, myId } from "../stores";
  import serverApi from "../utils/ServerApi";
  import AddAlarmByList from "./AddAlarmByList.svelte";
  import AddAlarmBySearch from "./AddAlarmBySearch.svelte";
  import Icon from "fa-svelte";
  import { faSearch } from "@fortawesome/free-solid-svg-icons/faSearch";
  import { faList } from "@fortawesome/free-solid-svg-icons/faList";

  export let active;

  let selectedLectureId = null;
  let search = false;
  let addMethodComponent = AddAlarmByList;

  $: disableAddAlarm = selectedLectureId === null;
  $: search, resetModal();

  const switchAddMethod = () => {
    search = !search;
  };

  const dispatch = createEventDispatcher();
  const closeModal = () => dispatch("close");

  const resetModal = () => {
    selectedLectureId = null;
  };

  const onLectureSelected = lectureId => {
    if (selectedLectureId === lectureId) selectedLectureId = null;
    else selectedLectureId = lectureId;
  };

  const { addNotification } = getNotificationsContext();

  const notifyAdd = () => {
    addNotification({
      text: "알람이 등록되었습니다.",
      position: "bottom-right",
      theme: "is-success",
      removeAfter: 4000
    });
  };

  const notifyError = () => {
    addNotification({
      text: "알람 등록에 실패했습니다.",
      position: "bottom-right",
      theme: "is-danger",
      removeAfter: 4000
    });
  };

  const addAlarm = async () => {
    if (disableAddAlarm) return;

    if ($myAlarms.length >= 10) {
      return addNotification({
        text: "알람은 10개까지 등록할 수 있습니다.",
        position: "bottom-right",
        theme: "is-warning",
        removeAfter: 4000
      });
    }

    try {
      const { lectures } = await serverApi.addAlarm($myId, selectedLectureId);
      console.log(lectures);
      myAlarms.set(lectures);

      resetModal();
      closeModal();
      notifyAdd();
    } catch (e) {
      console.error(e);
      notifyError(e.toString());
    }
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
  <div class="modal-background" on:click={closeModal} />

  <div class="modal-card modal-margin animated fadeInUp">
    <header class="modal-card-head">
      <p class="modal-card-title">알람 추가하기</p>
      <button class="button" on:click={switchAddMethod}>
        <Icon icon={search ? faList : faSearch} />

        <span class="ml-1">강의 {search ? '리스트' : '검색'}</span>
      </button>
    </header>

    <svelte:component
      this={search ? AddAlarmBySearch : AddAlarmByList}
      {onLectureSelected}
      {selectedLectureId} />

    <footer class="modal-card-foot">
      <button
        class="card-footer-item button is-success"
        on:click={addAlarm}
        disabled={disableAddAlarm}>
        추가
      </button>
      <button
        class="card-footer-item button"
        on:click={() => (closeModal(), resetModal())}>
        취소
      </button>
    </footer>
  </div>
</div>
