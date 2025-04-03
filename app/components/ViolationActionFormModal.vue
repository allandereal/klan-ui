<script setup lang="js">

const props = defineProps({
  violation: {
    type: Object,
    default: () => ({})
  }
})

const emit  = defineEmits(['close', 'actionTaken'])
const form = useTemplateRef('actionForm')
const config = useRuntimeConfig()
const toast = useToast()

const action = reactive({
  record_date: undefined,
  action_date: undefined,
  action_taken: undefined,
  other_remarks: undefined
})

async function onSubmit() {
  form.value.submit()

  if (form.value.errors.length ===  0) {
    await $fetch(`${config.public.appApiBase}/violation-actions`, {
      method: 'POST',
      body: {
        violation_id: props.violation.id,
        ...action
      },
      onResponse: ({response}) => {
        if(response.status === 201){
          emit('close', response._data);
          toast.add({ title: 'Success', description: 'Action has been take!.', color: 'success' })
        }
      }
    })
  }
}

const validate = (item) => {
  const errors = []

  Object.keys(action).filter(key => key !== 'other_remarks').forEach((key) => {
    if (!item[key]) errors.push({ name: key, message: 'Required' })
  })

  return errors
}

async function onError(event) {
  if (event?.errors?.[0]?.id) {
    const element = document.getElementById(event.errors[0].id)
    element?.focus()
    element?.scrollIntoView({ behavior: 'smooth', block: 'center' })
  }
}
</script>

<template>
  <UModal
      :close="{ onClick: () => emit('close', {}) }"
      :title="`Add Action for violation: (${violation.vcode} : ${violation.violation})`"
  >
    <template #body>
      <UForm ref="actionForm" :validate="validate" :state="action" class="space-y-4" @error="onError">
        <UFormField label="Record Date" name="record_date">
          <UInput v-model="action.record_date" type="datetime-local" class="w-full" required />
        </UFormField>

        <UFormField label="Action Date" name="action_date">
          <UInput v-model="action.action_date" type="datetime-local" class="w-full" required />
        </UFormField>

        <UFormField label="Action taken" name="action_taken">
          <UTextarea v-model="action.action_taken" :rows="2" class="w-full" required />
        </UFormField>

        <UFormField label="Other Remarks" name="other_remarks">
          <UTextarea v-model="action.other_remarks" :rows="4" class="w-full" required />
        </UFormField>
      </UForm>
    </template>
    <template #footer>
        <UButton label="Submit" @click="onSubmit" />
    </template>
  </UModal>
</template>
