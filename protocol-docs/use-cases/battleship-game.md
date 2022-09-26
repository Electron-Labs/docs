# Battleship Game

Let's see how we can build the classic game Battleship on a blockchain. If you have not played the game before - try it here - [https://www.battleshiponline.org/](https://www.battleshiponline.org/), it's a lot of fun. This tutorial assumes you know how the game works.

You can appreciate the fact that you cannot build Battleship without using privacy technology. This is because you need to hide the position of your ships. Let's get started.

#### Setup phase

Both players (Alice and Bob) arrange their ships using a UI in their browser. When they are satisfied with their setup, they can press submit. This will create a zk-proof of the initial game state of the player and submit it to the game managing smart contract. The properties of the zk-proof constructed are as follows -\
**Private Input** : `initial_game_state`\
**Business Logic implemented inside the ZK-program** :

1. Implement the following checks on `initial_game_state` -
   1. Have all the ships been placed?
   2. Are the ships placed valid (they are of the right length and breadth)?
   3. The location of the ships is within the designated squares?
   4. There is no overlapping of the ships?
2. Calculate `hash_game_state = SHA256(initial_game_state)`

**Public Output** : `hash_game_state`

Once both the proof commits are received on-chain (and successfully verify), the game starts. The `hash_game_state` of each player is stored on-chain for later use. The players must also maintain a local copy of their `initial_game_state`.

Bug: The signatures of the players also need to be added.

#### Gameplay (Steps to complete one volley)

1. Alice (through her UI) selects the square she wants to hit and makes an on-chain transaction to do the following:
   1. Set the value of the on-chain variable `attempt_location` to the square location `(x,y)` that Alice wants to hit.
   2. Set the value of the on-chain variable `attempt_result` as `pending` for this volley.
2. Bob reads this data from the smart contract, and then constructs a zk-proof that proves whether Alice's attempt was a successful hit or miss. Properties of this zk-proof are as follows -
   1. **Public Input1**: Bob's `hash_game_state`
   2. **Public Input2**: Alice's `attempt_location`for this volley
   3. **Public Input3**: Alice's `attempt_result` for this volley
   4. **Private Input1**: Bob's `initial_game_state`
   5. **Business Logic inside the ZK-program** :
      1. assert(`SHA256(initial_game_state) == hash_game_state`)
      2. assert(`attempt_result` == "pending")
      3. if (`initial_game_state` has a ship present at`attempt_location`) => assign `attempt_result = success`
      4. else => assign `attempt_result = fail`
   6. **Public Output** : `attempt_result`
3. Bob submits this on-chain where the smart contract does the following -
   1. Check that `hash_game_state` used in the zk-proof is the same as the one that Bob provided at the start of the game.
   2. Check that `attempt_location` used in the zk-proof is the same as the one provided by Alice in this volley.
   3. Check the zk-proof is valid.
   4. Update the on-chain value of `attempt_result` to the one specified as the Public Output of the zk-proof.

And this completes one volley!

Additions needed -\
Logic needs to be implemented about whether a ship was completely destroyed or not.\
The signature of the player also needs to be provided.
