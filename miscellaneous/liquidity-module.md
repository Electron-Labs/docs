# Liquidity Module

A big problem with existing bridges is that they give you these wrapped assets that have little utility. This problem does not exist in our bridge, as we give the user native assets on both the sides. We achieve this via the use of liquidity pools that can convert wrapped assets to native assets and vice-versa.

Detailed design of our Liquidity module is given below.

### Video Explainer -&#x20;

### ETH -> NEAR

{% embed url="https://youtu.be/Ai1BDBtootA" %}

### NEAR -> ETH

{% embed url="https://youtu.be/DWor4jG6zwY" %}

^^ please see the videos before proceeding

#### Important Terms

**The LP Contract:** This is the contract in which all Liquidity Providers (LP’s) add liquidity (zkUSDC or USDC.e). All LP’s add liquidity into the same LP contract.

**LP token:** Whenever LP’s deposit liquidity in the contract, they get LP tokens that represent their share in the pool. So if an LP supplied 20% of the total liquidity in the bridge, they will be given new LP tokens such that they own 20% of the LP tokens. New LP tokens are minted everytime someone deposits liquidity into the pool, and LP tokens must be burned to withdraw liquidity from the pool.

**Let us now try to understand how much tokens an LP will get when they add liquidity to a pre-existing pool.**

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

### Adding Liquidity

Consider the scenario when a new LP is adding new stable-coins. LP adds new stable coins $$v$$and gets LP tokens in return as $$t$$

Just _before_ the LP adds the liquidity, the supply of stable coin supply is = $$TVL_i$$ and the total LP coin supply is $$S_i$$

Just _after_ the LP adds the liquidity, the supply of stable coin supply is = $$TVL_f$$ and the total LP coin supply is $$S_f$$

Hence we can write ⇒

$$S_f = S_i + t$$

$$TVL_f = TVL_i + v$$

Furthermore, the % stakeholding in the LP token of the LP should be the same as the fraction of liquidity that belongs to him.

Hence, we can write ⇒

## $$\frac{t}{S_f}=\frac{v}{TVL_f}$$

## $$\frac{t}{t+S_i}=\frac{v}{TVL_i + v}$$

#### $$t(TVL_i + v) = vT + vS_i$$

## $$t=\frac{v * S_i}{TVL_i}$$

We can hence easily calculate the new tokens that need to be given to the LP when they add liquidity. Please note that new LP are minted and then given to the LP. Hence, the LP token supply is variable.

Let us try to calculate the price of the LP token before and after the liquidity is added.

Price just _before_ liquidity is added = $$P_i$$

Price just _after_ liquidity is added = $$P_f$$

We can calculate $$P_i$$ as ⇒

## $$P_i = \frac {TVL_i}{S_i}$$

And we can also write $$P_f$$ as ⇒

## $$P_f = \frac {TVL_f}{S_f} = \frac {TVL_i + v}{S_i + t} = \frac {TVL_i}{S_i}$$

Hence we can see that $$P_i == P_f$$

Hence price of the LP token is same before and after liquidity is added.

**As the pool accrues fees, the total liquidity in the pool will increase while LP tokens stay constant. Hence, the LP token will be an interest bearing token.**

Our model also ensures that LP only get the fees for the transactions that happened while the LP had locked the liquidity.

### Removing Liquidity

Initial liquidity in the pool _before_ LP burn = $$TVL_i$$

Final liquidity in the pool _after_ LP burn = $$TVL_f$$

Total supply of LP tokens _before_ LP burn = $$S_i$$

Total supply of LP tokens _after_ LP burn = $$S_f$$

Amount of LP tokens burnt in order to withdraw liquidity = $$t$$

Liquidity that LP withdraws from the protocol = $$v$$

Relation between $$TVL_i$$, $$TVL_f$$, $$S_i$$, $$S_f$$, $$t$$, and $$v$$:

$$(TVL)_f=(TVL)_i-v$$  .... (1)

$$S_f=S_i-t$$                       .... (2)



Since based on price equality, $$P_i = P_f$$

⇒ $$\frac{(TVL)_i}{S_i}=\frac{(TVL)_f}{S_f}$$

⇒ $$\frac{(TVL)_i}{S_i}=\frac{(TVL)_i-v}{S_i-t}$$ …. using (1) and (2)

### ⇒ $$v = \frac{(TVL)_i}{S_i}t$$



#### $$v$$ can be withdrawn in three ways:

1.  `function remove_liquidity(uint256 t)`\
    a mix of both $$v_{zkUSDC}$$ and $$v_{USDC.e}$$ such as

    $$v_{USDC.e} = \frac{{TVL}_{USDC.e}}{TVL_i}*v$$ ; and

    $$v_{zkUSDC} = \frac{{TVL}_{zkUSDC}}{TVL_i}*v$$

    No fee is deducted during this operation.
2. `function remove_and_swap(uint256 t, bool iszkusdc)`\
   100% in the form of zkUSDC then remove liquidity using above formula and swap $$v_{USDC.e}$$ with zkUSDC through the stableswap pool. During this swap, a fee `f` will be deducted.
   *   If in the pool,

       $$TVL_{zkUSDC} > TVL_{USDC.e}$$ ⇒ $$fee = 0$$

       Essentially LP is making pool healthy by removing his liquidity in the form of excess token.
   * If in the pool, $$TVL_{zkUSDC} \le TVL_{USDC.e}$$ ⇒ $$fee > 0$$ and will be calculated using **Fee Calculation** below. Essentially LP is making pool unhealthy by removing his liquidity.
3.  `function remove_and_swap(uint256 t, bool iszkusdc)`

    100% in the form of USDC.e then remove liquidity using above formula and swap $$v_{zkUSDC}$$ with USDC.e through the stableswap pool. During this swap, a fee `f` will be deducted.

    * If in the pool, $$TVL_{USDC.e} > TVL_{zkUSDC}$$ ⇒ $$fee = 0$$ . Essentially LP is making pool healthy by removing his liquidity in the form of excess token.
    * If in the pool, $$TVL_{USDC.e} \le TVL_{zkUSDC}$$ ⇒ $$fee > 0$$ and will be calculated using **Fee Calculation** below. Essentially LP is making pool unhealthy by removing his liquidity.



#### **Fees Calculation**

Let us say the fees for a swap is `f`, and the admin cut is `x%`, the fees given to admin is `f*x`, and the fees given to pool is `f*(1-x)`. Then for an incoming transaction of `U` zkUSDC, the following balance updates will happen -

1. $$pool_{zkUSDC} = pool_{zkUSDC} + (U-f)+f*(1-x\\%)$$
2. $$pool_{USDC.e} = pool_{USDC.e} - (U - f)$$
3. $$admin_{zkUSDC} = admin_{zkUSDC}+f*(x\%)$$

The fees collected by the admin is sent to another address (multi-sig).



Overtime, the ratio of zkUSDC:USDC.e could become skewed. Hence, LP maye need to re-balance from time to time.

**Re-balancing process for converting USDC to zkUSDC**

1. LP withdraws USDC.e from the NEAR LP contract.
2. LP takes this USDC.e to Ethereum via Rainbow, and get’s native USDC on Ethereum.
3. LP deposits the native USDC to Electron Bridge to get zkUSDC on NEAR side.
4. LP deposits this zkUSDC in the LP contract

**Re-balancing process for converting zkUSDC to USDC**

1. LP withdraws the zkUSDC from the NEAR LP contract.
2. LP takes this zkUSDC back to Ethereum via Electron Bridge, and get’s native USDC on Ethereum.
3. LP deposit’s the native USDC to Rainbow Bridge to get USDC.e to NEAR side.
4. LP deposits this USDC.e to LP contract.

