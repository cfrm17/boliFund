# BOLI/COLI Stable Value Wrap Model

The Insurance and Pension Solutions Group (IPS) stable value fund wrap model is presented. The modeling approach is to be used for several two basic contract types involving stable value protection on Bank Owned Life Insurance funds (BOLI), and Company Owned Life Insurance funds (COLI).  These products are fundamentally the same but differ with respect to tax treatment and/or the specific details regarding the redemption of the funds or portions thereof.

The model as tested, does not perform pricing, but rather makes an estimate of losses.  The other side of the pricing equation, namely the estimation of received fees, is not treated in the model, although the loss estimation appears to be, for the most part, consistent with the detailed structure of these contracts. Certain simplifications have, however, been made to facilitate the modeling process.  These simplifications do not appear to lead to major errors in loss estimation.  

The stable value model is aimed at estimating the value of providing protection on a portfolio consisting of fixed income and equity instruments.  The fund upon which stable value protection is provided is assumed to be under active management and thus to have a constant duration for the purposes of modeling.  The protection provided by the stable value contract is written on any shortfall between the market value of the fund and a defined “book value” which exists when redemptions of the fund are made.

The book value of the fund is computed as a function of the “crediting rate”, which itself is a function of previous market and book values as well as the equity and/or fixed income indices that indicate the market value of the fund.

There are thus several elements to the modeling problem:

•	The yield of the fixed income component of the protected fund
•	The yield of the equity component of the protected fund
•	The book value
•	The redemption rate
•	The discount curve (see https://finpricing.com/lib/IrCurveIntroduction.html)

The yield on the fixed income component of the protected fund is modeled as the change through time of a single rate which is considered to be the value of a fixed income yield index.  The model used is a single mean-reverting interest rate process given by

  (1)

where  ,  is the mean reversion speed,   is the log of the long-run interest rate,   is the volatility of  .  The parameters must be estimated by running a regression on historical data for the yield index in question.

The model supplied does not specify in any detail the model used for estimating the future values of the equity index.  The test model that has been implemented uses geometric Brownian motion given by

  (2)

where  is the drift and again   is the volatility.  For modeling, the drift should of course be equal to the risk-free forward rate.

The book value of the fund is a function of the crediting rate  , which is given by

  (3)

where
	  is the market value at time  
	  is the book value
	  is the fixed income yield from equation (1)
	 	is the crediting rate spread adjustment
	  is the duration of the protected portfolio

The crediting rate spread adjustment can include many different adjustments. At present it is given by

  (4)

where in turn  is the spread applied to the fixed income index yield,   and   are arbitrary fixed spreads,   is the “trigger” spread, and

  (5)

The market value is computed according to

  (6)

where   is the equity index return, and   is the fixed income index return, while   is the fraction of the portfolio which is held in equities.  Since the protected portfolio is assumed to have constant duration, the fixed income portion of the portfolio is modeled as “rolling over” at each time step in the simulation.  In this case the return on the fixed income portion of the portfolio can be written

  (7)

where   is the simulation time step or portfolio rollover time.

The book value of the protected fund is then given as

  (8)

where   is the exposure cap, which limits the amount of any payout under the contract to a percentage of the market value at any time.  Thus the crediting rate   for time   gives us the book value for time  , which in turn allows us to compute the crediting rate for time  .

Note that equation (8) is an approximation to the actual behavior of the protected fund.  In reality, a two-tiered strategy is used to accelerate the convergence of the market value toward book value in the event that certain exposure limits are breached.  When the book value exceeds market value by 10%, the portfolio must be reallocated to reduce the duration.  If the book and market values continue to diverge, when the book value exceeds market value by 20%, the portfolio is reallocated to strictly money-market instruments, and therefore has a duration of 0.25.  This can be represented by the following equation:

  (8a)

with no hard limit, as in equation  (8), on the book value.  The test model implements equations (8) and (8a) simultaneously.

Payouts under the contract occur only if redemptions of the protected fund are made by the policy holders.  Thus it is necessary to model the expected redemptions of the fund.  The model currently uses a probabilistic redemption model.  The redemption probability is given by specifying an annual redemption rate.  This annual redemption rate is then converted into a redemption probability for a time step according to

  (9)

where   is the annual redemption rate at time  .    In turn the redemption rate is given by

 . (10)

This allows the user to specify an initial redemption rate  , a final redemption rate  , and a time   over which the rate varies from initial to final.  This results in a probability of “survival”, or non-redemption given by

 . (11)

The model uses the redemption probability for a particular time step to make a random decision whether or not redemption occurs in that time step.  If the decision to redeem is made, then the model values the payout which must be made if the entire protected fund is liquidated.  Under this scheme, partial surrenders of the fund never occur.

It should be noted that the stochastic modeling of surrender events in this manner is not, strictly speaking, necessary.  Since surrender probabilities are non-stochastic, and completely independent of other parameters in the model, a more efficient methodology would be to simply multiply the payout amounts originating from the interest rate scenario by the probability of surrender in a given period.  This approach represents a considerable variance reduction over the current method.  The chosen method is correct, but perhaps a less than optimal choice.

A recommended improvement to the model is to allow the surrender rates to be stochastic.  It seems reasonable that the probability of surrender is not known exactly at any time, particularly several years into the future.  In conjunction with non-stochastic modeling of the surrender probability, this does not represent a dramatic increase in variance. 

BOLI/COLI policies are subject to taxation at both the federal and state level in the United States.  For business reasons, wrap contracts include provisions to shelter the wrap purchaser from the effect of these taxes.  There are two taxes that apply.  The first is premium tax.  This is a percentage of the initial premium that is claimed by the government.  Thus, the initial market value of the portfolio is not equal to the initial book value, but is reduced by the premium tax amount.  The amount of the premium tax is replaced over an amortization period, with the payment in each period representing an adjustment to the crediting rate.  The unadjusted crediting rate in each period, however, is calculated using the actual market value plus any outstanding premium tax.  Note that the outstanding premium tax is not itself adjusted for market value.

The other tax which applies to BOLI/COLI wraps is Deferred Amortization Cost (DAC) tax.  In this case, a certain percentage of the initial premium is effectively lent, at zero interest, to the government, which repays the sum according to a defined amortization schedule.  The wrap provider agrees to repay to the policy holder the interest (at the crediting rate) that the outstanding DAC amount would have accrued if it were present in the portfolio.  Again, this represents an adjustment to the effective crediting rate.

The IPS model implements these crediting rate adjustments assuming a constant crediting rate before adjustments.  Depending upon the behavior of actual interest rates, and therefore crediting rates, the crediting rate adjustments may be somewhat different than those calculated by the IPS model.  In order to test the assumption of constant crediting rate for the calculation of these tax adjustments, the test model implemented a method of computing the required amortization payments based on a variable, but known, interest rate scenario.  The expectation of these amortization payments can be computed as the model is estimating losses, and reported to the user.

The amortization of premium and DAC tax amounts is usually carried out by “charging” a fixed number of basis points  to the book value of the fund.  In a variable interest rate, and therefore crediting rate, environment, the correct  fee rate for tax amortization can be computed through the following formula:

  (12)

where   is the fee rate of interest,   is the amount that must be repaid,   is the initial value of the portfolio from which the payments are withdrawn, and the   is the interest rate in period  .  Equation (12) can be solved using an iterative technique and converges very rapidly.

