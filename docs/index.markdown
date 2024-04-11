---
layout: home
---

# LeadsCalendar Documentation

Welcome to the documentation for LeadsCalendar, a web application designed to facilitate event creation and payment processing. This documentation provides comprehensive guidance on integrating the Google Calendar API, PayPal REST API, and Binance Pay API into the LeadsCalendar project.

## Table of Contents

1. [Introduction](https://www.notion.so/LeadsCalendar-Documentation-55b444f70cd64ab1b22f4f5c4236545c?pvs=21)
2. [Technical Overview](https://www.notion.so/LeadsCalendar-Documentation-55b444f70cd64ab1b22f4f5c4236545c?pvs=21)
3. [API Integration](https://www.notion.so/LeadsCalendar-Documentation-55b444f70cd64ab1b22f4f5c4236545c?pvs=21)
    - [Google Calendar API](https://www.notion.so/LeadsCalendar-Documentation-55b444f70cd64ab1b22f4f5c4236545c?pvs=21)
    - [PayPal REST API](https://www.notion.so/LeadsCalendar-Documentation-55b444f70cd64ab1b22f4f5c4236545c?pvs=21)
    - [Binance Pay API](https://www.notion.so/LeadsCalendar-Documentation-55b444f70cd64ab1b22f4f5c4236545c?pvs=21)
4. [Functionality](https://www.notion.so/LeadsCalendar-Documentation-55b444f70cd64ab1b22f4f5c4236545c?pvs=21)
5. [Documentation Format](https://www.notion.so/LeadsCalendar-Documentation-55b444f70cd64ab1b22f4f5c4236545c?pvs=21)
6. [Deployment](https://www.notion.so/LeadsCalendar-Documentation-55b444f70cd64ab1b22f4f5c4236545c?pvs=21)
7. [Feedback](https://www.notion.so/LeadsCalendar-Documentation-55b444f70cd64ab1b22f4f5c4236545c?pvs=21)

## Introduction

LeadsCalendar is a web application that simplifies the process of event creation by integrating payment processing capabilities. Users can create events on Google Calendar through the LeadsCalendar interface, and upon event creation, they are prompted to make a payment of 1 USD or its equivalent in cryptocurrency. Events are confirmed only after successful payment processing.

## Technical Overview

The LeadsCalendar project utilizes three main APIs:

- **Google Calendar API**: Allows for the integration of Google Calendar functionalities into the LeadsCalendar application.
- **PayPal REST API**: Enables payment processing using PayPal as the payment gateway.
- **Binance Pay API**: Facilitates cryptocurrency payments through Binance Pay integration.

This documentation will guide you through the process of integrating these APIs into the LeadsCalendar project.

## API Integration

### Google Calendar API

The Google Calendar API provides functionality to create, modify, and delete events on Google Calendar programmatically. To integrate this API into the LeadsCalendar project, follow these steps:

1. **Set Up Google Cloud Platform Project**: 
    - Create a project on [Google Cloud Platform](https://console.developers.google.com/apis/dashboard), give it a name “example-app”.
    - After creation, the project becomes available in the list of projects - "example-app".
    - Enable the Google Calendar API (press ”Enable APIs and services” button and search for Google Calendar API).
    - Enter the product page and enable it.
    - After we go to the control panel using this API in our project https://console.developers.google.com/apis/api/calendar-json.googleapis.com/overview?project=YOUR_PROJECT&supportedpurview=project) , where we are told that to use the API we must create credentials.
    - Click “CREATE CREDENTIALS” and fill out a form that determines what type of credentials we need to access the API.
2. **Generate API Key:**
    - Based on the results of the survey, Google proposes to create a private key in JSON format, with which our application will connect to the Google Calendar API.
    - The biggest difficulty for me was the “Role” item. It looks like Google has even more roles than APIs you can connect to.
    - In general, if you select “Project / Owner”, they promise to provide full access to all resources. Click on the "Continue" button and receive the JSON file in the "Downloads" directory on your computer.
    
    The contents of the JSON file are something like this (I cut out the key):
    

```jsx
{
  "type": "service_account",
  ...
  "client_id": "YOUR_CLIENT_ID",
  ...
}
```

- In the API console, the "Habr Demo" application has a new service with the email `<name>.gserviceaccount.com`
1. **Authenticate Users and CRUD Operations**: 
    - To connect to APIs from a nodejs application, Google offers the googleapis library and an overview of its use. The review uses OAuth2 authentication and receiving a list of events from the calendar, but we need key authentication in the appropriate scopes and adding an event to the calendar. So the code looks something like this:

```jsx
const fs = require('fs');
const {google} = require('googleapis');

const CALENDAR_ID = 'client_or_our_calendar_id';
const KEYFILE = 'ExampleApp-4ec17ea5f8b6.json'; // path to JSON with private key been downloaded from Google
const SCOPE_CALENDAR = 'https://www.googleapis.com/auth/calendar'; // authorization scopes
const SCOPE_EVENTS = 'https://www.googleapis.com/auth/calendar.events';

(async function run() {
    // INNER FUNCTIONS
    async function readPrivateKey() {
        const content = fs.readFileSync(KEYFILE);
        return JSON.parse(content.toString());
    }

    async function authenticate(key) {
        const jwtClient = new google.auth.JWT(
            key.client_email,
            null,
            key.private_key,
            [SCOPE_CALENDAR, SCOPE_EVENTS]
        );
        await jwtClient.authorize();
        return jwtClient;
    }

    async function createEvent(auth) {
        const event = {
            'summary': 'Example App Post Demo',
            'description': 'Integration test of nodejs-app with Google Calendar API.',
            'start': {
                'dateTime': '2020-10-20T16:00:00+02:00',
                'timeZone': 'Europe/Riga',
            },
            'end': {
                'dateTime': '2020-10-20T18:00:00+02:00',
                'timeZone': 'Europe/Riga',
            }
        };

        let calendar = google.calendar('v3');
        await calendar.events.insert({
            auth: auth,
            calendarId: CALENDAR_ID,
            resource: event,
        });
    }

    // MAIN
    try {
        const key = await readPrivateKey();
        const auth = await authenticate(key);
        await createEvent(auth);
    } catch (e) {
        console.log('Error: ' + e);
    }
})();
```

- We run the code and receive a response from the Calendar API:

```jsx
{
  ...
  "status": 404,
  "statusText": "Not Found",
  ...
}
```

- We need to go to our calendar or client, and in the settings click “Access for individual users”, then “Add users” and enter the email address of our service, which will create events in the calendar.

For detailed instructions and code samples, refer to the [Google Calendar API documentation](https://developers.google.com/calendar/api/guides/overview).

### PayPal REST API

The PayPal REST API enables seamless payment processing within the LeadsCalendar application. Follow these steps to integrate PayPal into LeadsCalendar:

1. **Set Up PayPal Developer Account**:
    - Go to the [PayPal Developer website](https://developer.paypal.com/).
    - Sign up for a developer account or log in if you already have one.
    - Navigate to the Dashboard and create a new app to obtain API credentials (Client ID and Secret).
2. **Configure Webhooks**:
    - Create a webhook listener endpoint in your application to receive PayPal notifications.
    - Add the webhook URL to your PayPal developer account.
    - Verify and process incoming webhook events.
    
    Example webhook listener in Node.js with Express:
    
    ```jsx
    const express = require('express');
    const bodyParser = require('body-parser');
    const app = express();
    
    app.use(bodyParser.json());
    
    app.post('/webhook', (req, res) => {
        const webhookEvent = req.body;
    
        // Process webhook event
        console.log('Received webhook event:', webhookEvent);
    
        res.status(200).send('Webhook Received');
    });
    
    app.listen(3000, () => {
        console.log('Webhook listener started on port 3000');
    });
    
    ```
    
3. **Implement Payment Flows**:
    - Integrate PayPal Checkout SDK into your frontend to initiate payments.
    - Use the obtained Client ID to authenticate requests.
    
    Example of integrating PayPal Checkout SDK in HTML:
    
    ```html
    <script src="<https://www.paypal.com/sdk/js?client-id=YOUR_CLIENT_ID>"></script>
    <script>
        paypal.Buttons({
            createOrder: function(data, actions) {
                return actions.order.create({
                    purchase_units: [{
                        amount: {
                            value: '10.00' // Amount to charge
                        }
                    }]
                });
            },
            onApprove: function(data, actions) {
                return actions.order.capture().then(function(details) {
                    alert('Transaction completed by ' + details.payer.name.given_name);
                    // Proceed with event creation or any further action
                });
            }
        }).render('#paypal-button-container');
    </script>
    
    ```
    
4. **Handle Payment Responses**:
    - Process the payment response received after the user completes the payment.
    - Confirm event creation or any other desired action upon successful payment.
    
    Example backend code in Node.js to confirm event creation:
    
    ```jsx
    app.post('/paypal/success', (req, res) => {
        const { orderId, payerId } = req.body;
    
        // Verify payment status using orderId and payerId
        // Confirm event creation upon successful payment
        res.status(200).send('Payment successful, event created!');
    });
    
    ```
    

These steps should help you integrate PayPal API into your application. Remember to handle errors and edge cases appropriately in your implementation.

Detailed instructions and code samples can be found in the [PayPal REST API documentation](https://developer.paypal.com/api/rest/).

### Binance Pay API

The Binance Pay API facilitates cryptocurrency payments within the LeadsCalendar application. Follow these steps to integrate Binance Pay:

1. **Create Binance Developer Account**:
    - Go to the [Binance Developer website](https://www.binance.com/en/my/settings/api-management).
    - Sign up for a developer account or log in if you already have one.
    - Create a new API key pair to obtain API credentials (API Key and Secret Key).
2. **Configure Payment Methods**:
    - Determine which cryptocurrencies you want to accept as payment.
    - Set up wallets for each accepted cryptocurrency to receive payments.
3. **Implement Payment Gateway**:
    - Integrate Binance Pay SDK or API into your application to initiate cryptocurrency payments.
    - Use the obtained API Key to authenticate requests.
    
    Example of integrating Binance Pay SDK in HTML:
    
    ```html
    <!-- Include Binance Pay SDK script -->
    <script src="<https://www.binance.com/binance-merchant-web/static/bpm-web-sdk/bpm-web-sdk.js>"></script>
    <script>
        // Initialize Binance Pay
        const binancePay = new BinancePay();
    
        // Set up payment request
        const paymentRequest = {
            totalOrderAmount: {
                currency: 'USD', // Currency of the amount
                amount: '10.00' // Amount to charge
            },
            orderId: 'ORDER123' // Your order ID
        };
    
        // Open Binance Pay checkout flow
        binancePay.showBinancePay(paymentRequest).then((result) => {
            if (result.code === 'SUCCESS') {
                // Payment successful, handle response
                console.log('Payment successful:', result);
                // Proceed with event creation or any further action
            } else {
                // Payment failed, handle error
                console.error('Payment failed:', result);
            }
        }).catch((error) => {
            console.error('Error:', error);
        });
    </script>
    
    ```
    
4. **Handle Payment Responses**:
    - Process the payment response received after the user completes the cryptocurrency payment.
    - Confirm event creation or any other desired action upon successful payment.
    
    Example backend code in Node.js to confirm event creation:
    
    ```jsx
    app.post('/binancepay/success', (req, res) => {
        const { orderId, transactionId } = req.body;
    
        // Verify payment status using orderId and transactionId
        // Confirm event creation upon successful payment
        res.status(200).send('Payment successful, event created!');
    });
    
    ```
    

Refer to the [Binance Pay API documentation](https://developers.binance.com/docs/binance-pay/introduction) for detailed integration instructions and code samples.

## Functionality

LeadsCalendar offers the following functionality:

- **Event Creation**: Users can create events on Google Calendar through the LeadsCalendar interface.
- **Payment Processing**: Upon event creation, users are prompted to make a payment of 1 USD or its equivalent in cryptocurrency.
- **Confirmation**: Events are confirmed only after successful payment processing.

## Documentation Format

This documentation is structured logically to provide clear guidance on integrating the Google Calendar API, PayPal REST API, and Binance Pay API into the LeadsCalendar project. It includes detailed explanations, code samples, and configuration instructions for each API.

## Feedback

We value your feedback! If you have any suggestions, questions, or concerns regarding the LeadsCalendar documentation.

Thank you for using LeadsCalendar!