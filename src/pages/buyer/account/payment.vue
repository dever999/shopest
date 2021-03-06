<route lang="yaml">
meta:
  layout: buyer/account/LBDashboard
</route>

<script setup>
import { useUser } from '~/stores/user'
import { toast } from '~/stores/toast'
import AccountRequest from '~/services/account-request'

useHead({
  title: 'buyer | account payment',
})
const { t } = useI18n()
const useToast = toast()
const id = JSON.parse(localStorage.getItem('user')).data.id
const user = useUser()

onBeforeMount(() => {
  if (!localStorage.getItem('token'))
    router.push({ path: '/buyer/login' })
})

const payload = reactive({
  accountholder_name: '',
  identification_number: '',
  bank_name: '',
  bank_branch: '',
  account_number: '',
})

const isCreating = ref(true)
watch(async() => {
  const { data: bankData } = await AccountRequest.getBankAccount()
  user.payment = bankData[0]
  user.payment !== '' ? isCreating.value = false : isCreating.value = true
})

const handleCreate = async(e) => {
  e.preventDefault()
  await AccountRequest.createBankAccount(payload)
  useToast.updateToast('success', 'Payment method has been created successfully!', true)
}
const handleUpdate = async(e) => {
  e.preventDefault()
  await AccountRequest.updateBankAccountById(user.payment.id, payload)
  useToast.updateToast('success', 'Payment method has been updated successfully!', true)
}

const handleDelete = async() => {
  await AccountRequest.deleteBankAccountById(user.payment.id)
  useToast.updateToast('deleted', 'Payment method has been removed!', true)
}

</script>

<template>
  <div class="payment-container border-1 border-solid border-light-700 rounded-md p-5 bg-[#EBF6FC] dark:bg-cool-gray-800">
    <div class="border-b-1 border-b-solid border-b-light-700 py-3 font-medium flex justify-between items-center">
      <div class="flex items-center gap-1 font-medium ">
        <IBPayment />
        <h3 class="text-2xl">
          {{ t('account.payment-method') }}
        </h3>
      </div>
      <div class="flex items-center gap-5">
        <div class="text-blue-500 cursor-pointer" @click="isCreating = !isCreating">
          <IBCreate v-if="!isCreating" />
          <IEdit v-if="isCreating" />
        </div>
        <IBDelete class="text-red-400 cursor-pointer" @click="handleDelete" />
      </div>
    </div>
    <div v-if="user.payment !==''" class="payment_infor py-5 text-sm flex justify-around gap-10">
      <div>
        <p>
          {{ t('account.owner-account') }}: <span>{{ user.payment.accountholder_name }}</span>
        </p>
        <p>
          {{ t('account.identification-number') }}: <span>{{ user.payment.identification_number }}</span>
        </p>
        <p>
          {{ t('account.payment-id') }}: <span>{{ user.payment.id }}</span>
        </p>
      </div>
      <div>
        <p>
          {{ t('account.bank-name') }}: <span>{{ user.payment.bank_name }}</span>
        </p>
        <p>
          {{ t('account.bank-branch') }}: <span>{{ user.payment.bank_branch }}</span>
        </p>
        <p>
          {{ t('account.account-number') }}: <span>{{ user.payment.account_number }}</span>
        </p>
      </div>
    </div>
    <p v-else class="saved-message py-5 text-gray-400 text-sm font-medium">
      {{ t('account.not-payment') }}.
    </p>

    <form>
      <div>
        <label>{{ t('account.full-name') }}</label>
        <input v-model="payload.accountholder_name" type="text" required>
      </div>
      <div>
        <label>{{ t('account.identification-number') }} <span>(12 {{ t('account.digits') }})</span></label>
        <input v-model="payload.identification_number" type="text" required>
      </div>
      <div>
        <label>{{ t('account.bank-name') }}</label>
        <input v-model="payload.bank_name" type="text" required>
      </div>
      <div>
        <label>{{ t('account.bank-branch') }}</label>
        <input v-model="payload.bank_branch" type="text" required>
      </div>
      <div>
        <label>{{ t('account.card-number') }} <span>(9 digits)</span></label>
        <input v-model="payload.account_number" type="text" required>
      </div>
      <div class="pt-5 flex justify-end gap-5">
        <button v-if="!isCreating" type="submit" class="btn bg-black  duration-200 flex items-center gap-1 shadow-md shadow-gray-300 font-medium opacity-60" @click="handleUpdate">
          <ISave />{{ t('account.update-payment') }}
        </button>
        <button v-if="isCreating" type="submit" class="btn bg-black hover:bg-[#F33535] duration-200 flex items-center gap-1 shadow-md shadow-gray-300 font-medium" @click="handleCreate">
          <ISave />{{ t('account.create-payment') }}
        </button>
      </div>
    </form>
  </div>
</template>

<style scoped>
.payment_infor div p{
  margin: 0.5rem;
}
.payment_infor div p span{
  font-size: 1em;
  font-weight: 500;
  color: #ff3700d6;
}
input{
  width: 75%;
  outline: none;
  border: 1px solid rgba(233, 236, 239);
  border-radius: 0.3rem;
  padding: 0.3rem 1rem;
  transition: 0.2s linear;
  font-size: 0.9rem;
}
input:focus{
    box-shadow: 2px 2px 4px rgba(59, 175, 252, 0.8);
}
form > div:not(:last-child){
  padding: 0.5rem 0;
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 2rem;
}
form label span{
  font-size: 0.5em;
}
</style>
