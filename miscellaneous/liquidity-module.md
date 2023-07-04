# Liquidity Module

A big problem with bridges is that they give you these wrapped assets that have little utility. This problem does not exist in our bridge, as we give the user native assets on both the sides. We achieve this via the use of liquidity pools that can convert wrapped assets to native assets and vice-versa.

Detailed design of our Liquidity module is given below.

### Video Explainer -&#x20;

### ETH -> NEAR

{% embed url="https://youtu.be/Ai1BDBtootA" %}

### NEAR -> ETH

{% embed url="https://youtu.be/DWor4jG6zwY" %}

^^ please see the videos before proceeding

#### Important Terms

**The LP Contract:** This is the contract in which all Liquidity Providers (LP’s) add liquidity (zkUSDC or USDC). All LP’s add liquidity into the same LP contract.

**LP token:** Whenever LP’s deposit liquidity in the contract, they get LP tokens that represent their share in the pool. So if an LP supplied 20% of the total liquidity in the bridge, they will be given new LP tokens such that they own 20% of the LP tokens. New LP tokens are minted everytime someone deposits liquidity into the pool, and LP tokens must be burned to withdraw liquidity from the pool.

**Let us now try to understand how much tokens an LP will get when they add liquidity to a pre-existing pool.**

<figure><img src="../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>



### Adding Liquidity

Consider the scenario when a new LP is adding new stable-coins. LP adds new stable coins $$V$$and gets LP tokens in return as $$T$$

Just _before_ the LP adds the liquidity, the supply of stable coin supply is = $$TVL_i$$ and the total LP coin supply is $$S_i$$

Just _after_ the LP adds the liquidity, the supply of stable coin supply is = $$TVL_f$$ and the total LP coin supply is $$S_f$$

Hence we can write ⇒

$$S_f = S_i + T$$

$$TVL_f = TVL_i + V$$

Furthermore, the % stakeholding in the LP token of the LP should be the same as the fraction of liquidity that belongs to him.

Hence, we can write ⇒

## $$\frac{T}{S_f}=\frac{V}{TVL_f}$$

## $$\frac{T}{T+S_i}=\frac{V}{TVL_i + V}$$

#### $$T(TVL_i + V) = TV + S_iV$$

## $$T=\frac{V * S_i}{TVL_i}$$

We can hence easily calculate the new tokens that need to be given to the LP when they add liquidity. Please note that new LP are minted and then given to the LP. Hence, the LP token supply is variable.

Let us try to calculate the price of the LP token before and after the liquidity is added.

Price just _before_ liquidity is added = $$P_i$$

Price just _after_ liquidity is added = $$P_f$$

We can calculate $$P_i$$ as ⇒

## $$P_i = \frac {TVL_i}{S_i}$$

And we can also write $$P_f$$ as ⇒

## $$P_f = \frac {TVL_f}{S_f} = \frac {TVL_i + V}{S_i + T} = \frac {TVL_i}{S_i}$$

Hence we can see that $$P_i == P_f$$

Hence price of the LP token is same before and after liquidity is added.

**As the pool accrues fees, the total liquidity in the pool will increase while LP tokens stay constant. Hence, the LP token will be an interest bearing token.**

Our model also ensures that LP only get the fees for the transactions that happened while the LP had locked the liquidity.

### Removing Liquidity

