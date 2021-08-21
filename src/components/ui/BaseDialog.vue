<template>
  <teleport to="body">
    <div v-if="show" @click="tryClose" class="backdrop"></div>
    <transition name="dialog">
        <dialog open v-if="show">
          <header>
            <slot name="header">
              <h2>{{ title }}</h2>
              <div class="close-button"><span @click="exitButton">+</span></div>
            </slot>
          </header>
          <section>
            <slot></slot>
          </section>
          <menu>
            <slot name="actions">
              <base-button @click="tryClose">{{ button }}</base-button>
            </slot>
          </menu>
        </dialog>
    </transition>
  </teleport>
</template>

<script>
export default {
  props: {
    show: {
      type: Boolean,
      required: true,
    },
    title: {
      type: String,
      required: false,
    },
    fixed: {
      type: Boolean,
      required: false,
      default: false,
    },
    button: {
      type: String,
      required: false,
      default: "Close"
    }
  },
  emits: ['close', 'exit-button'],
  methods: {
    tryClose() {
      if (this.fixed) {
        return;
      }
      this.$emit('close');
    },
    exitButton() {
      if (this.fixed) {
        return;
      }
      this.$emit('exit-button');
    }
  },
};
</script>

<style scoped>
.close-button {
  position: relative;
  left: 46%;
  top: -2.25rem;
  font-size: 3rem;
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: -4rem;
  cursor: pointer;
  -webkit-transform: rotate(45deg);
  -moz-transform: rotate(45deg);
  transform: rotate(45deg);
  background-color: #2e2e2e;
}

.close-button:hover {
  background-color: #f16565;
  color: #2e2e2e;
}

.backdrop {
  position: fixed;
  top: 0;
  left: 0;
  height: 100vh;
  width: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  z-index: 10;
}

dialog {
  position: fixed;
  top: 20vh;
  left: 10%;
  width: 80%;
  z-index: 100;
  border-radius: 12px;
  border: none;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.26);
  padding: 0;
  margin: 0;
  overflow: hidden;
  background-color: white;
}

header {
  background-color: #c95454;
  color: white;
  width: 100%;
  padding: 1rem;
}

header h2 {
  margin: 0;
}

section {
  padding: 1rem;
}

menu {
  padding: 1rem;
  display: flex;
  justify-content: flex-end;
  margin: 0;
}

.dialog-enter-from,
.dialog-leave-to {
  opacity: 0;
  transform: scale(0.8);
}

.dialog-enter-active {
  transition: all 0.4s ease-in;
}

.dialog-leave-active {
  transition: all 0.4s ease-out;
}

.dialog-enter-to,
.dialog-leave-from {
  opacity: 1;
  transform: scale(1);
}

@media (min-width: 768px) {
  dialog {
    left: calc(50% - 20rem);
    width: 40rem;
  }
}
</style>