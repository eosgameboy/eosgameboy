# EOSGameboy - Rock scissors paper

## Overview

What is **EOSGameboy**?? EOSGameboy is a fully trustless and fair decentralized gambling platform and the first retro-style game which is built on the high-performance EOS.IO blockchain. No one can trust online game before a blockchain presence. Now anyone can inspect the design and probability of games in smart contracts. EOSGameboy offers fair games and provides a player full control of their account.

## EOS Rock-paper-scissors

### How to play

1. Make sure you have an EOS account. If you don't have any EOS account. download [EOSlyx](https://eoslynx.com) and create new one

2. Player can play RPS on Chrome [Scatter](https://chrome.google.com/webstore/detail/scatter/ammjpmhgckkpcamddpolhchgomcojkle) extension.

3. Set your BET AMOUNT for each game. Minimum bet is 0.1 EOS.

4. Now you're ready to play RPS now. Just choose among Rock, Paper, Scissors to win.


For each winning, Player will get up to 5 times EOS and up to 20 times BOY tokens by BONUS roulette game. When the game ends in ties, He will get the betting.

Each gameplay will create a new transaction, and each game result will return in **receipt** action. The receipt form is pre-defined as below.

```json
{
	"player": "gameboytest1",  // Player Account
	"bet_amount": "0.1000 EOS",  // Bet EOS
	"payout": "0.2000 EOS",  	 // Pay out EOS
	"seed": {  // SEED information
		"bet_id": "3985317911641843742",
		"block_prefix": -675144148,
		"block_num": 52728,
		"timestamp": 1540404318,
		"player_seed": 0
	},
	"player_won": 1,  // 1: Player win, 0: draw, -1: Player lose
	"player_rsp": 3,  // Player's move Rock(3), Sissors(2), Paper(1)
	"banker_rsp": 2,  // Opponent's move Rock(3), Sissors(2), Paper(1)
	"x_eos": 2,  //  EOS reward winning rate
	"x_token": 4,  // BOY token reward winning rate
	"roulette_idx": 4 // random reward index
}
```

Notice: If you get a notice that your transaction failed, please check that you have enough CPU & bandwidth to make the transaction! Please check on [EOSToolkit](https://eostoolkit.io/home).

## Fairness

**EOSGameboy** team had many kinds of research for the fairest game on EOS blockchain ecosystem.  Basically, It's impossible to generate a random number in EOS smart contract. Most of the smart-contract games import random seed from the outside of blockchain.
The problem with this method is that it is difficult to be intact with one smart contract from cheating. for an instance, Somebody can make security attacks like abusing or hacking when the smart-contract refer the random number.
**EOSGameboy** has designed to generate a random number in the only one smart contract. Therefore it is safe from manipulation, cheating or hacking from outside.
The random seed generation procedure is in the following order.

### 1. player_seed generation.
`transfer`(eosio.token) `action` get a player seed from Gameboy's web page in `memo` then it is delivered to **rockscissors** smart contract. Even a player can input a random seed whenever want.

### ENG 2. Seed generation for random number.

Gameboy uses several types of seeds as below.

```json
{
	"bet_id": "3985317911641843742",
	"block_prefix": -675144148,
	"block_num": 52728,
	"timestamp": 1540404318,
	"player_seed": 0
}
```

`bet_id`: Value from read_transaction() after hashing, *(refered from eosbet.io smart contract)*

```c++
  //generate bet id
  auto s = read_transaction(nullptr, 0);
  char *tx = (char *)malloc(s);
  read_transaction(tx, s);
  checksum256 tx_hash;
  sha256(tx, s, &tx_hash);

  const uint64_t bet_id = ((uint64_t)tx_hash.hash[0] << 56)
  + ((uint64_t)tx_hash.hash[1] << 48)
  + ((uint64_t)tx_hash.hash[2] << 40)
  + ((uint64_t)tx_hash.hash[3] << 32)
  + ((uint64_t)tx_hash.hash[4] << 24)
  + ((uint64_t)tx_hash.hash[5] << 16)
  + ((uint64_t)tx_hash.hash[6] << 8)
  + (uint64_t)tx_hash.hash[7];
```
`block_prefix`: [tapos_block_prefix()](https://developers.eos.io/eosio-cpp/reference#tapos_block_prefix), the Prefix in currently running transaction block.

`block_num`: [tapos_block_num()](https://developers.eos.io/eosio-cpp/reference#tapos_block_num), the Block number in currently running transaction.

`timestamp`: [now()](https://developers.eos.io/eosio-cpp/reference#now), Timestamp(sec)

`player_seed`: the seed from the webpage or the player's directly inserted seed.


### 3. Active random number table activation.

**rockscissors** contract has numerous number of random number table. Whenever the user plays the game, a seed is created by combining random seed materials described as above.

```c++
uint32_t seed() {
	return (uint32_t)((block_prefix + block_num) * timestamp + player_seed + bet_id);
}
```

And the smart-contract extracts Nth or lower index from the hashed seed. It gets more seeds from random number table by referring this index.
The final seed is generated from the two seeds as above. The random number is extracted based on the final seed.

The random number table will be updated for each game ends. Everytime newly updated random number table will be referred for the next transaction.
The random number table has a complex form and unpredictable algorithm, which makes it difficult to be hacked by outside and it demands a vast amount of computing or network power for a manipulation.
To ensure fairness, we plan to update the rules for updating the random number table every quarter, and to distribute a script that can verify the transactions that occurred for three months.

---

Still have questions? Reach out to us at gameboycenter@tuta.io and we’ll be happy to help!
>Warning: Please be careful to play too many games on blockchain may lead to a huge loss.
