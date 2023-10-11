# Secrets
Secrets is an Uqbar app that let's users post a pair of (`message`, `secret`) to the blockchain, where `message` is visible to everyone, and `secret` is hidden. Other users can then bid to have the poster reveal the `secret`. The poster can then reveal the secret on chain and collect the highest bid. Note that when they reveal the secret, it is impossible for them to change it to something they didn't commit to. The app has three components:
1. Solidity Contracts (in `/contracts`)
2. Uqbar App (in `/secrets-process`)
3. UI (in `/secrets-ui`)

## Solidity Contracts
These aren't that important for the purpose of understanding how an uqbar app is written. The only thing to understand are the three events listed at the top of `Secrets.sol`:
```
event NewSecret(bytes32 indexed messageHash, string message, bytes uqname);
event BidPlaced(bytes32 indexed messageHash, uint256 amount, bytes uqname);
event SecretRevealed(bytes32 indexed messageHash, string secret, bytes uqname);
```
These events should be self explanatory - `NewSecret` is emitted when someone posts a new secret. `BidPlaced` is emitted when someone posts a new offer to reveal the secret. `SecretRevealed` is emitted when the original poster has revealed the secret. Our Uqbar app watches for each of these events and indexes them to create a feed of all secrets - some of them revealed, some of them not revealed, and some of them with bids.

## Uqbar App
The code for this is well commented. The main things it is responsible for:
- Indexing events emitted by the contract
- When a user submits a `NewSecret` to chain, this app saves the secret to storage
- Serving the frontend
    - `/secrets` serves the ui
    - `/secrets/feed` serves a json list of all the indexed secrets (who said what, when was it emitted, are there bids for this secret, what is the top bid amount, etc.)
    - `/secrets/my-secrets` if the user has posted any secrets, the `message` is public while the `secret` is hidden, and can be accessed at this path
    - `/secrets/save-secret` a POST endpoint for the frontend to write new secrets to the backend

## UI
This is a fairly standard Web3 frontend - the only thing to note is that we are serving this from an uqbar node instead of a normal server.
