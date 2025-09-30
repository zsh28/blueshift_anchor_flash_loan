# Blueshift Anchor Flash Loan

A secure flash loan program built with Anchor framework demonstrating instruction introspection on Solana blockchain.

## 🌟 Overview

This project implements a flash loan system that allows users to borrow large amounts of tokens within a single transaction, with the requirement to repay the loan plus a fee before the transaction completes. The program uses **instruction introspection** to validate that borrowing and repayment occur atomically.

## 🔑 Key Features

- **Atomic Flash Loans**: Borrow and repay within a single transaction
- **Instruction Introspection**: Validates transaction structure before execution
- **5% Fee Structure**: 500 basis points fee on borrowed amounts
- **Zero Risk for Lenders**: Failed repayment reverts the entire transaction
- **Overflow Protection**: All arithmetic operations use checked math
- **Comprehensive Testing**: 6 test suites covering all functionality

## 🏗️ Architecture

### Core Instructions

#### 1. **Borrow Instruction**

```rust
pub fn borrow(ctx: Context<Loan>, borrow_amount: u64) -> Result<()>
```

- Transfers tokens from protocol to borrower
- Validates that a repay instruction exists at transaction end
- Ensures this is the first instruction in the transaction
- Checks account consistency between borrow and repay instructions

#### 2. **Repay Instruction**

```rust
pub fn repay(ctx: Context<Loan>) -> Result<()>
```

- Extracts borrowed amount from first instruction data
- Calculates 5% fee (500 basis points)
- Transfers borrowed amount + fee back to protocol

### Account Structure

```rust
#[derive(Accounts)]
pub struct Loan<'info> {
    #[account(mut)]
    pub borrower: Signer<'info>,

    #[account(seeds = [b"protocol"], bump)]
    pub protocol: SystemAccount<'info>,

    pub mint: Account<'info, Mint>,

    #[account(init_if_needed, payer = borrower,
              associated_token::mint = mint,
              associated_token::authority = borrower)]
    pub borrower_ata: Account<'info, TokenAccount>,

    #[account(mut, associated_token::mint = mint,
              associated_token::authority = protocol)]
    pub protocol_ata: Account<'info, TokenAccount>,

    #[account(address = INSTRUCTIONS_SYSVAR_ID)]
    /// CHECK: InstructionsSysvar account
    instructions: UncheckedAccount<'info>,

    pub token_program: Program<'info, Token>,
    pub associated_token_program: Program<'info, AssociatedToken>,
    pub system_program: Program<'info, System>
}
```

## 🔐 Security Features

### Instruction Introspection Validation

The program performs multiple security checks:

1. **Transaction Structure**: Ensures borrow is first instruction
2. **Repay Validation**: Confirms repay instruction exists at transaction end
3. **Account Consistency**: Validates same accounts used in both instructions
4. **Program Verification**: Checks repay instruction targets this program

### Arithmetic Safety

All calculations use checked arithmetic operations:

```rust
// Fee calculation with overflow protection
let fee = (amount_borrowed as u128)
    .checked_mul(500)
    .unwrap()
    .checked_div(10_000)
    .ok_or(ProtocolError::Overflow)? as u64;

amount_borrowed = amount_borrowed
    .checked_add(fee)
    .ok_or(ProtocolError::Overflow)?;
```

## 🚀 Getting Started

### Prerequisites

- [Rust](https://www.rust-lang.org/tools/install)
- [Solana CLI](https://docs.solana.com/cli/install-solana-cli-tools)
- [Anchor Framework](https://www.anchor-lang.com/docs/installation)

### Installation

1. Clone the repository:

```bash
git clone <repository-url>
cd blueshift_anchor_flash_loan
```

2. Install dependencies:

```bash
npm install
cargo add anchor-spl
```

3. Build the program:

```bash
anchor build
```

### Running Tests

Run the comprehensive test suite:

```bash
# Run all tests
cargo test

# Run specific test module
cargo test --test simple_tests
```

## 🧪 Test Coverage

The project includes 6 comprehensive test suites:

| Test                                                     | Description                                              | Coverage                             |
| -------------------------------------------------------- | -------------------------------------------------------- | ------------------------------------ |
| `test_challenge_1_borrow_instruction_structure`          | Validates borrow instruction encoding and discriminators | Instruction structure, data encoding |
| `test_challenge_2_repay_instruction_and_fee_calculation` | Tests fee calculation logic and repay structure          | Fee math, overflow protection        |
| `test_instruction_introspection_data_format`             | Verifies instruction introspection data layout           | Cross-instruction data access        |
| `test_flash_loan_transaction_structure`                  | Tests transaction ordering requirements                  | Transaction validation logic         |
| `test_protocol_pda_derivation`                           | Validates deterministic PDA generation                   | Account derivation                   |
| `test_complete_flash_loan_integration`                   | End-to-end integration test                              | Full flash loan flow                 |

### Test Results

```
running 6 tests
test tests::test_challenge_1_borrow_instruction_structure ... ok
test tests::test_challenge_2_repay_instruction_and_fee_calculation ... ok
test tests::test_instruction_introspection_data_format ... ok
test tests::test_flash_loan_transaction_structure ... ok
test tests::test_complete_flash_loan_integration ... ok
test tests::test_protocol_pda_derivation ... ok

test result: ok. 6 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

## 💡 How Flash Loans Work

### Transaction Flow

1. **Borrow**: User borrows tokens from protocol pool
2. **Use**: User performs operations with borrowed tokens (trading, arbitrage, etc.)
3. **Repay**: User repays borrowed amount + 5% fee
4. **Validation**: If any step fails, entire transaction reverts

## 📚 Learning Resources

- [Anchor Framework Documentation](https://www.anchor-lang.com/)
- [Blueshift Flash Loan Challenge](https://learn.blueshift.gg/en/challenges/anchor-flash-loan)

---

Built with ❤️ using [Anchor Framework](https://www.anchor-lang.com/) on [Solana](https://solana.com/)
