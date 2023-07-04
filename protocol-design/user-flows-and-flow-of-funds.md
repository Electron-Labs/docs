# User Flows and Flow of Funds

Our bridge supports USDC for now (although other tokens can be added very easily).

Quick summary of types of tokens -&#x20;

1. USDC - native Ethereum USDC
2. zkUSDC - wrapped asset of Electron zk-bridge (only on NEAR)
3. USDC.e - native USDC of NEAR

### Ethereum to NEAR

When a user locks USDC on Ethereum side, our bridge mints zkUSDC on NEAR side. zkUSDC is the wrapped asset provided by our bridge.

Then, using a liquidity pool attached to the bridge, the zkUSDC is automatically converted to USDC.e (the native coin on NEAR).

As a user, you have the choice to select whether you want zkUSDC or USDC.e on NEAR when bridging USDC from Ethereum.

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

### NEAR to Ethereum

When a user want to go from NEAR to Ethereum, they would bring their USDC.e coins. As the first step, the user would be required to convert their USDC.e to zkUSDC using the liquidity module.

Then, the user can burn zkUSDC on the bridge, and then the corresponding amount of USDC tokens are unlocked on Ethereum and transferred to the user.

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>





