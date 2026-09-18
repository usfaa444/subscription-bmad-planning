# System Idea — bf-recurring-payments

Build a platform in Burkina Faso that allows businesses to offer subscriptions and recurring payments to their customers. Example: a business owner might have a loyalty program or a service requiring clients to subscribe to a monthly payment. Right now there is no simple way for anyone in Burkina Faso to set this up; they would have to build the entire system themselves from scratch.

Solve this in two ways:

1) Businesses with websites/apps — API integration. When triggered from a merchant's app, launch a hosted checkout page (Stripe-like). Customer logs in, enters info, configures recurring payment. On completion, callback to the merchant app with status.

2) Businesses without a web app — offline service providers (e.g. weekly/monthly cleaning) register on our web portal, set business details, configure billing terms (frequency, due dates, penalties, payment collection preferences), register clients by phone number. Client gets SMS with instructions to access a portal and choose preferred payment method.

Billing and Payments: platform sends invoices and billing notifications to customers per agreed terms; plan to support automated debits from customer accounts to merchant accounts later.

Payment gateway flexibility is essential — switch/integrate multiple methods easily:
• Current popular in Burkina Faso: Orange Money, Moov Money, Wave
• Future: PI-SPI (BCEAO interoperable payment facilitator across West Africa; public API not out yet — architecture must be ready to adopt ASAP for seamless account-to-account transfers)

Long-term vision: launch Burkina Faso first, but architect from day one for multiple countries and currencies, expandable across West Africa, the continent, and globally.
