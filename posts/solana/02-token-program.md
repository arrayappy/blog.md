---
title: Solana Token Program
date: "2024-10-07"
category: Solana
tags: solana
summary: "Learn about SPL Tokens, and NFTs"
---

SOL is the 'native token' of Solana, but we can also create our own tokens. The Token Program is one of the most important native programs in Solana. It enables the creation and management of fungible and non-fungible tokens.

One advantage of Solana's account/program model is that we don't need a separate program for each token type (like ERC20 vs ERC721 in Ethereum). Instead, we can use the same program for both fungible and non-fungible tokens. All we need is to send a set of instructions to the Token Program.

#### Key Concepts

1. **Mint Account**: The source account that defines a token

   - Contains metadata like supply, decimals, mint authority
   - Created using `spl-token create-token`
   - Each token type has one mint account, like this [USDC](https://explorer.solana.com/address/EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v)

2. **Token Account**: Holds token balances for a specific mint

   - Each wallet needs a token account per token type
   - Created using `spl-token create-account`
   - Contains owner, mint, and balance information

3. **Associated Token Account (ATA)**:
   - Deterministic token account address that connects a specific wallet to a token
   - Created using `getAssociatedTokenAddress`
   - PDA([wallet, TOKEN_PROGRAM_ID, mint])

### Working with SPL Tokens

The best way to understand SPL tokens is to walk through creating and managing an SPL token using the Solana CLI ([installation guide](https://docs.anza.xyz/cli/install/)):

1. Create two new Solana wallets

```bash
solana-keygen new --force ~/my_solana_wallet1.json
solana-keygen new --force ~/my_solana_wallet2.json
```

2. Create a new token with the specified mint authority (mint token)

```bash
spl-token create-token --mint-authority ~/my_solana_wallet1.json
```

3. Store the token address in a variable in your terminal

```bash
TOKEN1=7gqyYWS8fpjvtbjWx2Qk1GPQpGHEasf7moBsns1gJkpf  # Replace with actual token address from the previous step
```

4. Check the mint token information

```bash
solana account $TOKEN1
```
It should return the token metadata like address, supply, decimals, and mint authority. You can check the same info on [Solana Explorer](https://explorer.solana.com/).

5. Store wallet addresses in variables in your terminal (these needs to be created before)

```bash
SOLADDR1=$(solana address -k ~/my_solana_wallet1.json)
SOLADDR2=$(solana address -k ~/my_solana_wallet2.json)
```

6. Set the configuration to use wallet1

```bash
solana config set --keypair ~/my_solana_wallet1.json 
```

7. Airdrop SOL to wallet1 and wallet2

```bash
solana airdrop 100 $SOLADDR1
solana airdrop 200 $SOLADDR2
```

8. Create an associated token account for wallet1 (token account) and store the address in a variable

```bash
TOKENACCT1=$(spl-token create-account $TOKEN1 --owner $SOLADDR1 --fee-payer ~/my_solana_wallet1.json | grep 'Creating account' | awk '{print $3}')
```


9. Mint 100 tokens to the token account (minting and transferring tokens)

```bash
spl-token mint $TOKEN1 100 --recipient $TOKENACCT1
```

10. Verify the token balance

```bash
spl-token supply $TOKEN1
spl-token balance $TOKENACCT1
```

11. Create an associated token account for wallet2 (token account) and store the address in a variable

```bash
TOKENACCT2=$(spl-token create-account $TOKEN1 --owner $SOLADDR2 --fee-payer ~/my_solana_wallet2.json | grep 'Creating account' | awk '{print $3}')
```

12. Transfer 25 tokens from wallet1 to wallet2 (transferring tokens)

```bash
spl-token transfer $TOKEN1 25 $TOKENACCT2 --owner ~/my_solana_wallet1.json --fee-payer ~/my_solana_wallet1.json
```

13. Check the balances of both accounts (checking balances)

```bash
spl-token balance $TOKENACCT1
spl-token balance $TOKENACCT2
```

14. Verify the token accounts (checking wallet associated token accounts)

```bash
spl-token accounts --owner $SOLADDR1
spl-token accounts --owner $SOLADDR2
```

## NFTs

NFTs (non-fungible tokens) are actually just SPL tokens with a few key constraints:

1. They use `--decimals 0` when creating the token, since there should be only one token
2. Only one token is minted, and then minting is disabled to ensure uniqueness

While you can create NFTs manually using the SPL token program this way, in practice most developers use tools like [Candy Machine](https://docs.metaplex.com/create-candy/introduction) to handle the image/metadata upload and minting process.