<template>
  <div
    class="relative bg-white text-gray-700 border border-gray-500 dark:bg-slate-600 dark:text-gray-300 dark:border-gray-500"
    :class="classes"
  >
    <input
      ref="input"
      type="text"
      autocomplete="off"
      class="input bg-transparent w-full px-3 py-2 top-0 border-0 focus:ring-0 z-10"
      :value="searchText"
      :disabled="disabled"
      @input="searchText = $event.target.value"
      @focus.prevent="openMenu"
      @blur="blurInput"
    />

    <div class="text absolute cursor-text left-1 top-0 p-2" @click="openMenu">{{ inputText }}</div>

    <span class="absolute inset-y-0 right-0 flex items-center pr-2 pointer-events-none">
      <ChevronUpDownIcon class="w-5 h-5 text-gray-400" aria-hidden="true" />
    </span>

    <div
      :class="[
        showMenu ? 'block' : 'hidden',
        'bg-white dark:bg-slate-600 border-gray-400 dark:border-gray-500 border-r border-l border-b absolute max-h-60 min-w-full overflow-x-hidden overflow-y-auto top-full z-50',
        menuClass,
      ]"
      @mousedown.prevent
    >
      <div
        v-for="(option, i) of filteredOptions"
        :key="i"
        :class="[
          'p-2 border-gray-400 border-t dark:border-gray-500 cursor-pointer hover:bg-[rgba(0,0,0,.05)]',
          optionClass,
        ]"
        @click.stop="selectItem(option)"
        @mousedown="mousedownItem"
      >
        {{ option.text }}
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed, ref, toRefs } from "vue";
import { ChevronUpDownIcon } from "@heroicons/vue/24/outline";

const props = defineProps({
  options: { type: Array, required: true },
  modelValue: { type: [String, Object], default: null },
  placeholder: { type: String, default: "" },
  classes: { type: String, default: "" },
  menuClass: { type: String, default: "" },
  optionClass: { type: String, default: "" },
  disabled: { type: Boolean, default: false },
});

const emit = defineEmits(["update:modelValue"]);

const { options, modelValue, placeholder } = toRefs(props);

const searchText = ref("");
const showMenu = ref(false);
const mousedownState = ref(false);

const input = ref(null);

const escapedRegExp = (str) => {
  return new RegExp(str.replace(/[.*+?^${}()|[\]\\]/g, "\\$&"), "i");
};

const filteredOptions = computed(() => {
  if (searchText.value) {
    const regex = escapedRegExp(searchText.value);

    return options.value.filter((o) => regex.test(o.text));
  }

  return options.value;
});

const selectedOption = computed(() =>
  options.value.find((option) => option.value === modelValue.value)
);

const inputText = computed(() => {
  if (searchText.value) {
    return "";
  } else {
    let text = placeholder.value;

    if (selectedOption.value) {
      text = selectedOption.value.text;
    }

    return text;
  }
});

const openMenu = () => {
  if (props.disabled) {
    return;
  }

  input.value.focus();

  showMenu.value = true;
  mousedownState.value = false;
};

const closeMenu = () => {
  showMenu.value = false;
};

const blurInput = () => {
  if (!mousedownState.value) {
    searchText.value = "";
    closeMenu();
  }
};

const selectItem = (option) => {
  searchText.value = "";

  closeMenu();

  if (typeof modelValue.value === "object" && modelValue.value) {
    emit("update:modelValue", option);
  } else {
    emit("update:modelValue", option.value);

    if (option.value === option.text) {
      searchText.value = option.value;
    }
  }

  input.value.blur();
};

const mousedownItem = () => {
  mousedownState.value = true;
};
</script>

<style scoped>
.input:focus + .text {
  color: rgba(115, 115, 115, 0.87);
}
</style>
