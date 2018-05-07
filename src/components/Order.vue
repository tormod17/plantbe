<template>
  <div class="container">
    <div class="columns is-vcentered">
      <div class='column is-6'>
        <img src="https://cdn.scotch.io/2842/b7yhhuUPSGO1fEkMHD6P_sticks.jpeg" alt="A bundle of sticks">
      </div>
      <div class='column is-6'>
        <div v-if="!submitted" class="payment">
          <div class='field'>
              <label>Name</label>
              <div class="control has-icons-left has-icons-right">
                  <input 
                    class="input" 
                    type="text" 
                    placeholder="First and Last"
                    v-model='name'
                  >
                  <span class="icon is-small is-left">
                    <i class="fa fa-user"></i>
                  </span>
              </div>
          </div>
          <div class='field'>
              <label for="email">Email</label>
              <div class="control has-icons-left has-icons-right">
                  <input 
                    class="input" 
                    type="email"
                    v-model='stripeEmail' 
                    placeholder="name@example.com"
                    id="email"
                  >
                  <span class="icon is-small is-left">
                    <i class="fa fa-at"></i>
                  </span>
              </div>
          </div>
          <label for="card">Credit Card</label>
          <p>Test using this credit card: <span class="cc-number">4242 4242 4242 4242</span>, and enter any 5 digits for the zip code</p>
          <div class='field'>
            <card 
              class='stripe-card'
              id="card"
              :class='{ complete }'
              stripe='pk_test_kF7bTdtzSjUxu3xnKlrFfkPt'
              :options='stripeOptions'
              @change='complete = $event.complete'
            />
          </div>
          <button 
            class='pay-with-stripe button is-primary is-large '
            @click='pay'
            :disabled='!complete || !stripeEmail'
          >
            Pay with credit card
          </button>
        </div>

        <div v-else class="statussubmit">
         
          <div v-if="status === 'failure'">
            <h3>Oh No!</h3>
            <p>{{ message || 'Something went wrong!'}} </p>
            <button @click="clearCart">Please try again</button>
          </div>
          <div v-else class="loadcontain">
            <h4 class="notification">Please hold, we're filling up your cart with goodies</h4>
            <intersecting-circles-spinner
              :animation-duration="1200"
              :size="70"
              color="#00d1b2"
            />
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { Card, createToken } from 'vue-stripe-elements-plus';
import { IntersectingCirclesSpinner } from 'epic-spinners'
import axios from 'axios';

export default {
  components: {
    Card,
    IntersectingCirclesSpinner,
  },
  props: {
    total: {
      type: [Number, String],
      default: '20.00'
    },
    success: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      submitted: false,
      complete: false,
      status: '',
      response: '',
      message: '',
      stripeOptions: {  
        // you can configure that cc element. I liked the default, but you can
        // see https://stripe.com/docs/stripe.js#element-options for details      
        iconStyle: 'solid',
        value: {
          postalCode: '12345',
        },
        style: {
          base: {
            color: 'blue',
            lineHeight: '2.2em',
            textAlign: 'center',
            fontSize: '16px',
          }
        }
      },
      stripeEmail: 'tormodsmith@gmail.com',
      name: 'Toriamos',
      amount: 50,
    };
  },
  methods: {
    pay() {
      createToken().then(data => {
        this.submitted = true;
        //this is a token we would use for the stripeToken below
        const request = {
          name: this.name,
          email: this.stripeEmail,
          card: data.card,
          amount: this.amount,
          token_from_stripe: data.token.id,
        };
        const code = '';
        
        axios
          .post(
            `${window.endpoint}/charge`,
            request,
            {
              headers: {
                'Content-Type': 'application/json'
              }
            }
          )
          .then(response => {
            const { data } = response
            this.status = data.statusCode;
            if (data.statusCode !== 200 ){
              this.status = 'failure'; 
              throw data.message;             
            };
            this.$router.push({ path: `order-complete/${data.id}` })
          })
          .catch(error => {
            this.status = 'failure';
            this.message = error;
            this.response = 'Error: ' + error;
          });
      });
    },
    clearCart() {
      this.submitted = false;
      this.status = '';
      this.complete = false;
      this.response = '';
    }
  }
};
</script>
<style lang="scss" scoped>
  @import '../css/order.scss';
</style>
