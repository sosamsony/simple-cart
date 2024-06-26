<template>
  <div class="receipt">
    <div class="receipt__form">
      <div class="header">
        <h2>Customer details</h2>
      </div>
      <div class="form-item">
        <label>Email</label>
        <input type="email" v-model="email">
      </div>
      <div class="form-item">
        <label>Address</label>
        <textarea type="email" v-model="address" />
      </div>
    </div>
    <div v-if="offline_reference" class="qr-container">
      <p>Type this reference: <strong>{{ offline_reference }}</strong> or scan this QR code to complete payment:</p>
      <img :src="qr" />
    </div>
    <div class="receipt__items">
      <div class="receipt-item">
        <span>Items subtotal</span>
        <span>{{ parseCurrency(subtotal) }}</span>
      </div>
      <div class="receipt-item">
        <span>Delivery fees</span>
        <span>{{ parseCurrency(deliveryFee) }}</span>
      </div>
      <div class="receipt-item">
        <span>VAT (7.5%)</span>
        <span>{{ parseCurrency(vat) }}</span>
      </div>
      <div class="receipt-total">
        <span>Total</span>
        <span>{{ parseCurrency(total) }}</span>
      </div>
    </div>
    <div class="receipt__checkout">
      <button :disabled="!isEmailValid" @click="createCustomer">Place order</button>
    </div>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'

const API_URL = import.meta.env.VITE_BASE_URL

export default {
  name: 'Receipt',
  data () {
    return {
      email: '',
      address: '',
      deliveryFee: 10,
      loading: false,
      offline_reference: '',
      qr: ''
    }
  },
  computed: {
    ...mapGetters('cart', {
      subtotal: 'cartTotalPrice'
    }),
    isEmailValid () {
      const regex = /\S+@\S+\.\S+/ // this is just a simple check

      return this.email && this.email.length > 0 && regex.test(this.email)
    },
    vat () {
      return 0.075 * this.subtotal
    },
    total () {
      return this.subtotal + this.deliveryFee + this.vat
    }
  },
  methods: {
    async createCustomer () {
      await fetch(`${API_URL}/create-customer`, {
        method: 'POST',
        body: JSON.stringify({ email: this.email})
      })
      .then(response => response.json())
      .then(data => {
        this.createInvoice(data.data.customer_code)
      })
      .catch((error) => {
        console.error('Error:', error);
      });
    },
    async createInvoice (customer) {
      const body = { 
        customer: customer, 
        amount: this.nairaToKobo(this.total)
      }

      await fetch(`${API_URL}/create-invoice`, {
        method: 'POST',
        body: JSON.stringify(body)
      })
      .then(response => response.json())
      .then(data => {
        this.offline_reference = data.offline_reference
        this.qr = data.qr
      })
      .catch((error) => {
        console.error('Error:', error);
      });
    },
    resetForm () {
      this.email = ''
      this.address = ''
    },
    nairaToKobo (amount) {
      return (amount * 100).toFixed(0)
    },
    parseCurrency (amount) {
      return new Intl.NumberFormat('en-AE', { style: 'currency', currency: 'AED' }).format(amount)
    }
  }
}


</script>

<style lang="scss" scoped>
$gray: #F2F5F7;

.receipt {
  display: flex;
  flex-direction: column;

  &__form {
    width: 100%;
    margin-bottom: 48px;
  }

  .header {
    margin-bottom: 50px;
  }

  &__items {
    border-top: 1px dashed #ececec;
    padding-top: 30px;
    margin-bottom: 30px;
  }

  &__checkout {
    button {
      width: 100%;
      background: #3bb75e;
      color: white;
      border-radius: 5px;
      border: none;
      font-size: 14px;
      font-weight: 500;
      text-transform: uppercase;
      height: 36px;
      padding-left: 10px;
      padding-right: 10px;
      cursor: pointer;

      &:disabled {
        background-color: rgba(59,183,94, 0.65);
        cursor: default;
      }

      &:focus {
        outline: none;
      }
    }
  }
}

.receipt-item {
  display: flex;
  justify-content: space-between;
  margin-bottom: 8px;

  span:first-child {
    margin-right: 24px;
  }
}

.form-item {
  display: flex;
  flex-direction: column;
  margin-bottom: 24px;

  label {
    margin-bottom: 8px;
    font-weight: 500;
  }

  input, textarea {
    font-size: 14px;
    color: #737575;
    padding: 10px;
    border: 1px solid $gray;
    box-sizing: border-box;
    box-shadow: 0px 1px 1px rgba(0, 0, 0, 0.03);
    border-radius: 6px;

    &:focus {
      border: 1px solid rgba(130, 130, 130, 0.2);
      outline: none;
    }
  }

  textarea {
    height: 80px;
    resize: none;
  }
}

.receipt-total {
  display: flex;
  justify-content: space-between;
  margin-top: 24px;
  font-size: 16px;
  font-weight: 600;

  span:first-child {
    margin-right: 24px;
  }
}

.qr-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 50px;
}
</style>
