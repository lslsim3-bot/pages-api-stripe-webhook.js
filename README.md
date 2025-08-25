import { buffer } from 'micro';
import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY);

export const config = {
  api: {
    bodyParser: false,
  },
};

export default async function handler(req, res) {
  if (req.method === 'POST') {
    const buf = await buffer(req);
    const sig = req.headers['stripe-signature'];

    let event;

    try {
      event = stripe.webhooks.constructEvent(
        buf,
        sig,
        process.env.STRIPE_WEBHOOK_SECRET
      );
    } catch (err) {
      console.error('❌ Error verifying Stripe webhook:', err.message);
      return res.status(400).send(`Webhook Error: ${err.message}`);
    }

    // 🎯 Handle event types
    if (event.type === 'checkout.session.completed') {
      const session = event.data.object;
      console.log('✅ Payment successful for:', session.customer_email);
      // 👉 Here you can give Boost Pack credits in Supabase
    }

    res.json({ received: true });
  } else {
    res.setHeader('Allow', 'POST');
    res.status(405).end('Method Not Allowed');
  }
}
