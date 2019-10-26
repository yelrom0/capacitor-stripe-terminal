<h3 align="center">capacitor-stripe-terminal</h3>

<div align="center">

![npm](https://img.shields.io/npm/v/capacitor-stripe-terminal.svg)
[![GitHub Issues](https://img.shields.io/github/issues/eventOneHQ/capacitor-stripe-terminal.svg)](https://github.com/eventOneHQ/capacitor-stripe-terminal/issues)
[![GitHub Pull Requests](https://img.shields.io/github/issues-pr/eventOneHQ/capacitor-stripe-terminal.svg)](https://github.com/eventOneHQ/capacitor-stripe-terminal/pulls)
[![GitHub license](https://img.shields.io/github/license/eventOneHQ/capacitor-stripe-terminal.svg)](https://github.com/eventOneHQ/capacitor-stripe-terminal/blob/master/LICENSE)
[![Slack](https://slack.event1.io/badge.svg)](https://slack.event1.io/)

</div>

---

<p align="center">Capacitor plugin for <a href="https://stripe.com/terminal">Stripe Terminal</a> (unofficial)
</p>

**[Current project status](https://github.com/eventOneHQ/capacitor-stripe-terminal/issues/2)**

**WARNING:** Until this project reaches 1.0, the API is subject to breaking changes.

## 📝 Table of Contents

- [Current Project Status](https://github.com/eventOneHQ/capacitor-stripe-terminal/issues/2)
- [Getting Started](#getting_started)
  - [iOS Setup](#ios_setup)
- [Usage](#usage) <!-- - [Contributing](CONTRIBUTING.md) -->
- [Authors](#authors)
- [Acknowledgments](#acknowledgement)

## 🏁 Getting Started <a name = "getting_started"></a>

### iOS Setup

First, follow all Stripe instructions under ["Configure your app"](https://stripe.com/docs/terminal/sdk/ios#configure). Then run the following in your Capacitor project:

```bash
# install the bridge
npm install eventOneHQ/capacitor-stripe-terminal#master

# sync the iOS project
npx cap sync ios
```

### Android Setup

(not supported yet)

## 🎈 Usage <a name="usage"></a>

```javascript
import {
  StripeTerminalPlugin,
  DeviceType,
  DiscoveryMethod
} from 'capacitor-stripe-terminal'

// First, initialize the SDK
const terminal = new StripeTerminal({
  fetchConnectionToken: () => {
    return fetch('https://your-backend.dev/token', { method: 'POST' })
      .then(resp => resp.json())
      .then(data => data.secret)
  }
})

// Start scanning for readers
// capacitor-stripe-terminal uses Observables for any data streams
// To stop scanning, unsubscribe from the Observable.
terminal
  .discoverReaders({
    simulated: false,
    deviceType: DeviceType.Chipper2X,
    discoveryMethod: DiscoveryMethod.BluetoothProximity
  })
  .subscribe(readers => {
    if (readers.length) {
      terminal
        .connectReader({ serialNumber: readers[0].serialNumber })
        .then(connectedReader => {
          // the reader is now connected and usable
        })
    }
  })

// Once the reader is connected, collect a payment intent!
;(async () => {
  // subscribe to user instructions
  const waitingSubscription = terminal
    .readerDisplayMessage()
    .subscribe(message => {
      console.log('readerDisplayMessage', message.text)
    })
  const inputSubscription = terminal.readerInput().subscribe(message => {
    console.log('readerInput', message.text)
  })

  // retrieve the payment intent
  const pi = await terminal.retrievePaymentIntent(
    'your client secret created server side'
  )

  // collect the payment method
  await this.terminal.collectPaymentMethod()

  // and finally, process the payment
  await this.terminal.processPayment()

  // once you are done, make sure to unsubscribe (e.g. in ngOnDestroy)
  waitingSubscription.unsubscribe()
  inputSubscription.unsubscribe()
})()
```

See the full docs [here](https://oss.eventone.page/capacitor-stripe-terminal).

## ✍️ Authors <a name = "authors"></a>

- [@nprail](https://github.com/nprail) - Maintainer

See also the list of [contributors](https://github.com/eventOneHQ/capacitor-stripe-terminal/contributors) who participated in this project.

## 🎉 Acknowledgements <a name = "acknowledgement"></a>

- Hat tip to anyone whose code was used
- Thanks [Stripe](https://stripe.com/terminal) for creating such an amazing product
- Thanks [react-native-stripe-terminal](https://github.com/theopolisme/react-native-stripe-terminal) for quite a few borrowed concepts
