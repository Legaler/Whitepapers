# CONTINUOUS LIQUIDITY POOLS

## Implementation specification for the continuous liquidity pools in Legaler

V0.1 December 2018

### Abstract 
>Legaler uses continuous liquidity pools to allow on-chain conversions of tokens into and out of LGR and a mathematical price. The continuous liquidity pools are permissionless; anyone can add or remove liquidity and anyone can use the pools to convert between assets. The pools rely on permissionless arbitrage to ensure correct market pricing of assets at any time. 


## Bonding Theory

Tokens on one side of the pool are bound to the tokens on the other. We can now determine the output given an input and pool depth.

| **Unit** | **Definition**                                | **Unit** | **Definition** |
|----------|-----------------------------------------------|----------|----------------|
| `X`        | Balance of TKN in the input side of the pool  | `x`        | Input          |
| `Y`        | Balance of TKN in the output side of the pool | `y`        | Output         |
| `K `       | Constant                                      |          |                |


**Equations:**

![X * Y = K](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20X%20*%20Y%20%3D%20K)

![\frac{y}{Y} = \frac{x}{x + X} \rightarrow y = \frac{xY}{x + X}  ](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20%5Cfrac%7By%7D%7BY%7D%20%3D%20%5Cfrac%7Bx%7D%7Bx%20&plus;%20X%7D%20%5Crightarrow%20y%20%3D%20%5Cfrac%7BxY%7D%7Bx%20&plus;%20X%7D)



## Prices and Slip
We can now determine the expected slip trade and the pool, based only on the input and the depth of the input side of the pool.

| **Unit** | **Definition** | **Unit**   | **Definition**                               |
|----------|----------------|------------|----------------------------------------------|
| `P0`       | Starting Price | `outputSlip` | Slip of the output compared to input         |
| `P1`       | Final Price    | `poolSlip`   | Slip of the pool after the output is removed |

**Equations:**

![P_0 = \frac{X}{Y}, P_1 = \frac{X+x}{Y-y}](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20P_0%20%3D%20%5Cfrac%7BX%7D%7BY%7D%2C%20P_1%20%3D%20%5Cfrac%7BX&plus;x%7D%7BY-y%7D)

![outputSlip = \frac{x/P_0 - t}{x/P_0}  = 1- \frac{Xy}{xY} = \frac{x}{x+X}](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20outputSlip%20%3D%20%5Cfrac%7Bx/P_0%20-%20t%7D%7Bx/P_0%7D%20%3D%201-%20%5Cfrac%7BXy%7D%7BxY%7D%20%3D%20%5Cfrac%7Bx%7D%7Bx&plus;X%7D)

![poolSlip = \frac{P_1 - P_0}{P_0} = \frac{xY + Xy}{XY - Xy} = \frac{x (2X + x)}{X^2}](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20poolSlip%20%3D%20%5Cfrac%7BP_1%20-%20P_0%7D%7BP_0%7D%20%3D%20%5Cfrac%7BxY%20&plus;%20Xy%7D%7BXY%20-%20Xy%7D%20%3D%20%5Cfrac%7Bx%20%282X%20&plus;%20x%29%7D%7BX%5E2%7D)



## Liquidity Fee

Stakers stake symmetrically and earn liquidity fees, which is proportional to slip. Slip is proportional to trade size and liquidity depth. Thus staking is incentivised in pools with out-sized trades. Instead of immediately emitting the bonded tokens, we calculate an appropriate fee, then emit tokens after the fee is removed. 

| **Unit**          | **Definition**                                               |
|-------------------|--------------------------------------------------------------|
| `tokensOutputted` | Tokens outputted from the formula before the fee is applied. |
| `tokensEmitted `    | Tokens emitted from the pool after the fee is applied.       | 
| `outputSlip` | The slip of price between input and output   |
|`tradeSlip`  | The slip of price between input and emission |
| `poolSlip`   | The slip of price in the pool after the swap |

**Equations:**

![liqFee = tradeSlip*tokensOutputted](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20liqFee%20%3D%20tradeSlip*tokensOutputted)

![liqFee = \frac{x}{x+X}*\frac{xY}{x + X} = \frac{x^2Y}{(x+X)^2} ](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20liqFee%20%3D%20%5Cfrac%7Bx%7D%7Bx&plus;X%7D*%5Cfrac%7BxY%7D%7Bx%20&plus;%20X%7D%20%3D%20%5Cfrac%7Bx%5E2Y%7D%7B%28x&plus;X%29%5E2%7D)

![tokensEmitted = tokensOutputted - liqFee](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20tokensEmitted%20%3D%20tokensOutputted%20-%20liqFee)

![tokensEmitted = \frac{x Y}{x + X}  - \frac{x^2Y}{(x+X)^2} = \frac{x Y X}{(x + X)^2}
](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20tokensEmitted%20%3D%20%5Cfrac%7Bx%20Y%7D%7Bx%20&plus;%20X%7D%20-%20%5Cfrac%7Bx%5E2Y%7D%7B%28x&plus;X%29%5E2%7D%20%3D%20%5Cfrac%7Bx%20Y%20X%7D%7B%28x%20&plus;%20X%29%5E2%7D)

![tradeSlip = \frac{xY/X - tokensEmitted}{xY/X}  = \frac{x (2 X + x)}{(x + X)^2}](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20tradeSlip%20%3D%20%5Cfrac%7BxY/X%20-%20tokensEmitted%7D%7BxY/X%7D%20%3D%20%5Cfrac%7Bx%20%282%20X%20&plus;%20x%29%7D%7B%28x%20&plus;%20X%29%5E2%7D)

Thus the final price that the user receives, as well as the final price of the pool, are:

![Price_{user} = Price_{0} * (1 - tradeSlip)](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20Price_%7Buser%7D%20%3D%20Price_%7B0%7D%20*%20%281%20-%20tradeSlip%29)

![Price_{pool} = Price_{0} * (1 - poolSlip)](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20Price_%7Bpool%7D%20%3D%20Price_%7B0%7D%20*%20%281%20-%20poolSlip%29)


## Atomic trade of a single pool

We have a single pool, TKN1, paired to LGR.  We wish to swap LGR to TKN1. 

| **Unit** | **Definition**                                | **Unit** | **Definition** |
|----------|-----------------------------------------------|----------|----------------|
| `X`        | Balance of TKN1 in the input side of the pool  | `x`        | Input of  LGR         |
| `Y`        | Balance of LGR in the output side of the pool | `y`        | Output of Token1        |

**Equations:**


![tokensOutputted = \frac{xY}{x + X}, outputSlip = \frac{x}{x+X}](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20tokensOutputted%20%3D%20%5Cfrac%7BxY%7D%7Bx%20&plus;%20X%7D%2C%20outputSlip%20%3D%20%5Cfrac%7Bx%7D%7Bx&plus;X%7D)

![liqFee = \frac{x^2Y}{(x+X)^2}](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20liqFee%20%3D%20%5Cfrac%7Bx%5E2Y%7D%7B%28x&plus;X%29%5E2%7D)

![tradeSlip = \frac{x (2X + x)}{(x + X)^2}, poolSlip = \frac{x (2X + x)}{X^2}](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20tradeSlip%20%3D%20%5Cfrac%7Bx%20%282X%20&plus;%20x%29%7D%7B%28x%20&plus;%20X%29%5E2%7D%2C%20poolSlip%20%3D%20%5Cfrac%7Bx%20%282X%20&plus;%20x%29%7D%7BX%5E2%7D)

![tokensEmitted = \frac{x Y X}{(x + X)^2}](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20tokensEmitted%20%3D%20%5Cfrac%7Bx%20Y%20X%7D%7B%28x%20&plus;%20X%29%5E2%7D)


## Atomic trades over two pools 

We have a two pools, TKN1 & TKN2, both paired to LGR.  We wish to swap TKN1 to TKN2.  

| **Unit** | **Definition**                                | **Unit** | **Definition** |
|----------|-----------------------------------------------|----------|----------------|
| `X`        | Balance of TKN1 in the input side of the pool  | `x`        | Input of Token1          |
| `Y`        | Balance of LGR in the output side of the pool | `y`        | Output of LGR        |
| `R`        | Balance of TKN1 in the input side of the pool  |         |           |
| `Z`        | Balance of LGR in the output side of the pool | `z`        | Output of Token2       |

**Equations:**


![tradeSlip_1 = \frac{x (2X + x)}{(x + X)^2}, liqFee_1 = \frac{x^2Y}{(x+X)^2}](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20tradeSlip_1%20%3D%20%5Cfrac%7Bx%20%282X%20&plus;%20x%29%7D%7B%28x%20&plus;%20X%29%5E2%7D%2C%20liqFee_1%20%3D%20%5Cfrac%7Bx%5E2Y%7D%7B%28x&plus;X%29%5E2%7D)

![int_{emission} = y = \frac{x Y X}{(x + X)^2}](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20int_%7Bemission%7D%20%3D%20y%20%3D%20%5Cfrac%7Bx%20Y%20X%7D%7B%28x%20&plus;%20X%29%5E2%7D)

Then:

![tradeSlip_2 = \frac{y (2R + y)}{(y + R)^2}, liqFee_2 = \frac{y^2Z}{(y+R)^2}](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20tradeSlip_2%20%3D%20%5Cfrac%7By%20%282R%20&plus;%20y%29%7D%7B%28y%20&plus;%20R%29%5E2%7D%2C%20liqFee_2%20%3D%20%5Cfrac%7By%5E2Z%7D%7B%28y&plus;R%29%5E2%7D)

![tokensEmitted_2 =  \frac{y Z R}{(y + R)^2}](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20tokensEmitted_2%20%3D%20%5Cfrac%7By%20Z%20R%7D%7B%28y%20&plus;%20R%29%5E2%7D)

Using just the pool depths, and the input, we can calculate the final output and slip:

![tokensEmitted_2 = z =  \frac{x X Y R Z (x + X)^2}{(x X Y + R x^2 + 2 R x X + R X^2)^2}](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20tokensEmitted_2%20%3D%20z%20%3D%20%5Cfrac%7Bx%20X%20Y%20R%20Z%20%28x%20&plus;%20X%29%5E2%7D%7B%28x%20X%20Y%20&plus;%20R%20x%5E2%20&plus;%202%20R%20x%20X%20&plus;%20R%20X%5E2%29%5E2%7D)

![P_{X0} = \frac{X}{Y}, P_{Z0} = \frac{R}{Z} \rightarrow P_0 = P_{X0} * P_{Z0}= \frac{XR}{YZ}](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20P_%7BX0%7D%20%3D%20%5Cfrac%7BX%7D%7BY%7D%2C%20P_%7BZ0%7D%20%3D%20%5Cfrac%7BR%7D%7BZ%7D%20%5Crightarrow%20P_0%20%3D%20P_%7BX0%7D%20*%20P_%7BZ0%7D%3D%20%5Cfrac%7BXR%7D%7BYZ%7D)

![finalSlip = \frac{x/P_0 - z}{x/P_0}  = \frac{\frac{xYZ}{XR}-z}{\frac{xYZ}{XR}} = 1 - \frac{R^2 X^2(x + X)^2}{(R (x + X)^2 + x X Y)^2}](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20finalSlip%20%3D%20%5Cfrac%7Bx/P_0%20-%20z%7D%7Bx/P_0%7D%20%3D%20%5Cfrac%7B%5Cfrac%7BxYZ%7D%7BXR%7D-z%7D%7B%5Cfrac%7BxYZ%7D%7BXR%7D%7D%20%3D%201%20-%20%5Cfrac%7BR%5E2%20X%5E2%28x%20&plus;%20X%29%5E2%7D%7B%28R%20%28x%20&plus;%20X%29%5E2%20&plus;%20x%20X%20Y%29%5E2%7D)


## Pool Staking

Stakers stake assets to earn a share of the pool. Stake average is the average of their two stakes (of LGR and TKN) at the time they staked.

| **Unit**   | **Definition**                      | **Unit**    | **Definition**                      |
|------------|-------------------------------------|-------------|-------------------------------------|
| `R`          | Balance of LGR in the pool         | `T `          | Balance of TKN in the pool          |
| `stakeR`     | Stake of LGR                       | `stakeT`      | Stake of TKN                        |
| `poolFeesR`  | Total accumulated fees in LGR      | `poolFeesTKN` | Total accumulated fees in TKN       |
| `stakeAveXi` | Averaged stake for staker X         | `stakeAveX`   | Sum of averaged stakes for staker X |
| `poolStakeX` | Sum of all stakes for staker X      | `poolTotal`   | Sum of pool stakes for all stakers  |
| `poolShareX` | Share of the pool for staker X      |             |                                     |
| `LGRFeesX`  | Share of the LGR fees for staker X | `TKNFeesX`    | Share of the TKN fees for staker X  |
| `LGRStakeX` | Share of the LGR fees for staker X | `TKNStakeX`   | Share of the TKN fees for staker X  |

**Equations:**

![stakeAve_{Xi} = (\frac{stake_{R}}{R + stake_R} + \frac{stake_{T}}{T+stake_T} ) * \frac{1}{2}](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20stakeAve_%7BXi%7D%20%3D%20%28%5Cfrac%7Bstake_%7BR%7D%7D%7BR%20&plus;%20stake_R%7D%20&plus;%20%5Cfrac%7Bstake_%7BT%7D%7D%7BT&plus;stake_T%7D%20%29%20*%20%5Cfrac%7B1%7D%7B2%7D)

![poolStake_{Xi} = stakeAve_{Xi} * (T+stake_T)](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20poolStake_%7BXi%7D%20%3D%20stakeAve_%7BXi%7D%20*%20%28T&plus;stake_T%29)

Users can only withdraw their stake partially or fully, or add more. This is tracked as the same as all stakes for that user:

![poolStake_{X} = [poolStake_{X0} + poolStake_{X1} + ... n]](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20poolStake_%7BX%7D%20%3D%20%5BpoolStake_%7BX0%7D%20&plus;%20poolStake_%7BX1%7D%20&plus;%20...%20n%5D)

A further number tracks the sum of every averaged stake from every user that has been made into the pool, including the first:

![poolTotal = \sum\limits_{i=0}^n  poolStake_{i} ](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20poolTotal%20%3D%20%5Csum%5Climits_%7Bi%3D0%7D%5En%20poolStake_%7Bi%7D)

![poolShare_{X} = \frac{poolStake_{X}}{pool_{total}}](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20poolShare_%7BX%7D%20%3D%20%5Cfrac%7BpoolStake_%7BX%7D%7D%7Bpool_%7Btotal%7D%7D)

Thus when a user X withdraws either the fees or their share of the pool, the following are the equations:

![TKNFees_{X} = poolShare_{X} * poolFees_{TKN}](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20TKNFees_%7BX%7D%20%3D%20poolShare_%7BX%7D%20*%20poolFees_%7BTKN%7D)

![LGRFees_{X} = poolShare_{X} * poolFees_{LGR}](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20LGRFees_%7BX%7D%20%3D%20poolShare_%7BX%7D%20*%20poolFees_%7BLGR%7D)

![TKNStake_{X} = poolShare_{X} * T](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20TKNStake_%7BX%7D%20%3D%20poolShare_%7BX%7D%20*%20T)

![LGRStake_{X} = poolShare_{X} * R](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20LGRStake_%7BX%7D%20%3D%20poolShare_%7BX%7D%20*%20R)

*For simplicity, stakes and fees can be set to be fully distributed when a staker withdraws. *


## Tracking Pool Metrics

The balance of every pool is the sum of all stakes and inputs, minus all emissions, on both sides. Fees are intrinsic to the pool until they are withdrawn by the stakers. 

![LGR_{pool1} = \sum\limits_{i=0}^n  stake_{i} + \sum\limits_{i=0}^n  inputs_{i} - \sum\limits_{i=0}^n  emissions_{i} - \sum\limits_{i=0}^n  feeWithdrawals_{i} - \sum\limits_{i=0}^n  stakeWithdrawals_{i}](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20LGR_%7Bpool1%7D%20%3D%20%5Csum%5Climits_%7Bi%3D0%7D%5En%20stake_%7Bi%7D%20&plus;%20%5Csum%5Climits_%7Bi%3D0%7D%5En%20inputs_%7Bi%7D%20-%20%5Csum%5Climits_%7Bi%3D0%7D%5En%20emissions_%7Bi%7D%20-%20%5Csum%5Climits_%7Bi%3D0%7D%5En%20feeWithdrawals_%7Bi%7D%20-%20%5Csum%5Climits_%7Bi%3D0%7D%5En%20stakeWithdrawals_%7Bi%7D)

Thus when a trade across a pool occurs (LGR -> TKN1), LGR and TKN1 balances are changed atomically:

![LGR_{pool1_{1}} = LGR_{pool1_{0}} + x](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20LGR_%7Bpool1_%7B1%7D%7D%20%3D%20LGR_%7Bpool1_%7B0%7D%7D%20&plus;%20x)

![TKN_{pool1_{1}} = TKN_{pool1_{0}} - y](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20TKN_%7Bpool1_%7B1%7D%7D%20%3D%20TKN_%7Bpool1_%7B0%7D%7D%20-%20y)

Across two pools, (TKN1 -> TKN2 via LGR), all balances are changed atomically:

![TKN_{pool1_{1}} = TKN_{pool1_{0}} + x](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20TKN_%7Bpool1_%7B1%7D%7D%20%3D%20TKN_%7Bpool1_%7B0%7D%7D%20&plus;%20x)

![LGR_{pool1_{1}} = LGR_{pool1_{0}} - y](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20LGR_%7Bpool1_%7B1%7D%7D%20%3D%20LGR_%7Bpool1_%7B0%7D%7D%20-%20y)

![LGR_{pool2_{1}} = LGR_{pool2_{0}} + y](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20LGR_%7Bpool2_%7B1%7D%7D%20%3D%20LGR_%7Bpool2_%7B0%7D%7D%20&plus;%20y)

![TKN_{pool2_{1}} = TKN_{pool2_{0}} - z](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20TKN_%7Bpool2_%7B1%7D%7D%20%3D%20TKN_%7Bpool2_%7B0%7D%7D%20-%20z)

## Arbitrage

Arbitrage is important to restore pool prices to the fair market price. To solve for the arbitrage amount, the fair market price `P_M` must equal the final pool price `P_1` after the trade:

![P_M = P_1 = \frac{X_1}{Y_1}](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20P_M%20%3D%20P_1%20%3D%20%5Cfrac%7BX_1%7D%7BY_1%7D)

![X_1 = X_0 + x, Y_1 = Y_0 - y](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20X_1%20%3D%20X_0%20&plus;%20x%2C%20Y_1%20%3D%20Y_0%20-%20y)

![P_M = \frac{X_0+x}{Y_0-y}, y = \frac{x Y X}{(x + X)^2}](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20P_M%20%3D%20%5Cfrac%7BX_0&plus;x%7D%7BY_0-y%7D%2C%20y%20%3D%20%5Cfrac%7Bx%20Y%20X%7D%7B%28x%20&plus;%20X%29%5E2%7D)

![1 - \frac{X+x}{P_M*Y-\frac{x Y X*P_M}{(x + X)^2}} = 0](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%201%20-%20%5Cfrac%7BX&plus;x%7D%7BP_M*Y-%5Cfrac%7Bx%20Y%20X*P_M%7D%7B%28x%20&plus;%20X%29%5E2%7D%7D%20%3D%200)

The real root of `x` is:

```
x = -(-2 P^3 Y^3 + 3 P^2 X Y^2 + 6 P^2 Y^3 + sqrt(81 P^4 X^2 Y^4 - 108 P^3 X^3 Y^3 - 270 P^3 X^2 Y^4
+ 540 P^2 X^4 Y^2 - 108 P^2 X^3 Y^3 + 297 P^2 X^2 Y^4 + 108 P X^5 Y - 324 P X^4 Y^2 + 324 P X^3 Y^3 - 108 P X^2 Y^4)
- 24 P X^2 Y + 3 P X Y^2 - 6 P Y^3 - 2 X^3 + 6 X^2 Y - 6 X Y^2 + 2 Y^3)^(1/3)/(3 2^(1/3))
+ (2^(1/3) (-3 (P X Y - X^2 - 2 X Y) - (P Y - 2 X - Y)^2))/(3 (-2 P^3 Y^3 + 3 P^2 X Y^2 + 6 P^2 Y^3
+ sqrt(81 P^4 X^2 Y^4 - 108 P^3 X^3 Y^3 - 270 P^3 X^2 Y^4 + 540 P^2 X^4 Y^2 - 108 P^2 X^3 Y^3
+ 297 P^2 X^2 Y^4 + 108 P X^5 Y - 324 P X^4 Y^2 + 324 P X^3 Y^3 - 108 P X^2 Y^4) - 24 P X^2 Y + 3 P X Y^2
- 6 P Y^3 - 2 X^3 + 6 X^2 Y - 6 X Y^2 + 2 Y^3)^(1/3)) + 1/3 (P Y - 2 X - Y);
```

This is a significantly complex solution, which means arbitrage will only be an approximate science in practise.

A significantly easier solution is to attempt to slip the price in the opposite direction but same amount as the premium, using the trade slip, instead of the pool slip. 

![P_M = \frac{X_0+x}{Y_0-y}, y = \frac{xY_0}{x + X_0} ](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20P_M%20%3D%20%5Cfrac%7BX_0&plus;x%7D%7BY_0-y%7D%2C%20y%20%3D%20%5Cfrac%7BxY_0%7D%7Bx%20&plus;%20X_0%7D)

![\frac{X_0+x}{Y_0*P_M-\frac{xY_0*P_M}{x + X_0} }](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20%5Cfrac%7BX_0&plus;x%7D%7BY_0*P_M-%5Cfrac%7BxY_0*P_M%7D%7Bx%20&plus;%20X_0%7D%20%7D)

![x = \sqrt{X} (\sqrt{P} \sqrt{Y} + \sqrt{X})](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20x%20%3D%20%5Csqrt%7BX%7D%20%28%5Csqrt%7BP%7D%20%5Csqrt%7BY%7D%20&plus;%20%5Csqrt%7BX%7D%29)

This solution results in a final price within 1% of the desired price, with premiums less than 20%. 

## Summary of Equations

### Onchain Equations:
The following equations are on-chain (they are processed at the protocol level).

#### Single Pool Trade

Caculation of the liquidity fee incurred in a trade:

![liqFee = \frac{x^2Y}{(x+X)^2}](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20liqFee%20%3D%20%5Cfrac%7Bx%5E2Y%7D%7B%28x&plus;X%29%5E2%7D)

Calculation of the final tokens to received in a single pool trade:

![tokensEmitted = \frac{x Y X}{(x + X)^2}](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20tokensEmitted%20%3D%20%5Cfrac%7Bx%20Y%20X%7D%7B%28x%20&plus;%20X%29%5E2%7D)

#### Double Pool Trade

As above for single pool trade, and:

Calculation of the intermediate liquidity fee incurred in a trade:

![liqFee_2 = \frac{y^2Z}{(y+R)^2}](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20liqFee_2%20%3D%20%5Cfrac%7By%5E2Z%7D%7B%28y&plus;R%29%5E2%7D)

Calculation of the final tokens to received in a double pool trade:

![tokensEmitted =  \frac{x X Y R Z (x + X)^2}{(x X Y + R x^2 + 2 R x X + R X^2)^2}](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20tokensEmitted%20%3D%20%5Cfrac%7Bx%20X%20Y%20R%20Z%20%28x%20&plus;%20X%29%5E2%7D%7B%28x%20X%20Y%20&plus;%20R%20x%5E2%20&plus;%202%20R%20x%20X%20&plus;%20R%20X%5E2%29%5E2%7D)

#### Staking

The stake from the staker is averaged and saved on-chain:

![stakeAve_{Xi} = (\frac{stake_{R}}{R + stake_R} + \frac{stake_{T}}{T+stake_T} ) * \frac{1}{2}](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20stakeAve_%7BXi%7D%20%3D%20%28%5Cfrac%7Bstake_%7BR%7D%7D%7BR%20&plus;%20stake_R%7D%20&plus;%20%5Cfrac%7Bstake_%7BT%7D%7D%7BT&plus;stake_T%7D%20%29%20*%20%5Cfrac%7B1%7D%7B2%7D)

![poolStake_{Xi} = stakeAve_{Xi} * (T+stake_T)](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20poolStake_%7BXi%7D%20%3D%20stakeAve_%7BXi%7D%20*%20%28T&plus;stake_T%29)

#### Withdrawing Fees and Stake

The proportion of fees and stake entitled to the staker is given by:

![poolShare_{X} = \frac{poolStake_{X}}{pool_{total}}](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20poolShare_%7BX%7D%20%3D%20%5Cfrac%7BpoolStake_%7BX%7D%7D%7Bpool_%7Btotal%7D%7D)

### Offchain Equations:
The following equations can be calculated off-chain (client-side) to allow the user to comprehend the pools and information whilst trading.

Trade slip for a single pool trade:

![tradeSlip = \frac{x (2X + x)}{(x + X)^2}](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20tradeSlip%20%3D%20%5Cfrac%7Bx%20%282X%20&plus;%20x%29%7D%7B%28x%20&plus;%20X%29%5E2%7D)

Trade slip for a double pool trade:

![tradeSlip = 1 - \frac{R^2 X^2(x + X)^2}{(R (x + X)^2 + x X Y)^2}](https://latex.codecogs.com/png.latex?%5Cdpi%7B100%7D%20%5Clarge%20tradeSlip%20%3D%201%20-%20%5Cfrac%7BR%5E2%20X%5E2%28x%20&plus;%20X%29%5E2%7D%7B%28R%20%28x%20&plus;%20X%29%5E2%20&plus;%20x%20X%20Y%29%5E2%7D)

## Conclusion

The implementation specification for Legaler’s CLPs is described. 



