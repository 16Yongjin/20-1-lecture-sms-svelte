<script>
  import { onMount } from "svelte";
  import { getNotificationsContext } from "svelte-notifications";
  import { myAlarms, myId } from "../stores";
  import Navbar from "../components/Navbar.svelte";
  import AlarmCard from "../components/AlarmCard.svelte";
  import AddAlarmModal from "./AddAlarmModal.svelte";
  import SignUpModal from "./SignUpModal.svelte";
  import serverApi from "../utils/ServerApi";

  const { addNotification } = getNotificationsContext();

  $: disableAdd = $myAlarms.length >= 10;

  const loadMyAlarms = async () => {
    if (!$myId) return activateSignUp();

    try {
      const { lectures, error, needAuth, message } = await serverApi.myAlarms(
        $myId
      );

      if (error) {
        console.log(message);
        notifyError(message);

        if (needAuth) return activateSignUp();
      }

      myAlarms.set(lectures);
      if ($myAlarms.length === 0) activateAdd();
    } catch (e) {
      console.log(e);
      notifyError(e.toString());
    }
  };

  const alarmUpdate = async () => {
    try {
      if ($myId) {
        const { lectures, error, needAuth, message } = await serverApi.myAlarms(
          $myId
        );

        if (error) {
          return;
        }

        const idMap = lectures
          .map(i => i.id)
          .reduce((acc, v) => ((acc[v] = true), acc), {});
        const completeAlarms = $myAlarms.filter(a => !idMap[a.id]);
        completeAlarms.forEach(a => {
          notifyAlarm(`${a.name} ${a.professor} ${a.time} 자리났어요.`);
        });

        myAlarms.set(lectures);
      }
    } catch (e) {
      console.log(e);
    }

    setTimeout(alarmUpdate, 3000);
  };

  const notifyDelete = () => {
    notifySuccess("알람이 삭제됐습니다.");
  };

  const notifyError = text => {
    addNotification({
      text,
      position: "bottom-right",
      theme: "is-danger",
      removeAfter: 5000
    });
  };

  const notifySuccess = text => {
    addNotification({
      text,
      position: "bottom-right",
      theme: "is-success",
      removeAfter: 5000
    });
  };

  const notifyAlarm = text => {
    addNotification({
      text,
      position: "top-center",
      theme: "is-warning",
      removeAfter: 5000
    });
  };

  onMount(async () => {
    await loadMyAlarms();
    alarmUpdate();
  });

  document.addEventListener("visibilitychange", () => {
    if (document.visibilityState === "visible") loadMyAlarms();
  });

  let activeAdd = false;
  let activeSignUp = false;

  const activateAdd = () => {
    activeAdd = true;
  };

  const closeAdd = () => {
    activeAdd = false;
  };

  const activateSignUp = () => {
    activeSignUp = true;
  };

  const closeSignUp = () => {
    activeSignUp = false;
  };

  const onAlarmDelete = async e => {
    const lectureId = e.detail.id;
    const { lectures } = await serverApi.deleteAlarm($myId, lectureId);
    myAlarms.set(lectures);
    notifyDelete();
  };
</script>

<style>
  .container-offset {
    padding-top: 52px;
    margin: 16px;
  }
</style>

<Navbar onClickAdd={activateAdd} {disableAdd} />

<div class="container is-fluid container-offset">
  <div class="title is-4">내가 등록한 알람 {$myAlarms.length} / 10개</div>
  <div class="columns is-tablet is-multiline">
    {#each $myAlarms as alarm}
      <AlarmCard {alarm} on:delete={onAlarmDelete} />
    {/each}
  </div>
  <AddAlarmModal active={activeAdd} on:close={closeAdd} />
  <SignUpModal active={activeSignUp} on:close={closeSignUp} />

</div>
