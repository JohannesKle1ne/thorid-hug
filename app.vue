<template>
  <div class="flex-down">
    <div
      :class="{
        center: true,
        name: true,
        'error-1': animate,
        'error-2': !animate,
      }"
      id="other"
    >
      {{ otherName }}
    </div>

    <div class="container" id="container">
      <!-- <div
        class="error-1"
        style="border: 1px solid black; width: 30px; height: 40px"
      ></div> -->
      <Transition name="hug-bottom">
        <div v-if="hugToggle" class="hug-bottom">Hug</div>
      </Transition>
      <Transition name="hug-bottom">
        <div v-if="!hugToggle" class="hug-bottom">Hug</div>
      </Transition>
      <Transition name="hug-top">
        <div v-if="huggedToggle" class="hug-top">Hug</div>
      </Transition>
      <Transition name="hug-top">
        <div v-if="!huggedToggle" class="hug-top">Hug</div>
      </Transition>
      <Transition name="hug-success">
        <div v-if="successfulHugging" class="hug-success">
          You hugged each other!
        </div>
      </Transition>
      <Transition name="hug-success">
        <div v-if="!successfulHugging" class="hug-success">
          You hugged each other!
        </div>
      </Transition>

      <!-- <div class="hug-bottom">Hug</div> -->
    </div>
    <div class="center name">{{ myName }}</div>
    <div class="center streak">Streak: {{ streak }}</div>

    <div class="center hug-button" @click="hug">Hug</div>
  </div>
</template>
<script lang="ts">
import { ref, onMounted, onUnmounted } from "vue";

export default defineComponent({
  name: "Counter",

  setup() {
    const hugToggle = ref<boolean>(false);
    const huggedToggle = ref<boolean>(false);
    const animate = ref<boolean>(false);
    const successfulHugging = ref<boolean>(false);
    const socket = ref<WebSocket | null>(null);
    const myName = ref<string>("");
    const streak = ref<number>(0);
    const otherName: ComputedRef<string> = computed((): string => {
      if (myName.value === "Orid") {
        return "Ohannes";
      }
      if (myName.value === "Ohannes") {
        return "Orid";
      }
      return "";
    });

    const isNumeric = (str: string) => {
      if (typeof str != "string") return false;
      return !isNaN(parseInt(str));
    };

    const connect = () => {
      socket.value = new WebSocket("ws://shard-server.onrender.com/");

      socket.value.addEventListener("open", () => {
        console.log("Connected");
      });

      socket.value.addEventListener("message", (event) => {
        console.log(event);
        if (event.data == "Orid" || event.data == "Ohannes") {
          myName.value = event.data;
        }
        if (event.data == "Hug") {
          getHugged();
        }
        if (event.data == "SuccessfulHugging") {
          successfulHugging.value = !successfulHugging.value;
        }
        if (isNumeric(event.data)) {
          streak.value = parseInt(event.data);
        }
      });

      socket.value.addEventListener("close", () => {
        console.log("Disconnected");
      });
    };

    const hug = () => {
      console.log("hug pressed");
      socket.value?.send("Hug");
      //hugToggle.value = !hugToggle.value;
      let element = document.createElement("div");
      element.classList.add("heart");
      let element2 = document.createElement("div");
      element2.classList.add("heart-writing");
      element2.innerText = "Hug";
      element.appendChild(element2);
      const container = document.getElementById("container");
      container?.appendChild(element);
      setTimeout(() => {
        container?.removeChild(element);
        animate.value = !animate.value;
      }, 2000);
    };

    const getHugged = () => {
      huggedToggle.value = !huggedToggle.value;
    };

    onMounted(() => {
      connect();
    });

    onUnmounted(() => {
      if (socket) {
        socket.value?.close();
      }
    });

    return {
      hugToggle,
      huggedToggle,
      hug,
      getHugged,
      myName,
      successfulHugging,
      streak,
      otherName,
      animate,
    };
  },
});
</script>

<style>
.wrapper,
html,
body {
  height: 100%;
  margin: 0;
  font-family: "Courier New", Courier, monospace;
  color: white;
}

.hug-button {
  background-color: #f8dda4;
  height: 15vh;
  margin-top: 2vh;
  width: 100%;
  border-radius: 20px;
  width: 80%;
  font-size: 80px;
}

.name {
  background-color: #52414c;
  height: 10vh;
  width: 100%;
  font-size: 40px;
}

.streak {
  background-color: #ba6e6e;
  width: 100%;
  height: 5vh;
  font-size: 20px;
}

.center {
  display: flex;
  align-items: center;
  justify-content: center;
}

.flex-down {
  position: relative;
  display: flex;
  flex-direction: column;
  height: 100%;
  margin: 0;
  align-items: center;
  justify-content: center;
  /*   top: 0px;
  bottom: 0px; */
}

.container {
  position: relative;
  height: 45vh;
  width: 100%;
  overflow: hidden;
  color: black;
}
.hug-top {
  position: absolute;
  top: 100%;
  left: 50%;
  transform: translateX(-50%);
  font-size: 0px;
}

.hug-bottom {
  position: absolute;
  top: 100%;
  left: 50%;
  transform: translateX(-50%);
}

.hug-success {
  position: absolute;
  top: 100%;
  left: 50%;
  transform: translateX(-50%);
}

@keyframes move-down {
  0% {
    top: -50px;
    font-size: 20px;
  }
  100% {
    top: 150px;
    font-size: 20px;
  }
}

@keyframes move-up {
  0% {
    top: 100%;
    font-size: 20px;
  }
  100% {
    top: 0%;
    font-size: 20px;
  }
}

@keyframes blow-up {
  0% {
    top: 50%;
    font-size: 20px;
  }
  100% {
    top: 50%;
    font-size: 60px;
  }
}

.hug-top-enter-active {
  animation: move-down 2s ease-in-out;
}

.hug-bottom-enter-active {
  animation: move-up 2s ease-in-out;
}

.hug-success-enter-active {
  animation: blow-up 2s ease-in-out;
}

.heart-writing {
  position: absolute;
  margin-bottom: 10px;
  z-index: 10;
}

.heart {
  position: absolute;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100px;
  height: 90px;
  margin-top: 10px;
  animation: move-up 2s ease-in-out;
}

.heart::before,
.heart::after {
  content: "";
  position: absolute;
  top: 0;
  width: 52px;
  height: 80px;
  border-radius: 50px 50px 0 0;
  background: #ba6e6e;
}

.heart::before {
  left: 50px;
  transform: rotate(-45deg);
  transform-origin: 0 100%;
}

.heart::after {
  left: 0;
  transform: rotate(45deg);
  transform-origin: 100% 100%;
}

/* source: https://stackoverflow.com/questions/36962903/javascript-shake-html-element */
@keyframes error-1 {
  0% {
    transform: translateX(0);
  }
  25% {
    transform: translateX(5px);
  }
  50% {
    transform: translateX(-5px);
  }
  75% {
    transform: translateX(5px);
  }
  100% {
    transform: translateX(0);
  }
}

@keyframes error-2 {
  0% {
    transform: translateX(0);
  }
  25% {
    transform: translateX(5px);
  }
  50% {
    transform: translateX(-5px);
  }
  75% {
    transform: translateX(5px);
  }
  100% {
    transform: translateX(0);
  }
}

.error-1 {
  /*  -webkit-animation-name: error-1;
  -webkit-animation-duration: 0.5s;
  -webkit-transform-origin: 50% 50%;
  -webkit-animation-iteration-count: 1; */
  animation: error-1 1s ease-in-out;
}

.error-2 {
  /*   -webkit-animation-name: error-2;
  -webkit-animation-duration: 0.5s;
  -webkit-transform-origin: 50% 50%;
  -webkit-animation-iteration-count: 1; */
  animation: error-2 1s ease-in-out;
}
</style>
