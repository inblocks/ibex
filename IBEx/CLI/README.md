# IBEx CLI helper

## Introduction

First of all CLI stands for Command Line Interface. If you do not have any idea of what this is, you can start [here](https://en.wikipedia.org/wiki/Command-line_interface) and come back whenever you are ready for this reading.

This documentation will help you to interact with IBEx API **for testing purpose only**.

The scope of this documentation is limited to our [testnet environment](https://testnet.ib.exchange) and to our Spot API.

We assume here that you already have a testnet account. If you do not, click [here](https://testnet.ib.exchange) and register yourself.

There is no guarantee that future IBEx API releases will be backward-compatible with what is described here.

The IBEx CLI relies on [Restish](https://rest.sh).

## Installation

### Local installation

To install Restish locally, please check out this [guide](https://rest.sh/#/guide).

### Using Docker

If you are a Docker user we can create this quite simple Dockerfile to build your image.

```
FROM golang:1.20.0-alpine3.17
RUN go install github.com/danielgtaylor/restish@latest
```

## Configuration

Create the following `apis.json` file and put it into your `${HOME}/.restish` directory.

```
{
  "ibex-testnet": {
    "base": "https://api-testnet.ib.exchange",
    "spec_files": [ "https://github.com/inblocks/ibex/raw/main/IBEx/CLI/ibex-api-0.1.yaml" ],
    "profiles": {
      "default": {
        "auth": {
          "name": "oauth-authorization-code",
          "params": {
            "audience": "account",
            "authorize_url": "https://auth-testnet.ib.exchange/auth/realms/inblocks-ibex/protocol/openid-connect/auth",
            "client_id": "ibex-cli",
            "scopes": "openid offline_access",
            "token_url": "https://auth-testnet.ib.exchange/auth/realms/inblocks-ibex/protocol/openid-connect/token"
          }
        }
      }
    }
  }
}
```

## First command

### Usage

```
> restish ibex-testnet
```

```
Usage:
  restish ibex-testnet [flags]
  restish ibex-testnet [command]

/Spot Commands:
  cancel-order
  create-order
  get-enabled-pairs
  get-orders
  get-price
```

When you will run the first command you will be asked to authenticate yourself on our testnet platform.

### Local installation

The OAuth2 authorization flow requires that you use your user credentials into the browser. Restify will open your browser so you can authenticate yourself and automaticcaly retrieve the code required for calling the API. You will need to enter your login and password only once because the resulting credentials will be stored for future use. You should not be asked to enter them again.

### Using Docker

When you run the command using Docker, your Docker instance cannot interact with your local browser and so cannot open the login page. You need to click on the link that is provided to you, authenticate yourself, and then, once done, you can get the `code` in your browser URL to provide it back into your terminal. You should only have to perform this operation once.

## First commands

If you are using a Docker instance in the following you should adapt the command by replacing the `restish` command by something like (assuming the Docker image is named `ibex-cli`):
```
docker run --rm \
  -v ${HOME}/.restish:/root/.restish \
  ibex-cli \
  restish
```

### get-enabled-pairs

```
> restish ibex-testnet get-enabled-pairs -f body
```

Response example:
```
[
  {
    checkout: "DISABLED"
    productCurrency: "POWER"
    productPrecision: 2
    productType: "MAIN"
    unitPriceCurrency: "IBXE"
    unitPricePrecision: 4
    unitPriceType: "MAIN"
  }
]
```

This response indicates that you can create buy and sell orders for the pair POWER / IBXE.

### get-price

For every command you can have its usage by using the `--help` option.

```
restish ibex-testnet get-price --help
```

```
Usage:
  restish ibex-testnet get-price [flags]

Aliases:
  get-price, getprice

Flags:
  -h, --help                         help for get-price
      --product-currency string
      --product-type string
      --unit-price-currency string
      --unit-price-type string
```

Let's now get the price for the pair POWER / IBXE

```
> restish ibex-testnet get-price \
  --product-currency POWER \
  --product-type MAIN \
  --unit-price-currency IBXE \
  --unit-price-type MAIN \
  -f body
```

The response is then:

```
{
  at: "2023-02-09T19:47:47.009899674Z"
  price: "4.2"
  productCurrency: "POWER"
  productType: "MAIN"
  unitPriceCurrency: "IBXE"
  unitPriceType: "MAIN"
}
```

### create-order

To buy 1 POWER at a unit price of 1 IBXE you can run the following command:

```
restish ibex-testnet create-order \
  'product{currency: POWER, type: MAIN, value: 1}, type: BUY, unitPrice{currency: IBXE, type: MAIN, value: 1}' \
  --code 000000 \
  -f body
```

```
{
  createdAt: "2023-02-10T12:37:09.528118725Z"
  id: "75baosrwoberj25y"
  ownerId: "abcd1234"
  product: {
    currency: "POWER"
    type: "MAIN"
    value: "1"
  }
  status: "OPEN"
  type: "BUY"
  unitPrice: {
    currency: "IBXE"
    type: "MAIN"
    value: "1"
  }
}
```

We just created a buy order whose id is `75baosrwoberj25y`.

`code` parameter is mandatory but its value can be any 6 digits in this environment.

`type` can be set with `BUY` (for buy orders) or `SELL` (for sell orders).

Three different responses can be obtained.

- `status = COMPLETED` means the order was entirely executed and so the order is not in the order book.
- `status = OPEN` means the order has been inserted into the order book. If the `product.value` field is different from the one sent in the request, this means the order was partially executed and so the response indicates which amount has been inserted into the book order.

### get-orders

```
restish ibex-testnet get-orders \
  --limit 10 \
  --offset 0 \
  --product-currency POWER \
  --sort unitPrice.value \
  --status OPEN \
  --unit-price-currency IBXE \
  -f body
```

Response example:

```
{
  offset: 0
  results: [
    {
      createdAt: "2023-02-10T12:37:09.528118725Z"
      id: "75baosrwoberj25y"
      ownerId: "abcd1234"
      product: {
        currency: "POWER"
        type: "MAIN"
        value: "1"
      }
      status: "OPEN"
      type: "BUY"
      unitPrice: {
        currency: "IBXE"
        type: "MAIN"
        value: "1"
      }
    }
    {
      createdAt: "2023-02-08T13:49:34.554674812Z"
      id: "1y69sn1djbtios5p"
      ownerId: "abcd1234"
      product: {
        currency: "POWER"
        type: "MAIN"
        value: "2"
      }
      status: "OPEN"
      type: "BUY"
      unitPrice: {
        currency: "IBXE"
        type: "MAIN"
        value: "2"
      }
      updatedAt: "2023-02-08T13:50:15.256767902Z"
    }
  ]
  size: 2
  total: 2
}
```

We can see the open buy order with id `75baosrwoberj25y`. For now it is part of the order book, waiting for a matching sell order.

### cancel-order

```
> restish ibex-testnet cancel-order 75baosrwoberj25y --code 000000 -f body
```

Response example:

```
{
  createdAt: "2023-02-10T12:37:09.528118725Z"
  id: "75baosrwoberj25y"
  ownerId: "abcd1234"
  product: {
    currency: "POWER"
    type: "MAIN"
    value: "0"
  }
  status: "CANCELED"
  type: "BUY"
  unitPrice: {
    currency: "IBXE"
    type: "MAIN"
    value: "1"
  }
  updatedAt: "2023-02-10T12:42:16.487413237Z"
}
```

The order is canceled and so removed from the order book.
