<template>
    <div class="page-checkout">
        <div class="columns is-multiline">
            <div class="column is-12">
                <h1 class="title">Checkout</h1>
            </div>

            <div class="column is-12 box">
                <table class="table is-fullwidth">
                    <thead>
                        <tr>
                            <th>Product</th>
                            <th>Price</th>
                            <th>Quantity</th>
                            <th>Total</th>
                        </tr>
                    </thead>

                    <tbody>
                        <tr v-for="item in cart.items" v-bind:key="item.product.id">
                            <td>{{ item.product.name }}</td>
                            <td>${{ item.product.price }}</td>
                            <td>{{ item.quantity }}</td>
                            <td>${{ getItemTotal(item).toFixed(2) }}</td>
                        </tr>
                    </tbody>

                    <tfoot>
                        <tr>
                            <td colspan="2">Total</td>
                            <td>{{ cartTotalLength }}</td>
                            <td>${{ cartTotalPrice.toFixed(2) }}</td>
                        </tr>
                    </tfoot>
                </table>
            </div>

            <div class="column is-12 box">
                <h2 class="subtitle">Shipping details</h2>

                <p class="has-text-grey mb-4">* All fields are required</p>

                <div class="columns is-multiline">
                    <div class="column is-6">
                        <div class="field">
                            <label>First name*</label>
                            <div class="control">
                                <input type="text" class="input" v-model="first_name">
                            </div>
                        </div>

                        <div class="field">
                            <label>Last name*</label>
                            <div class="control">
                                <input type="text" class="input" v-model="last_name">
                            </div>
                        </div>

                        <div class="field">
                            <label>E-mail*</label>
                            <div class="control">
                                <input type="email" class="input" v-model="email">
                            </div>
                        </div>

                        <div class="field">
                            <label>Phone* </label>
                            <div class="control">
                                <input type="text" class="input" v-model="phone">
                            </div>
                        </div>
                    </div>

                    <div class="column is-6">
                        <div class="field">
                            <label>Address*</label>
                            <div class="control">
                                <input type="text" class="input" v-model="address">
                            </div>
                        </div>

                        <div class="field">
                            <label>Zip code*</label>
                            <div class="control">
                                <input type="text" class="input" v-model="zipcode">
                            </div>
                        </div>

                        <div class="field">
                            <label>Place*</label>
                            <div class="control">
                                <input type="text" class="input" v-model="place">
                            </div>
                        </div>
                    </div>
                </div>

                <div class="notification is-danger mt-4" v-if="errors.length">
                    <p v-for="error in errors" v-bind:key="error">{{ error }}</p>
                </div>

                <hr>

                <div id="card-element" class="mb-5"></div>

                <template v-if="cartTotalLength">
                    <hr>

                    <button class="button is-dark" @click="submitForm">Pay with Stripe</button>
                </template>
            </div>
        </div>
    </div>
</template>

<script>
import axios from 'axios'

export default {
    name: 'Checkout',
    data() {
        return {
            cart: {
                items: []
            },
            stripe: {},
            card: {},
            first_name: '',
            last_name: '',
            email: '',
            phone: '',
            address: '',
            zipcode: '',
            place: '',
            errors: []
        }
    },
    mounted() {
        document.title = 'Checkout | Djackets'

        this.cart = this.$store.state.cart

        if (this.cartTotalLength > 0) {
            this.stripe = Stripe('pk_test_51Nl6ThJAfxjrK543npmEL0MzCG1gJFNnMtw6nIEs3BPaFqwryqRvtJzuoS8sqjJJ1lLOuAzDMz4xWLTvHtIeCFHS00DAH50m4L')
            const elements = this.stripe.elements();
            this.card = elements.create('card', { hidePostalCode: true })

            this.card.mount('#card-element')
        }
    },
    methods: {
        getItemTotal(item) {
            return item.quantity * item.product.price
        },
        submitForm() {
            this.errors = []

            const englishAlphabetPattern = /^[A-Za-z]+$/;
            if (this.first_name === '') {
                this.errors.push('The first name field is missing!')
            } 
            else if (this.first_name.length < 3) {
                this.errors.push('The first name should contain at least three characters.');
            } 
            else if (!englishAlphabetPattern.test(this.first_name)) {
                this.errors.push('The first name should only contain English characters.');
            }

            if (this.last_name === '') {
                this.errors.push('The last name field is missing!')
            }
            else if (this.last_name.length < 3) {
                this.errors.push('The last name should contain at least three characters.');
            } 
            else if (!englishAlphabetPattern.test(this.last_name)) {
                this.errors.push('The last name should only contain English characters.');
            }

            const emailPattern = /^[A-Za-z0-9]+@[A-Za-z0-9]+\.[A-Za-z0-9]+$/;
            if (this.email === '') {
                this.errors.push('The email field is missing!')
            }
            else if (!emailPattern.test(this.email)) {
                this.errors.push('The email is invalid');
            }

            const numericPattern = /^[0-9]+$/;
            if (this.phone === '') {
                this.errors.push('The phone field is missing!')
            }
            else if (!numericPattern.test(this.phone)) {
                this.errors.push('Phone number should contain numbers only');
            }
            else if (this.phone.length !== 11 ) {
                this.errors.push('Phone number should have 11 numerics');
            }

            const validCharacterPattern = /^[a-zA-Z0-9\s\.,#\-]+$/;
            if (this.address === '') {
                this.errors.push('The address field is missing!')
            }
            else if (!validCharacterPattern.test(this.address)) {
                this.errors.push('address should contain valid characters');
            }

            const zipPattern = /^[0-9]+$/;
            if (this.zipcode === '') {
                this.errors.push('The zip code field is missing!')
            }
            else if (!zipPattern.test(this.zipcode)) {
                this.errors.push('Zip Code should contain numbers only');
            }


            if (this.place === '') {
                this.errors.push('The place field is missing!')
            }
            else if (!englishAlphabetPattern.test(this.place)) {
                this.errors.push('Place should contain only English characters.');
            }

            if (!this.errors.length) {
                this.$store.commit('setIsLoading', true)

                this.stripe.createToken(this.card).then(result => {
                    if (result.error) {
                        this.$store.commit('setIsLoading', false)

                        this.errors.push(result.error.message)
                    } else {
                        this.stripeTokenHandler(result.token)
                    }
                })
            }
        },
        async stripeTokenHandler(token) {
            const items = []

            for (let i = 0; i < this.cart.items.length; i++) {
                const item = this.cart.items[i]
                const obj = {
                    product: item.product.id,
                    quantity: item.quantity,
                    price: item.product.price * item.quantity
                }

                items.push(obj)
            }

            const data = {
                'first_name': this.first_name,
                'last_name': this.last_name,
                'email': this.email,
                'address': this.address,
                'zipcode': this.zipcode,
                'place': this.place,
                'phone': this.phone,
                'items': items,
                'stripe_token': token.id
            }

            await axios
                .post('/api/v1/checkout/', data)
                .then(response => {
                    this.$store.commit('clearCart')
                    this.$router.push('/cart/success')
                    console.log(data)
                })
                .catch(error => {
                    this.errors.push('error  404')

                    console.log(error)
                })

            this.$store.commit('setIsLoading', false)
        }
    },
    computed: {
        cartTotalPrice() {
            return this.cart.items.reduce((acc, curVal) => {
                return acc += curVal.product.price * curVal.quantity
            }, 0)
        },
        cartTotalLength() {
            return this.cart.items.reduce((acc, curVal) => {
                return acc += curVal.quantity
            }, 0)
        }
    }
}
</script>
