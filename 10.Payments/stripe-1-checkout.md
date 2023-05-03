# Stripe Payments

## Installation

```
npm i stripe
```

## Inclusion

```js
import Stripe as 'stripe';
const stripe = new Stripe(process.env.STRIPE_SECRET_KEY);
```

## Create controller (single item)

```js
export getCheckoutSession = catchAsync(async (req, res, next) => {
  // check for item id
  if (!req.params.itemId)
    return //error

  // get the item details and check it exists
  const item = await Item.findById(req.params.id);
  if (!item)
    return //error

  // create a stripe session
  const session = await stripe.checkout.sessions.create({
    mode: 'payment', // subscription also available
    payment_method_types: ['card'],
    success_url: '', // url to redirect to on successful payment
    cancel_url: '', // url to redirect to if payment cancelled
    customer_email: req.user.email, // should be protected route, so user should exist
    client_reference_id: '', // only works in production
    line_items: [
      {
        quantity: 1,
        price_data: {
          currency: 'gbp',
          unit_amount: item.price * 100, // works in cents/pence
          product_data: {
            name: item.name,
            description: item.summary,
            images: ['image addresses']
          },
        },
      },
    ],
  });

  // direct user to checkout page
  // differs depending on whether using API / Frontend / Both
})
```

## Directing to Checkout

### Frontend Only

```js
res.redirect(303, session.url);
```

### API Only

```js
res.status(200).json({
  status: 'success',
  session,
});
```

The user will have access to the checkout link from within the json response under `session.url`.

### Both

If using both options, we use the `API response in the controller`. We then also need to ensure there's a payment button on the front end which we can use to start the payment:

Firstly, ensure that in the template view, the button has the item id in the dataset:

```pug
button#make-payment(data-item-id=`${item.id}`);
```

```js
// payments.js
import axios from 'axios';
import { showAlert } from './alerts';

export const makePayment = async (itemId) = {
  try {
    const session = await axios(`http://endpointAddress/${itemId}`);
    window.location.assign(session.data.session.url);
  } catch (err) {
    showAlert('error', err);
  }
}
```
