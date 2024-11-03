```rs
use anchor_lang::prelude::*;
// Please note that a program has a id also called an 'address' that will be automatically assigned to you when you deploy your Solana program using the Solana playground
declare_id!("");

pub const ANCHOR_DESCRIMINATOR_SIZE: usize = 8; // Your program will need 8 bytes + the size of whatever you saving on the Solana blockchain. 'a_d_s' is something that is written to every account on the Solana blockchain by an Anchor program. | ANCHOR_DESCRIMINATOR_SIZE Purpose: It specifies the type of account it is. It is used by anchor for some of its checks.

// Anchor lets you take a regular Rust program and convert it into a Anchor program using the Solana macro '#[program]'. For example:
#[program] // '#[program]' would even apply reasonable defualts
pub mod favorites { // a regular program called favorites
    use super::*;

    // Now this function 'set_favorites()' would take instructions provided by clients and run them and then save the results to the blockchain
    pub fn set_favorites() -> Return<()> {
        // Fill in later
    }
}

// This is what we what to save to the blockchain
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

// Note that when your 'set_favorites' function is called, tney will need to provide a list of accounts that they are going to modify on the blockchain
// Also note that Solana can process multiple transactions on different accounts at the same time

// When people call the function defined above 'set_favorites', they are going to need to provide account(s) they are going to modify on the blockchain. I guess the function 'set_favorite' needs one or more accounts to act upon

// Struct of accounts for the 'set_favorites' instruction handler
pub struct SetFavorites { // It is tradition to name the account that the instruction (e.g, 'set_favorites') would act upon as the name of the instruction handler function itself
    #[account(number)]
    pub user: Signer<'info> // lifetime `info is a Solana object
    // so we're gonna specify some options for this account
    // I'm gonna say that the signer is mutable. Why?
    // Because the person that signs the instruction to
    // ...set favorites is going to pay to create their
    // ...favorites account on the blockchain
    // You're gonna pay to store that information.
    //
    // Also, we'll need the person running 'set_favorites'
    // ...to specify the favorites account they want to write to.
    // It doesn't mean that we'll let them write to this account,
    // ...we just need them to specify the account.

}
```
