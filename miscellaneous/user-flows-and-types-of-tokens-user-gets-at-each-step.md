# User flows and types of tokens user gets at each step

When a user locks USDC on Ethereum side, our bridge mints zkUSDC on NEAR side. zkUSDC is the wrapped assets provided by our bridge.

Then, using a liquidity pool attached to the bridge, the zkUSDC is automatically converted to USDC.e (the native coin on NEAR)

As a user, you have the choice to select whether you want zkUSDC or USDC.e on NEAR fro bridging USDC from Eth.

When a user want to go from NEAR to Eth, they would bring their USDC.e coins. As the first step, the user would be required to convert their USDC.e to zkUSDC using the liquidity module.

Then, the user can burn zkUSDC on the bridge, and then the corresponding amount of USDC tokens are unlocked on Ethereum and transferred to the user.
