<route lang="yaml">
meta:
  layout: buyer/account/LBDashboard
</route>

<script setup>
import { useLoading } from '~/stores/loading'
import { useUser } from '~/stores/user'
import { toast } from '~/stores/toast'
import AccountRequest from '~/services/account-request'

useHead({
  title: 'buyer | account credit',
})

const { t } = useI18n()
const useToast = toast()
const user = useUser()
const loading = useLoading()
const isCreating = ref(true)

onBeforeMount(() => {
  if (!localStorage.getItem('token'))
    router.push({ path: '/buyer/login' })
})

watch(async() => {
  loading.isLoading = true
  const { data: creditData } = await AccountRequest.getCreditCard()
  user.credit = creditData[0]
  loading.isLoading = false
  user.credit !== '' ? isCreating.value = false : isCreating.value = true
})

const payload = reactive({
  cardholder_name: '',
  expiry_date: '',
  cvv: '',
  registration_address: '',
  postal_code: '',
  card_number: '',
})
const handleCreate = async(e) => {
  e.preventDefault()
  await AccountRequest.createCreditCard(payload)
  useToast.updateToast('sucess', 'your card has been created successfully!', true)
}
const handleUpdate = async(e) => {
  e.preventDefault()
  await AccountRequest.updateCreditCardById(user.credit.id, payload)
  useToast.updateToast('updated', 'Your card has been successfully!', true)
}

const handleDelete = async(e) => {
  e.preventDefault()
  await AccountRequest.deleteCreditCardById(user.credit.id)
  useToast.updateToast('deleted', 'Your card has been deleted!', true)
}

</script>

<template>
  <div class="payment-container border-1 border-solid border-light-700 rounded-md p-5 bg-[#EBF6FC] dark:bg-cool-gray-800">
    <div class="border-b-1 border-b-solid border-b-light-700 py-3 flex justify-between items-center">
      <div class="flex items-center gap-1 font-medium">
        <IBPayment />
        <h3 class="text-2xl">
          {{ t('account.credit-card') }}
        </h3>
      </div>
      <div class="flex item gap-5">
        <div class="text-blue-500 cursor-pointer" @click="isCreating = !isCreating">
          <IBCreate v-if="!isCreating" />
          <IEdit v-if="isCreating" />
        </div>
        <IBDelete class="text-red-400 cursor-pointer" @click="handleDelete" />
      </div>
    </div>
    <div class="credit_infor py-5 text-sm flex justify-around gap-10">
      <div>
        <p>
          {{ t('account.owner-account') }}: <span>{{ user.credit.cardholder_name }}</span>
        </p>
        <p>
          {{ t('account.address-register') }}: <span>{{ user.credit.registration_address }}</span>
        </p>
        <p>
          {{ t('account.zip-postal') }}: <span>{{ user.credit.postal_code }}</span>
        </p>
      </div>
      <div>
        <p>
          {{ t('account.id-card') }}: <span>{{ user.credit.id }}</span>
        </p>
        <p>
          {{ t('account.card-number') }}: <span>{{ user.credit.card_number }}</span>
        </p>
      </div>
      <div>
        <p>
          {{ t('account.expiry-date') }}: <span>{{ user.credit.expiry_date }}</span>
        </p>
        <p>
          {{ t('account.cvv-number') }}: <span>{{ user.credit.cvv }}</span>
        </p>
      </div>
    </div>
    <div class="saved-message py-5 text-gray-400 text-sm font-medium">
      <p v-if="user.payment !==''">
        You have 1 payment method
      </p>
      <p v-else>
        You not't saved your Payment method yet.
      </p>
    </div>

    <form>
      <div>
        <label>{{ t('account.card-name') }}</label>
        <!-- <input v-if="user.credit.cardholder_name" :value="user.credit.cardholder_name" type="text" required> -->
        <input v-model="payload.cardholder_name" type="text" required>
      </div>
      <div>
        <label>{{ t('account.card-number') }} <span>(16 digits)</span></label>
        <input v-model="payload.card_number" type="text" required>
      </div>
      <div>
        <label>{{ t('account.expiry-date') }}</label>
        <input v-model="payload.expiry_date " type="text" required>
      </div>
      <div>
        <label>Cvv <span>(3 digits)</span></label>
        <input v-model="payload.cvv" type="text" required>
      </div>
      <div>
        <label>{{ t('account.registration-address') }}</label>
        <input v-model="payload.registration_address" type="text" required>
      </div>
      <div>
        <label>{{ t('account.postal-zip-code') }}<span>(5 digits)</span></label>
        <input v-model="payload.postal_code" type="text" required>
      </div>
      <div class="pt-5 flex justify-end">
        <button v-if="!isCreating" type="submit" class="btn bg-black  duration-200 flex items-center gap-1 shadow-md shadow-gray-300 font-medium" @click="handleUpdate">
          <ISave />{{ t('account.update-card') }}
        </button>
        <button v-if="isCreating" type="submit" class="btn bg-black hover:bg-[#F33535] duration-200 flex items-center gap-1 shadow-md shadow-gray-300 font-medium" @click="handleCreate">
          <ISave />{{ t('account.create-card') }}
        </button>
      </div>
    </form>
  </div>
</template>

<style scoped>
.credit_infor div p{
  margin: 0.5rem;
}
.credit_infor div p span{
  font-size: 1em;
  font-weight: 500;
  color: #ff3700d6;
}
input{
  width: 80%;
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
