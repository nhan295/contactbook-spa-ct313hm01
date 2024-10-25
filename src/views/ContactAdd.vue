<script setup>
import { ref } from 'vue';
import ContactForm from '@/components/ContactForm.vue';
import contactsService from '@/services/contacts.service';

import { useMutation } from '@tanstack/vue-query';

const contact = ref({
    name: '',
    email: '',
    address: '',
    phone: '',
    favorite: 0,
    avatar: null,
});
const message = ref('')

const mutation = useMutation({
    mutationFn: async (newContact) => {
        await contactsService.createContact(newContact);
    },
    onSuccess: () => {
        message.value = 'Liên hệ được thêm thành công!';
    },
    onError: () => {
        message.value = 'Lỗi khi thêm liên hệ';
    },
});

async function addContact(contact) {
    await mutation.mutateAsync(contact);
}
</script>

<template>
    <div v-if="contact" class="page">
    <h4>Thêm liên hệ</h4>
    <ContactForm
      :contact="contact"
      @submit:contact="addContact"
    />
    <p>{{ message }}</p>
  </div>
</template>