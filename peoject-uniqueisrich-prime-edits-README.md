Prime code structures 

1. The NEAR NFT Smart Contract (Rust)
This contract implements the standard NEAR NEP-171 NFT framework. It explicitly locks the mint functionality behind a truth_gate_key parameter which must evaluate to 1301



use near_contract_standards::non_fungible_token::metadata::{
    NFTContractMetadata, NonFungibleTokenMetadataProvider, TokenMetadata,
};
use near_contract_standards::non_fungible_token::NonFungibleToken;
use near_contract_standards::non_fungible_token::TokenId;
use near_sdk::borsh::{self, BorshDeserialize, BorshSerialize};
use near_sdk::collections::UnorderedMap;
use near_sdk::{
    env, near_bindgen, AccountId, Balance, PanicOnDefault, Promise, PromiseOrValue,
};

#[near_bindgen]
#[derive(BorshDeserialize, BorshSerialize, PanicOnDefault)]
pub struct UniqueIsRichContract {
    tokens: NonFungibleToken,
    metadata: LazyOption<NFTContractMetadata>,
    
    // Core parameters tracking Aderibigbe Richard's origin profile
    pub truth_gate_key: u32,
    pub max_supply: u64,
    pub mint_price: Balance,
    pub token_adaptability_tiers: UnorderedMap<TokenId, u8>,
}

#[near_bindgen]
impl UniqueIsRichContract {
    #[init]
    pub fn new_default_meta(owner_id: AccountId) -> Self {
        Self::new(
            owner_id,
            NFTContractMetadata {
                spec: "nft-1.0.0".to_string(),
                name: "Aderibigbe Richard: Genesis Node".to_string(),
                symbol: "RICH".to_string(),
                icon: None,
                base_uri: Some("https://ipfs.io".to_string()),
                reference: None,
                reference_hash: None,
            },
        )
    }

    #[init]
    pub fn new(owner_id: AccountId, metadata: NFTContractMetadata) -> Self {
        assert!(!env::state_exists(), "Already initialized");
        Self {
            tokens: NonFungibleToken::new(
                b"t".to_vec(),
                owner_id,
                Some(b"g".to_vec()),
                Some(b"m".to_vec()),
                Some(b"c".to_vec()),
            ),
            metadata: LazyOption::new(b"m".to_vec(), Some(&metadata)),
            truth_gate_key: 1301, // 1:01 PM Birth Moment
            max_supply: 1995,    // Birth Year Max Supply
            mint_price: 22_000_000_000_000_000_000_000, // 0.022 NEAR in yoctoNEAR
            token_adaptability_tiers: UnorderedMap::new(b"a".to_vec()),
        }
    }

    // REALITY-BASED TRUTH GATE: Blocks minting unless the historical key matches
    #[payable]
    pub fn mint_origin_node(&mut self, token_id: TokenId, receiver_id: AccountId, truth_gate_key: u32) -> Token {
        let current_supply = self.tokens.owner_by_id.len();
        assert!(current_supply < self.max_supply, "Max supply reached");
        
        // The Truth Gate Validation
        assert_eq!(truth_gate_key, self.truth_gate_key, "🚨 Access Denied: Reality verification failed.");
        
        let attached_deposit = env::attached_deposit();
        assert!(attached_deposit >= self.mint_price, "Insufficient NEAR deposit attached for transaction");

        // Set initial Positron Adaptability state to Tier 0 (Ibadan Core Baseline)
        self.token_adaptability_tiers.insert(&token_id, &0);

        let token_metadata = TokenMetadata {
            title: Some(format!("UniqueIsRich Genesis Node #{}", token_id)),
            description: Some("An adaptive node reflecting the timeline of Aderibigbe Richard. Born in Ibadan, optimized in Lagos.".to_string()),
            media: Some(format!("{}_tier_0.png", token_id)), // Defaults to baseline state image
            media_hash: None,
            copies: Some(1),
            issued_at: Some(env::block_timestamp().to_string()),
            expires_at: None,
            starts_at: None,
            updated_at: None,
            extra: Some("{\"identity\":\"uniqueisrich\", \"origin\":\"Ibadan\", \"transit\":\"Lagos\"}".to_string()),
            reference: None,
            reference_hash: None,
        };

        self.tokens.internal_mint(token_id, receiver_id, Some(token_metadata))
    }

    // POSITRON ADAPTABILITY: Dynamically shift properties as the project narrative matures
    pub fn adapt_node_state(&mut self, token_id: TokenId, new_tier: u8) {
        assert_eq!(env::predecessor_account_id(), self.tokens.owner_id, "Unauthorized administrative call");
        assert!(self.tokens.owner_by_id.get(&token_id).is_some(), "Token does not exist");
        
        self.token_adaptability_tiers.insert(&token_id, &new_tier);
        
        if let Some(mut token_metadata) = self.tokens.token_metadata_by_id.as_mut().unwrap().get(&token_id) {
            token_metadata.media = Some(format!("{}_tier_{}.png", token_id, new_tier));
            token_metadata.extra = Some(format!(
                "{{\"identity\":\"uniqueisrich\", \"origin\":\"Ibadan\", \"transit\":\"Lagos\", \"positron_tier\":{}}}", 
                new_tier
            ));
            self.tokens.token_metadata_by_id.as_mut().unwrap().insert(&token_id, &token_metadata);
        }
    }
}

near_contract_standards::impl_non_fungible_token_core!(UniqueIsRichContract, tokens);
near_contract_standards::impl_non_fungible_token_metadata!(UniqueIsRichContract, metadata);


2. Interactive Frontend Web3 Connection (NEAR API JS)
To interact with this contract from your launchpad website or your interactive graphic novel UI, use this JavaScript pattern to trigger the validation gateway


import * as nearAPI from "near-api-js";

const { connect, WalletConnection, Contract, utils } = nearAPI;

const config = {
    networkId: "mainnet",
    keyStore: new nearAPI.keyStores.BrowserLocalStorageKeyStore(),
    nodeUrl: "https://near.org",
    walletUrl: "https://near.org",
    contractName: "uniqueisrich.near" // Your named contract account on NEAR
};

async function initializeMintGate() {
    const near = await connect(config);
    const wallet = new WalletConnection(near, "uniqueisrich-app");

    if (!wallet.isSignedIn()) {
        wallet.requestSignIn(config.contractName);
        return;
    }

    const contract = new Contract(wallet.account(), config.contractName, {
        viewMethods: [],
        changeMethods: ["mint_origin_node"],
    });

    const userInputKey = parseInt(document.getElementById("truthGateInput").value); // User enters 1301

    if (userInputKey !== 1301) {
        alert("🚨 ACCESS DENIED: The Truth Gate requires Richard's true birth minute code.");
        return;
    }

    try {
        // Call contract with parameters and deposit network funds (0.022 NEAR)
        await contract.mint_origin_node(
            {
                token_id: "node-1",
                receiver_id: wallet.getAccountId(),
                truth_gate_key: userInputKey
            },
            "300000000000000", // Gas limit
            utils.format.parseNearAmount("0.022") // Attached mint cost
        );
        alert("🎉 Integration Unlocked: uniqueisrich node successfully added to your ledger.");
    } catch (err) {
        console.error("NEAR Execution Error:", err);
    }
}





