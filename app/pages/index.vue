<script setup lang="js">
import {getPaginationRowModel} from '@tanstack/vue-table'
import {h} from 'vue'
import ViolationActionFormModal from "~/components/ViolationActionFormModal.vue";

const table = useTemplateRef('table')
const overlay = useOverlay()

const modal = overlay.create(ViolationActionFormModal, {
  props: {
    violation: null
  },
})

async function open(violation) {

  const actionTaken = await modal.open({
    violation: violation
  })

  if (actionTaken){
    violations.value = violations.value.map(item =>
        actionTaken.violation_id == item.id ? {...item, actionTaken: actionTaken} : item
    )
  }
}

const { data: violations, status } = await useFetch('https://cms.klanlogistics.com:8443/api/wylon-apis/protected?passcode=dataView123', {
  key: 'table-violations',
  transform: (data) => {
    return (
        data?.data?.map((violation) => ({
          id: violation.id,
          ...violation.attributes,
        })) || []
    )
  },
  lazy: true
})

const { data: actionsTaken, status: actionsTakenStatus } = await useFetch('http://127.0.0.1:8000'+'/api/violation-actions', {
  key: 'table-actions-taken',
  transform: (data) => {
    return data?.data|| []
  },
  lazy: true
})

const violationsWithActions = computed(()=>{
  return violations.value.map((violation) => {
    return {
      actionTaken: actionsTaken.value?.find((action) => action.violation_id == violation.id),
      ...violation,
    }
  })
})

const columns = [
  {accessorKey: 'vcode', header: 'VCODE'},
  {accessorKey: 'violation', header: 'Violation',},
  {
    accessorKey: 'beginningtime',
    header: 'Beginning Time',
  },
  {
    accessorKey: 'initiallocation',
    header: 'Initial Location',
  },
  {
    accessorKey: 'endtime',
    header: 'End Time',
  },
  {
    accessorKey: 'finallocation',
    header: 'Final Location',
  },
  {
    accessorKey: 'value',
    header: 'Value',
  },
  {
    accessorKey: 'averagespeed',
    header: 'Final Speed',
  },
  {
    accessorKey: 'count',
    header: 'Count',
  },{
    accessorKey: 'createdAt',
    header: 'Created At',
  },{
    accessorKey: 'updatedAt',
    header: 'Updated At',
  },{
    accessorKey: 'publishedAt',
    header: 'Published At',
  },
  {
    accessorKey: 'actionTaken',
    header: 'Actions Taken',
    cell: ({ row }) => {
      return h('div', { class: 'font-medium' }, row.getValue('actionTaken')?.action_taken)
    }
  },
  {
    id: 'action',
    header: 'Action'
  }
]

const pagination = ref({
  pageIndex: 0,
  pageSize: 10
})
</script>

<template>

  <div class="w-full space-y-4 pb-4">
    <UTable
        ref="table"
        v-model:pagination="pagination"
        :data="violationsWithActions"
        :columns="columns"
        :pagination-options="{
        getPaginationRowModel: getPaginationRowModel()
      }"
        class="flex-1"
    >
      <template #action-cell="{ row }">
        <UButton label="Action" color="neutral" variant="subtle" @click="open(row.original)" />
      </template>
    </UTable>

    <div class="flex justify-center border-t border-(--ui-border) pt-4">
      <UPagination
          :default-page="(table?.tableApi?.getState().pagination.pageIndex || 0) + 1"
          :items-per-page="table?.tableApi?.getState().pagination.pageSize"
          :total="table?.tableApi?.getFilteredRowModel().rows.length"
          @update:page="(p) => table?.tableApi?.setPageIndex(p - 1)"
      />
    </div>
  </div>
</template>