<h3>Solana Program teaser</h3>

Here is a quick and immersive introduction to the first Solana program developed in the Solana Bootcamp - 2024.

The Solana program below allows a user to save a favorite to the blockchain. The user can sign and run a favorite saving transaction, and would get charged for it.

```rs
use anchor_lang::prelude::*;
// Please note that a program has a id also called an 'address' that will be automatically assigned to you when you deploy your Solana program using the Solana playground
declare_id!("");

pub const ANCHOR_DESCRIMINATOR_SIZE: usize = 8; // Your program will need 8 bytes + the size of whatever you saving on the Solana blockchain. 'a_d_s' is something that is written to every account on the Solana blockchain by an Anchor program. | ANCHOR_DESCRIMINATOR_SIZE Purpose: It specifies the type of account it is. It is used by anchor for some of its checks.

// Anchor lets you take a regular Rust program and convert it into a Anchor program using the Solana macro '#[program]'. For example:
#[program] // '#[program]' would even apply reasonable defualts
pub mod favorites { // a regular program called favorites
    use super::*;

    // Now this function 'set_favorites()' would take instructions provided by clients, run them, and then save the results to the blockchain
    pub fn set_favorites() -> Result<()> {
        // Fill in later
    }
}

// 'Favorites' is what we what to save to the blockchain
#[account] // This macro specifies to save struct 'Favorites' to an account
#[derive(InitSpace)] // This macro lets Anchor know how big the data is, and gives all the intances of the data (in this case, struct Favorites) the initspace attribute which is the total space used by all the items inside
pub struct Favorites {
    pub number: u64,

    // You do need to also specify the size of each field in the struct. This is especially the case for a String data type which essentially has no default max size
    #[max_len(50)] // (50) as in 50 bytes
    pub color: String,

    #[max_len(5, 50)] // (5, 50) as in, 5 strings max, with each string being 50 bytes max
    pub hobbies: Vec<String>,
}

// Note that when your 'set_favorites' function is called, the instruction runner will need to provide a list of accounts involved with what they are going to modify on the blockchain - the account to be modified, the account that would sign the modifications, e.t.c
// Also note that Solana can process multiple transactions on different accounts at the same time

// Struct of accounts for the 'set_favorites' instruction handler
#[derive(Accounts)]
pub struct SetFavorites<'info> { // this struct holds every meta data needed for a successful 'set_favorites' action - the destination account to add a favorite to, the signer (instruction runner)

    #[account(mut)] // the account of the instruction runner needs to be mutable since they will be paying to execute the 'set_favorites' instruction handler
    pub user: Signer<'info>,

    // the actual account of all favorites set by users of our 'set_favorites' instruction handler
    #[account(
        init_if_needed, // make a new 'favorites' account if it doesn't already exist
        payer = user, // the payer signs and gets charged for the transaction of adding a favorite
        space = ANCHOR_DESCRIMINATOR_SIZE + Favorites::INIT_SPACE, // how much space favorites need
        seeds = [b"favorites", user.key().as_ref()], // this is a PDA used to give a non-user account (e.g, favorites) an address on the blockchain
        bump // Used to calculate the seed provided above
    )]
    pub favorites: Account<'info, Favorites>,

    pub system_program: Program<'info, System>,
}

```

Phew! Although fun, the program above contained a lot of comments. This was done to provide as much base understanding as possible and needed to make sense of fundamental Solana patterns. The remainder of this guide book does not contain nearly as much comments.
