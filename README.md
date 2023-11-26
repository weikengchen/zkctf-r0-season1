# Catch the flag competition over RISC Zero

This repo contains a catch-the-flag challenge in RISC Zero. 

Contestants are asked to design inputs to the guest program running inside the RISC Zero VM. The goal is to let `main.rs` output "You have successfully solved this RISC Zero challenge."

Note that this means that a correct receipt (i.e., a zero-knowledge proof) is needed in order to have RISC Zero accept the submission.

If you solve this challenge, send your solution to Weikeng Chen at w.k@berkeley.edu . 

## Prize
I am prepared to give out some small cash prizes (around $20 each) to the earliest 20 qualified submissions.

## Code
```rust
use risc0_zkvm::{default_prover, ExecutorEnv};

const PROGRAM_CODE: &'static [u8] = include_bytes!("code");
const PROGRAM_ID: [u32; 8] = [
    4089023768u32,
    3145240833u32,
    1747178994u32,
    3214649236u32,
    3131642364u32,
    1474811960u32,
    1082230595u32,
    3832750910u32,
];

fn main() {
    /***************************************************************/

    /* Edit this area to provide your input to the guest program. */

    let mut input = Vec::<u8>::with_capacity(64);
    for i in 0..64 {
        input.push(i as u8);
    }

    /***************************************************************/

    let env = ExecutorEnv::builder()
        .write(&input)
        .unwrap()
        .build()
        .unwrap();

    let prover = default_prover();

    if prover.get_name() != "bonsai" {
        println!("You are recommended to use the Bonsai API for best experience if you have the Bonsai API key.");
    }

    let mut success_flag = true;

    let receipt = prover.prove_elf(env, PROGRAM_CODE);

    let expected = b"You have successfully solved this RISC Zero challenge.".to_vec();

    if receipt.is_err() {
        success_flag = false;
    } else {
        let receipt = receipt.unwrap();

        let verify_result = receipt.verify(PROGRAM_ID);
        if verify_result.is_err() {
            success_flag = false;
        }

        let data = receipt.journal.bytes;
        if data != expected {
            success_flag = false;
        }
    }

    if success_flag {
        println!("You have successfully solved this RISC Zero challenge.");
    } else {
        println!("Try again!");
    }
}

```

## Hints
Although I am not going to give out problem-specific hints, people who want to understand RISC Zero, particularly low-level details, may benefit from this blog article:

**Tech Deep Dive: Verifying FHE in RISC Zero, Part I**

https://l2ivresearch.substack.com/p/tech-deep-dive-verifying-fhe-in-risc
