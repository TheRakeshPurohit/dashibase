<template>
  <div class="sm:rounded-lg border border-[#EAEAEA] dark:border-[#2F2F2F] shadow overflow-y-hidden w-full">
    <table class="w-full transition text-primary dark:text-primary-dark overflow-x-auto">
      <thead class="drop-shadow dark:border-b-2 dark:border-b-neutral-750">
        <tr class="transition bg-table-hover dark:bg-table-hover-dark">
          <th class="hidden sm:table-cell px-1 py-2 text-center text-xs font-medium uppercase tracking-wider w-[1rem] text-tertiary dark:text-tertiary-dark group">
            <span v-if="!selected.length" :class="props.readonly ? '' : 'group-hover:hidden'">#</span>
            <input v-if="!selected.length" type="checkbox"
              class="hidden cursor-pointer focus:outline-none focus:ring-0 focus:ring-offset-0 focus:border-0 h-4 w-4 rounded text-neutral-700 border-neutral-300 dark:bg-neutral-900 dark:border-neutral-600"
              @click="selectAll" :checked="selected.length === items.length" :class="props.readonly ? '' : 'group-hover:inline-block'" />
            <input v-if="selected.length" type="checkbox"
              class="cursor-pointer focus:outline-none focus:ring-0 focus:ring-offset-0 focus:border-0 h-4 w-4 rounded text-neutral-700 border-neutral-300 dark:bg-neutral-900 dark:border-neutral-600"
              @click="selectAll" :checked="selected.length === items.length" />
          </th>
          <th v-for="attribute in attributes" :key="attribute.id" class="px-2 py-[0.55rem] text-left text-xs font-medium uppercase tracking-wider min-w-[6rem] whitespace-nowrap">
            <span class="lg:pl-2">{{ attribute.label }}</span>
          </th>
        </tr>
      </thead>
      <tbody class="divide-y transition text-primary divide-neutral-200 bg-surface dark:bg-surface-dark dark:text-primary-dark dark:divide-neutral-750">
        <div v-if="items.length === 0" class="table-row text-sm bg-input-disabled dark:bg-input-disabled-dark text-secondary dark:text-secondary-dark">
          <td></td>
          <td class="px-2 py-2 max-w-0 whitespace-nowrap text-sm">No rows found.</td>
          <td v-for="attribute, i in attributes.slice(1)" :key="i"></td>
        </div>
        <a v-for="item, i in items" :key="i" class="table-row cursor-pointer hover:bg-table-hover dark:hover:bg-table-hover-dark text-primary dark:text-primary-dark"
          @click.exact="event => viewItem(i, event)"
          @click.shift.left.exact="event => selectRow(i, event)">
          <td class="hidden sm:table-cell w-10 px-1 py-2 whitespace-nowrap text-sm text-center transition group text-tertiary dark:text-tertiary-dark">
            <span v-if="!selected.length" :class="props.readonly ? '' : 'group-hover:hidden'">{{ i + 1 + countFrom }}</span>
            <input type="checkbox"
              class="hidden cursor-pointer focus:outline-none focus:ring-0 focus:ring-offset-0 focus:border-0 h-4 w-4 rounded text-neutral-700 border-neutral-300 dark:bg-neutral-900 dark:border-neutral-600"
              :class="props.readonly ? '' : 'group-hover:inline-block'" @click="event => selectRow(i, event)" :checked="selected.includes(i)" />
            <input v-if="selected.length" type="checkbox"
              class="cursor-pointer focus:outline-none focus:ring-0 focus:ring-offset-0 focus:border-0 h-4 w-4 rounded text-neutral-700 border-neutral-300 dark:bg-neutral-900 dark:border-neutral-600" :class="props.readonly ? '' : 'group-hover:hidden'"
              :checked="selected.includes(i)" />
          </td>
          <td v-for="attribute, i in attributes" :key="attribute.id"
            class="px-2 py-1 max-w-0 text-sm overflow-hidden sm:table-cell align-top" :class="i === 0 ? 'font-medium' : ''">
            <div class="flex lg:pl-2 pt-1">
              <Badge v-if="(attribute.type === AttributeType.Enum && item[attribute.id]) || (attribute.type === AttributeType.Bool && ['true', 'false'].includes(String(item[attribute.id])))" :size="'sm'">
                {{ item[attribute.id] }}
              </Badge>
              <div v-else-if="attribute.type === AttributeType.Join && item[attribute.id] && item[attribute.id].constructor === Array" class="pt-[0.05rem] w-full">
                <Badge v-for="i in item[attribute.id]" :title="i" :size="'sm'" class="mr-1 mb-1">
                  {{ i }}
                </Badge>
              </div>
              <div v-else-if="item[attribute.id] && linkify.test(item[attribute.id].toString())" class="truncate" :title="item[attribute.id]">
                <a :href="linkify.find(item[attribute.id])[0].href" target="_blank" class="underline hover:text-neutral-900" @click="$event.stopImmediatePropagation()">{{ item[attribute.id] }}</a>
              </div>
              <!-- <div v-else-if="item[attribute.id] && !isNaN((new Date(item[attribute.id].toString())).getTime())" class="truncate" :title="item[attribute.id]">
                {{ (new Date(item[attribute.id])).toLocaleString() }}
              </div> -->
              <div v-else class="truncate" :title="item[attribute.id]">
                {{ item[attribute.id] }}
              </div>
            </div>
          </td>
        </a>
      </tbody>
    </table>
  </div>
</template>

<script setup lang="ts">
import { ref, PropType } from 'vue'
import * as linkify from 'linkifyjs'
import { Attribute, AttributeType } from '@/utils/config'
import Badge from '../elements/Badge.vue'

const props = defineProps({
  attributes: {
    type: Array as PropType<Attribute[]>,
    default: [],
  },
  items: {
    type: Array as PropType<any[]>,
    default: [],
  },
  countFrom: {
    type: Number,
    default: 0,
  },
  readonly: {
    type: Boolean,
    default: false,
  }
})
const emit = defineEmits(['createItem', 'viewItem', 'deleteItem'])

const selected = ref([] as number[])

function viewItem (itemIdx:number, event:Event) {
  event.preventDefault()
  emit('viewItem', itemIdx)
}

function selectRow (idx:number, event:Event) {
  event.stopPropagation()
  if (props.readonly) return
  if (!selected.value.includes(idx)) selected.value.push(idx)
  else selected.value.splice(selected.value.indexOf(idx), 1)
}

function selectAll () {
  if (props.readonly) return
  if (selected.value.length !== props.items.length) selected.value = [...Array(props.items.length).keys()]
  else selected.value = []
}

defineExpose({
  selected,
})
</script>
