<script setup lang="ts">
import {
  BIconFlagFill,
  BIconStarFill,
  BIconCheck2,
  BIconFileEarmarkPdf,
  BIconGithub,
} from "bootstrap-icons-vue";
import { computed, PropType } from "vue";

import { getCategorizerString, getPubTypeString } from "@/base/string";
import { PaperEntity } from "@/models/paper-entity";
import { FieldTemplate, ItemField } from "@/renderer/types/data-view";

const props = defineProps({
  item: {
    type: Object as PropType<PaperEntity>,
    required: true,
  },
  fieldTemplates: {
    type: Object as PropType<Map<string, FieldTemplate>>,
    required: true,
  },
  active: {
    type: Boolean,
    default: false,
  },
  striped: {
    type: Boolean,
    required: true,
  },
});

const fields = computed(() => {
  const fields: ItemField[] = [];

  for (const [fieldKey, fieldTemplate] of props.fieldTemplates.entries()) {
    let value = props.item[fieldKey];
    if (value === undefined) {
      continue;
    }

    switch (fieldKey) {
      case "addTime": {
        value = new Date(value).toLocaleString().slice(0, 10);
        break;
      }
      case "tags":
      case "folders": {
        value = getCategorizerString(
          value,
          fieldTemplate.sortBy || "addTime",
          fieldTemplate.sortOrder || "desc"
        );
        break;
      }
      case "pubType": {
        value = getPubTypeString(value);
        break;
      }
      case "mainURL": {
        value = value !== "" && value !== undefined;
        break;
      }
      case "codes": {
        value = value.length > 0;
        break;
      }
      case "supURLs": {
        value = value.length > 0;
        break;
      }
    }

    const field = {
      type: fieldTemplate.type,
      value: value,
      width: fieldTemplate.width,
    };
    fields.push(field);
  }

  return fields;
});
</script>

<template>
  <div
    class="flex w-full py-1 text-xs rounded-md select-none cursor-pointer h-7"
    :class="
      (striped && !active
        ? `bg-neutral-100 dark:bg-neutral-700 dark:bg-opacity-40`
        : ``) +
      (active
        ? `bg-accentlight dark:bg-accentdark dark:bg-opacity-100 text-white`
        : ``)
    "
  >
    <div
      class="my-auto px-2 flex"
      v-for="(field, index) in fields"
      :style="`width: ${field.width}%`"
    >
      <span class="my-auto truncate" v-if="field.type === 'string'">{{
        field.value
      }}</span>
      <span
        class="my-auto truncate"
        v-html="field.value"
        v-else-if="field.type === 'html'"
      ></span>
      <span class="my-auto" v-else-if="field.type === 'flag'">
        <BIconFlagFill
          class="my-auto text-xxs"
          :calss="active ? 'text-white' : ''"
          v-if="field.value"
        />
      </span>
      <span class="my-auto text-xxxs flex" v-else-if="field.type === 'rating'">
        <BIconStarFill v-for="_ in field.value" class="my-auto mr-[1px]" />
      </span>
      <span class="my-auto" v-else-if="field.type === 'boolean'">
        <BIconCheck2
          class="my-auto text-xxs"
          :calss="active ? 'text-white' : ''"
          v-if="field.value"
        />
      </span>
      <span class="my-auto" v-else-if="field.type === 'file'">
        <BIconFileEarmarkPdf
          class="my-auto text-sm"
          :calss="active ? 'text-white' : ''"
          v-if="field.value"
        />
      </span>
      <span class="my-auto" v-else-if="field.type === 'code'">
        <BIconGithub
          class="my-auto text-sm"
          :calss="active ? 'text-white' : ''"
          v-if="field.value"
        />
      </span>
      <span class="my-auto truncate" v-else>
        {{ field.value }}
      </span>
    </div>
  </div>
</template>