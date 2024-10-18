---
title: Sealevel Attacks
date: "2024-10-18"
category: Solana
tags: solana, security
summary: "Overview of Anchor's list of footguns/attacks and protections in the Solana programming model"
---

Anchor is a framework for building Solana programs. It provides a lot of features and abstractions that make it easier to build programs. This post is a summary of Anchor's [sealevel-attacks](https://github.com/coral-xyz/sealevel-attacks/tree/master/programs), list of footguns/attacks and protections in the Solana programming model.

## 1. Signer Authorization

One of the most critical security checks is verifying that an "authority" account has actually signed the transaction.

Without proper checks, anyone could pass an authority account without proving ownership.

⚠️ Vulnerable: The authority account is not verified as a transaction signer
```rust
pub struct User<'info> {
    authority: AccountInfo<'info>,
}
```

✓ Secure: The authority must be a transaction signer
```rust
pub struct User<'info> {
    authority: Signer<'info>,
}
```

## 2. Account Data Validation

Ensure that the accounts contain the valid data.

For instance, when working with token accounts, verify the token account contains owner, mint, and amount fields.

⚠️ Vulnerable: No validation of token account data structure
```rust
pub struct User<'info> {
    token: AccountInfo<'info>,
    authority: Signer<'info>,
}
```

✓ Secure: Add a constraint to verify that the token account is owned by the authority
```rust
pub struct User<'info> {
    #[account(constraint = authority.key == &token.owner)]
    token: Account<'info, TokenAccount>,
    authority: Signer<'info>,
}
```

## 3. Checking Account Ownership

Ensure that passed-in accounts are owned by the expected program.

For example, token accounts must be owned by the SPL Token program to prevent manipulation with fake accounts.

⚠️ Vulnerable: No verification of token account program ownership
```rust
let token = SplTokenAccount::unpack(&ctx.accounts.token.data.borrow())?;
if ctx.accounts.authority.key != &token.owner {
    return Err(ProgramError::InvalidAccountData);
}
msg!("Your account balance is: {}", token.amount)?;
```

✓ Secure: Add an Anchor constraint to verify program ownership
```rust
pub struct User<'info> {
    #[account(constraint = authority.key == &token.owner)]
    token: Account<'info, TokenAccount>,
    authority: Signer<'info>,
}
```

## 4. Account Type Confusion

Prevent account type confusion by ensuring different account types can be distinguished from each other.

Without proper type discrimination, one account type could be mistaken for another.

⚠️ Vulnerable: No way to distinguish between User and Metadata accounts
```rust
#[derive(BorshSerialize, BorshDeserialize)]
pub struct User {
    authority: Pubkey,
}

#[derive(BorshSerialize, BorshDeserialize)]
pub struct Metadata {
    account: Pubkey,
}
```

Manual fix using discriminants:
```rust
#[derive(BorshSerialize, BorshDeserialize)]
pub struct User {
    discriminant: AccountDiscriminant,
    authority: Pubkey,
}

#[derive(BorshSerialize, BorshDeserialize)]
pub struct Metadata {
    discriminant: AccountDiscriminant,
    account: Pubkey,
}

#[derive(BorshSerialize, BorshDeserialize, PartialEq)]
pub enum AccountDiscriminant {
    User,
    Metadata,
}
```

✓ Secure: Using Anchor's `#[account]` macro for automatic discriminators (where Anchor adds an 8-byte discriminator)
```rust
#[account]
pub struct User {
    authority: Pubkey,
}

#[account]
pub struct Metadata {
    account: Pubkey,
}
```

## 5. Account Initialization

When creating new accounts, proper initialization with discriminators is important.

Otherwise, this could lead to both incorrect account initialization and unnecessary re-initialization.

⚠️ Vulnerable: No discriminator and allows re-initialization
```rust
#[derive(Accounts)]
pub struct Initialize<'info> {
    user: AccountInfo<'info>,
    authority: Signer<'info>,
}

#[derive(BorshSerialize, BorshDeserialize)]
pub struct User {
    authority: Pubkey,
}

pub fn initialize(ctx: Context<Initialize>) -> ProgramResult {
    let mut user = User::try_from_slice(&ctx.accounts.user.data.borrow()).unwrap();
    user.authority = ctx.accounts.authority.key();
    let mut storage = ctx.accounts.user.try_borrow_mut_data()?;
    user.serialize(&mut storage.deref_mut()).unwrap();
    Ok(())
}
```

✓ Secure: Anchor's `#[account(init)]` constraint handles initialization safely
```rust
#[derive(Accounts)]
pub struct Init<'info> {
    #[account(init, payer = authority, space = 8+32)]
    user: Account<'info, User>,
    authority: Signer<'info>,
    system_program: Program<'info, System>,
}
```

## 6. Arbitrary CPI

When performing CPIs, make sure you're invoking the correct program.

⚠️ Vulnerable: No verification of token program address
```rust
pub fn cpi(ctx: Context<Cpi>, amount: u64) -> ProgramResult {
    solana_program::program::invoke(
        &spl_token::instruction::transfer(
            ctx.accounts.token_program.key,
            ctx.accounts.source.key,
            ctx.accounts.destination.key,
            ctx.accounts.authority.key,
            &[],
            amount,
        )?,
        &[
            ctx.accounts.source.clone(),
            ctx.accounts.destination.clone(),
            ctx.accounts.authority.clone(),
        ],
    )
}
```

Manual program ID verification:
```rust
pub fn cpi_secure(ctx: Context<Cpi>, amount: u64) -> ProgramResult {
    if &spl_token::ID != ctx.accounts.token_program.key {
        return Err(ProgramError::IncorrectProgramId);
    }
    solana_program::program::invoke(
        &spl_token::instruction::transfer(
            ctx.accounts.token_program.key,
            ctx.accounts.source.key,
            ctx.accounts.destination.key,
            ctx.accounts.authority.key,
            &[],
            amount,
        )?,
        &[
            ctx.accounts.source.clone(),
            ctx.accounts.destination.clone(),
            ctx.accounts.authority.clone(),
        ],
    )
}
```

✓ Secure: Using Anchor's wrapper of the SPL token program
```rust
use anchor_spl::token::{self, Token, TokenAccount};

#[program]
pub mod arbitrary_cpi_recommended {
    use super::*;
    
    pub fn cpi(ctx: Context<Cpi>, amount: u64) -> ProgramResult {
        token::transfer(ctx.accounts.transfer_ctx(), amount)
    }
}

#[derive(Accounts)]
pub struct Cpi<'info> {
    source: Account<'info, TokenAccount>,
    destination: Account<'info, TokenAccount>,
    authority: Signer<'info>,
    token_program: Program<'info, Token>,
}

impl<'info> Cpi<'info> {
    pub fn transfer_ctx(&self) -> CpiContext<'_, '_, '_, 'info, token::Transfer<'info>> {
        let program = self.token_program.to_account_info();
        let accounts = token::Transfer {
            from: self.source.to_account_info(),
            to: self.destination.to_account_info(),
            authority: self.authority.to_account_info(),
        };
        CpiContext::new(program, accounts)
    }
}
```

## 7. Duplicate Mutable Accounts

When working with multiple mutable accounts, ensure they're distinct.

⚠️ Vulnerable: Same account could be used for both user_a and user_b
```rust
pub fn update(ctx: Context<Update>, a: u64, b: u64) -> ProgramResult {
    let user_a = &mut ctx.accounts.user_a;
    let user_b = &mut ctx.accounts.user_b;
    user_a.data = a;
    user_b.data = b;
    Ok(())
}

#[derive(Accounts)]
pub struct Update<'info> {
    user_a: Account<'info, User>,
    user_b: Account<'info, User>,
}
```

✓ Secure: Enforce unique accounts using constraints
```rust
#[derive(Accounts)]
pub struct Update<'info> {
    #[account(constraint = user_a.key() != user_b.key())]
    user_a: Account<'info, User>,
    user_b: Account<'info, User>,
}
```

## 8. PDA Bump Seed Canonicalization

When working with Program Derived Addresses (PDAs), always use the canonical bump seed to ensure uniqueness.

Using `create_program_address` with user-provided bumps is not recommended because, multiple valid PDAs could exist for the same seeds.

Always verify the canonical bump either by storing it and re-using it with `create_program_address` or by using `Pubkey::find_program_address` and comparing the expected bump.

⚠️ Vulnerable: User-provided bump allows multiple valid PDAs
```rust
pub fn set_value(ctx: Context<BumpSeed>, key: u64, new_value: u64, bump: u8) -> ProgramResult {
    let address = Pubkey::create_program_address(
        &[key.to_le_bytes().as_ref(), &[bump]], 
        ctx.program_id
    )?;
    if address != ctx.accounts.data.key() {
        return Err(ProgramError::InvalidArgument);
    }

    ctx.accounts.data.value = new_value;
    Ok(())
}
```

✓ Secure: Verify canonical bump seed
```rust
pub fn set_value_secure(
    ctx: Context<BumpSeed>,
    key: u64,
    new_value: u64,
    bump: u8,
) -> ProgramResult {
    let (address, expected_bump) = Pubkey::find_program_address(
        &[key.to_le_bytes().as_ref()],
        ctx.program_id
    );
    if address != ctx.accounts.data.key() {
        return Err(ProgramError::InvalidArgument);
    }
    if expected_bump != bump {
        return Err(ProgramError::InvalidArgument);
    }

    ctx.accounts.data.value = new_value;
    Ok(())
}
```

## 9. PDA Sharing

Use unique PDAs for different authority domains to maintain proper security boundaries.

For example, in DeFi applications, each liquidity pool should have its own unique PDA authority derived from its specific parameters (like token pair addresses).

## 10. Closing Accounts

When an account is no longer needed, close it properly to reclaim rent and prevent security issues.

✓ Recommended: Use Anchor's close constraint
```rust
#[derive(Accounts)]
pub struct Close<'info> {
    #[account(mut, close = destination)]
    account: Account<'info, Data>,
    destination: AccountInfo<'info>,
}
```