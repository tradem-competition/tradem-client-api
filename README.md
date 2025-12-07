# trade'm client API guide
Enabling algorithmic trading on the trade'm platform

## Exchange rates data
In order to get the information about exchange rates, you need to integrate in your algorithm a data source like [CoinBase](https://docs.cloud.coinbase.com/sign-in-with-coinbase/docs) or [CoinGecko](https://www.coingecko.com/api/documentation).

## Transaction request

### POST http://platform.tradem.online/api/transaction
#### Request body: {

`sourceWalletId`: Source wallet ID available from the UI (see below)

`destWalletId`: Destination wallet ID available from the UI (see below)

`amountFromSourceWallet`: [excludable with `amountToDestWallet`] Number of the currency units to sell from the source wallet

`amountToDestWallet`: [excludable with `amountFromSourceWallet`] Number of the currency units to buy

`exchangeRate`: [*OPTIONAL*] Transaction will be rejected if (at the moment of execution) the current exchange rate is lower than given `exchangeRate` (it's always the rate *SOURCE_CURRENCY/DESTINATION_CURRENCY*, see examples below)

#### }

#### Header:

`Authorization`: `Bearer <id_token>` (see Python code example)

## Wallet valuation request

### GET http://platform.tradem.online/api/account/:account_id/wallet/:wallet_id/valuation
#### Query parameters:

`limit`: [*OPTIONAL*] Number of data points to return (to get the latest valuation just pass `?limit=1`, see the example code below)

`fromTime`: [*OPTIONAL*] Get data points after the given time

`toTime`: [*OPTIONAL*] Get data points before the given time

`offset`: [*OPTIONAL*] Omit the given number number of earlier data points to get some later ones

#### Header:

`Authorization`: `Bearer <id_token>` (see Python code example)

## Where to find account ID (from URL or UI)
![Account ID](account-id.png "Where to find account ID")

## Where to find wallet IDs
![Wallet IDs](wallet-ids.png "Where to find wallet IDs")

## Alternatively, where to find wallet ID (from URL or UI)
![Wallet ID](wallet-id.png "Alternatively, where to find wallet ID")

## Examples
### Buying BTC for 10 USD
POST http://platform.tradem.online/api/transaction \
Request body: { \
`sourceWalletId`: `<USD_wallet_ID>` \
`destWalletId`: `<BTC_wallet_ID>` \
`amountFromSourceWallet`: 10 \
`exchangeRate`: `<lowest_acceptable_exchange_rate> [USD/BTC rate]` \
}

### Buying 5 BTC for USD
POST http://platform.tradem.online/api/transaction \
Request body: { \
`sourceWalletId`: `<USD_wallet_ID>` \
`destWalletId`: `<BTC_wallet_ID>` \
`amountToDestWallet`: 5 \
`exchangeRate`: `<lowest_acceptable_exchange_rate> [USD/BTC rate]` \
}

### Buying USD for 20 ETH
POST http://platform.tradem.online/api/transaction \
Request body: { \
`sourceWalletId`: `<ETH_wallet_ID>` \
`destWalletId`: `<USD_wallet_ID>` \
`amountFromSourceWallet`: 20 \
`exchangeRate`: `<lowest_acceptable_exchange_rate> [ETH/USD rate]` \
}

### Buying 15 USD for ETH
POST http://platform.tradem.online/api/transaction \
Request body: { \
`sourceWalletId`: `<ETH_wallet_ID>` \
`destWalletId`: `<USD_wallet_ID>` \
`amountToDestWallet`: 15 \
`exchangeRate`: `<lowest_acceptable_exchange_rate> [ETH/USD rate]` \
}

## Example code
**In order to run the example code, you need to set the `TRADEM_EMAIL` and `TRADEM_PASSWORD` environment variables with your login credentials to the trade'm platform. You can see the `export TRADEM_EMAIL=<your_email>` and `export TRADEM_PASSWORD=<your_password>` commands in BASH.**

[example.py](example.py)
```python
from os import environ
import requests

API_KEY = 'AIzaSyBOEvN4OzAePlFp1fSRKWJlioA9r2WPZHw'
AUTH_URL = 'https://www.googleapis.com/identitytoolkit/v3/relyingparty/verifyPassword'
TRANSACTION_URL = 'http://platform.tradem.online/api/transaction'
ACCOUNT_ID = 'f24802e6-e2a2-46ad-ba25-caf423b73c70'
USD_WALLET_ID = '1d0e3ee6-1fac-4246-a869-9c383ae8fa9c'
BTC_WALLET_ID = '030bcff8-1bde-4b89-b814-b9c75cfda896'
VALUATION_URL = f'https://platform.tradem.online/api/account/{ACCOUNT_ID}/wallet/{BTC_WALLET_ID}/valuation'

data = {
    "email": environ['TRADEM_EMAIL'],
    "password": environ['TRADEM_PASSWORD'],
    "returnSecureToken": True,
}
res = requests.post(AUTH_URL, json=data, params={'key': API_KEY}).json()
id_token = res['idToken']

res = requests.post(TRANSACTION_URL, headers={
    'Authorization': f'Bearer {id_token}'
}, json={
    "sourceWalletId": USD_WALLET_ID,
    "destWalletId": BTC_WALLET_ID,
    "amountFromSourceWallet": 10,
    "exchangeRate": 0.000023, # the lowest acceptable USD/BTC exchange rate
}, verify=False)

print(res.json())

"""
{
	'jsonapi': {
		'version': '1.0'
	},
	'data': [
		{
			'type': 'Transaction',
			'id': 'db383586-4347-4a6a-93a9-b7b8aa75a422',
			'attributes': {
				'sourceWalletId': '1d0e3ee6-1fac-4246-a869-9c383ae8fa9c',
				'amountFromSourceWallet': '10.00',
				'destWalletId': '030bcff8-1bde-4b89-b814-b9c75cfda896',
				'amountToDestWallet': '0.00022802',
				'exchangeRate': '0.000022801824328360865',
				'creatorId': '4774e5c8-4359-4a38-8205-9561187834f9',
				'state': 'done',
				'createdAt': '2023-12-09T10:00:51.587Z',
				'updatedAt': '2023-12-09T10:00:51.589Z',
				'sourceWalletVersion': '24',
				'sourceWalletBalance': '999770.00',
				'destWalletVersion': '24',
				'destWalletBalance': '0.00526128',
				'comment': None
			}
		}
	]
}
"""

res = requests.get(VALUATION_URL, headers={
    'Authorization': f'Bearer {id_token}'
}, params={
    'limit': 1,
}, verify=False)

print(res.json())

"""
[
	{
		'id': '1abd2171-6d86-4966-bf19-54408789a625',
		'balance': '0.00526128',
		'baseCurrencyBalance': '230.73943225920002',
		'createdAt': '2023-12-09T10:00:51.797Z',
		'updatedAt': '2023-12-09T10:00:51.797Z',
		'walletId': '030bcff8-1bde-4b89-b814-b9c75cfda896',
		'accountValuationId': '4502ae1d-0abf-4656-b871-2b87d56d43c8'
	}
]
"""
```
