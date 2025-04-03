<script setup lang="js">

const props = defineProps({
  violation: Object
})

const emit  = defineEmits(['close', 'actionTaken'])
const form = useTemplateRef('form')

const action = reactive({
  record_date: undefined,
  action_date: undefined,
  action_taken: undefined,
  other_remarks: undefined,
})

async function handleFormSubmit() {
  await $fetch('http://127.0.0.1:8000'+'/api/violation-actions', {
    method: 'POST',
    body: {
      violation_id: props.violation.id,
      ...action
    },
    onResponse: ({response}) => {
      if(response.status === 201){
        emit('close', response._data);
      }
    }
  })
}

const validate = (state) => {
  const errors = []
  if (!state.email) errors.push({ name: 'email', message: 'Required' })
  if (!state.password) errors.push({ name: 'password', message: 'Required' })
  return errors
}

const toast = useToast()
async function onSubmit(event) {
  form.value.submit()
  await handleFormSubmit()
  //toast.add({ title: 'Success', description: 'The form has been submitted.', color: 'success' })
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
      :title="`Add Action for violation: ${violation.vcode}`"
  >
    <template #body>
      <UForm ref="form" :validate="validate" :state="action" class="space-y-4" @submit="onSubmit" @error="onError">
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
