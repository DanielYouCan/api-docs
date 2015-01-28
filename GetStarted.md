# Get Started

In order to use our API, you will need a sandbox account and an API key.

Please note that anything you do on the sandbox account will be in test mode, so
there will be no real money involved and all shipments created and labels
generated will be fake.

This document provides a list of steps to perform in order to get both a sandbox
account and an API key.

## Create a Sandbox account

First of all, you will need a sandbox account:

1. Go to `https://sandbox.parcelbright.com/get-started`
2. Fill in your account details
3. Add a collection address as per instructions on screen

At this point, you have a sandbox account with a company and a user.

You can use this interface to keep track of the shipments created via the API
and also to topup your account when your funds are low.

## Topup your account

In order to topup your account:

1. Login to `https://sandbox.parcelbright.com`
2. Click the "Add Funds" button at the top of the page
3. Select an amount to topup and click 'Add funds now'
4. When redirected to PayPal's sandbox site, use the following credentials:
  - "sandbox@parcelbright.com" as the username
  - "parcelbright" as the password
5. Confirm the payment on PayPal's sandbox account
6. PayPal will redirect you back to the ParcelBright's sandbox site and the
   funds should be on your account

At this point, your account's balance should reflect the topup made.

## Get an API key

The final step is to get an API key to start using the API. When you have
completed the steps above, please email us to support at parcelbright dot com
with the email address used on the sandbox account so that we can generate an
API token. We will then send it to you.

All API tokens can be revoked and re-issued, so plesse let us know if you have
any concerns that your token is being used maliciously.

## Production

When you are happy with your sandbox integration and would like to go live, the
process will be exactly the same, except that:

1. The production site is `https://www.parcelbright.com`
2. The PayPal details are your own details
