<script setup>
import { ref } from 'vue';
import ContactForm from '@/components/ContactForm.vue';
import contactsService from '@/services/contacts.service';

const contact = ref({
    name: '',
    email: '',
    address: '',
    phone: '',
    favorite: 0,
    avatar: null,
});
const message = ref('')

async function addContact(contact) {
    try {
        await contactsService.createContact(contact)
        message.value = 'Liên hệ được thêm thành công!';
    }
    catch {
        message.value = 'Lỗi khi thêm liên hệ';
    }
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