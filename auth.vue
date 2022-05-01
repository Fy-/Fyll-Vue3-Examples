<template>
  <div class="" v-if="apiResult">
    <h1 v-if="apiResult.data.title">{{ apiResult.data.title }}</h1>
    <p v-if="apiResult.data.subTitle" class="info">
      {{ apiResult.data.subTitle }}
    </p>

    <div>
      <p class="error" v-if="apiResult.data.error">
        {{ apiResult.data.error }}
      </p>
      <template v-for="field in apiResult.data.fields">
        <button class="authBtn" v-if="field.type == 'button'">
          <img
            :src="field.svg"
            class="w-10 h-10 rounded-full p-2 border"
            :style="`background: ${field.background}`"
          />
          <div class="ml-4">{{ field.label }}</div>
        </button>
        <template v-if="field.type == 'form'">
          <form
            class="border border-gray-400 bg-gray-50 rounded my-2 p-2"
            @submit.prevent="userFlow(true)"
          >
            {{ field.label }}
            <div class="inputGroup" v-for="input in field.fields">
              <label :for="input.id">{{ input.label }}</label>
              <input
                :id="input.id"
                :placeholder="input.placeholder"
                v-model="fieldsInput[input.id]"
                :autocomplete="input.autocomplete"
                :name="input.id"
                :type="input.type"
                :class="
                  fieldsErrors && fieldsErrors[input.id] ? 'inputError' : ''
                "
              />
              <div
                class="inputErrorInfo"
                v-if="fieldsErrors && fieldsErrors[input.id]"
              >
                {{ fieldsErrors[input.id] }}
              </div>
            </div>
            <button type="submit" class="btn-default p-2">
              {{ $t("next") }}
            </button>
          </form>
        </template>
      </template>
    </div>
  </div>
</template>
<script setup>
import { useRest } from "../helpers";
import { useRouter } from "vue-router";
import { ref, onMounted } from "vue";

const rest = useRest();
const router = useRouter();
const fieldsInput = {};
const fieldsErrors = {};
const apiResult = ref(null);
const lastAction = ref();

const userFlow = async (actionRequired = false) => {
  const data = { ...fieldsInput };
  if (actionRequired) data.action = lastAction.value;

  const response = await rest.post("User/flow", data);
  apiResult.value = response.data;
  if (apiResult.value.success) {
    localStorage.setItem("session", apiResult.value.data.uuid);
    localStorage.setItem("authenticated", true);
    lastAction.value = apiResult.value.data.actionRequired;
    rest.defaults.headers.common["Authorization"] = apiResult.value.data.uuid;
    if (apiResult.value.data.done) {
      router.push("/");
      return;
    }
  } else {
    fieldsErrors.value = apiResult.value.data.fields
      ? apiResult.value.data.fields
      : [];
  }
};
onMounted(async () => {
  userFlow();
});
</script>
