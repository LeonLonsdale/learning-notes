# Stripe Checkout

## Stripe Installation

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
    client_reference_id: '', // put product ID here we can use for DB storage later.
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
};

// index.js

import { makePayment } from './payment';

const payBtn = document.getElementById('make-payment');

if (payBtn) {
  payBtn.addEventListener('click', async e => {
    e.target.textContent = 'Processing...';
    const { itemId } = e.target.dataset;
    makePayment(itemId);
  })
}
```

## Handling success webhook

## Create a Webhook

Head over to Stripes dashboard and create a webhook for the event `checkout.session.completed`. When creating this webhook, we need to provide an endpoint that we can use to build the completion event. Once the webhook is completed, it will provide a secret webhook key we can use to ensure this process is secure.

### Create an Endpoint

Upon payment success, stripe will send a post request to our end point.

The endpoint will need to provide the body in raw format. This means we need to create it before we call `app.use(express.json())` in our `app.js` (or ts).

```js
app.post(
  '/webhook-checkout',
  express.raw({type: 'application/json'}),
  webhookCheckout // controller function for this example
);
```

### Create a Controller for the endpoint

The successful payment will redirect the user to the `success_url` specified in the session used when creating the payment. When redirecting, stripe will set a header of `stripe-signature`. This is used as a security measure, which we can use, along with our webhook secret key to construct the payment event.

```js
export const webhookCheckout = catchAsync(async (req, res, next) => {
  const signature = req.headers['stripe-signature'];

  try {
    event = stripe.webhooks.constructEvent(req.body, signature, process.env.STRIPE_WH_SECRET);
  } catch (err) {
    return res.status(400).send(`Webhook error: ${err.message}`);
    // this error will be sent to stripe, visible from the dashboard
  }

  // register this in our database if necessary
  // event.data.object is the updated session
  if (event.type === 'checkout.session.completed') await createPurchaseCheckout(event.data.object);

  res.status(200).json({received: true});
  // again, this will be sent back to Stripe.
});
```

Once this is done, stripe will redirect to the `success_url` provided earlier.

Create the function to record the purchase to our desired database:

```js
// example using mongoDB parent referencing.

const createPurchaseCheckout = async (session) {
  const user = await User.findOne({ email: session.customer_email });
  const userId = user._id;
  const product = session.client_reference_id;
  const price = session.amount_total / 100;
  await Model.create({ product, user: userId, price });
}
```
